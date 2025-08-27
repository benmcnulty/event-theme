

# event-theme — Plasma & Glass Web Style System

**Purpose:** Provide a reproducible, scalable, and accessible theme inspired by two visual motifs in the provided screenshot:
1) **Neon‑plasma logo glow** (cool blue core → mint cushion → lemon rim with a whisper of warm edge),
2) **Liquid‑glass mark** (pearl base with faint iridescent edges).
This README is the **single source of truth** the coding agent should use to scaffold a working demo site with live, self‑explaining examples and controls.

---

## TL;DR for implementers

- Build a small demo site (Vite + TypeScript + vanilla CSS) that showcases **Plasma** and **Glass** primitives as composable utilities.  
- All sizes are **relative**; use `clamp()`, `%`, `vi/vb/svh`, and CSS variables.  
- Provide **dark & light themes**, **reduced‑motion support**, and pass WCAG contrast for text.  
- Expose a **Controls panel** to tune hues/strength/blur for each primitive; changes update CSS variables in real time and persist to `localStorage`.  
- Ship **Playwright** visual snapshots (1×/2× DPR) and **unit tests** for utilities.

---

## 1) Project structure (authoritative)

Create exactly this tree:

```
event-theme/
  public/
    logo.svg                 # simple placeholder logo mask for halo demos
  src/
    styles/
      tokens.css             # design tokens (colors, spacing, type)
      base.css               # resets + typography + layout helpers
      gradients.css          # plasma gradient/glow utilities
      glass.css              # liquid-glass utilities
    components/
      controls.ts            # UI for sliders/toggles; binds to CSS vars
      preview.ts             # small helpers for demos (mask loader, etc.)
      a11y.ts                # contrast checks, prefers-reduced-motion helpers
    main.ts                  # bootstraps controls & demo sections
  index.html                 # demo page
  vite.config.ts
  package.json
  tsconfig.json
  playwright.config.ts
  tests/
    visual.spec.ts
    a11y.spec.ts
  README.md (this file)
```

**Stack**: Vite, TypeScript, Playwright. No CSS frameworks; this repo demonstrates raw CSS custom properties.

---

## 2) Design tokens (copy verbatim)

Create `src/styles/tokens.css` with:

```css
:root {
  /* Fluid base (1rem → 1.125rem across viewport widths) */
  --size: clamp(1rem, 0.94rem + 0.3vw, 1.125rem);

  /* Type ramp relative to --size */
  --font--1: clamp(0.8125rem, 0.78rem + 0.2vw, 0.9rem);
  --font-0:  var(--size);
  --font-1:  clamp(1.125rem, 1rem + 0.6vw, 1.375rem);
  --font-2:  clamp(1.35rem, 1.1rem + 1.2vw, 1.75rem);
  --font-3:  clamp(1.6rem, 1.2rem + 2vw, 2.25rem);
  --font-4:  clamp(2rem, 1.4rem + 3vw, 3rem);

  /* Spacing & radii (fluid) */
  --space-1: clamp(0.375rem, 0.3rem + 0.3vw, 0.5rem);
  --space-2: clamp(0.5rem, 0.4rem + 0.4vw, 0.75rem);
  --space-3: clamp(0.75rem, 0.6rem + 0.6vw, 1rem);
  --space-4: clamp(1rem, 0.8rem + 0.8vw, 1.5rem);
  --space-6: clamp(1.5rem, 1rem + 1.3vw, 2.25rem);
  --radius-1: 0.5rem;
  --radius-2: 0.875rem;
  --radius-3: 1.25rem;

  /* Light theme defaults (dark overrides below) */
  --bg: hsl(0 0% 100%);
  --surface: hsl(240 14% 97%);
  --text: hsl(230 36% 6%);
  --muted: hsl(230 12% 36%);
  --hairline: hsl(230 12% 86%);

  /* Plasma spectrum (rounded from screenshot) */
  --plasma-blue:  hsl(213 74% 53%);
  --plasma-royal: hsl(225 60% 38%);
  --plasma-cyan:  hsl(190 46% 64%);
  --plasma-mint:  hsl(150 35% 72%);
  --plasma-lemon: hsl(50 76% 62%);
  --plasma-warm:  hsl(18 60% 56%);

  /* Iridescent tints (low-chroma, for edges) */
  --iri-cyan:  hsla(190 80% 86% / 0.55);
  --iri-pink:  hsla(330 76% 86% / 0.55);
  --iri-lime:  hsla(85  72% 85% / 0.55);
  --iri-lav:   hsla(260 70% 88% / 0.55);
  --glass:     hsla(0 0% 100% / 0.35);
  --glass-2:   hsla(0 0% 100% / 0.08);

  /* Tunable glow geometry (percentages so it scales) */
  --plasma-core-size: 55%;
  --plasma-ring-size: 78%;
  --plasma-bloom-size: 110%;
}

/* Auto dark-mode; also support [data-theme] override */
@media (prefers-color-scheme: dark) {
  :root {
    --bg: hsl(230 36% 6%);
    --surface: hsl(230 30% 9%);
    --text: hsl(0 0% 98%);
    --muted: hsl(220 14% 70%);
    --hairline: hsl(220 18% 22%);
  }
}
[data-theme="dark"] {
  --bg: hsl(230 36% 6%);
  --surface: hsl(230 30% 9%);
  --text: hsl(0 0% 98%);
  --muted: hsl(220 14% 70%);
  --hairline: hsl(220 18% 22%);
}
[data-theme="light"] {
  --bg: hsl(0 0% 100%);
  --surface: hsl(240 14% 97%);
  --text: hsl(230 36% 6%);
  --muted: hsl(230 12% 36%);
  --hairline: hsl(230 12% 86%);
}
```

---

## 3) Base styles (copy verbatim)

Create `src/styles/base.css`:

```css
:root { color-scheme: light dark; }
html, body { height: 100%; }
body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font: 400 var(--font-0)/1.35 system-ui, -apple-system, "Inter", "SF Pro", Segoe UI, Roboto, sans-serif;
  text-rendering: optimizeLegibility;
}
h1 { font-size: var(--font-4); line-height: 1.1; letter-spacing: -0.01em; margin: 0 0 var(--space-3); }
h2 { font-size: var(--font-3); line-height: 1.1; letter-spacing: -0.01em; margin: 0 0 var(--space-2); }
p  { margin: 0 0 var(--space-2); color: var(--text); }
.prose { max-inline-size: 72ch; }

.container { inline-size: min(1200px, 94vi); margin-inline: auto; padding: var(--space-6) var(--space-4); }
.hr { block-size: 1px; background: var(--hairline); margin: var(--space-6) 0; }

:focus-visible {
  outline: 2px solid color-mix(in oklab, var(--plasma-blue), white 20%);
  outline-offset: 3px;
}

/* Reduced motion rules */
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}
```

---

## 4) Plasma gradients & halo utilities

Create `src/styles/gradients.css`:

```css
/* Main hero background: cool core -> mint cushion -> lemon rim -> warm fringe */
.plasma-bg {
  background-color: var(--bg);
  background-image:
    radial-gradient(closest-side at 50% 52%,
      color-mix(in oklab, var(--plasma-blue), transparent 80%) 0%,
      transparent var(--plasma-bloom-size)
    ),
    radial-gradient(closest-side at 50% 48%,
      var(--plasma-royal) 0%,
      var(--plasma-blue) 35%,
      var(--plasma-cyan) 60%,
      transparent calc(var(--plasma-core-size) + 2%)
    ),
    radial-gradient(closest-side at 50% 50%,
      var(--plasma-mint) 0%,
      color-mix(in oklab, var(--plasma-mint), transparent 60%) calc(var(--plasma-core-size) - 4%),
      transparent var(--plasma-core-size)
    ),
    conic-gradient(from 210deg at 50% 50%,
      transparent 0 75%,
      var(--plasma-lemon) 78% 86%,
      color-mix(in oklab, var(--plasma-warm), transparent 20%) 88% 94%,
      transparent 96% 360deg
    );
  background-repeat: no-repeat;
  background-size: 100% 100%;
  filter: saturate(1.05);
}

/* Gentle drift to avoid static posterization */
@keyframes plasma-drift {
  0%, 100% { background-position: 50% 50%, 50% 48%, 50% 50%, 50% 50%; }
  50%      { background-position: 51% 49%, 49% 50%, 50% 51%, 50% 50%; }
}
@media (prefers-reduced-motion: no-preference) {
  .plasma-bg { animation: plasma-drift 9s ease-in-out infinite; }
}

/* Shape-agnostic halo for logos; provide a mask via CSS or inline SVG */
.halo {
  position: relative;
  isolation: isolate;
}
.halo::before {
  content: "";
  position: absolute; inset: -8%;
  background:
    radial-gradient(closest-side at 50% 48%, var(--plasma-blue), transparent 65%),
    radial-gradient(closest-side at 52% 52%, color-mix(in oklab, var(--plasma-lemon), transparent 20%), transparent 70%);
  filter: blur(18px) saturate(1.1);
  z-index: -1;
  mask: var(--mask, none);
  -webkit-mask: var(--mask, none);
}
```

---

## 5) Liquid‑glass utilities

Create `src/styles/glass.css`:

```css
.glass {
  position: relative;
  background:
    radial-gradient(120% 100% at 50% 10%, var(--glass) 0 30%, transparent 60%),
    conic-gradient(from 0.25turn at 50% 50%,
      var(--iri-cyan) 0 20%,
      var(--iri-pink) 20% 40%,
      var(--iri-lime) 40% 60%,
      var(--iri-lav)  60% 80%,
      transparent 80% 100%);
  background-blend-mode: screen;
  backdrop-filter: blur(16px) saturate(1.1);
  -webkit-backdrop-filter: blur(16px) saturate(1.1);
  border: 0.75px solid color-mix(in oklab, var(--hairline), white 30%);
  border-radius: var(--radius-3);
  box-shadow:
    0 0.5px 0.25px hsla(0 0% 0% / 0.06) inset,
    0 0 0 1px color-mix(in oklab, var(--hairline), white 20%),
    0 6px 24px -8px hsla(0 0% 0% / 0.25);
}
.glass::before {
  content: "";
  position: absolute; inset: 0;
  border-radius: inherit;
  box-shadow:
    inset 0 12px 30px -18px hsla(220 20% 20% / 0.25),
    inset 0 -10px 24px -20px hsla(220 20% 10% / 0.25);
  pointer-events: none;
}
.glass::after {
  content: "";
  position: absolute; inset: 0;
  background:
    radial-gradient(40% 18% at 68% 22%, hsla(0 0% 100% / 0.55), transparent 60%),
    radial-gradient(30% 14% at 28% 78%, hsla(0 0% 100% / 0.35), transparent 70%);
  border-radius: inherit;
  mix-blend-mode: screen;
  pointer-events: none;
}
```

---

## 6) Demo page requirements

Create `index.html` with:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>event-theme — Plasma & Glass</title>
  </head>
  <body>
    <main class="container">
      <section class="hero plasma-bg" style="min-height: 60svh; display:grid; place-items:center;">
        <div class="prose readable">
          <h1>Plasma + Glass</h1>
          <p>Tunable neon-plasma and liquid-glass primitives with live controls.</p>
        </div>
      </section>

      <div class="hr"></div>

      <section id="controls" class="glass" style="padding: var(--space-6); margin-bottom: var(--space-6);">
        <h2>Controls</h2>
        <!-- controls.ts will populate -->
      </section>

      <section id="examples" style="display:grid; gap: var(--space-4); grid-template-columns: repeat(auto-fit, minmax(22ch, 1fr));">
        <div class="glass" style="padding: var(--space-6);">
          <h3>Glass Card</h3>
          <p>Edge iridescence with frosted interior. Use for overlays or panels.</p>
        </div>
        <div class="glass halo" style="padding: var(--space-6);">
          <h3>Logo Halo</h3>
          <p>An event-style glow applied via mask.</p>
          <div id="logo" class="logo" style="--mask: url(#logo-mask); inline-size: clamp(6rem, 20vw, 14rem); aspect-ratio: 1 / 1;"></div>
          <!-- inline SVG mask will be injected by preview.ts -->
        </div>
      </section>
    </main>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

Create `src/main.ts` with:

```ts
import './styles/tokens.css';
import './styles/base.css';
import './styles/gradients.css';
import './styles/glass.css';
import { mountControls } from './components/controls';
import { mountLogoMask } from './components/preview';

mountControls(document.getElementById('controls')!);
mountLogoMask(document.getElementById('logo')!);
```

---

## 7) Controls panel specification

**Goal:** Real-time editing of the following variables with sane ranges; values persist to `localStorage` and are restored on load.

| Control | CSS Var | Range / Step | Notes |
|---|---|---|---|
| Theme | `[data-theme]` | `light | dark | system` | Apply attribute on `<html>`; if `system`, remove attribute. |
| Bloom strength | `--plasma-bloom-size` | 90–140% / 1% | Larger value = wider soft outer glow. |
| Core size | `--plasma-core-size` | 45–65% / 1% | Size of cool core. |
| Rim hue | `--plasma-lemon`(h) | 30–70 / 1 | Adjust hue only; keep s=76, l=62 to avoid banding. |
| Blue hue | `--plasma-blue`(h) | 205–220 / 1 | Subtle shifts only. |
| Saturation global | *applied via* `filter: saturate()` | 0.8–1.2 / 0.01 | Apply on `.plasma-bg`. |
| Motion | `animation` toggle | on/off | Respect `prefers-reduced-motion` by default. |
| Glass blur | backdrop blur px | 8–24 / 1 | Set on `.glass` backdrop-filter. |
| Iridescence intensity | alpha factor | 0.2–0.8 / 0.05 | Scale alphas of `--iri-*` tokens. |

Implementation notes (create `src/components/controls.ts`):
- Build semantic form controls with labels; reflect changes using `document.documentElement.style.setProperty`.  
- For hue controls, parse `hsl()` and rewrite only the `h` channel.  
- Save a JSON object of overridden vars in `localStorage["event-theme:vars"]`.  
- Provide a **Reset** button to clear overrides.  
- Add a **Copy CSS** button that prints current variable overrides as a snippet.

---

## 8) Logo masking (preview helper)

Create `src/components/preview.ts`:

```ts
export function mountLogoMask(host: HTMLElement) {
  const svg = `
    <svg width="0" height="0" style="position:absolute;">
      <defs>
        <mask id="logo-mask">
          <rect width="100%" height="100%" fill="white"/>
          <!-- Simple rounded apple-like blob for demo; replace as needed -->
        </mask>
      </defs>
    </svg>`;
  document.body.insertAdjacentHTML('afterbegin', svg);
  // If you have an actual mask path, inject it into the <mask>.
}
```

*(The halo works with any silhouette; we deliberately keep the glow independent of the mark.)*

---

## 9) Accessibility rules

- Do not place long text directly on `.plasma-bg`; wrap content in a `.glass` panel or `.readable` overlay:
  ```css
  .readable {
    background: linear-gradient(hsla(230 36% 6% / 0.66), hsla(230 36% 6% / 0.66));
    backdrop-filter: blur(2px);
    padding: var(--space-3) var(--space-4);
    border-radius: var(--radius-2);
  }
  ```
- Body text contrast on `--surface` must be ≥ **4.5:1**; headings ≥ **3:1**.  
- Keyboard focus uses the tokenized ring defined in `base.css`.  
- Honor `prefers-reduced-motion`; the **Motion** toggle must not override the user preference—only reduce further.

Add `src/components/a11y.ts` with helpers to compute contrast (sRGB) and to warn in the demo UI if a user’s overrides drop contrast below thresholds.

---

## 10) Testing

### Playwright visual snapshots
- Capture the hero and a glass card in light/dark at 1× and 2× DPR.
- Mask dynamic text regions to reduce flakiness.

`tests/visual.spec.ts` (sketch):

```ts
import { test, expect } from '@playwright/test';

test.describe('visuals', () => {
  test('hero plasma light/dark', async ({ page, browserName }) => {
    await page.goto('/');
    await expect(page).toHaveScreenshot('hero-light.png', { fullPage: false });
    await page.emulateMedia({ colorScheme: 'dark' });
    await expect(page).toHaveScreenshot('hero-dark.png', { fullPage: false });
  });
});
```

### A11y checks
- Verify focus visibility and that body text contrast ≥ 4.5:1 on both themes.

`tests/a11y.spec.ts` (sketch):
```ts
import { test, expect } from '@playwright/test';
test('page has main landmarks', async ({ page }) => {
  await page.goto('/');
  await expect(page.locator('main')).toBeVisible();
});
```

---

## 11) Build & run

Create `package.json` with scripts:

```json
{
  "name": "event-theme",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "playwright test"
  },
  "devDependencies": {
    "typescript": "^5.6.0",
    "vite": "^5.4.0",
    "@playwright/test": "^1.46.0"
  }
}
```

Initialize Vite/TS config with defaults. Start:
```
npm i
npm run dev
```

---

## 12) Definition of Done

- [ ] Demo loads with **Plasma hero**, **Glass cards**, **Logo halo** example.  
- [ ] **Controls** panel adjusts variables and persists overrides; **Reset** works.  
- [ ] **Dark/Light/System** theme switching works and is reflected in UI.  
- [ ] Text never sits directly on plasma without a readability layer.  
- [ ] All units are relative; no hardcoded `px` for layout/typography.  
- [ ] Playwright snapshots pass on CI (1×/2× DPR).  
- [ ] No console errors; Lighthouse a11y score ≥ 95.

---

## 13) Notes on fidelity & nuance

- **Banding control:** each ring is separated by a mint “cushion” layer; gradients use `color-mix(in oklab, …)` for perceptual blending.  
- **Scale correctness:** geometry uses percentages and `clamp()` so the effect is consistent from icon size to hero banners.  
- **Iridescence realism:** keep edge colorants subtle; center should remain nearly neutral.  
- **Browser support:** `color-mix()` with `oklab` ships in modern browsers; if needed, provide an HSLA fallback layer (non‑blocking).

---

## 14) License

MIT — see `LICENSE` (to be added).
