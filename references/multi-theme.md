# Sources with more than one theme

Read this when the source declares more than one appearance of the same design: light and dark, a
high-contrast mode, a brand skin the user can switch. The rest of `SKILL.md` assumes one value per
role, and a two-theme source has two. This file says how to read both and where to put them.

## A theme is a declared condition, not a disagreement

The distinction decides which rule applies, so make it first.

- A **declared condition** is one the source states: a `@media (prefers-color-scheme: dark)` block, a
  `[data-theme="dark"]` or `.dark` selector, a `color-scheme` declaration, a visible theme switch in
  the UI. Both values are correct, each under its own condition.
- A **conflict** is two sources claiming the same condition and disagreeing - two screenshots of the
  same screen in the same theme showing a different primary button. That is the case Step 4's edge
  case covers: list both against their source and ask which is canonical.

Reading a light and a dark palette as a conflict and asking the user to pick one deletes half of a
two-theme design. Asking which of two same-condition shades is canonical is the correct question.
Nothing is inferred here: if the source does not declare a second condition, there is not one.

## Reading both

- **URL.** A computed value resolves under one color scheme, so a single fetch returns one theme and
  says nothing about the other. Read the declared blocks rather than the computed ones for anything
  inside a scheme query or a theme selector, or fetch twice under each preference. Note which scheme
  the fetch ran under whenever you report a computed value.
- **CSS file.** The conditions are visible in the file. This is the highest-confidence source for a
  themed design, because the mechanism is written down rather than reconstructed.
- **Screenshots.** Two screenshots are a theme pair only when the user says so or the UI shows the
  switch. Two shots that merely look different are the conflict case, not this one.

## What actually varies

Emit a second set only for the groups that differ. Color and shadow usually do; typography, spacing
and radii usually do not, and duplicating them makes two copies of one fact that drift apart later.
A group identical across themes stays in the base set and is not repeated.

## CSS shape

The base block is the default theme. Every additional theme is a second block keyed to the mechanism
the source itself uses - the same media query or the same selector, never one you invent - carrying
only the tokens that change. Token names stay identical across themes, because a name is a role and
the role does not change when the theme does.

```css
:root {
  /* Palette (light, the source default) */
  --bg-primary: #FFFFFF;
  --text-primary: #0F172A;
  --accent: #4F46E5;

  /* Spacing - identical in both themes, declared once */
  --space-2: 8px;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg-primary: #0F172A;
    --text-primary: #F8FAFC;
    --accent: #818CF8;
  }
}
```

## JSON shape

The W3C draft gives a token one `$value`, so a themed set cannot be one flat group. Emit the base
set, then a sibling group per additional theme holding only the tokens that differ, and say in the
group's `$description` what condition it applies under and that it is an override rather than a
complete set. The overriding group repeats the base token names exactly.

```json
{
  "color": {
    "bg-primary": { "$value": "#FFFFFF", "$type": "color" },
    "accent": { "$value": "#4F46E5", "$type": "color" }
  },
  "color-dark": {
    "$description": "overrides for prefers-color-scheme: dark - only the tokens that differ",
    "bg-primary": { "$value": "#0F172A", "$type": "color" },
    "accent": { "$value": "#818CF8", "$type": "color" }
  }
}
```

## Source footer

Name the condition each set was read under, next to where it came from:

```
Source: palette from https://example.com (computed styles, fetched under prefers-color-scheme: light); dark palette from the same page's @media (prefers-color-scheme: dark) block, read as declared; spacing + radii identical in both themes.
```

## Do not

- Do not merge two themes into one set by picking the one you fetched under. The other theme is not
  absent from the source, only from your fetch.
- Do not average two themed values, or place a themed pair on the long-tail list from Step 3. A dark
  background and a light background are not near-duplicate colors, they are one role under two
  conditions.
- Do not rename a token per theme (`--bg-primary-dark` as a separate role). The role is the same; the
  condition is what changed, and the block or group it sits in is what states that.
- Do not emit a second theme the source never declared, and do not compute one by inverting the first.
