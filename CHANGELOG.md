# Changelog

## [1.2.0] - 2026-08-14

- Step 4 now states the shape of the not-extracted marker in each output, because the literal string it prescribed was invalid in both. In CSS, bare text inside `:root { }` is not a declaration: the parser discards it and everything up to the next semicolon, so a not-extracted group deletes the first real token after it. The marker is a comment.
- In JSON, the marker is an empty group carrying `$description`, not a token `$value`. The draft requires `$value` on every token and requires it to follow the rules for its `$type`, so a sentence under `$type: duration` fails validation - and a group with no source data has no token name to put it under. Empty groups are allowed by the draft as placeholder structure.
- A not-extracted group is never omitted. A missing group reads as "this design has none", which is a claim about the source rather than an admission that nothing was read.
- The README example was the bug in miniature: its CSS emitted the marker as bare text, and its JSON dropped the motion group entirely while the surrounding prose promised the marker in both. Both fixed, with the What-it-does bullet, the How-it-works bullet and the FAQ answer brought into agreement.

## [1.1.0] - 2026-08-08

- Added Step 8, unit fidelity: a length authored in `rem`, `em`, `ch`, `%`, `vw` or `vh` keeps that unit in the token. Computed values are still used to settle which declaration wins, but a computed pixel is one snapshot of one root font size and one viewport, and storing it drops the resize behavior the source had (WCAG 2.2, SC 1.4.4).
- Fluid values (`clamp()`, `min()`, `max()`) are stored as the whole expression rather than read at the fetch viewport. Since the W3C draft's `dimension` type takes a single number with a `px` or `rem` unit, the JSON gets the floor and ceiling as two dimension tokens plus the expression in `$description`.
- Source footer now names the root font size whenever the set contains `rem` values - a source using the `font-size: 62.5%` trick makes `1rem` equal 10px.
- Screenshot capture note: a picture cannot show a unit, so every length read from one is a pixel estimate and must be stated as such.
- New edge cases for rem/em/clamp sources and for a root font size other than 16px; examples in `SKILL.md` and `README.md` updated to a rem scale with one fluid step.

## [1.0.0] - 2026-07-12

- Initial release: SKILL.md covers URL, screenshot, and CSS-file capture across palette, typography, spacing, radii, shadows, and motion.
- Output contract fixed to always emit a `:root` CSS block plus a W3C Design Tokens Community Group JSON block, with a source footer.
- Long-tail handling for palettes past 8 colors, gradient stop preservation, and a not-extracted marker for unconfirmed values.
- Complete-set re-emission rule for updates, so a follow-up edit never silently drops an existing token.
