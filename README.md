# Bekk design tokens

Design tokens for Bekk, kept in sync with Figma (the **Tokens Master** file) through the **Token Spark** plugin. The JSON in `tokens/` is the source of truth; `dist/tokens.css` is generated from it.

```
tokens/                Source of truth (W3C DTCG JSON)
  primitives/
    color.json           The colour palette (12-step scales + profile colours)
    sizes/               Font sizes and dimensions: mobile.json / desktop.json
  semantic/
    themes/              One file per theme: Vann, Jord, Ild, Luft
    light.json           Role-based colours for light mode
    dark.json            …and for dark mode
    global/              Typography styles, radius, border width, spacing, fonts
  metadata.json          Themes, colour schemes, sizes, Figma mapping, output formats
dist/
  tokens.css             Generated – the only output enabled today
```

## How the colours fit together

```
primitives          →   theme (Vann, light)               →   semantic
color.grey.100          color.light.dominant.base.default     color.dominant.base.default
```

A **theme** decides which palette colours play which role, once for light and once for dark. **Semantic** tokens are what you use in code. They follow whichever theme and colour scheme is active, so components never reference a theme, a mode or a palette colour directly.

### Naming: `color.{role}.{slot}.{state}`

| Role | What it's for |
| --- | --- |
| `neutral` | Page backgrounds, ordinary surfaces, body text, borders |
| `dominant` | The Bekk identity: grey + rebell (#D4FF26). **The same in every theme.** Its `base` is rebell in both light and dark. |
| `accent` | The theme colour – **the only role that changes between Vann, Jord, Ild and Luft** |
| `interactive` | Links and focus |
| `success` · `danger` · `warning` · `info` | Status |

| Slot | States | Use |
| --- | --- | --- |
| `background` | `default` (neutral), `tinted` (dominant, accent) | Page / section background |
| `surface` | `default`, `tinted`, `hover`, `active` | Cards, panels, list rows |
| `border` | `default` | Borders and dividers |
| `text` | `default` | Text and icons on `background` / `surface` |
| `base` | `default`, `hover`, `active` | Solid fills – buttons, selected checkboxes and toggles, badges |
| `base-contrast` | `default` | Text and icons on `base` |

`neutral` only has what's shared: `background.default`, `surface.default`, `border.default`, `text.default` and `text.subtle`, plus `disabled` states for `surface`, `base`, `border` and `text`. For hover and active surfaces, use `dominant` or `accent`. Status roles and `interactive` have no `background` of their own and use `neutral.background.default`. Mix roles instead of looking for a duplicate token: a card can be `neutral.surface.default` with an `accent.base.default` button, and it still works when the theme changes.

### How the steps are picked

`accent`, the status roles and `interactive` all use the same steps on their own palette, so the four surface states never collapse into each other:

| | `surface.default` | `surface.tinted` | `surface.hover` | `surface.active` |
| --- | --- | --- | --- | --- |
| Light | 50 | 100 | 200 | 300 |
| Dark | 800 | 700 | 600 | 900 |

`accent.background.tinted` is 50 in light (the theme's light profile colour) and 900 in dark.

`dominant` is set by hand to match bekk.no and forside.bekk.no:

| | Light | Dark |
| --- | --- | --- |
| `background.tinted` | grey 25 (Dag) | grey 900 |
| `surface.default` | grey 50 | grey 700 |
| `surface.tinted` | rebell 50 | rebell 800 |
| `surface.hover` / `active` | rebell 100 / 25 | rebell 700 / 900 |
| `base.default` / `hover` / `active` | rebell 50 / 100 / 200 | rebell 50 / 100 / 200 |

`base.hover` (rebell 100) is one step darker than `surface.tinted` (rebell 50), so a button on a rebell surface still shows its hover state.

**One rule for contrast, no exceptions:**

- on `background` or `surface` → use `{role}.text.default`
- on `base` → use `{role}.base-contrast.default`

All pairs pass WCAG AA (4.5:1) in every theme and in both light and dark.

### Locking a section to one colour scheme

Some sections should look the same whatever the page's mode – the rebell highlight card, or a dark footer. Don't invent tokens for these: set `data-color-scheme` on the section to the mode it was designed in, and use the ordinary tokens inside it.

```html
<!-- Rebell card: identical on light and dark pages -->
<article class="card" data-color-scheme="light">…</article>

<!-- Dark footer on a light page -->
<footer data-color-scheme="dark">…</footer>
```

```css
.card {
  background: var(--color-dominant-surface-tinted);   /* rebell 50 */
  color: var(--color-dominant-text-default);           /* grey 800 */
  border-top: var(--border-width-default) solid var(--color-dominant-border-default);
}
```

Use `base` only for buttons and small elements. Large areas use `background` or `surface`, locked to a scheme if needed. In Figma, the same thing is done by setting the frame's `semantic` mode to Light or Dark.

### Data colours

`color.data.blue` (#0142FE) and `color.data.red` (#FC411E) are for charts and data visualisation only – don't use them for UI. They're the same in every theme and mode.

## Using it

### Web

```html
<link rel="stylesheet" href="dist/tokens.css" />
<html data-color-scheme="auto" data-theme="ild"></html>
```

```css
.card {
  background: var(--color-neutral-surface-default);
  color: var(--color-neutral-text-default);
  border: var(--border-width-default) solid var(--color-neutral-border-default);
  border-radius: var(--radius-md);
  padding: calc(var(--spacing-unit) * 4);
}

.button-secondary {
  background: var(--color-accent-base-default);
  color: var(--color-accent-base-contrast-default);
}
.button-secondary:hover {
  background: var(--color-accent-base-hover);
}

.checkbox[data-state="checked"] {
  background: var(--color-dominant-base-default);      /* rebell */
  color: var(--color-dominant-base-contrast-default);
}
```

| Attribute | Values | Notes |
| --- | --- | --- |
| `data-color-scheme` | `light` · `dark` · `auto` | **Required** – semantic colours only exist inside it. `auto` follows the OS setting. Can be nested to lock a section (see above). |
| `data-theme` | `vann` · `jord` · `ild` · `luft` | Optional – `vann` is the default. To theme just one section, put `data-theme` **and** `data-color-scheme` on it; `data-theme` alone doesn't change the semantic colours. |
| `data-size` | `desktop` | Optional – desktop sizes switch on automatically from 768px; set this to use them at any width. |

### What's in `tokens.css`

| Group | Variables | Notes |
| --- | --- | --- |
| Semantic colours | `--color-{role}-{slot}-{state}` | Use these. |
| Theme colours | `--color-light-…`, `--color-dark-…` | Internal plumbing behind the semantic colours – don't use directly. |
| Palette | `--color-{palette}-{step}` (`grey`, `black`, `white`, `rebell`, `vann`, `jord`, `ild`, `luft`, `blue`, `green`, `red`, `orange`, `purple`) | Avoid in components. Profile colours from the brand guide are marked in the Figma variable descriptions (e.g. `rebell-50` = rebell, `grey-800` = Natt, `grey-25` = Dag). |
| Radius | `--radius-none` · `sm` · `md` · `lg` · `xl` · `full` | 0 / 2 / 4 / 8 / 12 / 9999px |
| Border width | `--border-width-default` · `highlight` · `focus` | 1 / 2 / 4px |
| Spacing | `--spacing-unit` | Base step; multiply with `calc()` |
| Font family | `--font-family-base` · `display` · `action` | All GT America today |
| Font weight | `--font-weight-{name}` | See typography below |
| Sizes | `--primitive-font-size-*`, `--primitive-dimension-*` | Change between mobile and desktop |
| Typography | `--typography-{style}-{property}` | See below |

### Typography

Each text style from Figma becomes a set of variables with valid CSS values:

```css
.label {
  font-family: var(--typography-action-fontFamily);
  font-size: var(--typography-action-fontSize);
  font-weight: var(--typography-action-fontWeight);       /* 500 */
  font-stretch: var(--typography-action-fontStretch);     /* 125% */
  line-height: var(--typography-action-lineHeight);       /* 1.4 */
  letter-spacing: var(--typography-action-letterSpacing); /* 0.04em */
  text-transform: var(--typography-action-textCase);      /* uppercase */
}
```

| Style | Size (mobile / desktop) | Line height | Notes |
| --- | --- | --- | --- |
| `heading-lg` | 40 / 56 | 1.3 | |
| `heading-md` | 32 / 48 | 1.3 | |
| `heading-sm` | 20 / 24 | 1.4 | |
| `leading` | 24 / 32 | 1.4 | Intro text |
| `body` | 16 / 20 | 1.4 | |
| `title` | 16 / 20 | 1.4 | Medium |
| `small` | 14 / 18 | 1.4 | |
| `caption` | 12 / 16 | 1.4 | |
| `action` | 12 / 16 | 1.4 | Expanded Medium, uppercase, 0.04em – buttons and labels |

How Figma values are translated (by Token Spark, CSS output only – the JSON keeps Figma's values so syncing stays lossless):

- A font style name becomes weight, width and italic: `"Expanded Medium"` → `fontWeight: 500` plus `fontStretch: 125%`; `"Light Italic"` → `300` plus `fontStyle: italic`. The `-fontStretch` and `-fontStyle` variables only exist when they aren't the default (`100%`, `normal`), so use a fallback: `font-stretch: var(--typography-body-fontStretch, 100%)`. The global `--font-weight-*` tokens get the same `-fontStretch` / `-fontStyle` siblings.
- Line height and letter spacing are stored as percent: `140` → `1.4`, `4` → `0.04em`.
- Text case: `upper` → `uppercase`, `title` → `capitalize`, `original` → `none`.

The full GT America family is licensed for web. Expanded and italic styles only render correctly if the matching font files are loaded with `@font-face` (with `font-stretch` / `font-style` set) – the tokens describe the style, they don't load the font. If a style name can't be parsed, Token Spark falls back to weight 400 with a warning; custom names can be mapped with `fontStyles` and `fontStretchKeywords` in `metadata.json`.

### Not in the tokens

- **Motion and shadow** live in code only. Figma can't represent them as more than text strings.
- **Fluid type** (`clamp()`) and ready-made typography classes are not generated – font sizes step between mobile and desktop.

### Other platforms

Only CSS is enabled. Token Spark can also generate JS, TS, Dart (Flutter) and Swift (iOS) – turn them on under `platforms` in `tokens/metadata.json` and they'll appear on the next push. Those outputs carry the font weight but not width or italic.

## Changing tokens

Designers change variables and text styles in Figma, then run **Token Spark → Push to GitHub**. That opens a pull request with the updated `tokens/` and the regenerated `dist/` – review and merge it like any other change.

`dist/` is only regenerated by a push, and the next push overwrites any hand edits – so change tokens in Figma (or edit the JSON and **Pull** it into Figma), not in the generated files.
