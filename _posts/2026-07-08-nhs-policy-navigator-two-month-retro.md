---
layout: post
type: reflection
title: "From Build-in-a-Day to v1.1.0: Two Months of NHS Policy Navigator"
date: 2026-07-08
categories: [rag, engineering, retrospective]
tags: [nhs-policy-navigator, RAP, mongodb, gemini, agentic-rag, ci, retrospective]
summary: "A retro on the two-month journey of NHS Policy Navigator — from a one-day MongoDB hackathon build in May, through the move off the sponsored stack onto free tiers, to a Gold RAP product with a Query Router (v1.1.0) presented as a prototype in July."
---

In May 2026 I walked into the MongoDB Agentic Evolution Hackathon in London on the strength of a one-line pitch: *"Build in a day. Shape the future of AI Agents."* Two months later, [**NHS Policy Navigator**](https://github.com/chaeyoonyunakim/nhs-policy-navigator) — an adaptive retrieval agent over the NHS 10 Year Health Plan — is a documented, tested, CI-gated product at **v1.1.0**, running at zero cost, and I've just presented it as a prototype. This is the retro on those two months: what happened in each phase, and what each one taught me.

## Phase 1: the day it started (May)

The hackathon brief came with the stack chosen for me: the sponsored database and infrastructure, and a limited LLM model. That meant my first ever day on **MongoDB Atlas** and **AWS**, with **ElevenLabs** in the sponsor mix too — the only familiar face was LangSmith.

What made a one-day build survivable was everything transferable underneath: the AI engineering fundamentals didn't change just because the logos did. Within about an hour I was comfortable wiring an MVP dashboard, and the day produced a working proof of concept — a FastAPI backend, MongoDB Atlas vector and full-text search, Gemini for embeddings and generation, and a single-file front end.

But it was hackathon code: one flat folder, no tests, no CI, `print()` statements for logging, and an "adaptive" story that was more aspiration than implementation.

**Lesson: the stack is swappable; the fundamentals are the asset.** Being new to Atlas and AWS on the same morning cost me an hour, not the day.

## Phase 2: leaving the hackathon workspace (about two days)

The first post-hackathon job wasn't a feature — it was making the project *continuable* outside the sponsored environment, as I moved development towards an NHS tenant context.

- **Kept MongoDB Atlas unchanged** — the free M0 tier does vector and full-text search, so the core of the retrieval design survived intact. It also connected me with a team at NHS England already working with MongoDB, which turned a hackathon dependency into a professional conversation.
- **Dropped the AWS dependencies** entirely.
- **Pivoted the pricy design to free tier.** The hackathon configuration cost about £0.50 per run — trivial in a demo, disqualifying for continuous development on a side project. Rebuilt on the Gemini free tier for both embeddings and generation, the whole system now runs at **zero external cost**, with no paid observability platform and no text-to-speech provider. It can be handed over without anyone procuring anything.
- **Let the front end stay at hack level while the backend was overhauled.** A deliberate imbalance: the UI could wait; the foundations couldn't.

**Lesson: portability is a feature you build on purpose.** The £0.50-per-run design would quietly have killed the project; the free-tier constraint is why it still exists.

## Phase 3: structure before features (June, v1.0.0)

With the project able to live anywhere, I resisted the urge to build the shiny feature and instead repackaged everything to **Gold RAP** — the NHS Reproducible Analytical Pipeline standard: a proper `src/` package, environment-driven configuration with no hard-coded secrets, structured logging, a pytest suite, GitHub Actions CI, a changelog, and documentation that lives next to the code. That shipped as **v1.0.0** on 24 June.

It felt slow. It wasn't. Every feature afterwards landed on a base that tested itself across three Python versions (3.10–3.12) and refused to merge if anything broke. The test count went from 0 to 29 in that first PR and climbed to 49 by the end — all mocked, no network or database required — and not once did I ship a regression I had to chase in production. **The boring work bought me speed later.**

## Phase 4: the Query Router, and naming what's actually adaptive (June, v1.1.0)

The headline feature — shipped as **v1.1.0** the day after v1.0.0, which tells you how much the groundwork helped — was the **Query Router**. Every question is classified (factual, conceptual, comparative, or gap analysis), routed to the right sources, and given a retrieval strategy — keyword, vector, or hybrid — that can *switch* to whichever strategy has scored best historically for that kind of question, with the decision trail logged to MongoDB. Alongside it, every query is tagged with NHS-domain facets (care setting and professional group), and near-identical questions are deduplicated into a categorised digest: the front page shows the top-10 most-asked topics per category, while a separate Previous Queries tab keeps the full append-only history. A supervising LLM produces the executive summary; live news and publication feeds fill in what happened after the plan was published.

Partway through I asked a blunt question of my own codebase: *what here is genuinely adaptive, and what just looks it?* The honest answer was humbling. The strategy selection is real — it reads logged outcomes and changes behaviour — but it's **greedy** (once a strategy wins, the others stop being tried), it learns from the **model grading itself** rather than from users, and it averages over all time with no sense of recency. It's a bandit with the exploration switched off.

Writing that down, without flattering the work, turned out to be the most valuable artefact of the two months. It's the backlog for what comes next.

## Phase 5: process lessons the hard way

**Separate capability from presentation.** I tried to ship the Query Router as one pull request, bundling the backend feature and the UI on top of it. Splitting them into two reviewable PRs hit the obvious-in-hindsight wall: the UI commits *depended* on the feature, so they couldn't merge independently. The fix was a stacked pair — feature PR into `main`, UI PR retargeted after it merged — plus a rebase and a careful tree-equality check. **Decide your review boundaries before you write the code, not after.**

**Iterate the UI deliberately, not accidentally.** I'd taken inspiration from how the Gmail team uses iterative design alongside machine learning, and from the NHS England data visualisation community of practice. The iteration worked — annotated screenshots drove real improvements (dropping the example pills and stats panels, a plain-English "This query" legend, top-10 per category) — but it churned across several PRs. A front-loaded UI spec would have turned four rounds into one.

**The tooling will surprise you.** Importing the UI design from Claude Design didn't work in the running session, so the handoff became a plain git patch; publishing the `v1.1.0` git tag was blocked by the environment, so tag and release notes go out manually. Neither was a disaster — both cost time I hadn't budgeted. **Confirm the mechanics of your handoffs and your release process early**, while they're cheap to fix.

## What I'm proud of

- A one-day hackathon build is now a documented, tested, CI-gated, versioned product — **v1.1.0** — that still costs nothing to run.
- The documentation tells the truth about what the system does, including where the "adaptive" label is aspirational.
- Graceful degradation everywhere: tagging, dedup routing, news and publication fetches, and re-ranking all fail soft — a failure never blocks an answer.
- Small, green, reversible increments the whole way.

## What's next

Presenting the prototype produced its own backlog of open questions — the ones I put on my final slide:

- **Feedback from potential users** — the single most important input, and the missing reward signal.
- **More example questions**, and possibly a **TL;DR answer** mode.
- **Benchmark / evaluation** — an offline harness with a labelled query set, so retrieval changes are validated rather than self-reported.
- **Functions beyond summarisation**, and additional dashboard components.
- **Extending the corpus with the 10 Year Workforce Plan** — and deciding the right pairing: workforce plan alongside the 10 Year Health Plan, or bridging back to the Long Term Workforce Plan.

Under the hood, the adaptive honesty audit sets the engineering order: a genuine user feedback signal instead of the model marking its own homework, ε-greedy exploration so strategies keep being sampled, recency-weighted scoring so the system tracks drift, and reciprocal-rank fusion in place of concatenate-and-dedupe.

The demo answered questions. The product should get better at answering them.

---

*NHS Policy Navigator is built on free-tier MongoDB Atlas and Google Gemini. Live demo: <https://nhs-policy-navigator.vercel.app>*
