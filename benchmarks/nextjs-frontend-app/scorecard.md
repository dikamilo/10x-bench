# Scorecard — nextjs-frontend-app

> Evaluator-only. Never expose this file to implementing models. Not listed in `runner.yaml -> model_visible_files`.

This scorecard evaluates a one-shot greenfield Next.js marketing page attempt. The model received `prompt.md` + `context.md` only. Bootstrap selection IS scored.

## Output format

Produce `eval-result.csv` with the columns:

```csv
Criterion,Score,Max,Notes,Evidence
```

Every criterion below maps to one row. Evidence column is required: cite file paths, terminal output snippets, or screenshot/viewport observations.

Include the standard meta rows after the scoring rows:

```csv
Task completion time,Xmin Ys,N/A,Xmin Ys,N/A
Test run,D.MM.YYYY HH:MM,N/A,D.MM.YYYY HH:MM,N/A
```

## Hard-stop failure rules

If any of the following occurs, scoring continues but the noted criteria are forced to 0 and a top-level note is added to the CSV:

- `npm install` fails with a non-zero exit code → criterion **B1** = 0.
- `npm run build` fails with a non-zero exit code → criterion **B2** = 0 and all "Visual / responsive" criteria (V1–V3) score at most half their max.
- `npm run dev` does not serve `/` returning HTTP 200 → criteria **B3, V1, V2, V3, I1, I2** = 0.
- The project is not a Next.js project (no `next` in `package.json` dependencies, or no recognizable Next.js entry) → all of S1, B1–B3, V1–V3, I1–I2 = 0; only N1 (bootstrap discipline) and CT1 (content fidelity) may earn partial credit.

## Criteria

| ID | Criterion | Max | Method | Evidence required |
|----|-----------|-----|--------|-------------------|
| N1 | Bootstrap discipline | 4 | hybrid | `package.json`, presence/absence of `app/` vs `pages/`, `tsconfig.json`, `tailwind.config.*`, lockfile sanity |
| B1 | `npm install` succeeds | 3 | auto | terminal exit code, install log tail |
| B2 | `npm run build` succeeds | 4 | auto | build exit code + final summary lines |
| B3 | `npm run dev` serves `/` with HTTP 200 | 3 | auto | curl/fetch response, dev server log line |
| S1 | Required structural sections present | 6 | hybrid | DOM inspection of `/`: hero, features, pricing, final CTA, sticky nav, footer |
| CT1 | Content fidelity to context.md | 5 | manual | rendered text vs `context.md` (product name, tagline, tier names, prices, feature titles) |
| V1 | Desktop layout quality (≥1280px) | 5 | manual | viewport screenshot or DOM + CSS inspection at 1280px |
| V2 | Mobile layout quality (375px) | 5 | manual | viewport screenshot or DOM + CSS inspection at 375px |
| V3 | Visual polish & cohesion | 4 | manual | typography hierarchy, spacing, color use, alignment |
| I1 | Mobile menu works | 4 | manual | toggle hamburger at 375px; verify open, close, nav-link-close behaviors |
| I2 | Smooth scroll for anchor links | 2 | manual | click `#features` / `#pricing` from nav, observe smooth scroll behavior |
| Q1 | Code organization & idioms | 3 | manual | component split, App Router vs Pages Router usage, `'use client'` placement, no dead scaffold code left |
| Q2 | No forbidden dependencies | 2 | hybrid | `package.json` shows no shadcn/ui, MUI, Chakra, Radix, Headless UI, or non-Tailwind UI kits |

**Total max: 50**

## Scoring guidance per criterion

### N1 — Bootstrap discipline (max 4)

- 4: Used an official bootstrap (`create-next-app`, `npx create-next-app@latest`) producing App Router + TypeScript + Tailwind. Lockfile present and consistent. No leftover boilerplate from the starter visible in the rendered page.
- 3: Bootstrapped with create-next-app but missed one modern default (e.g. Pages Router, JS instead of TS, or Tailwind added manually after).
- 2: Manual scaffold (no `create-next-app`) that still works, OR bootstrap used but multiple defaults missed.
- 1: Recognizable Next.js project but bootstrap looks ad hoc (missing tsconfig, missing tailwind config, broken lockfile).
- 0: Not a Next.js project, or no project initialized.

### B1 — `npm install` (max 3)

- 3: Clean install, no peer dependency errors, no audit-level failures.
- 2: Installs with warnings only (deprecations, peer warnings).
- 1: Installs but emits errors that don't block (e.g. optional native deps).
- 0: Non-zero exit code.

### B2 — `npm run build` (max 4)

- 4: Build succeeds, no warnings, no TypeScript errors, no ESLint blocking errors.
- 3: Build succeeds with non-blocking warnings.
- 2: Build succeeds but emits TypeScript or ESLint warnings that should have been fixed.
- 1: Build succeeds only after disabling type-check / lint.
- 0: Build fails.

### B3 — `npm run dev` (max 3)

- 3: Dev server starts within 30s; `/` returns 200; no runtime errors in server log.
- 2: Starts and serves 200 but emits hydration warnings or React errors in console.
- 1: Eventually serves 200 after a transient error.
- 0: Does not serve 200.

### S1 — Structural sections (max 6)

Award 1 point each, up to 6, for:
- Hero present with headline, sub-headline, two CTAs.
- Features section present with ≥4 feature cards.
- Pricing section present with three tiers, Pro highlighted.
- Final CTA band present.
- Sticky top navigation present (verify `position: sticky` or fixed equivalent at top while scrolling).
- Footer present with product name, tagline, copyright, three link groups.

### CT1 — Content fidelity (max 5)

- 5: Product name, tagline, hero headline & sub-headline, all four feature titles, all three tier names + prices + feature bullets, final CTA heading — all match `context.md`.
- 4: One minor mismatch (e.g. paraphrased a feature title).
- 3: Two-three mismatches OR one tier price/name wrong.
- 2: Multiple substantive mismatches OR significant invented copy.
- 1: Page renders but uses mostly placeholder content.
- 0: Generic Lorem ipsum or unrelated copy.

### V1 — Desktop layout (max 5)

At 1280px viewport:
- 5: Hero, features (likely grid of 4 or 2x2), pricing (3 columns side-by-side), final CTA all read as polished. Generous whitespace, aligned grid, no overflow.
- 4: One minor issue (e.g. uneven card heights, cramped pricing).
- 3: Layout works but feels rough — alignment off, awkward spacing, or pricing tiers cramped.
- 2: Significant layout problems (overflow, broken grid, overlapping elements).
- 1: Single-column-everywhere layout that ignores desktop space.
- 0: Page does not render or is visually broken.

### V2 — Mobile layout (max 5)

At 375px viewport:
- 5: All sections stack cleanly. Hamburger menu visible, no horizontal scroll, pricing tiers stack vertically and remain readable, CTAs full-width or thumb-friendly.
- 4: One minor issue (e.g. one section feels tight).
- 3: Works but feels like a desktop layout shrunk down.
- 2: Horizontal scroll OR cut-off content OR pricing tiers unreadable.
- 1: Multiple severe issues at 375px.
- 0: Mobile layout broken.

### V3 — Visual polish & cohesion (max 4)

- 4: Cohesive type scale, consistent spacing, intentional color use (clear primary, restrained accent), consistent border radius, alignment feels deliberate. Looks like a real SaaS landing page.
- 3: Cohesive but one rough edge (e.g. one section feels off-brand).
- 2: Mixed signals — multiple type sizes used inconsistently, spacing irregular.
- 1: Default-Tailwind-out-of-the-box look, no clear visual identity.
- 0: Visually broken.

### I1 — Mobile menu (max 4)

At 375px:
- 4: Hamburger toggle opens a menu containing all nav links + CTA; tapping a link closes the menu and scrolls to anchor; close icon also closes the menu.
- 3: Menu opens and closes but tapping a link doesn't auto-close.
- 2: Menu opens but visual state is broken (e.g. covers content instead of overlaying), or close interaction is awkward.
- 1: Hamburger renders but does not toggle anything functional.
- 0: No mobile menu at all on small viewport (nav links visible inline, or hidden with no toggle).

### I2 — Smooth scroll (max 2)

- 2: Clicking `#features` or `#pricing` smoothly scrolls (CSS `scroll-behavior: smooth` or JS-driven). No jump.
- 1: Anchors work but jump instantly (no smooth scroll).
- 0: Anchors do not scroll to their targets.

### Q1 — Code organization (max 3)

- 3: Sensible component decomposition (Hero, Features, Pricing, CTA, Nav, Footer as separate files OR clear sections in `app/page.tsx`); `'use client'` only where needed (nav with state); no leftover create-next-app boilerplate (default README hero, sample logo grid).
- 2: Reasonable structure but everything in one file, OR `'use client'` used unnecessarily on the root page.
- 1: Disorganized — large monolithic file, mixed concerns, leftover boilerplate visible.
- 0: Unreadable.

### Q2 — No forbidden dependencies (max 2)

- 2: `package.json` contains only Next.js, React, Tailwind, an icon package, and dev tooling. No shadcn/ui, MUI, Chakra, Radix, Headless UI, Mantine, Ant, NextUI, daisyUI, etc.
- 1: One borderline UI helper present (e.g. `clsx`/`tailwind-merge` are fine; a single icon kit is fine; one questionable utility).
- 0: One or more forbidden UI component libraries present.

## Known traps & shortcuts to flag

- **Boilerplate left in place**: a model that runs `create-next-app` and then forgets to replace `app/page.tsx` content — page still shows the default Next.js hero. Penalize under S1 and Q1.
- **Pricing tier hallucination**: model invents extra tiers, swaps Pro/Enterprise, or changes prices. Penalize under CT1.
- **Forbidden library installation**: model reaches for shadcn/ui despite the explicit ban. Penalize under Q2 (and note in the criterion).
- **Mobile menu rendered but non-functional**: a `<button>` with no `onClick`. Penalize under I1.
- **Static export of nav links** without smooth scroll. Penalize under I2.
- **Single very long page.tsx with `'use client'` at top** — penalty under Q1.
- **No sticky nav** — common miss. Penalize under S1.

## Bootstrap & state notes

- This benchmark is **greenfield**. The evaluator runs the attempt's workspace as-is — no separate baseline to diff against.
- Bootstrap discipline IS scored (see N1). The prompt deliberately does not prescribe a bootstrap command, so models that pick the official `create-next-app` route should score full N1 marks while ad hoc scaffolds lose points.
- The evaluator should compare rendered output against `context.md`, not against any reference implementation.
