---
layout: post
type: reflection
title: "Turning a Hackathon RAG Demo into a Reproducible Product — NHS Policy Navigator Sprint 01 Retro"
date: 2026-07-08
categories: [rag, engineering, retrospective]
tags: [nhs-policy-navigator, RAP, mongodb, gemini, agentic-rag, ci, retrospective]
summary: "Sprint 01 of NHS Policy Navigator is done: v1.0.0 (Gold RAP repackaging) and v1.1.0 (Query Router) shipped to main on zero running cost. The retro, honestly told — including which parts of 'adaptive' are real and which are still aspiration."
---

I came out of the MongoDB Agentic Evolution Hackathon (London, May 2026) with a working proof of concept: [**NHS Policy Navigator**](https://github.com/chaeyoonyunakim/nhs-policy-navigator), an adaptive retrieval agent over the NHS 10 Year Health Plan. It did the job in a demo — but it was hackathon code. One flat folder, no tests, no CI, `print()` statements for logging, and an "adaptive" story that was more aspiration than implementation.

Sprint 01 had two goals: make it a product I'd be comfortable handing to someone else, and actually build the adaptive layer the theme promised — all while keeping the project at **zero running cost** (free-tier MongoDB Atlas and Gemini only).

The sprint is now closing, with **v1.0.0** (the Gold RAP repackaging) and **v1.1.0** (the Query Router) merged to `main` and deployed on Vercel across eight PRs. This is the retro.

## Lesson 1: structure before features

The first thing I did was resist the urge to build the shiny feature. Instead I repackaged the project to **Gold RAP** — the NHS Reproducible Analytical Pipeline standard: a proper `src/` package, environment-driven configuration with no hard-coded secrets, structured logging, a pytest suite, GitHub Actions CI, a changelog, and documentation that lives next to the code.

It felt slow. It wasn't. Every feature I built afterwards landed on a base that tested itself across three Python versions (3.10, 3.11, 3.12) and refused to merge if anything broke. The test count went from 0 to 29 in that first PR and climbed to 49 by the end of the sprint — all mocked, no network or database required — and not once did I ship a regression I had to chase in production. **The boring work bought me speed later.**

## Lesson 2: name what's actually adaptive

The headline feature was the **Query Router**. Every question is classified, routed to the right sources, and given a retrieval strategy — keyword, vector, or hybrid — that can *switch* from a sensible default to whichever strategy has scored best historically for that kind of question, with the decision trail logged to MongoDB. Near-identical questions are deduplicated into a categorised digest so the front page shows the top-10 most-asked topics per NHS care setting, while a separate tab keeps the full append-only history. Keeping those two surfaces apart — the curated digest and the complete log — avoided conflating "highlights" with "history".

Partway through I asked a blunt question of my own codebase: *what here is genuinely adaptive, and what just looks it?* The honest answer was humbling. The strategy selection is real — it reads logged outcomes and changes behaviour — but it's **greedy** (once a strategy wins, the others stop being tried), it learns from the **model grading itself** rather than from users, and it averages over all time with no sense of recency. It's a bandit with the exploration switched off.

Writing that down, without flattering the work, turned out to be the most valuable artefact of the sprint. It's the backlog for Sprint 02.

## Lesson 3: separate capability from presentation

I tried to ship the Query Router as one pull request. It bundled the backend feature and the UI on top of it, and when I wanted to split them into two reviewable PRs I hit the obvious-in-hindsight wall: the UI commits *depended* on the feature, so they couldn't merge independently.

The fix was a **stacked pair** — the feature PR into `main`, the UI PR based on the feature branch and retargeted once it merged. It worked, but it needed a rebase and a careful tree-equality check to be sure the stack still added up to the same code. The cleaner move would have been to keep the capability and its presentation on separate commits from the start. **Decide your review boundaries before you write the code, not after.**

The UI itself churned more than it needed to, for a related reason: several changes (dropping the example pills, dropping the stats panels, replacing the counter with a "This query" legend, top-10 per category) came from annotated screenshots, iterated PR by PR. Screenshot-driven iteration is fast and effective — but a front-loaded UI spec would have turned four rounds of churn into one.

## Lesson 4: the tooling will surprise you

Two things I assumed were automatic weren't. Importing the UI design from Claude Design didn't work in the running web session — the design connector couldn't authorise, and "Send to Claude Code Web" seeds a *new* session rather than the one doing the work — so the handoff became a plain git patch. And when cutting the release, publishing the `v1.1.0` git tag was blocked by the environment, so the tag and release notes have to be published manually. Neither was a disaster — but both cost time I hadn't budgeted. **Confirm the mechanics of your handoffs and your release process early**, while they're cheap to fix.

## What I'm proud of

- A hackathon demo is now a documented, tested, CI-gated, versioned product — shipped as **v1.1.0** — that still costs nothing to run.
- The documentation tells the truth about what the system does, including where the "adaptive" label is aspirational.
- Graceful degradation everywhere: tagging, dedup routing, news and publication fetches, and re-ranking all fail soft — a failure never blocks an answer.
- Small, green, reversible increments the whole way.

## What Sprint 02 is for

Make the adaptation real, roughly in this order:

1. **A genuine user feedback signal** — thumbs-up/down that writes a real reward field, instead of the model marking its own homework.
2. **Exploration in strategy selection** — replace the greedy "pick the best average" with an ε-greedy / bandit approach so strategies keep being sampled.
3. **Recency-weighted scoring** — decay old scores so the system tracks drift instead of averaging over all time.
4. **Proper hybrid fusion** — reciprocal-rank fusion instead of concatenate-and-dedupe.
5. **An offline evaluation harness** — a labelled query set and a CI metric, so retrieval changes are validated rather than self-reported.

Plus two pieces of hygiene carried over: lock the UI spec before implementation, and publish the `v1.1.0` tag and release to close the signposting gap.

The demo answered questions. The product should get better at answering them.

---

*NHS Policy Navigator is built on free-tier MongoDB Atlas and Google Gemini. Live demo: <https://nhs-policy-navigator.vercel.app>*
