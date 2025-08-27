# Repository Guidelines

## Project Structure & Module Organization
- Source lives in `src/` with CSS under `src/styles/` and TS in `src/components/` plus `src/main.ts`. Demo HTML is `index.html`; static assets go in `public/`.
- Tests live in `tests/` (Playwright specs like `tests/visual.spec.ts`, `tests/a11y.spec.ts`).
- The README is the single source of truth for required files (see the scaffolded tree and copy‑verbatim sections).

## Build, Test, and Development Commands
- `npm run dev`: Start Vite dev server for the demo site.
- `npm run build`: Production build via Vite.
- `npm run preview`: Serve the built site locally for verification.
- `npm test`: Run Playwright tests (visual and basic a11y).

## Coding Style & Naming Conventions
- TypeScript ES modules; 2‑space indentation; no frameworks.
- CSS uses custom properties as the API. Keep units relative (`clamp()`, `%`, `vi/vb/svh`); avoid layout `px`.
- File names: CSS in kebab case (`tokens.css`, `gradients.css`); TS modules lowercase, one concept per file (`controls.ts`, `preview.ts`, `a11y.ts`).
- Class names are kebab case (e.g., `.plasma-bg`, `.glass`, `.readable`). Do not introduce BEM unless necessary.

## Testing Guidelines
- Visuals: Playwright snapshots for hero and glass card in light/dark at 1×/2× DPR; mask dynamic regions. Example: `tests/visual.spec.ts`.
- A11y: Verify focus visibility and contrast thresholds via helpers in `src/components/a11y.ts`.
- Naming: `*.spec.ts` under `tests/`. Run with `npm test`.

## Commit & Pull Request Guidelines
- Commits: Imperative, concise subjects; describe scope when relevant. Example: `feat(controls): persist overrides to localStorage`.
- PRs: Include a clear description, linked issues, before/after screenshots of the demo (light/dark), and test results. Keep PRs focused; update README if structure or tokens change.

## Security & Configuration Tips
- Persist UI state only to `localStorage["event-theme:vars"]`; do not store secrets.
- Respect `prefers-reduced-motion` and theme selection (`[data-theme]`). Avoid introducing third‑party CSS/JS that conflict with tokens.
