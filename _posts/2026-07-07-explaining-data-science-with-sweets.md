---
layout: post
type: reflection
title: "Candy, Sweets, and Jelly: Explaining Data Science at a Local School (and Getting My Labels Wrong)"
date: 2026-07-07
categories: [outreach, machine-learning]
tags: [volunteering, classification, STEM-outreach, labelling, british-english]
summary: "I volunteered to explain what a data scientist does to three very different age groups, using a sweet-sorting classifier as the running example — and accidentally demonstrated why label vocabulary matters more than model choice."
---

I recently volunteered at a local school to talk about what I do as a data scientist and what you study to become one. The brief was unusual: three sessions, three wildly different audiences — Year 1 and 2, Year 7 and 8, and Year 12 and 13. The same job, explained to a six-year-old, a twelve-year-old, and someone choosing university courses.

I built the whole talk around one running example: sorting sweets with a classification model. It mostly worked. Where it went wrong taught me something I should have known already — and it turned out to be the best data science lesson in the room.

---

## The setup: one example, three altitudes

I opened every session with the same slide — a collage of things they'd all recognise from the corner shop: Dairy Milk, Maltesers, Skittles, Haribo Goldbears, Jelly Babies, M&M's, Smarties, lollipops, a chocolate bunny.

![Slide 1 — a collage of familiar British confectionery: Skittles, Smarties, M&M's, Maltesers, Galaxy, Dairy Milk, Aero, Haribo Goldbears, Jelly Babies, Kinder Surprise, lollipops and more](/assets/img/school-talk-sweets-collage.png)

Then the second slide: what my job actually looks like, stripped of jargon.

![Slide 2 — "Work!" slide: a dog sitting in a suitcase next to paper bags of assorted sweets, with bullet points: using a computer and internet, having a super sorting power, making decisions with supporting data](/assets/img/school-talk-work-slide.png)

- Using a computer and the internet
- Having a super sorting power
- Making decisions with supporting data

"Super sorting power" is not a bad one-line job description for classification, honestly.

### The binary classifier: can the dog have it?

The first model was a yes/no question every child immediately understood: **can I share this packet with a dog?**

Chocolate is toxic to dogs, so a Dairy Milk bar, Maltesers, or the chocolate bunny gets a hard **No**. The answer isn't a matter of opinion — it comes from evidence about what theobromine does to dogs. That's the third bullet point doing its work: *making decisions with supporting data*. (And for the record, the safe answer for the fruit gummies is also no — but the point was that the model draws a line, and the line comes from data, not vibes.)

For Year 1 and 2, that was the whole lesson: a machine can learn to answer yes-or-no questions, and the answers have to come from evidence, because a wrong answer here really hurts someone (or some dog).

### The multi-class classifier: candy or chocolate?

For the older groups I extended it: instead of two answers, sort every item on the collage slide into classes — **candy** or **chocolate**. Skittles go one way, Aero goes the other, and then you hit the fun edge cases: what about chocolate-covered things? M&M's? A Kinder Surprise, which is chocolate wrapped around a toy? Suddenly we were talking about ambiguous examples, fuzzy class boundaries, and who gets to decide the ground truth — which is most of applied machine learning, delivered via a bag of Haribo.

With Year 12 and 13 that opened the door to the "what do I study for this" conversation: maths and statistics for the decision boundaries, computing for the pipelines, and — as I was about to inadvertently demonstrate — a lot of careful thinking about how you define your labels in the first place.

---

## Where my labels didn't sync with the room

Here's what I got wrong, and I didn't notice until afterwards.

**"Candy" is not the word.** I'd built the whole taxonomy around *candy* vs *chocolate* — but British pupils don't say candy. They say **sweets**. Candy is what American YouTube says; sweets is what you ask for at the corner shop. Every time I said "candy," there was a tiny translation step happening in the room that I hadn't budgeted for. My class labels were in the wrong dialect.

**"Jelly" made it worse.** Trying to be more granular, I introduced a third label — *jelly* — for the Haribo-type gummies. Reasonable, I thought: they're gummy, they're jelly-ish, Jelly Babies are literally right there on the slide. But in a British school, **jelly means the wobbly dessert** — the 🍮 pudding-bowl kind that comes with ice cream at a birthday party. So when I said "these go in the jelly class," a fair chunk of the room pictured a dessert bowl, not a Goldbear. The younger pupils and the sixth formers alike were quietly resolving my labels against a different ontology.

The class didn't fall apart — children are generous, and sweets on a slide buy you a lot of goodwill. But the labelling scheme and the audience's vocabulary never fully synced, and I was the last person in the room to realise it.

---

## The accidental lesson: label taxonomy is a localisation problem

The irony is that this is *exactly* the kind of failure I deal with professionally. I stood in front of three classes explaining that classifiers are only as good as their labels — while using labels that didn't match the domain vocabulary of my users.

A few things I'd tell past-me before the next school visit:

**1. Labels are user-facing vocabulary, not internal convenience.** If the people consuming your classifier's output call the thing *sweets*, your class is called `sweets`, not `candy`. This is the same discipline as matching clinical terminology in health data: the label set has to come from the domain, not from the modeller's head.

**2. Test your taxonomy on a native speaker of the domain.** Five minutes with any British eight-year-old — or honestly, any British colleague — would have caught both problems. That's a label review, and it's cheaper than discovering the mismatch live in front of Year 2.

**3. Ambiguous labels don't just confuse models — they confuse annotators.** If I'd asked the pupils to sort the collage themselves (which would have been a great activity), the *jelly* class would have collected inconsistent items depending on what each child thought jelly meant. That's exactly how noisy training labels happen in real projects: not from carelessness, but from a label name that means different things to different annotators.

**4. The corrected taxonomy was sitting on my own slide.** *Sweets* / *chocolate* / and, if you want the third class, *gummies* or just *Haribo* — a brand name that every child in Britain resolves to precisely the right concept. The best label is the one your users already use.

---

## What I'd run next time

The talk I want to give next time is the same talk, with one change: hand the sorting to the room. Give each table the collage, ask them to invent their *own* class names, and compare the taxonomies between tables. Year 1 will produce "yummy / super yummy". Year 8 will argue about whether a Kinder Surprise is chocolate or a toy. Year 12 will reinvent hierarchical classification without being told.

And every one of those outcomes teaches the real lesson better than my slides did: before you train the model, agree on what the labels mean — with the people who'll actually use them.

The dog still doesn't get any chocolate, though. Some classifications are non-negotiable.
