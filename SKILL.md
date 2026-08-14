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

- **URL**: fetch the page, read linked CSS and inline styles, prefer computed values over declared ones when a variable gets overridden downstream. Computed values settle which declaration wins - they do not settle what the token says. For lengths, record the unit the winning declaration was authored in (Step 8).
- **Screenshot**: read colors and proportions visually. State in the source footer that these are visual estimates, not exact hex/px reads. A picture cannot show a unit either: every length read off a screenshot is a pixel estimate, and a `rem` or fluid scale looks identical to a fixed one - say that rather than presenting px as the authored value.
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

If a group has no usable source data, do not estimate. Mark the group not extracted, per group rather than for the whole output. A source with a clear palette but no visible motion still yields five clean groups and one honest gap.

The marker has one shape per output, and neither is free-form prose. A sentence dropped into a CSS block or a JSON `$value` does not survive the file it is written into.

**In CSS it is a comment.** Bare text inside `:root { }` is not a declaration, so the parser discards it along with everything up to the next semicolon - and that is the next token in the block. A not-extracted palette written as bare text deletes the first real token after it, silently. The marker only looks harmless when it lands on the last group in the block, where nothing follows it to lose.

```css
  /* Motion: not extracted - provide a screenshot/URL and re-run */
```

**In JSON it is an empty group carrying `$description`.** The group keeps its key and states the reason:

```json
"motion": { "$description": "not extracted - provide a screenshot/URL and re-run" }
```

Never hang the sentence on a leaf token. The draft requires `$value` on every token and requires that value to follow the rules for its `$type`, so `{ "$value": "(not extracted...)", "$type": "duration" }` fails validation and takes the rest of the file down with it. A group with no source data has no token names to put it under in the first place. An empty group is explicitly allowed by the draft as placeholder structure, which is exactly what this is.

**Never drop the group from either output.** A missing `motion` key does not read as "not measured", it reads as "this design has no motion" - a claim the source never made. That is the same failure this step exists to prevent, arriving by omission rather than by estimate.

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

  /* Typography (authored in rem, root 16px) */
  --font-heading: 'Inter', system-ui, sans-serif;
  --font-body: 'Inter', system-ui, sans-serif;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.25rem;
  --text-xl: 2rem;

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
    "text-base": { "$value": "1rem", "$type": "dimension" }
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
Source: palette + typography from https://example.com (computed styles; type scale authored in rem, root font size 16px); spacing + radii from screenshot (dashboard-2026-07.png, visual estimate, units assumed px); motion not extracted.
```

## Step 7 - updating an existing token set

If the user already has a token set from a previous run and asks to add, change, or re-check one group, re-emit the **complete** CSS block and complete JSON block, not just the changed group. A partial answer silently deletes every token left out. Carry forward every token you are not explicitly changing, byte-for-byte.

## Step 8 - keep the authored unit, not the computed pixel

A computed value is a measurement taken under one condition: one root font size, one viewport width, one zoom level. Storing that number as the token throws away the behavior the source actually had.

- **Relative lengths stay relative.** When the winning declaration is authored in `rem`, `em`, `ch`, `%`, `vw` or `vh`, the token carries that unit. A type scale authored in `rem` and recorded as `--text-base: 16px` hard-codes the browser default and drops the reader's own font-size setting - the thing the original respected (WCAG 2.2, SC 1.4.4 Resize text).
- **Fluid values stay whole.** `clamp(1rem, 2.5vw, 2rem)` is a rule, not a number. Store the expression: `--text-xl: clamp(1rem, 2.5vw, 2rem)`. Reading it at whatever viewport you fetched at produces a value no other viewport agrees with, and a second run at a different width silently "corrects" the token.
- **Give fluid values a valid JSON form.** The W3C draft's `dimension` type is a single number with a `px` or `rem` unit, so a `clamp()` expression is not a valid dimension token. Emit the floor and ceiling as two dimension tokens (`text-xl-min`, `text-xl-max`) and put the full expression in the token's `$description` - tooling gets something valid, a human still sees the real rule.
- **Name the root font size** in the source footer whenever the set contains `rem` values. A source using the `font-size: 62.5%` trick makes `1rem` equal 10px, and rem tokens read against the wrong root are wrong everywhere at once.
- **Pixels are right when the source authored pixels.** Hairline borders, radii and shadow offsets are often fixed on purpose. The rule is to keep what the source said, not to convert everything to `rem`.

## Edge cases

- **Source authored in `rem`, `em`, or `clamp()`**: see Step 8. Never store the computed pixel in place of the authored value.
- **Root font size is not 16px** (an `html { font-size: 62.5% }` or similar): record the root value in the source footer - every `rem` token in the set depends on it.
- **Conflicting values across multiple screenshots** (two screenshots show a different shade of the "same" primary button): do not average or silently pick one. List both values against their source and ask which is canonical.
- **More than 8 palette colors**: see Step 3.
- **Gradients**: see Step 5.
- **A whole group has no usable source data**: see Step 4. The marker is a CSS comment and an empty JSON group with `$description` - never bare text in the `:root` block, never a token `$value`, and never a group quietly left out.
- **No URL, screenshot, or CSS given**: ask for one of the three. Do not fabricate a plausible-looking palette.
- **URL fetch fails or the page is behind auth**: say so, and ask for a screenshot instead.
- **Screenshot too small or too compressed to read colors reliably**: say which groups you can still extract with confidence and which need a cleaner screenshot.

## Do not

- Do not name tokens after their value (`--blue-500`, `--16px`).
- Do not convert an authored `rem`, `em`, or `clamp()` value into a fixed pixel number.
- Do not invent a hex code, font name, or spacing value that is not visible in the source.
- Do not skip the JSON block or the CSS block - always output both.
- Do not write the not-extracted marker as bare text in the CSS block or as a token `$value` - it is a comment in one and an empty group's `$description` in the other (Step 4).
- Do not leave a not-extracted group out of the JSON. Silence reads as "this design has none", which is a claim about the source.
- Do not average or guess between conflicting sources.
- Do not pad a group to a round number - a third shadow that is not in the source, added just to reach 3, is a fabrication.
