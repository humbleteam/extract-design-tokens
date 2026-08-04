# Changelog

## [1.0.0] - 2026-07-12

- Initial release: SKILL.md covers URL, screenshot, and CSS-file capture across palette, typography, spacing, radii, shadows, and motion.
- Output contract fixed to always emit a `:root` CSS block plus a W3C Design Tokens Community Group JSON block, with a source footer.
- Long-tail handling for palettes past 8 colors, gradient stop preservation, and a not-extracted marker for unconfirmed values.
- Complete-set re-emission rule for updates, so a follow-up edit never silently drops an existing token.
