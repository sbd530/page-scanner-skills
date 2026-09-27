# DESIGN.md template

Fill every part from the files `extract_design` wrote. Delete a part the files say nothing about
rather than filling it with a guess; say what is missing under Gaps.

```markdown
# DESIGN.md

The design system of <site>, measured on <n> pages of <host> with Page Scanner on <date>. It is
what the site draws, not its source code: values are computed styles, and names in `code` with
`--` are the site's own custom properties.

## Provenance

| What            | Where it came from                                                                                               |
| --------------- | ---------------------------------------------------------------------------------------------------------------- |
| Pages read      | <the list from audit.md>                                                                                         |
| Declared tokens | the site's own custom properties, <count> of them drawn with (<unusedDeclared> declared colors unused, left out) |
| Inferred tokens | values used on at least <minUses, 2 unless changed> elements, named here by role                                 |
| Components      | repeated structures and controls on those pages, states forced with `:hover` and `:focus-visible`                |

Names given here to inferred values:

| Name in this file | Measured as | Uses | Used as |
| ----------------- | ----------- | ---- | ------- |
| `text-primary`    | `color-1`   | 412  | text    |

## Color

### Surfaces and text

| Token          | Light     | Dark      | Role                          |
| -------------- | --------- | --------- | ----------------------------- |
| `--background` | `#fdfdfd` | `#131413` | page background (61 elements) |

### Borders

### Brand and accents

### Status

## Typography

| Role | Family | Size / line height | Weight | Letter spacing | Uses  |
| ---- | ------ | ------------------ | ------ | -------------- | ----- |
| Body | Inter  | 16px / 24px        | 400    | normal         | 1,203 |

## Shape and space

- Radius: <values by use>.
- Spacing scale: <values by use>.
- Shadows: <values>.
- Motion: <durations and easings>.
- Breakpoints: <widths>.

## Components

### Button

<instances> instances on <pages>.

| Variant | Look                                                                 | Hover        | Focus                | Disabled |
| ------- | -------------------------------------------------------------------- | ------------ | -------------------- | -------- |
| Filled  | bg `#3ecf8e`, text `#1c1c1c`, radius 6px, padding 8px 16px, 14px/500 | bg `#36b37e` | 2px border `#1c1c1c` | 1 of 2   |

## Do and don't

- Do <rule>. (<which file shows it>)
- Don't <rule>. (<which file shows it>)

## Gaps

- <what was not measured: states behind a click, pages not read, text over pictures not measured>
```

## Notes on each part

- **Provenance** is what lets a reader trust the rest. Keep the name mapping complete: every
  name you gave, the number it had, its uses.
- **Color** groups tokens by `usedAs` and by what they sit on, not alphabetically. Put the dark
  value beside the light one when `dark` is set, and the token's `aliases` in its Role cell
  ("also `--card-foreground`"), so a reader who finds either name in the site's CSS finds it here.
- **Typography** comes from the font-size, line-height, weight and family tokens and from the
  text pairs in `extract.json`; a role is a size and weight used for one kind of text.
- **Components** keep only what `components.json` has. A variant used once may be a one-off; say
  so rather than making it a documented variant.
- **Do and don't** cites its evidence: `contrast.md` for a failing pair, `audit.md` for nearly
  equal values, `components.json` for a component with more variants than it needs.
