# Tasks

## 1. Shortened metadata chips

- [ ] 1.1 Add a metadata-chip variant to `ui::detail` that takes `(key, value)` pairs and shortens
  any value longer than 100 characters or spanning more than one line to its first 20 characters of
  the first line plus an ellipsis, counting characters rather than bytes. Verify: unit tests for a
  multi-line value, a 300-character single-line value, a value of exactly 100 characters, and a
  multi-byte value cut on a character boundary.
- [ ] 1.2 Give a shortened chip a hover tooltip with the full value, wrapped and width-capped, and
  no tooltip on a short chip. Verify: a render test that a shortened chip has a tooltip carrying the
  full value and a short chip has none.
- [ ] 1.3 Switch the pod detail panel's and object viewer's label and annotation chips to the new
  variant, leaving capacity chips on the existing path. Verify: render tests in both panels that a
  long annotation shows only the preview, and that a Secret's redacted last-applied-configuration
  annotation shows no value in the chip or tooltip.

## 2. Verification

- [ ] 2.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`.
- [ ] 2.2 Manual check: open a pod with a multi-line JSON annotation and a long single-line
  annotation; confirm both chips are one line with a short preview, hovering shows the full value,
  and the YAML view still shows them in full.
