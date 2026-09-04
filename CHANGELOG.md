# Changelog

## [1.3.0] - 2026-09-04

- Added `references/multi-theme.md`, the repo's first reference file, for sources that declare the same design twice - light and dark, a high-contrast mode, a switchable skin. Nothing in the skill handled them, and every rule in it assumes one value per role.
- The gap had teeth in three places. Step 1 says to prefer computed values on a URL, and a computed value resolves under one color scheme, so a single fetch of a themed site returned one palette with nothing to say the other existed. The output shapes have room for one value per token, so there was nowhere to put a second set. Worst of the three, the edge case for conflicting screenshots fires exactly on a light and dark pair of the same screen - "two screenshots show a different shade of the same primary button" - and told the model to ask which one is canonical. Following it deletes half of a two-theme design, and the answer to the question is "both".
- A conflict is now defined: two sources claiming the same condition and disagreeing. Two values under two declared conditions are not one. A theme has to be declared by the source - a scheme query, a theme selector, a visible switch - and is never inferred from two screenshots that merely look different.
- The reference file carries the shapes: a second CSS block keyed to the source's own mechanism rather than one the skill invents, a sibling JSON group holding only the tokens that differ, identical token names across themes because the role does not change when the theme does, no second set for groups that do not vary, and a footer line naming the condition each set was read under.
- A themed pair also stops landing on the Step 3 long tail. A dark background and a light background are not near-duplicate colors competing for one role.

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
