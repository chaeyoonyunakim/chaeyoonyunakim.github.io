---
layout: post
type: reflection
title: "Fixing a Sentiment Analysis Pipeline: A Checklist for Next Time"
date: 2026-07-01
categories: [nlp, data-analysis, postmortem]
tags: [python, nltk, textblob, vader, sentiment-analysis]
---

In [What I Got Wrong Building a Sentiment Analysis Pipeline for Survey Data]({% post_url 2026-07-01-what-i-got-wrong-sentiment-analysis %}), I traced why a survey sentiment pipeline came back suspiciously flat: stemmed text fed into a lexicon scorer, negation words stripped out by a stopword list, a filtering step that silently discarded the zero-scored rows instead of investigating them, and a scoring tool that was never validated against real judgments. This post is the other half — a concrete checklist for building the pipeline correctly the next time, rather than patching the same bugs after the fact.

## 1. Never share preprocessing between bag-of-words and sentiment scoring

Stemming and lemmatization exist to collapse word variants for frequency counting. Lexicon-based sentiment tools need the opposite — intact words that match dictionary entries. The fix isn't a smarter stemmer; it's two separate preprocessing branches from the same raw text:

```python
raw_text = response.strip()

# Branch 1: word-cloud / frequency analysis
bow_text = lemmatize(remove_stopwords(tokenize(raw_text.lower())))

# Branch 2: sentiment scoring — lightly cleaned only
sentiment_text = expand_contractions(raw_text.lower())
sentiment_text = strip_punctuation(sentiment_text, keep_negation_markers=True)
```

Treat "text prepared for sentiment scoring" and "text prepared for word frequency" as two different artifacts with two different names in code, not two call sites sharing one `clean()` function. If a future refactor tries to merge them again, that naming makes the mistake visible in review.

## 2. Build negation handling in from the start, not as a patch

Don't start a stopword list from a generic NLTK/spaCy default and extend it without auditing what's already in there. Before any stopword list touches text destined for sentiment scoring:

```python
NEGATIONS = {'no', 'nor', 'not', 'never', 'neither', 'none', 'nothing', 'nowhere'}
sentiment_stopwords = [w for w in default_stopwords if w not in NEGATIONS]
```

And expand contractions *before* tokenizing, not after — a tokenizer that splits `"didn't"` into `"do"` + `"n't"` will drop `"n't"` as non-alphabetic before any stopword filter even runs:

```python
sentiment_text = contractions.fix(raw_text)  # "didn't" -> "did not"
```

Write a small unit test that locks this in permanently:

```python
def test_negation_survives_pipeline():
    assert score("The team is not happy") < 0
    assert score("Leadership didn't communicate well") < 0
```

A test like this catches a regression the moment someone edits the stopword list, instead of six months later when the summary stats look "suspiciously flat" again.

## 3. Treat a zero score as a measurement gap, not a data point to drop

`0.0` from a lexicon-based scorer means "no recognizable sentiment-bearing words were found," not "neutral." Never filter it out of an average silently. Instead, report it:

```python
zero_rate = (scores == 0.0).mean()
print(f"{zero_rate:.1%} of responses had no detected sentiment words")
```

If that rate is high (double digits, say), that's a signal the *coverage* of the tool is the problem, not that respondents lack opinions. Publish the coverage number alongside the average sentiment score every time — it's the diagnostic that would have caught all three bugs immediately, before ever tracing an individual response by hand.

## 4. Pick a tool suited to the text, and prove it before trusting it

TextBlob's `pattern.en` lexicon is ~2,900 hand-tagged adjectives, tuned for movie reviews. Domain words like "understaffed," "burnout," or "morale" — nouns and verbs that carry the emotional weight in workplace feedback — aren't scored at all. Two concrete upgrades, in order of effort:

- **VADER** (`vaderSentiment`) — rule-based, handles negation and intensifiers ("very," "!!", ALL CAPS) natively, and is tuned for short informal text rather than long-form reviews. It's a drop-in replacement with meaningfully better coverage for survey responses.
- **A transformer-based classifier** (e.g. a fine-tuned `distilbert-base-uncased-finetuned-sst-2-english` or similar) — higher accuracy, but adds a model dependency and inference cost. Worth it if sentiment scoring feeds a decision, not just a dashboard.

Whichever tool is chosen, validate it before trusting its output:

```python
sample = df.sample(50, random_state=0)
# hand-label sample['human_label'] as positive/negative/neutral
from sklearn.metrics import classification_report
print(classification_report(sample['human_label'], sample['model_label']))
```

No pipeline should ship a sentiment number to a stakeholder without at least one hand-labeled validation pass behind it. That's the check that would have surfaced all of this before the report went out, not after.

## 5. Add a pipeline sanity check, not just a code review

The root cause underneath all four bugs was the same: nothing in the pipeline would flag itself as broken. A cheap guardrail catches this class of bug automatically:

```python
def sanity_check(df):
    assert df['sentiment'].std() > 0.05, "suspiciously low variance — check preprocessing"
    known_negative = "This was a terrible and frustrating experience"
    assert score(known_negative) < -0.3, "known-negative probe scored near zero"
```

Run it as part of the pipeline, not as a one-off debugging step. A probe sentence with obvious, known sentiment is a canary — if it stops registering, the pipeline breaks loudly instead of quietly returning plausible-looking zeros.

## The takeaway

Every fix above is cheap relative to the cost of shipping a wrong conclusion — "the team feels neutral about the reorg" — to people making decisions on it. The general pattern: separate preprocessing paths by purpose, protect the tokens the model actually depends on, treat missing signal as missing (not neutral), and validate the tool against ground truth before trusting its numbers. None of it requires a bigger model — just checking the pipeline's work before believing it.
