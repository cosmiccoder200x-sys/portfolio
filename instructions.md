# Instructions — cosmiccoder200x-sys/portfolio

Living project guide. Read before editing; keep it updated when architecture or
conventions change.

## What this is

A **single-file static portfolio site** (`index.html`, ~1200 lines) with no build
step: all CSS lives in one `<style>` block, all JS in one IIFE `<script>` block.
It is deployed to **GitHub Pages** via `.github/workflows/static.yml` (push to
`main` → Actions → Pages). `CNAME` holds a custom domain; `README.md` also
documents an optional Vercel deploy (framework preset: Other, no build command).

## Architecture

- **`index.html`** — the whole site:
  - `<style>` — design tokens (`:root` CSS variables), section styles, responsive
    breakpoints at 1024 / 860 / 520 px.
  - Body markup — semantic sections: `header#home` (hero), `#about`, `#projects`,
    `#github`, `#skills`, `#contact`, `<footer>`.
  - `<script>` IIFE — contains `PORTFOLIO_DATA` (single source of truth for
    profile, education, skills, socials, **projects**), `renderAll()` which
    renders every dynamic section from that object, small UI behaviours
    (typed text, particles, cursor glow, scroll progress, reveal-on-scroll via
    IntersectionObserver, mobile nav, contact form → Google Forms), and `fetchGH()`
    which enriches the GitHub section with **live API data plus graceful
    offline fallback**.
- **`.github/workflows/static.yml`** — Pages deployment; uploads the repo root.

### Data flow rules

- All site content (names, project descriptions, links) lives in
  `PORTFOLIO_DATA`. **Never hard-code project cards in HTML** — render from data.
- The featured-projects grid is **fully static** (rendered from `PORTFOLIO_DATA`
  at load) and must render even with no network. The live GitHub API is used only
  to *enrich* (stats, recent-repo cards) and must always keep the offline
  fallback path working.
- Live GitHub enrichment is unauthenticated `api.github.com` (60 req/h per IP).
  Never add tokens to client code.

## Coding conventions

- Vanilla JS, ES2017+, single IIFE, `'use strict'`. No frameworks, no bundler,
  no new dependencies without an explicit request.
- All dynamic HTML goes through `esc()` (HTML-escape) to prevent XSS; keep doing
  this for any interpolated data, including URLs.
- External links: `target="_blank" rel="noopener"`.
- Element IDs are kebab-case; dynamic containers are `*-grid`, `*-container`.
- Comments mark sections with `── Section ──` box drawing.

## Design rules

- Preserve the existing visual identity: dark theme (`--bg: #04050a`), accent
  blue `--accent: #6c8cff` / purple `--accent-2: #a78bfa` / green `--accent-3`,
  Inter + JetBrains Mono fonts, 18–36 px radii, `cubic-bezier(.2,.8,.2,1)`
  easing, `.reveal` scroll animations with `reveal-delay-1..3` staggering.
- Project grid: 3 columns ≥1025 px, 2 columns 861–1024 px, 1 column ≤860 px
  (featured spans 1 column there). `.project-card.featured` spans 2 columns on
  desktop/tablet; keep the featured/regular mix such that total spans are a
  multiple of the column count so rows pack exactly.
- `grid-auto-flow: dense` fills gaps when a category filter hides cards.
- Interaction polish stays restrained: hover lift `-6px`, 200–300 ms transitions,
  visible `:focus-visible` outlines on every interactive element.
- No fabricated metrics, testimonials, badges, or completion claims. Status and
  demos are shown only when verified.

## Content / project-data rules

- **The curated grid lists exactly these 5 projects, in this order: Modulr,
  Local Brain MCP, FocusFlow Extension, E--learning, Gullylabs.** Other repos
  are deliberately not detailed on the site; do not add project entries beyond
  this list without an explicit request.
- Every project entry needs: `name`, `description` (based on the repo's real
  implementation), `technologies` (verified from the repo), `github` (repo URL),
  `demo` (**only if verified live**, otherwise `null`), `featured` (boolean),
  `icon`, `category` (one of `ai`, `web`, `devtools`, `productivity`).
- Source of truth for project facts is the GitHub repo (README + code), never
  memory. Repos that are forks, experiments, course notes, or near-duplicates of
  an already-listed project are documented in the audit but not listed.
- The `prioritizedRepos` array inside `fetchGH()` must list repo names in the
  same priority order as the curated grid; keep both in sync when adding projects.

## Security requirements

- No API keys, tokens, or secrets in the repo or in client-side code.
- Contact form posts directly to a Google Forms `formResponse` endpoint via a
  hidden iframe; no backend, no stored credentials.
- Keep `esc()` around all interpolated data.

## Testing procedure (no test framework exists)

1. Syntax-check the inline script: extract the `<script>` contents and run
   `node --check` on it.
2. DOM test with jsdom in a scratch directory (not committed): load `index.html`
   with scripts enabled, stub `IntersectionObserver`, canvas `getContext`, and
   `fetch`; assert card count, filter behaviour, link hygiene
   (`rel="noopener"`, `target="_blank"`), no duplicate projects, and that the
   GitHub fallback path renders when `fetch` fails.
3. Verify every external URL with `curl -s -o /dev/null -w "%{http_code}" -L`.
   A demo link may only ship when it returns 200.
4. Sanity-check responsive behaviour by CSS inspection (breakpoints listed above);
   no build output to validate.

## Deployment constraints

- Pushing to `main` auto-deploys to GitHub Pages. **Never push without explicit
  user authorization.**
- `CNAME` must keep the custom domain exactly as configured in the GitHub repo's
  Pages settings.
- The site is intentionally dependency-free: do not introduce a backend, build
  step, or framework.

## How to add a project

1. Verify the repo: README, primary languages, and (if any) the live demo URL
   (`curl -L` must return 200).
2. Add one object to `PORTFOLIO_DATA.projects` (fields listed above), placed so
   featured and regular cards alternate, and set its `category`.
3. Re-check that the featured/regular spans still pack the grid, and insert the
   repo name into `prioritizedRepos` in `fetchGH()` at the matching position.
4. Run the testing steps above, then commit/push only with authorization.

## Redesign update — Midnight Systems (2026-10-31)

The `redesign/midnight-blue-immersive` branch introduces a visual-only design layer
inside `index.html` (`#midnight-blue-redesign` style block). This intentionally
keeps the dependency-free single-file architecture and preserves the existing
markup, `PORTFOLIO_DATA`, render functions, GitHub API enrichment, contact form,
command palette, mobile navigation, terminal REPL, project filters, and reveal
behaviour.

### Current visual direction

- Midnight navy background with electric-blue / sky-blue accents.
- Inter + Space Grotesk + JetBrains Mono remain the typography system.
- Restrained blue ambient lighting, grid, and cursor glow.
- Immersive but purposeful hover depth on the terminal and project cards.
- Blue contribution heatmap levels and blue focus-visible states.
- Responsive layout retained at the existing breakpoints.
- `prefers-reduced-motion` disables decorative motion and transforms.

### Content accuracy

- Use the full branch designation **Electrical & Computer Engineering** for TKMCE.
- Do not label the user's branch as ECE or Electronics & Communication.
- Keep the IIT Madras BS Data Science context.
- Do not invent project metrics, testimonials, skills, or deployment URLs.

### Maintenance

When changing the design, update the appended override block rather than
reintroducing the old violet/cyan palette into the original CSS. Keep the original
styles as a fallback foundation. Run the existing script syntax, DOM, link,
responsive, and GitHub offline-fallback checks before merging.


## Professional portfolio update — 2026-10-31

The professional pass keeps the Midnight Blue visual foundation and the existing dependency-free,
single-file architecture.

### Content and sections

- The positioning is software development, AI systems, and data science; do not present the
  Electrical & Computer Engineering degree as the user's intended career direction.
- Keep the exact TKMCE branch designation **Electrical & Computer Engineering**.
- Keep the IIT Madras BS Data Science & Applications programme as ongoing study.
- The curated project grid remains exactly five projects, in its existing order.
- Each project has a case-study modal with an overview, problem, implementation approach,
  technology stack, architecture summary, and source/demo links where available.
- The Milestones section records the user's HackerRank AI Orchestrator challenge score
  as **2,798 / 4,096** (a score, not a rank or award) and the Local Brain MCP npm publication.
- A resume PDF is not present in the repository. Use a mailto **Request Resume** link until
  the user supplies a current PDF; never point a download button at a missing file.

### Live data and forms

- GitHub profile, repository, and contribution data must come from live responses.
- Never synthesize contribution calendars, guessed totals, cached star counts, or substitute
  curated project cards for live repository data.
- On network/API failure, show a clear unavailable state and retain the curated project grid.
- Contribution totals must use the current calendar year and returned data only.
- The contact form posts to the existing Google Forms endpoint through a hidden iframe.
  Do not use a fixed-delay timer to claim success. Only update the status after the response
  iframe loads, and phrase the message as a returned response rather than guaranteed delivery.

### Visual and accessibility

- Keep the midnight navy / electric-blue palette consistent, including SVG heatmap cells.
- Keep motion restrained and honor `prefers-reduced-motion`.
- Maintain visible focus states, keyboard-operable project filters/modals, and responsive
  navigation including the Milestones anchor.
- Do not add unverified technology names to the marquee or skill list.

### Verification after edits

1. Extract the inline JavaScript and run `node --check`.
2. Confirm exactly five curated project data entries and five case-study specifications.
3. Confirm no generated GitHub contribution fallback or fake "verified" repository stars remain.
4. Confirm the contact form has no fixed-delay success handler.
5. Check all nav anchors resolve to section IDs, especially `#milestones`.
6. Check the project links, demo URLs, mailto links, and npm package URL.
7. Inspect responsive styles and reduced-motion behavior. Browser-based visual QA is still
   required before merging or deploying.


## LinkedIn + GitHub presentation update — 2026-10-31

- LinkedIn profile: https://www.linkedin.com/in/sreerang-p
- Current verified highlights: TKM Notes launch post; Google 5-Day AI Agents: Intensive Vibe Coding Course (issued June 2026); Gemini Certified University Student (issued April 2026, expires April 2029).
- Do not display follower/connection counts or other scraped LinkedIn metrics; public search snapshots may be stale.
- The GitHub section contains the contribution heatmap only. Do not restore profile counters, repository cards, stars, followers, following, or API sync timestamps.
- Do not fetch GitHub profile or repository APIs from the portfolio. The public contribution endpoint is used only for the heatmap.
- Keep the contribution scale consistent: slate → deep blue → cobalt → electric blue → ice blue. Avoid purple/violet remnants in heatmap cells, hover glows, labels, and legend.
- The decorative 120 FPS label and live clock were removed to keep navigation free of simulated telemetry.
- Keep milestone/credential facts tied to the actual public LinkedIn profile/post and retain issuer and issue/expiry dates.
