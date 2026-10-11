# Intent — Unmarkdown rewrite

Confirmed 2026-10-11 via interview-me.

- Outcome:      Rewrite Unmarkdown in Astro + React islands, deployed on Cloudflare Pages
- User:         Solo dev now, collaborators later
- Why now:      Single file is messy; clean ultrafast base that invites contributions
- Success:      Same UX (two-step input→reading, TOC rail, word count + read time) with near-zero-JS shell + one React island, live on Pages
- Constraint:   Static-only, local-only (no backend, no uploads), `npm run dev/build` acceptable
- Out of scope: New user-facing features in the rewrite; backend/sync; non-React islands

## Decisions already locked
- Stack: Astro + React island + TypeScript
- Hosting: Cloudflare Pages (`npm run build` → `dist/`)
- Font: JetBrains Mono everywhere (body, headings, UI, code)
- Legacy reference: `unmarkdown/md-view.html` (single-file v1, keep until rewrite ships)
