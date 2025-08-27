# Repository Guidelines

## Project Structure & Module Organization
- Source lives in `src/` with CSS under `src/styles/` and TS in `src/components/` plus `src/main.ts`. Demo HTML is `index.html`; static assets go in `public/`.
- Tests live in `tests/` (Playwright specs like `tests/visual.spec.ts`, `tests/a11y.spec.ts`).
- The README is the single source of truth for required files (see the scaffolded tree and copy‑verbatim sections).

## Architecture Overview
- Vite + TypeScript; no CSS frameworks. CSS Custom Properties are the API.
- Two primitives: Plasma gradients/halos and Glass surfaces composed via utility classes.
- Accessibility: dark/light themes, reduced‑motion, contrast helpers.

## Build, Test, and Development Commands
- `npm run dev`: Start Vite dev server for the demo site.
- `npm run build`: Production build via Vite.
- `npm run preview`: Serve the built site locally for verification.
- `npm test`: Run Playwright tests (builds first via `pretest`).

## Coding Style & Naming Conventions
- TypeScript ES modules; 2‑space indentation; no frameworks.
- CSS custom properties are the API. Use relative units (`clamp()`, `%`, `vi/vb/svh`); avoid layout `px`.
- File names: CSS kebab case (`tokens.css`); TS lowercase modules (`controls.ts`, `preview.ts`, `a11y.ts`).
- Class names kebab case (e.g., `.plasma-bg`, `.glass`, `.readable`).

## Testing Guidelines
- Visuals: Playwright snapshots for hero and glass card in light/dark at 1×/2× DPR; mask dynamic regions.
- A11y: Verify focus visibility and contrast thresholds via `src/components/a11y.ts`.
- Naming: `*.spec.ts` under `tests/`. Run with `npm test`.
- Coverage: Add tests with any feature/style API change; update snapshots only for intended visual diffs.

## Commit & Pull Request Guidelines
- Conventional Commits: `feat:`, `fix:`, `chore:`, or scoped e.g. `feat(controls): ...`.
- Branching: Short‑lived feature branches; open PRs against `main`; squash‑merge.
- PRs: Include a clear description, linked issues, before/after screenshots (light/dark), and test results. Keep PRs focused; update README if structure or tokens change.

## Security & Configuration Tips
- Persist UI state only to `localStorage["event-theme:vars"]`; do not store secrets.
- Respect `prefers-reduced-motion` and theme selection (`[data-theme]`). Avoid introducing third‑party CSS/JS that conflict with tokens.

## Agent Workflow
- Treat README as the spec; implement copy‑verbatim sections exactly.
- Prefer minimal, surgical changes; avoid unrelated refactors. Update docs when behavior or structure changes.

## Continuous Integration (CI)
- GitHub Actions workflow: `.github/workflows/ci.yml` runs on push/PR to `main`.
- Node 20 on Ubuntu; caches npm; installs Playwright browsers.
- Steps: install deps → `npm run build` → `npm test` (headless Playwright with Vite preview server).
- Artifacts: upload Playwright report/screenshots on failure for review.
