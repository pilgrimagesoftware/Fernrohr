# Tasks

## 1. Tokens

- [x] 1.1 Add `ui::style`: `accent`, `accent_subtle`, `surface_raised`, `surface_card`, `stripe`,
  `status(Tone)`, and `contrast(a, b)`. Verify: unit tests for the WCAG ratio against known pairs,
  and for surface derivation in both modes.
- [x] 1.2 Override the six theme tokens after each theme change (`ui::theme::apply`), with the
  accent darkened or lightened until it meets 3:1. Verify: contrast tests over both modes' themes,
  including a deliberately too-light accent.

## 2. Chrome and selection

- [x] 2.1 Panel headings and hint bars on `surface_raised`; table headers and Resource panel section
  headers likewise; selected rows on the accent tint. Verify: render tests that the tokens are used
  (no raw `theme.muted` left in those views).

## 3. Status colour

- [x] 3.1 `BadgeTone::Bad`, and pod phase and waiting reason to tone in the Pods table projection;
  status cell, Ready dot and restart count coloured. Verify: projection tests for each phase and
  for `CrashLoopBackOff`/`ImagePullBackOff`.

## 4. Detail views

- [x] 4.1 Section headings with an accent bar, alternating row stripe, quieter label column, and
  cards on `surface_card`, in `ui::detail` (so pod and object detail both get them). Verify: pod and
  object detail render tests still pass; a test that rows alternate the stripe.

## 5. Review

- [x] 5.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`.
- [ ] 5.2 Before/after screenshots of the Pods list, a pod's detail and the Resource panel, in light
  and dark mode, reviewed with the user. **Needs user sign-off.**
