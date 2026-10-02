# Tasks

## 1. Rendering spike

- [x] 1.1 Render three of the set's unlabeled SVGs through GPUI's `img()` at 16, 20 and 24px on 1x
  and 2x displays and compare against the source artwork. Verify: screenshots in the PR, and a note
  here on whether colour and edges are faithful (if not, fall back to pre-rasterized PNGs at
  build time and record that decision).

  **Findings.** `img()` with an SVG source keeps colour (#326CE5) but not edges: it rasterizes once
  at the SVG's intrinsic size x2 (about 136x132 px for this set) and lets the GPU downsample with no
  mipmaps, 4-8x at 16 px. At 1x that turns the Pod cube into a blob. Rasterizing at the exact
  device size with GPUI's own `svg_renderer().render_parsed` and drawing it 1:1 matches a Chrome
  reference: antialiased-edge share 0.198 vs Chrome's 0.182 at 2x and 0.372 vs 0.334 at 1x, against
  0.131 and 0.163 for `img()`. Captured through the Metal renderer at 16/20/24 px for pod, deploy
  and cm. Screenshots: `assets/compare-1x-zoom.png`, `assets/compare-2x-zoom.png`.

  **Decision:** neither `img()`-with-SVG nor build-time PNGs (which would need every size x scale
  x text-size step). Rasterize at runtime at the exact device size, cache by (icon, device px).

## 2. Icon set and lookup

- [x] 2.1 Vendor the set's unlabeled SVGs and its licence under `app/assets/icons/kubernetes/`,
  plus container, custom-resource and generic fallbacks in the same style. Verify: a test that every
  bundled icon file loads and parses.
- [x] 2.2 Add `ui::icon::for_kind(group, kind)`, covering every built-in kind the set has and
  falling back otherwise. Verify: unit tests for core kinds, an apps-group kind, a CRD named like a
  built-in kind in another group, and a container.

## 3. Where icons appear

- [ ] 3.1 Panel title bars and tabs. Verify: a render test that a Pods tab and a ConfigMap tab
  each show their icon before the title, and a `simulate_keystrokes` test that tab order is
  unchanged.
- [ ] 3.2 The Resource panel's kind list, including Custom Resources subgroups. Verify: a render
  test for a core kind and a CRD.
- [ ] 3.3 Configuration-tab and object-viewer cards, container cards, and resource links. Verify:
  render tests for each.
- [ ] 3.4 Size icons to the adjacent text and scale them with the text-size preference (if
  `visual-refresh-typography-spacing` has landed; otherwise to the current text size). Verify: a
  test that icon size follows text size.

## 4. Credit

- [ ] 4.1 Ship the licence with the app bundle and credit the set in the About window. Verify: a
  render test of the About credits.

## 5. Verification

- [ ] 5.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`.
- [ ] 5.2 Manual check: a busy window with several kinds open; icons are recognizable, crisp at the
  default and a larger text size, and don't distract from the accent colour.
