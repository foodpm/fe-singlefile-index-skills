# fe-singlefile-index

Bundle a frontend build output into a single `dist/index.html` (inline JS/CSS and optionally other assets). Useful for single-file deployment, offline demos, or delivering one HTML artifact.

## When To Use
- Deliver a demo as “one HTML file” (can be opened directly)
- Publish a static site as a single file (avoid an `assets/` directory)
- Inline page/resources to share via email/IM as one file

## Acceptance Criteria
- Produces `dist/index.html`
- Default: inline JS/CSS into `dist/index.html`
- Strict single-file mode: inline images/fonts where possible (data URIs) so `dist/` only contains `index.html`

## Process

### 1) Detect The Build Tool
Check in this order:
1. Vite: `vite.config.*` exists, or `package.json` scripts contain `vite build`
2. Webpack: `webpack.config.*` exists
3. Next.js: `next.config.*` exists
4. Other: start with the post-processing script approach

### 2) Vite Approach (Preferred: Inline Via A Plugin)
1. Install a single-file plugin (prefer `vite-plugin-singlefile`; reuse an existing one if the repo already uses it)
2. Add the plugin to `vite.config.*` and minimize multi-file output:
   - Disable CSS splitting: `build.cssCodeSplit = false`
   - Avoid multiple chunks: merge dynamic imports / disable manual chunks (minimal changes, follow existing config style)
   - Increase asset inlining threshold: `build.assetsInlineLimit` (set very high for strict single-file; otherwise keep default or moderate)
3. Add a script in `package.json` (do not break the existing `build`):
   - `build:single`: generates the single-file output
4. Check the output:
   - `dist/index.html` contains inlined `<style>` and `<script>` (or equivalent)
   - In strict mode, `dist/` should not contain `assets/` (if it does, use the post-processing fallback)

### 3) Post-Processing Fallback (Strict Single-File Or Non-Vite)
1. Keep your normal build step to produce `dist/`
2. Add a Node script (for example `scripts/inline-dist.mjs`) to:
   - Read `dist/index.html`
   - Inline CSS referenced by `<link rel="stylesheet" href="...">` into `<style>`
   - Inline JS referenced by `<script type="module" src="...">` into `<script>`
   - Optional: convert `<img src>` and CSS `url(...)` assets into data URIs
   - Optional: remove `dist/assets/` so only `dist/index.html` remains
3. Add a script in `package.json`:
   - `build:single`: `build && node scripts/inline-dist.mjs`

## Notes
- The single HTML file will be much larger and loses long-term caching benefits of separate `assets/*.js`. Best for demos and single-file delivery.
- If you rely on Service Workers, Web Workers, or many static assets, strict single-file output typically requires the post-processing fallback.

## Output (Report At The End)
- Which approach you used (Vite plugin / post-processing script)
- Which scripts you added/changed (for example `build:single`)
- What the build output looks like (which files are in `dist/`)
