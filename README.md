<div align="center">

<h1>Extract design tokens</h1>

**Pull a named design-token set - palette, type, spacing, radii, shadows, and motion - from a live URL, a screenshot, or a CSS file, and get back CSS custom properties plus W3C-shaped JSON, built for design teams and the engineers who work with them.**

[![CI](https://img.shields.io/github/actions/workflow/status/humbleteam/extract-design-tokens/validate.yml?branch=main&style=for-the-badge&logo=github&label=CI)](https://github.com/humbleteam/extract-design-tokens/actions/workflows/validate.yml)
[![GitHub stars](https://img.shields.io/github/stars/humbleteam/extract-design-tokens?style=for-the-badge&logo=github&color=181717)](https://github.com/humbleteam/extract-design-tokens/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/humbleteam/extract-design-tokens?style=for-the-badge&color=339933)](https://github.com/humbleteam/extract-design-tokens/commits/main)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-skill-D97757?style=for-the-badge&logo=anthropic&logoColor=white)](https://github.com/humbleteam/extract-design-tokens/blob/main/SKILL.md)

</div>

`extract-design-tokens` turns a production URL, screenshots, or a raw CSS file into a design-token set: role-named CSS custom properties plus JSON in the W3C Design Tokens Community Group format. The rule that keeps it honest: any value not visible in the source is marked not extracted, never guessed.

## Table of contents

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Usage](#usage)
- [Example output](#example-output)
- [How it works](#how-it-works)
- [How is this different from just asking the model?](#how-is-this-different-from-just-asking-the-model)
- [FAQ](#faq)
- [Related skills](#related-skills)
- [Who maintains this](#who-maintains-this)

## What it does

- Reads a live URL, screenshots, or a CSS/SCSS file and pulls out color, typography, spacing, radii, shadow, and motion values.
- Names every token by role (`--accent`, `--text-secondary`), not by value (`--blue-500`), so the set survives a rebrand.
- Outputs a ready-to-paste `:root` CSS block and JSON in the W3C Design Tokens Community Group draft shape, every time.
- Groups any color past the first 5-8 structural ones into a flagged long tail, instead of padding the main palette with near-duplicates.
- Marks every unconfirmed value as `(not extracted - provide a screenshot/URL and re-run)` instead of guessing.
- Re-emits the complete token set on every update, so an edit never silently drops a token you did not ask to change.

## Quick start

**Personal (all projects):**

```bash
git clone https://github.com/humbleteam/extract-design-tokens ~/.claude/skills/extract-design-tokens
```

**Project only:**

```bash
git clone https://github.com/humbleteam/extract-design-tokens .claude/skills/extract-design-tokens
```

**Other agents (Cursor, Codex, or any LLM agent):** this skill is plain markdown per the Agent Skills format. Paste the contents of `SKILL.md` into the system prompt.

Restart Claude Code after cloning, then confirm the skill is listed - it loads from `~/.claude/skills/` and `.claude/skills/` at startup.

## Usage

- "Extract the design tokens from https://example.com" - reads computed styles and linked CSS, returns the CSS and JSON blocks.
- "Pull the design tokens out of this screenshot" (image attached) - reads colors and proportions visually, flags them as estimates.
- "Add a warning color token to the set we extracted earlier" - re-emits the complete set with the new token folded in.

## Example output

Below is an example run against a fictional product, Northwind Analytics, from its marketing homepage.

```css
:root {
  /* Palette */
  --bg-primary: #0B1220;
  --bg-secondary: #141C2E;
  --accent: #4F7CFF;
  --accent-hover: #3A63E0;
  --text-primary: #F4F6FB;
  --text-secondary: #8993A8;
  --success: #34C77B;
  --danger: #E4574C;

  /* Typography */
  --font-heading: 'Sora', system-ui, sans-serif;
  --font-body: 'Inter', system-ui, sans-serif;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 22px;
  --text-xl: 40px;

  /* Spacing (base 4px) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 16px;
  --space-4: 24px;

  /* Radii */
  --radius-sm: 4px;
  --radius-md: 10px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(4, 8, 20, 0.24);

  /* Motion */
  (not extracted - provide a screenshot/URL and re-run)
}
```

```json
{
  "color": {
    "bg-primary": { "$value": "#0B1220", "$type": "color" },
    "accent": { "$value": "#4F7CFF", "$type": "color" }
  },
  "typography": {
    "font-heading": { "$value": "Sora, system-ui, sans-serif", "$type": "fontFamily" },
    "text-base": { "$value": "16px", "$type": "dimension" }
  },
  "spacing": {
    "space-2": { "$value": "8px", "$type": "dimension" }
  },
  "radii": {
    "radius-md": { "$value": "10px", "$type": "dimension" }
  },
  "shadow": {
    "shadow-sm": { "$value": "0 1px 2px rgba(4, 8, 20, 0.24)", "$type": "shadow" }
  }
}
```

```
Source: palette, typography, spacing, and radii from https://northwind-analytics.example (computed styles); motion not extracted - no transitions or hover states were visible in the fetched CSS.
```

## How it works

- **Fixed capture order.** Palette, typography, spacing, radii, shadows, motion, every run, so nothing gets skipped by accident.
- **Role-first naming.** Tokens are named for what they do (`--bg-primary`), not what they equal (`--slate-900`), so a rebrand is a value swap, not a rename across the codebase.
- **A palette cap with an escape hatch.** The main palette stops at 5-8 colors with a clear role. Everything past that goes into a flagged long tail.
- **No invented values, ever.** A group with no usable source data gets `(not extracted - provide a screenshot/URL and re-run)` in both the CSS and the JSON, per group.
- **Two outputs, always.** A `:root` CSS block for immediate use, and JSON in the W3C Design Tokens Community Group draft shape (`$value` / `$type`) for tooling.
- **A source footer on every answer.** Each group traces back to a URL, a named screenshot file, or a CSS file.
- **Complete-set re-emission.** Updating one token means re-emitting the whole set - a partial answer would silently delete every token left out.
- **Gradients are preserved as stops**, not flattened to a single color, since a flattened gradient loses information a developer needs to rebuild it.

## How is this different from just asking the model?

A bare prompt like "extract the colors from this site" tends to return a rounded, plausible palette padded to a tidy number, with values invented where the model could not confirm them. It rarely distinguishes a value it read from one it estimated, and a follow-up edit ("add a warning color") often returns just the new line, dropping the rest of the set. This skill pins the capture order, the role-based naming, and the rule that every update re-emits the complete set. The judgment calls - which color deserves which role - still come from the model; the skill fixes the structure around them.

## FAQ

**How do I extract design tokens from a website?**
Give Claude the URL and ask it to extract design tokens. With this skill installed, it reads computed styles and linked CSS, then returns a `:root` CSS block plus a JSON file, grouped into palette, typography, spacing, radii, shadows, and motion.

**Can AI create a design system from screenshots?**
It can extract a starting token set from screenshots, with colors, spacing, and type read visually and marked as estimates. Pairing a screenshot with the live URL or CSS, when available, gives higher-confidence values.

**What format should design tokens be in?**
CSS custom properties for immediate use, plus JSON in the W3C Design Tokens Community Group draft shape (`$value` and `$type` per token), the format most token-consuming build tools expect.

**How many colors should a design system have?**
There is no fixed number, but a palette past 8 structural colors usually signals drift, not intentional variety. This skill caps the main palette at 5-8 roles and flags the rest as a long tail for cleanup.

**Can I re-run this after the design changes?**
Yes. Give it the updated URL, screenshot, or CSS and ask for an update. It re-emits the complete token set with the new values folded in, not just what changed.

**Does this work with Figma files directly?**
Not directly - it reads rendered output (a URL, a screenshot) or raw CSS, not the Figma file format. Export or screenshot the relevant frames first.

**What happens if a value cannot be confirmed?**
It is marked `(not extracted - provide a screenshot/URL and re-run)` rather than estimated. That marker tells you exactly what to supply to complete the set.

## Related skills

Part of a 10-skill open-source kit for design teams by Humbleteam.

- [design-review](https://github.com/humbleteam/design-review) - structured UX critique with a 0-4 score, Before/After/Why fixes, and a citation for every claim.
- [ascii-wireframes](https://github.com/humbleteam/ascii-wireframes) - three distinct layout hypotheses as ASCII wireframes before any hi-fi work.
- [html-mockup](https://github.com/humbleteam/html-mockup) - census-first HTML mockups that match a reference screenshot: exact palette, item counts, component states.
- [audit-design-tokens](https://github.com/humbleteam/audit-design-tokens) - find token drift in a codebase: raw hex values, off-scale spacing, near-duplicate colors.
- [design-qa](https://github.com/humbleteam/design-qa) - a pre-ship design QA gate: states, contrast, touch targets, breakpoints, keyboard paths.
- [design-handoff](https://github.com/humbleteam/design-handoff) - turn a finished mockup into a dev-ready spec: tokens, states, accessibility annotations, open questions.
- [accessibility-audit](https://github.com/humbleteam/accessibility-audit) - WCAG 2.2-grounded accessibility review with success-criterion citations and severity levels.
- [ux-writing](https://github.com/humbleteam/ux-writing) - interface copy that reads human: plain-verb microcopy rules and an AI-tell strip pass.
- [design-brief](https://github.com/humbleteam/design-brief) - extract a 5-bullet design brief from messy project inputs, with a gap report for what is missing.

## Who maintains this

Maintained by [Humbleteam](https://humbleteam.com/ai), a design and AI-engineering studio that builds AI infrastructure for design teams. This skill is distilled from the internal playbooks we run on client work. Issues and PRs welcome.

MIT - see [LICENSE](LICENSE).
