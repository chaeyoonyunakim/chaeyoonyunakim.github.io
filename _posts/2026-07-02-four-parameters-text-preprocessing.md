---
layout: post
type: reflection
title: "Four Knobs in My Text Preprocessing Pipeline (and What Each One Actually Does)"
date: 2026-07-02
categories: [nlp, data-analysis]
tags: [python, nltk, text-preprocessing, contractions, stopwords, stemming, lemmatization]
---

In [What I Got Wrong Building a Sentiment Analysis Pipeline for Survey Data]({% post_url 2026-07-01-what-i-got-wrong-sentiment-analysis %}) and the [follow-up checklist]({% post_url 2026-07-01-fixing-sentiment-analysis-pipeline-next-time %}), I wrote about the bugs that came from treating "normalize text" as a single, undifferentiated step. This post is the reference I wish I'd had at the time: a walkthrough of the `TextPreprocessor` class behind that pipeline, the four parameters that control what happens to each response before it's counted, and — at the end — why the sentiment-scoring step needs a different combination of those same four parameters than the word cloud does. All the code is collected in the [Appendix](#appendix-full-source) at the bottom; the sections below discuss it and link to the relevant part rather than repeating it inline.

Four things happen to a raw survey response before it becomes a token in a word cloud: contraction expansion, stopword removal, stemming, and lemmatization. `contractions`, `rm_stopwords`, `stemming`, and `lemmatization` are all independent booleans on the class ([Appendix A](#a-the-textpreprocessor-class)), which makes it easy to turn any of them on or off — but "independent" also means it's on you to know what combining them does. Here's each one.

## 1. Contractions — expanded before tokenizing, not after

When `self.contractions` is true, `contractions.fix(sent)` runs first, on the raw sentence string, before `word_tokenize` ever sees it — turning `"Leadership didn't communicate well"` into `"Leadership did not communicate well"` ([Appendix B](#b-contraction-expansion)).

Order matters here specifically because of how NLTK's tokenizer handles contractions. `word_tokenize("didn't")` splits into `["did", "n't"]`, and `"n't"` then fails the `w.isalpha()` filter two lines later — so if contractions aren't expanded first, the negation particle is silently dropped before the stopword list is even consulted. Expanding contractions on the full string first turns `"didn't"` into two ordinary words, `"did"` and `"not"`, both of which survive tokenization intact. Note the naming: the `contractions` parameter (a bool) and the `contractions` module (imported at the top) share a name, but Python resolves them separately — the parameter is a local name in `__init__`'s scope, while the module reference inside `normalise_text` resolves normally through the enclosing module scope, so `contractions.fix(sent)` still calls the library function correctly.

## 2. Stopwords — what's added, and what's deliberately kept

`self.stopwords` starts from NLTK's default English list (~180 words: "the", "is", "and", "a", ...) and is extended with domain-specific noise words that showed up constantly in this survey's free text without carrying much signal for a word cloud ([Appendix C](#c-stopword-additions-and-the-negation-carve-out)): mostly NHS/organisational shorthand and boilerplate ("wte" = whole-time equivalent, "slt" = senior leadership team) plus words so common in *this specific survey's prompt* ("team", "work", "feel") that they'd dominate every word cloud regardless of what respondents actually said — the kind of stopwords you only discover by looking at your first output and asking "why is 'team' the biggest word in every single response category."

The one deliberate subtraction is negation. NLTK's default list includes "no", "not", "nor" — removing them would silently flip meaning downstream (see the [postmortem post]({% post_url 2026-07-01-what-i-got-wrong-sentiment-analysis %}) for what that cost when it happened to a sentiment score). This class produces bag-of-words output, not a sentiment score, so a stray "not" doesn't flip a polarity number here — but it does still change what a word cloud or bigram frequency table implies. "Not supported" and "supported" are different signals even in a frequency count, so negation words are carved out of the stopword list before it's used anywhere, not just before sentiment scoring.

## 3. Stemming — Porter's rule-based truncation

`PorterStemmer` chops words down to a root form using a fixed set of suffix-stripping rules, without checking whether the result is a real word — `"communication"` becomes `"commun"`, `"frustrating"` becomes `"frustrat"`, `"supportive"` becomes `"support"` ([Appendix D](#d-stemming-examples)).

It's fast and dependency-free, which is why it's useful for collapsing word variants before counting frequencies — `"support"`, `"supports"`, `"supported"`, and `"supportive"` all reduce to something starting with `"support"`, so they group together in a frequency count instead of splitting into four separate low-count entries. The tradeoff is that the output isn't always readable English (`"commun"`, `"frustrat"`), which is fine for counting but useless for anything that needs to match against a dictionary — which is exactly why this same normalized text should never be handed to a lexicon-based sentiment tool.

## 4. Lemmatization — dictionary-aware, and run three times on purpose

`WordNetLemmatizer` looks up the correct base form using an actual dictionary rather than truncating by rule, but it needs to know the part of speech to do that correctly — a lemmatizer given no POS hint defaults to treating everything as a noun. That's why `lemmatize` is called three times in sequence, once per part of speech: nouns, then verbs, then adjectives ([Appendix E](#e-lemmatization-three-pass)).

Each pass is a no-op for words that don't match that part of speech, so running all three in sequence is a cheap way to catch noun, verb, and adjective inflections without a separate POS tagger — at the cost of being approximate (a word that's actually a noun still gets passed through the verb and adjective lemmatizers, which usually leaves it unchanged, but isn't guaranteed to for every edge case).

## 5. Polarity scoring — the one configuration that doesn't erase the signal

The scoring function and the `df_norm` it runs on are in [Appendix F](#f-polarity-scoring). The short version: `df_norm` isn't built with the same flags I'd use for a word cloud. It comes from the same `TextPreprocessor` class, but instantiated as `TextPreprocessor(contractions=True, rm_stopwords=True, stemming=False, lemmatization=False)`.

In terms of the four knobs above, that's **(1) contractions = True, (2) stopwords = True, (3) stemming = False, (4) lemmatization = False.** Here's why each one has to land exactly there for polarity scoring to work:

**(1) Contractions: True — this one's non-negotiable.** `TextBlob("not happy")` and `TextBlob("happy")` score oppositely, so the negation particle has to survive as a real token. Leaving `contractions` off would mean `word_tokenize("didn't")` splits into `["did", "n't"]`, `"n't"` fails the alphabetic filter, and the negation disappears before it ever reaches TextBlob — silently turning a negative response into a positive-scoring one. `contractions=True` is what expands it to `"did"` + `"not"` first, so both survive.

**(2) Stopwords: True — safe, because the list only removes words TextBlob was never going to score.** `pattern.en` scores by matching individual words — mostly adjectives — against a fixed lexicon. Removing "the", "is", "would", "team", "leadership", and the rest of the extended stopword list doesn't touch anything TextBlob would have picked up, because none of those words are in its adjective dictionary to begin with. And because negation words were already carved back out of `self.stopwords` in `__init__`, turning `rm_stopwords` on here strips noise for free without risking a flipped polarity.

**(3) Stemming: False — this is the one that actually matters.** This is Mistake 1 from the postmortem, restated as a parameter choice: `ps.stem("frustrating")` returns `"frustrat"`, which isn't a word TextBlob's dictionary recognizes, so `TextBlob("frustrat").sentiment.polarity` comes back `0.0` — not because the response was neutral, but because the word that would have scored it no longer exists after stemming. Turning `stemming` off is what keeps `"frustrating"`, `"unsupported"`, and `"disorganised"` intact and scoreable.

**(4) Lemmatization: False — for the same reason, even though it's gentler.** Lemmatization produces real dictionary words, which makes it look safer than stemming, but it still changes the exact form of the word being checked against TextBlob's lexicon — and that lexicon is small (~2,900 hand-tagged entries), built against whatever inflected forms its authors happened to tag, not systematically against every lemma. Swapping `"worse"` for `"bad"` or `"frustrating"` for `"frustrate"` risks landing on a form that scores differently, or isn't in the dictionary at all, for no benefit — a word cloud needs canonical forms to group counts; a fixed-dictionary scorer needs the original inflection, because that's the form it was hand-labelled against.

The pattern underneath all four: for a lexicon-based scorer, every transformation is a *risk of losing a dictionary match*, not a neutral cleanup step. Stopword removal is safe because it only ever removes words the lexicon wasn't going to score anyway. Stemming and lemmatization are unsafe because they change the very word the lexicon is trying to look up. `(1)=True, (2)=True, (3)=False, (4)=False` isn't an arbitrary choice — it's "keep everything TextBlob might match, strip everything it definitely won't."

## The one gotcha: don't turn on both 3 and 4 at once

`stemming` and `lemmatization` are independent flags, but stacking them isn't "get the benefits of both" — Porter stemming runs first and already mangles words into non-dictionary fragments, so by the time `WordNetLemmatizer` runs, most inputs are things like `"commun"` or `"frustrat"` that aren't in WordNetLemmatizer's dictionary and pass through unchanged. In practice, pick one: stemming for a faster, coarser grouping, or lemmatization alone for output that stays human-readable in a word cloud. Running both isn't wrong, exactly — it's just paying for a step that mostly does nothing.

## The general shape

All four parameters exist to answer the same question differently: which word variants should collapse into the same bucket when you're counting what people said? Contractions and negation-aware stopwords protect meaning; stemming and lemmatization control how aggressively variants get merged. The same class can serve both the word cloud and the sentiment scorer, but never with the same flags: `stemming`/`lemmatization` on (or either one) for frequency counts, both off for polarity scoring. The parameter that matters most isn't any single flag — it's remembering which pipeline the output is about to feed before you decide how aggressively to transform it.

## Appendix: full source

### A. The `TextPreprocessor` class

*Custom project code — no official docs; it's built on the libraries referenced in B–F below.*

```python
class TextPreprocessor:
    """
    Process text in columns of a DataFrame.

    Args:
        contractions: Set True to expand contractions in normalization.
        rm_stopwords: Set True to apply stopwords in normalization.
        stemming: Set True to apply ps in normalization.
        lemmatization: Set True to apply wordnet_lemmatizer in normalization.

    Attributes:
        stopwords: A customised set of words to be removed from the text.
        ps: An instance of a stemmer for reducing words to their root form.
        wordnet_lemmatizer: An instance of a lemmatizer for reducing words to their base form.

    Methods:
        process_text(df, columns): Normalizes text columns in a DataFrame.
    """

    def __init__(self, contractions, rm_stopwords, stemming, lemmatization):
        self.stopwords = stopwords.words('english')
        self.stopwords.extend(['would', 'wte', 'slt', 'senior', 'leader',
                                'leadership', 'sub', 'team', 'member', 'feel',
                                'like', 'work'])

        negations = {'no', 'nor', 'not', 'never', 'neither', 'none'}
        self.stopwords = [w for w in self.stopwords if w not in negations]

        self.ps = PorterStemmer()
        self.wordnet_lemmatizer = WordNetLemmatizer()

        self.contractions = contractions
        self.rm_stopwords = rm_stopwords
        self.stemming = stemming
        self.lemmatization = lemmatization

    def process_text(self, df, columns):
        def normalise_text(sent):
            if self.contractions:
                sent = contractions.fix(sent)
            words = word_tokenize(sent)
            words = [w.lower() for w in words if w.isalpha()]

            if self.rm_stopwords and self.stopwords:
                words = [w for w in words if w not in self.stopwords]

            if self.stemming:
                words = [self.ps.stem(w) for w in words]

            if self.lemmatization:
                words = [self.wordnet_lemmatizer.lemmatize(w, pos="n") for w in words]
                words = [self.wordnet_lemmatizer.lemmatize(w, pos="v") for w in words]
                words = [self.wordnet_lemmatizer.lemmatize(w, pos="a") for w in words]

            return " ".join(words)

        for col in columns:
            df[col] = df[col].fillna('').apply(normalise_text)

        return df
```

### B. Contraction expansion

*Docs: [`contractions` on GitHub](https://github.com/kootenpv/contractions) — no separate docs site; the README is the reference.*

```python
contractions.fix("Leadership didn't communicate well")
# -> "Leadership did not communicate well"
```

### C. Stopword additions and the negation carve-out

*Docs: [NLTK Stopwords Corpus](https://www.nltk.org/nltk_data/)*

```python
self.stopwords.extend(['would', 'wte', 'slt', 'senior', 'leader',
                        'leadership', 'sub', 'team', 'member', 'feel',
                        'like', 'work'])

negations = {'no', 'nor', 'not', 'never', 'neither', 'none'}
self.stopwords = [w for w in self.stopwords if w not in negations]
```

### D. Stemming examples

*Docs: [`nltk.stem.porter` — PorterStemmer](https://www.nltk.org/api/nltk.stem.porter.html)*

```python
ps.stem("communication")  # -> "commun"
ps.stem("leadership")     # -> "leadership"  (already in the extended stopword list, so moot here)
ps.stem("frustrating")    # -> "frustrat"
ps.stem("supportive")     # -> "support"
```

### E. Lemmatization three-pass

*Docs: [`nltk.stem.wordnet` — WordNetLemmatizer](https://www.nltk.org/api/nltk.stem.wordnet.html)*

```python
words = [wordnet_lemmatizer.lemmatize(w, pos="n") for w in words]  # nouns:  "leaders" -> "leader"
words = [wordnet_lemmatizer.lemmatize(w, pos="v") for w in words]  # verbs:  "communicated" -> "communicate"
words = [wordnet_lemmatizer.lemmatize(w, pos="a") for w in words]  # adjectives: "worse" -> "bad"
```

### F. Polarity scoring

*Docs: [TextBlob Quickstart — Sentiment Analysis](https://textblob.readthedocs.io/en/latest/quickstart.html#sentiment-analysis)*

```python
preprocessor = TextPreprocessor(contractions=True, rm_stopwords=True, stemming=False, lemmatization=False)
df_norm = preprocessor.process_text(df, columns)

def get_sentiment(text):
    blob = TextBlob(text)
    return blob.sentiment.polarity

sentiment_scores = df_norm.map(get_sentiment)
```
