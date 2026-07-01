---
layout: post
title: "What I Got Wrong Building a Sentiment Analysis Pipeline for Survey Data"
date: 2026-07-01
categories: [nlp, data-analysis, postmortem]
tags: [python, nltk, textblob, sentiment-analysis]
---

I ran a sentiment analysis pipeline over free-text responses from an internal survey asking people for their thoughts on a team management change. Given that kind of prompt, I expected a real spread of emotion — people are rarely neutral about being reorganized. Instead, almost every score came back at or near zero.

My first instinct was "maybe people really were neutral." They weren't. When I went back through the pipeline line by line, I found three self-inflicted bugs that were quietly erasing the signal before it ever reached the sentiment model — plus a fourth, more fundamental issue with the tool I'd chosen. Here's the self-critique.

## Mistake 1: I scored sentiment on text I had already destroyed

My pipeline had one preprocessing step that did double duty: it produced normalized text for both a bag-of-words/word-cloud step *and* the sentiment analysis step. That normalization applied stopword removal, Porter stemming, and lemmatization — all sensible for word-frequency analysis, all fatal for a lexicon-based sentiment tool.

`TextBlob`'s polarity scoring works by matching words against a fixed dictionary of known English words ("terrible", "frustrating", "supportive"). Stemming mangles words into fragments that aren't in that dictionary at all:

```
"Communication from senior leadership has been terrible and frustrating"
  raw polarity:    -0.70
  after stemming:  "commun senior leadership terribl frustrat"
  stemmed polarity: 0.00
```

Every negative example I tested collapsed to exactly `0.0` once stemmed. That was the main source of the "neutral" result — not an absence of emotion in the responses, but a preprocessing step silently deleting the words the model needed to detect it.

**The fix:** never feed the same normalized text into both a bag-of-words pipeline and a sentiment pipeline. Run sentiment scoring on the original (or only lightly cleaned — lowercased, punctuation-stripped) text, and reserve stemming/lemmatization for the word-cloud/frequency analysis where destroying word forms doesn't matter.

## Mistake 2: I stripped the words that carry negation

My stopword list started from NLTK's default English stopword set, which includes "not", "no", and "nor" (plus contracted forms like "don't", "isn't", "wasn't"). I extended it with some domain-specific words without checking what was already baked in.

TextBlob actually handles negation reasonably well on its own:

```
TextBlob("happy").sentiment.polarity      ->  0.8
TextBlob("not happy").sentiment.polarity  -> -0.4
```

But if "not" gets removed before scoring, "not happy" becomes "happy" — a full sentiment flip, not just a dropped word. Anyone who wrote "not supported" or "leadership didn't communicate well" would have scored as neutral-to-positive instead of negative.

**The fix:** explicitly filter negation words out of the stopword list before it's used anywhere near sentiment scoring:

```python
negations = {'no', 'nor', 'not', 'never', 'neither', 'none'}
self.stopwords = [w for w in self.stopwords if w not in negations]
```

There's a subtlety here too: my tokenizer split contractions like "don't" into "do" + "n't", and then filtered out "n't" because it isn't alphabetic — so it was gone before the stopword list was even consulted. Fixing the stopword list alone wasn't enough; I also needed to expand contractions *before* tokenizing (e.g. `"didn't communicate"` → `"did not communicate"`) so the negation survived as a real token.

## Mistake 3: my "average" quietly threw out most of the data

To compute a mean sentiment score per question, I filtered out rows with a score of exactly `0.0` before averaging — the idea being that a `0.0` meant "neutral," so it shouldn't count. But per Mistake 1, `0.0` didn't mean neutral — it meant "the model found no recognizable words." Filtering those rows out meant my final averages were computed from a small, non-representative leftover of responses (mostly ones with short, unstemmed common words like "good" or "low" that survived by luck), not the dataset as a whole.

**The fix:** treat `0.0` scores as a coverage problem, not a neutral judgment. Report "% of responses with zero detected sentiment words" as a diagnostic, and investigate *why* they scored zero, rather than silently dropping them from the average.

## Mistake 4: the tool itself wasn't a great fit, and I never checked

Even with the above fixed, TextBlob's built-in lexicon (`pattern.en`) is small (~2,900 hand-tagged adjectives) and generic — tuned more for movie-review-style text than workplace feedback. Words like "understaffed," "burnout," "restructure," or "morale" simply aren't in it, so plenty of genuinely emotional responses would still score `0.0` because TextBlob only scores the adjectives it recognizes, ignoring the nouns and verbs that carry most of the weight in this kind of feedback.

More importantly: I never validated the model's output against a hand-labeled sample. I had no F1 score, no precision/recall, nothing — just an assumption that "TextBlob's number" was "the sentiment." That's the real root cause underneath all three bugs above: I didn't build in any way to notice the pipeline was broken until I went looking.

**The fix going forward:**
- Swap in something better suited to short, informal text — VADER (rule-based, built-in negation/intensifier handling) as a low-effort upgrade, or a transformer-based sentiment classifier for real accuracy gains.
- Hand-label a small sample and actually measure agreement before trusting the pipeline's output.

## The general lesson

Every one of these bugs was invisible from the summary statistics alone — the numbers looked plausible (small, near-zero, "neutral"), just wrong. The only way I caught them was by tracing a few individual responses through every transformation step and checking the *intermediate* output, not just the final score. If a sentiment pipeline on emotionally loaded text comes back suspiciously flat, that's a signal to go inspect the preprocessing, not to conclude the respondents didn't feel anything.

For the concrete fixes I'm carrying into the next pipeline — code snippets included — see the follow-up post: [Fixing a Sentiment Analysis Pipeline: A Checklist for Next Time]({% post_url 2026-07-01-fixing-sentiment-analysis-pipeline-next-time %}).
