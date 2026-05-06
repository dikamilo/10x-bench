# Task: Build a Marketing Landing Page in Next.js

You will create, from scratch, a single-page marketing website for a fictional SaaS startup using Next.js. The product details, copy, and assets you need are in `context.md`. Treat that file as the single source of truth for all visible content.

## State mode

**Greenfield.** The workspace starts empty. You must initialize the Next.js project yourself before writing any feature code. Choosing a sensible bootstrap is part of the task.

## Required stack

- **Next.js** (latest stable; App Router preferred)
- **TypeScript**
- **Tailwind CSS** for styling
- **React 18+ / 19** as bundled by Next.js

Do not introduce additional UI component libraries (no shadcn/ui, MUI, Chakra, Radix, Headless UI, etc.). Tailwind utility classes only. Lucide / Heroicons-style SVG icons inlined or imported from a single icon package are acceptable.

## Required page sections

The landing page lives at `/` and must contain these four sections, in this order:

1. **Hero** — product name, headline, sub-headline, two CTAs (primary "Start free trial", secondary "View pricing"), and a hero visual placeholder.
2. **Features** — at least four feature cards, each with an icon, title, and short description. Use the feature list provided in `context.md`.
3. **Pricing** — three pricing tiers (Starter, Pro, Enterprise) presented side-by-side, with the middle tier visually highlighted as recommended. Use the exact tier names, prices, and feature lists from `context.md`.
4. **Final CTA** — a closing call-to-action band repeating the primary CTA with a short reinforcing message.

In addition, the page must include:

- A **sticky top navigation bar** with the product name/logo on the left and links to `#features`, `#pricing`, and a primary CTA on the right.
- A **footer** with the product name, a one-line tagline, a copyright line, and three plain-text link groups (Product, Company, Legal — at least three items each, hrefs may be `#`).

## Required interactivity

- The top navigation must collapse into a **hamburger / mobile menu** on small viewports (≤ 768px wide). Tapping the hamburger toggles a panel exposing the same nav links plus the CTA. Tapping a link or the close icon closes the panel.
- All in-page anchor links (`#features`, `#pricing`, etc.) must use **smooth scrolling** to their targets.

These are the only interactive behaviors required. No forms, no animations beyond smooth scroll, no client data fetching.

## Responsive design

The page must look polished and intentional at both:

- **Mobile**: 375px wide
- **Desktop**: 1280px wide and above

No content may be cut off, overflow horizontally, or overlap at either breakpoint. Mobile and desktop layouts may differ — they should both feel finished, not just shrunk down.

## Deliverables

A single Next.js project at the workspace root that:

- Installs cleanly with `npm install` (or the package manager the bootstrap chose).
- Builds successfully with `npm run build` (or equivalent).
- Runs in development with `npm run dev` (or equivalent) and serves the landing page at `http://localhost:3000/`.
- Renders the four required sections, the sticky nav with working mobile menu, and the footer.
- Uses content drawn from `context.md`.

## Constraints

- This is a **one-shot** task. Do not ask follow-up questions during the attempt. Make reasonable decisions and proceed.
- Do not include placeholder Lorem ipsum where `context.md` provides real copy. Where `context.md` is silent (e.g. footer link labels), invent reasonable, professional content.
- Do not deploy anywhere. The deliverable is local code only.
- Do not write tests unless they are part of the bootstrap output you choose to keep.

## Definition of done

- `npm install` succeeds.
- `npm run build` exits with code 0.
- `npm run dev` starts a server on port 3000 and `/` returns HTTP 200 with the rendered landing page markup.
- The four sections, nav, mobile menu, and footer are all visible and styled.
