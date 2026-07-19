# Design file — "From Guardrails to Governance" talk deck

NHS England colour theme and slide styles for the presentation
**From Guardrails to Governance: Teaching Reasoning About AI-Assisted Code in Safety-critical Contexts**
(Chaeyoon Kim, NHS England).

This file is the single source of truth for the deck's visual language.
When populating or adding slides in `index.html`, follow the tokens and
layout rules below — do not invent new colours or type sizes.

---

## 1. Colour tokens (NHS identity palette)

All colours come from the NHS visual identity guidelines. They are defined
as CSS custom properties on `:root` in `index.html`.

### Primary

| Token | Hex | Use |
|---|---|---|
| `--nhs-blue` | `#005EB8` | Primary brand colour: title slide, headers, links, key accents |
| `--nhs-dark-blue` | `#003087` | Section-divider slides, footer bar, emphasis backgrounds |
| `--nhs-bright-blue` | `#0072CE` | Interactive states, secondary accents |
| `--nhs-white` | `#FFFFFF` | Slide backgrounds, text on dark backgrounds |

### Neutrals

| Token | Hex | Use |
|---|---|---|
| `--nhs-black` | `#231F20` | Body text on light backgrounds |
| `--nhs-dark-grey` | `#425563` | Secondary text, captions, slide numbers |
| `--nhs-mid-grey` | `#768692` | Borders, dividers, de-emphasised text |
| `--nhs-pale-grey` | `#E8EDEE` | Card backgrounds, code blocks, alternating panels |

### Support (use sparingly — one accent per slide)

| Token | Hex | Use in this deck |
|---|---|---|
| `--nhs-green` | `#009639` | Positive findings, "what worked" |
| `--nhs-aqua-green` | `#00A499` | HeartLink repo accent |
| `--nhs-purple` | `#330072` | EU AI Act / governance framing accent |
| `--nhs-pink` | `#AE2573` | NHS Policy Navigator repo accent |
| `--nhs-warm-yellow` | `#FFB81C` | Warnings, risk callouts, highlight underlines |
| `--nhs-emergency` | `#DA291C` | Reserved: failure/risk statements only (e.g. "invisible until fabricated silicon") |
| `--nhs-light-blue` | `#41B6E6` | RAP pipeline repo accent, charts |

Contrast rules: body text is `--nhs-black` on white or white on
`--nhs-blue`/`--nhs-dark-blue`. Never place `--nhs-light-blue` or
`--nhs-warm-yellow` text on white; use them for fills, rules and tags only.

## 2. Typography

NHS brand typeface is Frutiger (licensed). The web-safe fallback stack used
throughout, matching the nhs.uk frontend:

```css
font-family: "Frutiger W01", Arial, Helvetica, sans-serif;
```

Type scale (slides are 1280×720 design units; sizes in `rem` at 16px base):

| Role | Size | Weight | Colour |
|---|---|---|---|
| Deck title (title slide) | 3.25rem | 700 | white on NHS Blue |
| Section divider heading | 2.75rem | 700 | white on NHS Dark Blue |
| Slide heading (h2) | 2.125rem | 700 | `--nhs-blue` |
| Sub-heading (h3) | 1.375rem | 700 | `--nhs-dark-grey` |
| Body / bullets | 1.25rem | 400 | `--nhs-black` |
| Callout / quote | 1.5rem | 400 italic | `--nhs-dark-blue` |
| Caption, footer, slide no. | 0.875rem | 400 | `--nhs-dark-grey` |

Bullets: maximum 5 per slide, sentence case, no full stops on fragments.

## 3. Slide layouts

Each slide is a `<section class="slide">` plus one layout modifier:

| Class | Purpose |
|---|---|
| `slide--title` | Title slide: NHS Blue background, white text, speaker block |
| `slide--section` | Section divider: NHS Dark Blue background, big heading, section number |
| `slide--content` | Default: white background, blue h2 with yellow underline rule |
| `slide--two-col` | Content slide with `.col` children in a 2-column grid |
| `slide--closing` | Thank-you/references slide: NHS Blue background |

Reusable components (see `index.html` for markup):

- `.tag` — small pill label; colour-coded per repo (`.tag--heartlink`,
  `.tag--rap`, `.tag--navigator`) and per theme (`.tag--ethics`, `.tag--risk`)
- `.card` — pale-grey panel with 4px left rule in an accent colour
- `.callout` — quote/finding block, NHS Blue left border
- `.risk` — risk statement block, emergency-red left border (use once, maybe twice, in the whole deck)
- `.principles` — 5-across grid for the EU AI Act principles
- `.kbd-hint`, `.slide-no`, `.footer-bar` — chrome; do not edit per slide

## 4. Populating the deck

- Placeholder text is wrapped in `[square brackets]` — replace it, keep the
  surrounding markup.
- Keep one idea per slide; move overflow to a new `slide--content` section
  rather than shrinking type.
- Accent discipline: repo slides use their assigned accent; findings slides
  use green; risk slides use emergency red; everything else stays blue.
- Navigation: ← → / space / click; `p` toggles print view (one slide per
  page, for PDF export via the browser's print dialog).

## 5. Narrative arc (slide order)

1. Title
2. The structural misalignment (problem)
3. Why it matters: safety-critical stakes (NHS + semiconductor)
4. Section — The initiative
5. Cohort & setting (5 learners, open NHS data)
6. Two research questions
7. Section — Three teaching repositories
8. HeartLink (Kiro, spec-driven)
9. RAP analytical pipeline (supervised Claude Code)
10. NHS Policy Navigator (debugging-in-dialogue)
11. Section — What we learned
12. EU AI Act as motivating framework (5 principles)
13. Findings: governance framing lowers the barrier
14. Findings: domain expertise as pedagogical resource
15. Conclusion: correctness reasoning *is* AI ethics
16. Beyond the NHS + thank you / references
