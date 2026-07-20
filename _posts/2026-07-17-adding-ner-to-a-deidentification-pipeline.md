---
layout: post
type: reflection
title: "Adding NER to a De-identification Pipeline — Without Betting the Guarantee on It"
date: 2026-07-17
categories: [healthcare-ai, engineering]
tags: [nhs, de-identification, ner, presidio, spacy, nlp, healthcare-ai]
summary: "How NoteGuard uses Presidio + spaCy (en_core_web_md), why it's an optional extra, and the gotchas nobody warns you about."
---

> A companion to my [NoteGuard one-month retro](https://chaeyoonyunakim.github.io/2026/07/10/noteguard-one-month-retro/).
> In that post I mentioned "turning on NER" almost in passing. This is the zoom-in: what
> that actually took, and why the whole system is built so it *doesn't* depend on it.

[NoteGuard](https://github.com/chaeyoonyunakim/noteguard-agent)'s job is to strip patient
identifiers out of NHS clinical free-text *before* an LLM ever sees it. Rules and a
ground-truth vault get you a long way — NHS numbers, GMC/NMC IDs, postcodes, emails,
dates. What they can't do is catch a **free-text name the vault has never heard of**:
"Reviewed by Dr Ethel Joanne Duffy." That's the gap NER fills — and, more importantly,
here's how I made sure the system doesn't *depend* on it.

## The gap

The de-identification core is rules + a vault. The vault is built from the dataset's
structured tables, so any patient or clinician name that appears there gets redacted
deterministically. But a discharge note is free text. It's full of names that were never
in a table:

```
Reviewed by Dr Ethel Joanne Duffy. Nurse Jasmine Freda Murray assessed at triage.
```

The rule layer is deliberately **name-agnostic** — it matches formats (NHS numbers,
postcodes), not arbitrary capitalised words, because "match any Title-Case token" would
redact half of every clinical note. So those two names sail straight through to the
model. My trust panel would *report* the leak (that's a separate audit — see the retro),
but reporting a leak isn't the same as not leaking. I needed something that actually
recognises names.

That something is **Named Entity Recognition** — and the pragmatic, no-training-required
option is [Microsoft Presidio](https://microsoft.github.io/presidio/) sitting on top of a
spaCy model.

## Rule #1: NER is a *recall boost*, never a hard dependency

Before writing a line of it, I set one constraint: **the app must still run, and still
de-identify, if NER isn't installed at all.** The rule/vault layer is the floor; NER only
ever raises the ceiling. That constraint shaped everything.

So NER lives behind a tiny interface with a no-op default:

```python
class _Detector:
    """Stub detector — no-op when Presidio/spaCy is not installed."""
    def detect_persons(self, text: str) -> list[str]:
        return []
```

and a builder that upgrades to the real thing *only if it imports cleanly*:

```python
def _build_detector() -> _Detector:
    try:
        from presidio_analyzer import AnalyzerEngine
        from presidio_analyzer.nlp_engine import NlpEngineProvider

        model = os.getenv("NOTEGUARD_SPACY_MODEL", "en_core_web_md")
        provider = NlpEngineProvider(nlp_configuration={
            "nlp_engine_name": "spacy",
            "models": [{"lang_code": "en", "model_name": model}],
        })
        engine = AnalyzerEngine(nlp_engine=provider.create_engine(),
                                supported_languages=["en"])

        class _PresidioDetector(_Detector):
            def detect_persons(self, text: str) -> list[str]:
                results = engine.analyze(text, language="en",
                                         entities=["PERSON", "LOCATION"])
                return [text[r.start:r.end] for r in results if r.end > r.start]

        return _PresidioDetector()
    except Exception:
        return _Detector()          # graceful fallback — rules + vault still run
```

That `except Exception: return _Detector()` is the whole philosophy in two lines. CI
doesn't install NER, so CI runs the stub. My laptop doesn't always have the model, so it
runs the stub. The deployed image *does* have it, so it runs the real detector. Same code
path, three environments, zero crashes.

### The `NlpEngineProvider` detail that cost me an afternoon

The naive way to build a Presidio engine is `AnalyzerEngine()` with no arguments. Don't.
The default configuration wants **`en_core_web_lg`** — a ~560 MB model — and if it's not
present you get a confusing failure at *import time*, not call time. The fix is to
configure the NLP engine explicitly with the model you actually shipped, via
`NlpEngineProvider`. That also makes the model a one-line, env-configurable choice
(`NOTEGUARD_SPACY_MODEL`), which matters for the next section.

## Shipping it: the `[nlp]` extra + the Docker image

The Python deps are an **optional extra** in `pyproject.toml`, so a plain install stays
lean and only people who want NER pull the weight:

```toml
[project.optional-dependencies]
# pip install -e ".[nlp]"  — Presidio + spaCy NER for free-text PERSON/LOCATION.
# Also needs a spaCy model:  python -m spacy download en_core_web_md
nlp = ["presidio-analyzer>=2.2.0", "spacy>=3.7.0,<4"]
```

The spaCy *model* isn't a normal pip dependency — it's a separate download — so the
deployed Docker image installs the extra and then fetches the model at build time:

```dockerfile
RUN pip install --no-cache-dir ... "presidio-analyzer>=2.2.0" "spacy>=3.7.0,<4"
RUN python -m spacy download en_core_web_md
ENV NOTEGUARD_SPACY_MODEL=en_core_web_md
```

Baking the model into the image (rather than downloading at boot) keeps cold starts
predictable and avoids a runtime network dependency.

## Choosing the model: `sm` vs `md` vs `lg`

This is the decision people hand-wave, so here's the actual data. I ran all three spaCy
English models on the note that broke things:

```
"Contacted GP Dr. Ethel Joanne Duffy. Nurse Jasmine Freda Murray reviewed.
 Seen by Dr Sarah Chen."

en_core_web_sm  → ['Ethel Joanne Duffy', 'Murray', 'Sarah Chen']
en_core_web_md  → ['Ethel Joanne Duffy', 'Nurse Jasmine', 'Freda Murray', 'Sarah Chen']
```

- **`sm`** (~12 MB) caught "Ethel Joanne Duffy" fully but only picked up **"Murray"** from
  the second name — leaving "Jasmine Freda" exposed.
- **`md`** (~40 MB) covered the second full surname (across two spans) — meaningfully
  better recall.
- **`lg`** (~560 MB) is better still, but it's **~14× the size** of `md` for a demo Space.

`md` is the sweet spot: materially better recall than `sm`, a fraction of `lg`. And
because the model is env-configurable, anyone who wants maximum recall can set
`NOTEGUARD_SPACY_MODEL=en_core_web_lg` and bake that in instead. Recall/size is a dial,
not a hard-coded decision.

## The gotcha reel

NER is not a "drop it in and you're done" component. Three things bit me:

### 1. Presidio scores every spaCy entity the *same*
I assumed I could filter false positives by confidence. Nope — Presidio assigns spaCy
entities a **flat 0.85 score**. A real name and a garbage detection get the *identical*
score, so a threshold buys you nothing. This is important to know before you design any
"only redact if confidence > X" logic: with the spaCy recogniser, there is no X.

### 2. It thinks clinical abbreviations are people
`en_core_web_md` confidently tagged **"Subcut"** (subcutaneous) as a `PERSON`. So the
de-id turned "Subcut emphysema" into "`[PERSON_3]` emphysema" — safe, but wrong and noisy.
With no confidence signal to lean on (see #1), the pragmatic fix is a small
**clinical-term allow-list** applied at the detector boundary:

```python
_NER_STOPWORDS = frozenset({"subcut", "subcutaneous", "obs", "sats", "nad",
                            "afebrile", "pneumomediastinum", ...})

@staticmethod
def _detect_names(text: str) -> list[str]:
    return [n for n in _DETECTOR.detect_persons(text)
            if n and len(n) > 2 and n.lower() not in _NER_STOPWORDS]
```

It's a stop-list, so it's inherently incomplete — but it's targeted, explainable, and it
never touches a real name.

### 3. NER makes your tests non-deterministic
The moment NER is installed, `deidentify()` output depends on whether the model is
present — which breaks unit tests that assume the rule/vault layer in isolation. The fix
is an autouse fixture that pins the stub by default, so the suite is deterministic whether
or not the `[nlp]` extra is installed; the one NER-path test injects its own fake detector
on top. Tests should test the layer they mean to test.

## Where NER sits in the stack

The thing I'd most want a past-me to internalise: **NER is one layer of four, and it's the
only probabilistic one.** The design leans on that:

| Layer | Kind | Job |
|---|---|---|
| Rules + vault | Deterministic | NHS numbers, IDs, known names — exact, measurable |
| **Presidio NER** | **Probabilistic** | **Free-text names the vault never saw — recall boost** |
| `scan_pii` audit | Deterministic, high-precision | Reports anything that *still* leaked (incl. NER's misses) |
| `assert_clean()` | Hard gate | Raises before the model sees anything with a known identifier |

NER improves the *odds*. The audit tells the truth about what got through. The gate is the
guarantee. If NER were the guarantee, a single missed name would be a silent PHI leak — so
it isn't, and it never will be.

## The honest limitation

`en_core_web_md` is trained largely on Western/English text, so **name recall is lower for
names of non-English origin.** In a de-identification tool, under-detection isn't a neutral
miss — it means those patients carry a *higher* residual re-identification risk. That's an
equity problem, not a footnote. My mitigations are the deterministic vault (which doesn't
care about a name's origin), the `scan_pii` audit that *surfaces* residual names to the
clinician rather than hiding the gap, and a plan to evaluate recall **stratified by name
origin** before this ever touches real Trust data. Naming the limitation out loud is part
of the job.

## Takeaways

- **Make NER optional.** Graceful fallback to the rule layer means one code path across
  CI, local, and prod — and no crash when the model isn't there.
- **Configure the spaCy model explicitly** (`NlpEngineProvider`) instead of letting
  Presidio default to `lg`, and make it an env var so recall/size is a dial.
- **You can't threshold spaCy confidence** in Presidio — plan for an allow-list, not a
  cutoff.
- **NER recalls; it doesn't guarantee.** Pair it with a precise audit and a hard gate, and
  be honest about whose names it misses.

*This is a companion to the [NoteGuard one-month retro](https://chaeyoonyunakim.github.io/2026/07/10/noteguard-one-month-retro/).
Code on [GitHub](https://github.com/chaeyoonyunakim/noteguard-agent); live on Hugging Face Spaces.*
