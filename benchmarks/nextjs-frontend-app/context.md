# Product Context: Pulseboard

This is the content you must use for the landing page. Use the copy verbatim where provided. You may rewrite filler descriptive sentences for flow, but you MUST preserve product name, tier names, prices, and the feature list verbatim.

## Brand

- **Product name**: Pulseboard
- **Tagline (one-liner)**: "Real-time analytics for product teams who hate dashboards."
- **Tone**: confident, slightly irreverent, technical but not jargon-heavy
- **Color direction**: deep indigo primary (`#4F46E5`-ish), near-black backgrounds for hero/CTA bands, white surfaces for feature and pricing cards, soft slate text. The model may pick the exact palette as long as contrast is accessible.

## Hero section

- **Headline**: "Ship the metric that matters. Skip the dashboard."
- **Sub-headline**: "Pulseboard turns your product events into a single shared signal — one number, updated every second, visible to your whole team."
- **Primary CTA label**: "Start free trial"
- **Secondary CTA label**: "View pricing"
- **Hero visual**: a placeholder is fine — a styled mock chart, a gradient block, a stylized SVG. Do not embed external images that require network access at build time.

## Features section

- **Section eyebrow**: "Why Pulseboard"
- **Section heading**: "Built for teams that already drowned in dashboards."

Use these four features. Keep titles exact; descriptions may be lightly polished.

1. **Live signal, not stale charts** — Every metric is a real-time stream. No 24-hour delay, no nightly batch jobs, no "last refreshed" disclaimer.
2. **One number per goal** — Each team picks the single metric that defines success this quarter. Pulseboard makes that number unmissable.
3. **Wired into your stack** — Drop-in SDKs for Node, Python, Go, and Rust. Send an event in 30 seconds, see it on the board immediately.
4. **Alerts that don't cry wolf** — Threshold and anomaly alerts route to Slack, email, or webhook — but only when the signal sustains, not on a single spike.

Pick any clean line-icon style (e.g. lucide-react, heroicons, or inline SVGs you draw yourself). Do not use emoji as feature icons.

## Pricing section

- **Section eyebrow**: "Pricing"
- **Section heading**: "One signal. Three plans."
- **Section sub-heading**: "Start free. Upgrade when your team outgrows it."

Three tiers. Use these names, prices, and feature lists verbatim. Highlight **Pro** as the recommended tier.

### Starter — $0 / month

- 1 project
- 100k events / month
- 7-day retention
- Slack alerts
- Community support

### Pro — $49 / month per seat (recommended)

- Unlimited projects
- 10M events / month
- 90-day retention
- Slack, email, and webhook alerts
- Anomaly detection
- Priority email support

### Enterprise — Custom

- Unlimited events
- Custom retention
- SSO and SCIM
- Dedicated VPC option
- 99.99% SLA
- Named customer success engineer

CTA labels per tier: Starter → "Start free", Pro → "Start 14-day trial", Enterprise → "Contact sales". Pricing links may all point to `#`.

## Final CTA section

- **Heading**: "Stop staring at dashboards. Ship the number."
- **Body**: "Free for 14 days. No credit card. Two-minute setup."
- **Primary CTA label**: "Start free trial"

## Footer

- **Tagline line under product name**: "Real-time analytics for product teams."
- **Copyright**: "© 2026 Pulseboard, Inc."
- Three link groups, three+ items each. Suggested labels (you may adjust):
  - **Product**: Features, Pricing, Changelog, Status
  - **Company**: About, Careers, Blog, Contact
  - **Legal**: Privacy, Terms, Security, DPA

All link `href` values may be `#`.

## Non-goals (do not implement)

- No authentication, no signup form, no actual newsletter capture.
- No backend, no API routes (this benchmark is frontend-only).
- No CMS integration. Content is hardcoded in the components.
- No internationalization. English only.
- No dark/light mode toggle. Pick one cohesive theme and ship it.
- No tests, no Storybook, no Playwright, no Cypress.
- No analytics scripts, no third-party tag managers, no external font CDNs that require network access during build (Next.js' `next/font` is fine).
