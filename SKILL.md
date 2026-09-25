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

1. **Palette** - the hex colors that carry a structural role, each with a role label (naming rule in Step 2). Typically 5 to 8. That is the range to expect, not a count to hit: a source outside it in either direction is Step 3.
2. **Typography** - 1 to 2 font families plus the type scale (the distinct font sizes in use, largest to smallest).
3. **Spacing** - the base unit (commonly 4px or 8px) and the ramp built from it.
4. **Radii** - up to 3 border-radius values, smallest to largest.
5. **Shadows** - up to 3 box-shadow values, smallest to largest.
6. **Motion** - transition/animation durations and easing curves, only if the source shows motion (hover states, CSS transitions, a GIF or video the user provides). If the source gives no signal that motion exists at all, mark it not extracted instead (Step 4).

Source-specific reading:

- **URL**: fetch the page, read linked CSS and inline styles, prefer computed values over declared ones when a variable gets overridden downstream. Computed values settle which declaration wins - they do not settle what the token says. For lengths, record the unit the winning declaration was authored in (Step 8). A computed value also resolves under one color scheme, so a page with a dark theme returns only the theme you fetched under: check for a `prefers-color-scheme` block or a theme selector before treating one palette as the palette (Step 9).
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

## Step 3 - a palette outside 5 to 8

### More than 8 colors

If the source has more than 8 genuinely distinct colors:

1. Keep the 5-8 that carry a clear structural role: background, text, one or two accents, one or two status colors, border.
2. List every other color under a `## Long tail` note, with its hex and where it appeared, flagged for cleanup.
3. Tell the user this points at token-set consolidation, and that the `audit-design-tokens` skill is built for finding and merging near-duplicate colors across a whole codebase.

### Fewer than 5 colors

Some designs are built on three colors and a lot of whitespace. Emit the three. The 5 to 8 in Step 1 is the range a typical source falls in, not a quota the output has to reach, and a palette padded up to it invents every color past the source's own - which Step 4 and the Do-not list forbid in the same words.

Padding is tempting at this end because it does not look like invention. Deriving `--text-secondary` as the text color at 60% opacity, or a `--border` as some light grey mixed between background and text, produces plausible values by a plausible method, and the values still go in the set without the source ever showing them. A derived color is harder to catch than a guessed one for exactly that reason. If the source displays no secondary text color, it does not have one.

Two things keep a small palette honest:

- **One distinct value, one token.** Where the source uses the same hex for two structural roles - a page background and a card surface both `#FFFFFF` - name it for the role it plainly serves and alias the second to it, rather than declaring the same color twice. Two independent tokens claim a distinction the source does not make, and the day one of them moves the design changes in a way the source never showed.

```css
  --bg-primary: #FFFFFF;
  --surface: var(--bg-primary);
```

```json
"surface": { "$value": "{color.bg-primary}", "$type": "color" }
```

- **Say the size in the footer.** `palette: 3 colors, the full set the source uses`. Without it a three-token palette reads as an extraction that stopped early, and the next person re-runs the job hunting for what you missed. This is not the Step 4 marker and must never be written as one: not extracted means the source could not be read, and a source with three colors was read completely.

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

If a background or fill is a gradient, do not collapse it to one flat hex. Store the whole expression under one role name, stops in order:

```css
--accent-gradient: linear-gradient(135deg, #6366F1 0%, #8B5CF6 50%, #EC4899 100%);
```

A gradient is one role, not one role per stop. Its stops are not palette entries: they do not count toward the 5 to 8 colors of Step 1, and they never appear on the Step 3 long tail. Three stops of one gradient are one thing the source shows in one place, not three colors competing for a role, and listing them for cleanup asks the user to merge a gradient into itself. The one token sits with the palette in both outputs: under the `/* Palette */` header in CSS, in the `color` group in JSON.

In JSON, use `"$type": "gradient"` with `"$value"` as an ordered array of stops, each `{ "color": ..., "position": ... }` with the position as a number from 0 at the start of the gradient's axis to 1 at the end. Convert the CSS percentages: `50%` is `0.5`.

That array carries the stops and nothing else, so everything outside the stop list has nowhere to go - the direction (`135deg`), the gradient function itself, any repeat or size argument. Dropped, a `linear-gradient(135deg, ...)` and a `radial-gradient(...)` over the same stops emit the same token, and nobody reading the JSON can rebuild either one. Put the full CSS expression in the token's `$description`, the same remedy Step 8 uses for a `clamp()` the `dimension` type cannot hold: tooling gets a valid gradient, a human still sees the real rule.

```json
"accent-gradient": {
  "$type": "gradient",
  "$value": [
    { "color": "#6366F1", "position": 0 },
    { "color": "#8B5CF6", "position": 0.5 },
    { "color": "#EC4899", "position": 1 }
  ],
  "$description": "linear-gradient(135deg, #6366F1 0%, #8B5CF6 50%, #EC4899 100%)"
}
```

The CSS token and the JSON token describe the same gradient, so read them against each other before delivering: same stop colors in the same order, the same positions, and an expression in `$description` that matches the custom property byte for byte.

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
    "shadow-sm": {
      "$type": "shadow",
      "$value": {
        "color": "rgba(0, 0, 0, 0.06)",
        "offsetX": "0px",
        "offsetY": "1px",
        "blur": "2px",
        "spread": "0px"
      },
      "$description": "0 1px 2px rgba(0, 0, 0, 0.06)"
    }
  },
  "motion": {
    "duration-fast": { "$value": "150ms", "$type": "duration" }
  }
}
```

### Composite types carry an object, not the CSS string

`color`, `dimension`, `fontFamily` and `duration` take the value as authored. `shadow` does not. The draft defines it as a composite of `color`, `offsetX`, `offsetY`, `blur` and `spread`, so `"$value": "0 1px 2px rgba(0, 0, 0, 0.06)"` under `"$type": "shadow"` is a CSS declaration sitting in a slot that does not accept one - the same error as a `clamp()` stored as a `dimension` (Step 8) and a gradient flattened to its stops (Step 5), and the only one of the three this skill used to commit in its own worked example.

Three things the split gets wrong on the way out:

- **Every sub-value is a dimension, so it carries a unit.** A shadow with no spread writes `"spread": "0px"` - never a bare `0`, never the key left out. The draft wants all five, and a unitless number is not a dimension any more than `16` is a font size.
- **A layered `box-shadow` is one token, not one per layer.** Two comma-separated shadows under one role become an array of two shadow objects under a single `$value`. One role, one token: the rule Step 5 states for a gradient's stops, arriving on the other composite. Splitting them invents a `--shadow-sm-1` and a `--shadow-sm-2` the source never had, and Step 2 forbids the names on top of that.
- **`$description` still carries the CSS expression, byte for byte.** It is the remedy Step 5 and Step 8 already use, and it is what keeps whatever the five sub-values do not hold - an `inset` keyword among them - readable after the split. Read the two forms against each other the way Step 5 asks of a gradient: same color, same offsets, same blur, and an expression in `$description` that matches the custom property exactly.

The token count is untouched by any of this. A composite is one leaf token with one `$value`, whatever that value contains, so five sub-values are not five tokens and two layers are not two.

### The two blocks carry the same set

Read them against each other before delivering, the way Step 5 already asks of a gradient's two forms. Every custom property in the CSS block has its token in the JSON, every group in one is a group in the other, and the not-extracted markers stand in both. There is exactly one legal divergence in the whole set: a `clamp()` is one custom property in CSS and a floor plus a ceiling in JSON, because the draft's `dimension` type cannot hold the expression (Step 8). Anything else on one side and not the other is a token that block deletes - the failure Step 7 describes, arriving inside a single answer instead of across two.

The count is the fast version of the check. Custom properties in the CSS block, leaf tokens in the JSON, and the two numbers differ by one for each `clamp()` in the set and by nothing else. A composite changes the shape of one side and not the set: `--shadow-sm` is one custom property and one leaf token whether its `$value` is an object, an array of two, or the string it should never have been.

### Source footer

Close every output with a plain-text line naming where each group came from. Every group the blocks carry is on it, the not-extracted ones included: a group emitted and never traced is the same silence Step 4 refuses, arriving in the footer instead of in the JSON.

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

## Step 9 - a source with more than one theme

Some sources declare the same design twice: light and dark, a high-contrast mode, a switchable skin.
Every rule above assumes one value per role, and these sources have two, so they need their own shape:
which values to read, where the second set goes in each output, and why a themed pair is not a
conflict. That is [references/multi-theme.md](references/multi-theme.md).

The one rule to carry without opening the file: a theme is a condition the source declares, and both
of its values are correct. Never collapse a themed pair to the one you happened to fetch under, and
never invent a second theme the source does not declare.

## Edge cases

- **Source authored in `rem`, `em`, or `clamp()`**: see Step 8. Never store the computed pixel in place of the authored value.
- **Root font size is not 16px** (an `html { font-size: 62.5% }` or similar): record the root value in the source footer - every `rem` token in the set depends on it.
- **Conflicting values across multiple screenshots** (two screenshots show a different shade of the "same" primary button): do not average or silently pick one. List both values against their source and ask which is canonical. A conflict is two sources claiming the *same* condition and disagreeing - if the two shots are the light and dark version of one screen, that is two declared conditions and both values are right, so it is Step 9, not this. Asking which of a themed pair is canonical deletes half the design.
- **Source declares light and dark, a high-contrast mode, or a switchable skin**: see Step 9 and `references/multi-theme.md`. The second set goes in a block keyed to the source's own mechanism, with the same token names, carrying only the tokens that differ.
- **More than 8 palette colors**: see Step 3.
- **Fewer than 5 palette colors**: see Step 3. Emit what the source has. The 5 to 8 range is what a typical source yields, not a floor the output has to reach, and a color derived from another one - a secondary text at 60% opacity, a border grey mixed between background and text - is invented by a method that makes it look measured. Where one hex serves two roles, alias the second to the first instead of declaring the color twice, and say the palette's size in the footer so a small set does not read as an incomplete one.
- **Gradients**: see Step 5. One role, one token, in both outputs. Stops never count toward the Step 1 palette or land on the Step 3 long tail, and the JSON token carries the full CSS expression in `$description`, since its stop array cannot hold the gradient's direction.
- **Shadows in the JSON block**: `shadow` is a composite type, so its `$value` is an object of `color`, `offsetX`, `offsetY`, `blur` and `spread`, every sub-value carrying a unit with `0px` written out, and a layered `box-shadow` is one token holding an array of them. The CSS expression goes in `$description`, as it does for a gradient and a `clamp()`. See Step 6.
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
- Do not flatten a source's second theme into one set, and do not file a themed pair as near-duplicate colors on the long-tail list - one role under two declared conditions is not two colors competing for one role.
- Do not put a CSS `box-shadow` string in a `shadow` token's `$value`, and do not split a layered one into a token per layer. It is a composite type: the string is the same invalid-by-type error as a `clamp()` stored as a `dimension`, and the split is the gradient's stops failure on the other composite (Step 6).
- Do not split a gradient into its stops - not as separate palette tokens, not as long-tail entries, and not as a JSON token whose direction was dropped on the way in.
- Do not pad a group to a round number - a third shadow that is not in the source, added just to reach 3, is a fabrication. The same goes for padding up to the bottom of a range: a palette lifted from three colors to five invents two, and derives them from the real ones so they read as measured (Step 3).

## Reference material

See [references/multi-theme.md](references/multi-theme.md) for sources that declare more than one
theme: how to read both without a second fetch overwriting the first, the CSS and JSON shapes for a
themed set, the footer line that names the condition each set was read under, and the line between a
declared condition and a genuine conflict.
