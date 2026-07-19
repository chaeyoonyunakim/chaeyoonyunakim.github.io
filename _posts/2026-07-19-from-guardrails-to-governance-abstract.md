---
layout: post
type: reflection
title: "From Guardrails to Governance: Teaching Reasoning About AI-Assisted Code in Safety-critical Contexts"
date: 2026-07-19
categories: [ai-engineering, education, governance]
tags: [eu-ai-act, ai-ethics, nhs, RAP, formal-methods, agentic-ai, teaching, kiro, claude-code]
summary: "Talk abstract: an AI Engineering teaching initiative in the NHS where specification, invariants, and testing are taught through the lens of regulatory accountability — reframing formal methods from arcane overhead into necessary practice."
---

*This is the abstract for my talk "From Guardrails to Governance: Teaching Reasoning About AI-Assisted Code in Safety-critical Contexts" (Chaeyoon Kim, NHS England).*

## Abstract

As AI coding assistants permeate software development, programming education faces a structural misalignment: learners increasingly produce syntactically correct code without being able to reason about its correctness, safety, or regulatory implications. This gap is consequential across safety-critical sectors. In public-sector contexts such as the UK National Health Service (NHS), AI-generated code may encode bias, violate clinical guardrails, or fail EU AI Act compliance obligations for high-risk healthcare systems. The same reasoning deficit carries similarly serious consequences in high-confidentiality private-sector domains such as semiconductor design and manufacturing, where IP protection, supply-chain integrity, and the correctness of AI-driven control systems are subject to stringent regulatory and export-control obligations – and where an unverified pipeline's failure may be invisible until it propagates into fabricated silicon.

This talk reports on an AI Engineering teaching initiative using open-source NHS data as primary instructional artefacts, with a cohort of five learners united by NHS experience but occupying markedly different positions within it. It addresses two questions: first, whether traditional teaching practices can be taught alongside agentic AI tools rather than in opposition to them; and second, whether grounding these practices in an AI ethics framework drawn from the EU AI Act (accountability, transparency, fairness, human oversight, and auditability) makes correctness reasoning more motivating when learners already inhabit the safety-critical domain the code must serve.

Several teaching repositories operationalised the approach, featuring clinical and political guardrails, requirement-to-task traceability, and a prototype dashboard with user journey for different personas. HeartLink introduces a spec-driven prediction modelling pipeline built with the specification workflow on AWS's AI IDE (Kiro). The second is an analytical pipeline built under supervised Claude Code agentic development, following NHS England's Reproducible Analytical Pipeline principles. Lastly, the NHS Policy Navigator supports an agentic system teaching on how to debug an error while a loop of conversation with AI.

Across the cohort, framing correctness requirements as governance artefacts substantially reduced the perceived barrier to formal thinking, regardless of academic stage or professional seniority. Existing NHS domain knowledge (clinical, operational, or analytical) accelerated engagement with property-based tests and reproducibility manifests, as learners could readily articulate what a pipeline failure would mean in practice. Pre/post-conditions and invariants became legible when expressed as technical evidence for documented acceptance criteria and regulatory obligations where applicable. Learners became more confident evaluating AI-generated code critically when they could name which specification property or EU AI Act obligation a given output violated.

Reasoning about program correctness is not separate from AI ethics. Teaching specification, invariants, and testing through the lens of regulatory accountability reframes formal methods from arcane overhead into necessary practice. The cohort described here suggests that existing domain expertise in safety-critical settings is itself a pedagogical resource: learners who already understand what is at stake when a pipeline fails are strongly motivated to acquire the formal vocabulary to prevent it. When AI ethics is introduced as the motivating framework, learners are not just asked whether their code works, but whether they can prove it, audit it, and defend it to a clinician, a policymaker, or a regulator after experiencing a replicable model for programming education in the age of responsible AI. Although this talk is limited to present the educational experience conducted within the NHS, there is a large possibility to extend the precursors for the other public-sector area and the high-confidentiality private-sector domains.

## References

1. [heartlink-kiro-nhs](https://github.com/chaeyoonyunakim/heartlink-kiro-nhs)
2. [pwr-workforce-elasticity-modelling](https://github.com/chaeyoonyunakim/pwr-workforce-elasticity-modelling)
3. [nhs-policy-navigator](https://github.com/chaeyoonyunakim/nhs-policy-navigator)

## Biography

Data Scientist based in London. Certified AI Ethicist with four years' experience in the NHS ecosystem. Lead Developer for community pharmacy modelling on the 10-Year Workforce Plan, working jointly with NHS England and the Department of Health and Social Care. Specialised in AI engineering.
