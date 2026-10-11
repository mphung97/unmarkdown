# Unmarkdown — SPEC (Astro + React rewrite)

## 1. Objective
Rewrite the single-file `md-view.html` prototype as an Astro + React-islands app
deployed on Cloudflare Pages. Same UX, clean component structure, ultrafast
initial load. No backend; everything stays in the browser.

## 2. UX contract (must-match v1)
- **Step 1 — Input:** brand header (Unmarkdown + tagline), textarea, Sample /
  Clear / Render buttons. Last paste restored from localStorage.
- **Step 2 — Reading (on Render):** full-width doc (1120px, same as input),
  brand header kept, Edit / New paste / Contents buttons.
- **Markdown rendering:** h1–h3 with anchor ids, task checkboxes (`- [x]`),
  tables, fenced code, blockquotes, `**bold**`, `` `code` ``, links.
- **TOC:** built from h1–h3 (shown when 2+ sections). Desktop ≥1280px: floating
  sticky rail right of viewport + Contents toggle. Below: collapsible
  `<details>` above doc. Stats (`N words · M min read · L lines`) live inside
  the TOC block; slim fallback line above doc when no TOC.
- **Stats:** words exclude fenced code blocks; read time = ceil(words/200),
  min 1 min.
- **Theme:** single "9009" theme only (see §4); JetBrains Mono everywhere.

## 3. Architecture
- Astro static shell (near-zero JS) + one React island (`MarkdownApp`)
  owning input state, render, TOC, stats.
- Components: `Editor` / `Reader` / `TocRail` + `TocTop` / `Stats` /
  `lib/markdown.ts` (parser port from v1) / `lib/stats.ts`.
- State: `view: 'in' | 'out'`, `markdown: string`, derived `doc + toc + stats`.
  localStorage key `md-presenter:last` unchanged.
- No router, no backend, no auth. `output: 'static'`, Cloudflare Pages
  adapter not needed (pure static).

## 4. Styling — Tailwind CSS v4, full utilities
- **Stack:** `tailwindcss@4` + `@tailwindcss/vite` plugin.
- **No hand-written CSS files.** Every style is a Tailwind utility. The v1
  `<style>` block is deleted. The markdown renderer emits utility classes
  directly in its HTML strings (headings, lists, task boxes, tables, code,
  quotes) — no `@tailwindcss/typography`, no global `.doc` selectors.
- **Theme tokens** (`@theme` in the global stylesheet) — the "9009" theme,
  single theme, no variants:
  ```
  --color-bg:     #eeebe2;  /* page background            */
  --color-ink:    #080909;  /* primary text (text + main) */
  --color-sub:    #99947f;  /* secondary text             */
  --color-faint:  #d3cfc1;  /* subAlt: borders, muted bg  */
  --color-accent: #7fa480;  /* caret: links, focus, active*/
  --color-error:  #c87e74;  /* errors                    */
  --font-mono: "JetBrains Mono Variable", ui-monospace, SFMono-Regular, Menlo, monospace;
  ```
  Accent usage (replaces v1's clay): links, TOC hover/active, focus rings,
  task-checkbox fill. `sub` for meta labels and secondary text, `faint`
  for borders and muted surfaces (replaces v1's gray-150/300 roles).
- **Breakpoints:** TOC rail ≥1280px (`min-[1280px]:` variants); disclosure
  below. Content column `max-w-[1120px]` both views.
- **Code style example** (renderer output):
  ```tsx
  `<h2 id="${id}" class="mt-8 text-[26px] font-bold leading-[1.2] tracking-[-0.025em] text-ink">` +
  `<pre class="overflow-x-auto bg-ink p-4 font-mono text-[13px] leading-[1.7] text-bg">`
  ```
  Naming: `kebab-case` components, `lib/` pure functions, no default exports.

## 5. Non-goals
No multi-doc history, no search, no additional themes (single "9009" theme
only), no export, no server rendering of user content, no non-React islands.

## 6. Testing
- Unit: parser (headings/ids, tasks, tables, code, quotes), stats
  (word count excludes code, min-1-min), slug uniqueness.
- Visual parity: rendered sample compared against v1 `md-view.html`
  (same spacing/type scale).
- Manual: paste sample → Render → TOC rail links jump; narrow viewport →
  disclosure on top; reload restores input; <2 headings → fallback stats line.
- Deploy preview on every PR (Cloudflare Pages), merge only when green.

## 7. Open questions
- Confirm accent mapping: `caret` green for links/focus/active — or should
  `main` black carry links with green reserved for focus only?
