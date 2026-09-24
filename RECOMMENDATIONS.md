# Repository Review Checklist

A senior-developer review of `sf-stairs-map`, in priority order. Check items off as they land.

## What you're working with

- **Stack:** TypeScript, Vite and Leaflet in the browser. A Cloudflare Worker (`worker/index.ts`) only serves the static files. Python scripts generate the data ahead of time.
- **Data flow:** `data/` (CSV, KML and caches) → `scripts/*.py` → `src/stairs.json` and `src/stairway-metrics.json`, bundled into the site. The live site makes no API calls.
- **Tests:** 13 Playwright end-to-end tests in `tests/map.spec.ts`. A few also test plain logic, such as the escalator scoring.
- **Git:** `origin` is the fork (`garikcareer`), `upstream` is `elizabethsiegle`.
- **State at review time:** `npm run build` passes, `npm audit` reports 0 vulnerabilities, dependencies are only slightly out of date.

## Day 1: setup

- [ ] Run `npm ci`, `npx playwright install chromium`, then `npm test`. Confirm everything passes before changing anything.
- [ ] Run `npm run dev` and try every feature: search, rating chips, neighborhood picker, "near me", route planner, escalator filter, about dialog.
- [ ] Read all of `README.md`. It explains how the data is sourced and credited, which matters more than the code.
- [ ] Decide what to do with the untracked folders at the root (`.claude/`, `.codex/`, `.cursor/`, `.gemini/`, `.opencode/`, `.pi/`, `.factory/`, `.idea/`, `gh/`, `new-entire-demo/`): add them to `.gitignore` or commit them on purpose.
- [ ] Run `git fetch upstream` before starting a branch.
- [ ] On Windows, `python3` may open the Microsoft Store stub. If `npm run import:data` fails, run `python scripts/import-data.py` instead.

## High-value fixes

- [ ] **Type-check the Worker.** `tsconfig.json` only includes `src/`, so the build never checks `worker/index.ts`. Checked on its own, it can't find `Fetcher` or `ExportedHandler`. Add `@cloudflare/workers-types` (or run `wrangler types`) and a small `tsconfig` for `worker/`.
- [ ] **Remove or use dead code.** Nothing imports `src/routing.ts`, yet `public/walking-network.json` and `scripts/import-network.py` still exist for it. Connect it to the route planner or delete all three.
- [ ] **Add CI.** `.github/` only has Entire hooks. Add a GitHub Action that runs `npm ci`, `npm run build` and `npm test` on every PR.
- [ ] **Fix the route planner label.** `src/main.ts:114` always says "best rated first", even when "Closest mix of ratings" is selected, and `#route-distance` never shows a distance.
- [ ] **Add a Content-Security-Policy** to `public/_headers`. The site only loads from itself, OSM tiles and Google Fonts, so a strict policy is easy.

## Tests

- [ ] Replace hard-coded counts (`1,123`, `80`, `82`, `183`, `181`) with values computed from `stairs.json`, or document them as a snapshot to update on every data refresh.
- [ ] Mock OpenStreetMap tiles with `page.route()` so tests don't fail when OSM is slow.
- [ ] Move pure-logic tests (escalator scoring, `measurements.ts`) to Vitest.
- [ ] Add accessibility checks with `@axe-core/playwright`.

## Code quality

- [ ] Split `src/main.ts` (one 260-line file with large HTML strings, global state and long one-liners at lines 104, 113, 114, 171) into modules such as `state.ts`, `render/list.ts`, `render/detail.ts`, `route-planner.ts`, `filters.ts`, `geo.ts`. Lean on the E2E tests while refactoring.
- [ ] Add ESLint and Prettier.
- [ ] Keep every value in `innerHTML` templates passing through `escape()`. Switch to a templating helper if user-entered content is ever added.
- [ ] Replace the biased shuffle `sort(() => Math.random() - .5)` (`src/main.ts:50`) with Fisher–Yates.

## Performance

- [ ] The JS is about 790 KB raw (about 175 KB gzipped), mostly JSON data. Serve `stairs.json` as a cacheable file from `public/`, or drop unused fields in the importer.
- [ ] Convert `public/photos/` to WebP/AVIF with `srcset`, or use Cloudflare Images.

## Data pipeline

- [ ] Add a `.python-version` and a check that fails when regenerating the JSON produces a different file than the committed one.
- [ ] Have the importer write the snapshot date into `stairs.json` instead of hard-coding "September 14, 2026" in `main.ts` and the README.

## Workflow

- [ ] Delete merged remote branches on `origin`.
- [ ] Keep PRs small, one feature per branch (`sf-stairs/<feature>`).
- [ ] Read the `.entire/runners/*` check results (review, security, drift, slop gate) on each PR.
- [ ] Add a `CLAUDE.md` with these commands and conventions so AI agents follow them.

**If you only do three things:** add CI, type-check the Worker, and resolve `routing.ts`. Then split `main.ts` before adding features.
