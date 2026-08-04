---
name: extract-design-tokens
description: Extracts a design-token set - palette, type, spacing, radii, shadows, motion - from a live URL, screenshots, or a CSS file, into CSS custom properties plus a W3C Design Tokens JSON file. Trigger on "extract design tokens from this site", "pull the tokens from this screenshot", "turn this CSS into tokens", "build a token file from this URL". Do not use to find token drift in an existing codebase - use audit-design-tokens for that.
---

# Extract design tokens

Pull a named, source-linked design-token set out of a URL, a set of screenshots, or a CSS dump. Output both a `:root` CSS block and a JSON file in the W3C Design Tokens Community Group draft shape.

## When to run this

Run when the user gives you one or more of:

- A live URL - fetch it, read linked stylesheets, inline styles, and computed values.
- One or more screenshots - read colors, proportions, and type visually.
- A CSS or SCSS file, pasted or attached.

If none of the three are present, ask for one. Do not guess a palette or scale from a text description alone.

## Step 1 - capture, in this order

Work through these six groups. A group that is hard to read is not a reason to skip it - mark it not extracted instead (Step 4).

1. **Palette** - 5 to 8 hex colors, each with a role label (naming rule in Step 2).
2. **Typography** - 1 to 2 font families plus the type scale (the distinct font sizes in use, largest to smallest).
3. **Spacing** - the base unit (commonly 4px or 8px) and the ramp built from it.
4. **Radii** - up to 3 border-radius values, smallest to largest.
5. **Shadows** - up to 3 box-shadow values, smallest to largest.
6. **Motion** - transition/animation durations and easing curves, only if the source shows motion (hover states, CSS transitions, a GIF or video the user provides). If the source gives no signal that motion exists at all, mark it not extracted instead (Step 4).

Source-specific reading:

- **URL**: fetch the page, read linked CSS and inline styles, prefer computed values over declared ones when a variable gets overridden downstream.
- **Screenshot**: read colors and proportions visually. State in the source footer that these are visual estimates, not exact hex/px reads.
- **CSS file**: read custom properties and literal values directly. This is the highest-confidence source - prefer it over a screenshot of the same page when both are available.

## Step 2 - name every token by role, not by value

Never name a token after its value. `--blue-500` breaks the day the brand color changes; `--accent` does not.

Use this naming pattern, adapted to what the source actually contains:

- Color: `--bg-primary`, `--bg-secondary`, `--surface`, `--text-primary`, `--text-secondary`, `--accent`, `--accent-hover`, `--success`, `--danger`, `--warning`, `--border`
- Typography: `--font-heading`, `--font-body`, `--text-xs` through `--text-xl` (only the steps actually present)
- Spacing: `--space-1` through `--space-n`, built off the base unit
- Radii: `--radius-sm`, `--radius-md`, `--radius-lg`
- Shadow: `--shadow-sm`, `--shadow-md`, `--shadow-lg`
- Motion: `--duration-fast`, `--duration-slow`, `--ease-standard`

A source with only 2 shadow levels gets 2 tokens, not 3 padded ones. If the source's own class or variable names already suggest a role - a class called `.btn-primary` using a specific blue - use that signal for the role label.

## Step 3 - more than 8 colors

If the source has more than 8 genuinely distinct colors:

1. Keep the 5-8 that carry a clear structural role: background, text, one or two accents, one or two status colors, border.
2. List every other color under a `## Long tail` note, with its hex and where it appeared, flagged for cleanup.
3. Tell the user this points at token-set consolidation, and that the `audit-design-tokens` skill is built for finding and merging near-duplicate colors across a whole codebase.

## Step 4 - never invent a missing value

If a group has no usable source data, do not estimate. Write the literal string, in both the CSS comment and the JSON `$value`:

```
(not extracted - provide a screenshot/URL and re-run)
```

Do this per group, not for the whole output. A source with a clear palette but no visible motion still yields five clean groups and one honest gap.

## Step 5 - gradients

If a background or fill is a gradient, do not collapse it to one flat hex. Store the stops in order:

```css
--accent-gradient: linear-gradient(135deg, #6366F1 0%, #8B5CF6 50%, #EC4899 100%);
```

In JSON, use `"$type": "gradient"` with `"$value"` as an ordered array of `{ "color": ..., "position": ... }` stops.

## Step 6 - output format

Always output all three parts, in this order: CSS block, JSON block, source footer. Never output only one or two.

### CSS block

A single `:root { }` block, grouped by section with a comment header per group, in the Step 1 capture order:

```css
:root {
  /* Palette */
  --bg-primary: #0F172A;
  --bg-secondary: #1E293B;
  --accent: #6366F1;
  --text-primary: #F8FAFC;
  --text-secondary: #94A3B8;

  /* Typography */
  --font-heading: 'Inter', system-ui, sans-serif;
  --font-body: 'Inter', system-ui, sans-serif;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 20px;
  --text-xl: 32px;

  /* Spacing (base 4px) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 16px;
  --space-4: 24px;

  /* Radii */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 16px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.06);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.10);

  /* Motion */
  --duration-fast: 150ms;
  --ease-standard: cubic-bezier(0.4, 0, 0.2, 1);
}
```

### JSON block

Follows the W3C Design Tokens Community Group draft shape: every leaf token is an object with `$value` and `$type`, grouped by the same sections as the CSS.

```json
{
  "color": {
    "bg-primary": { "$value": "#0F172A", "$type": "color" },
    "accent": { "$value": "#6366F1", "$type": "color" }
  },
  "typography": {
    "font-heading": { "$value": "Inter, system-ui, sans-serif", "$type": "fontFamily" },
    "text-base": { "$value": "16px", "$type": "dimension" }
  },
  "spacing": {
    "space-2": { "$value": "8px", "$type": "dimension" }
  },
  "radii": {
    "radius-md": { "$value": "8px", "$type": "dimension" }
  },
  "shadow": {
    "shadow-sm": { "$value": "0 1px 2px rgba(0, 0, 0, 0.06)", "$type": "shadow" }
  },
  "motion": {
    "duration-fast": { "$value": "150ms", "$type": "duration" }
  }
}
```

### Source footer

Close every output with a plain-text line naming where each group came from:

```
Source: palette + typography from https://example.com (computed styles); spacing + radii from screenshot (dashboard-2026-07.png, visual estimate); motion not extracted.
```

## Step 7 - updating an existing token set

If the user already has a token set from a previous run and asks to add, change, or re-check one group, re-emit the **complete** CSS block and complete JSON block, not just the changed group. A partial answer silently deletes every token left out. Carry forward every token you are not explicitly changing, byte-for-byte.

## Edge cases

- **Conflicting values across multiple screenshots** (two screenshots show a different shade of the "same" primary button): do not average or silently pick one. List both values against their source and ask which is canonical.
- **More than 8 palette colors**: see Step 3.
- **Gradients**: see Step 5.
- **No URL, screenshot, or CSS given**: ask for one of the three. Do not fabricate a plausible-looking palette.
- **URL fetch fails or the page is behind auth**: say so, and ask for a screenshot instead.
- **Screenshot too small or too compressed to read colors reliably**: say which groups you can still extract with confidence and which need a cleaner screenshot.

## Do not

- Do not name tokens after their value (`--blue-500`, `--16px`).
- Do not invent a hex code, font name, or spacing value that is not visible in the source.
- Do not skip the JSON block or the CSS block - always output both.
- Do not average or guess between conflicting sources.
- Do not pad a group to a round number - a third shadow that is not in the source, added just to reach 3, is a fabrication.
