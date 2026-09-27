---
name: design-md
description: Write a website's design system down as a DESIGN.md (colors, type, spacing, radii, motion, components, and do and don't), with an optional Tailwind config and React components, from what Page Scanner measured on the live site with extract_design or the page-scanner design command. Use when the user wants a site's design system documented, reverse-engineered from the site, audited, or handed to an agent to build new pages in the same style. Every value comes from the measured files; nothing is guessed from screenshots.
license: Apache-2.0
compatibility: Google Chrome with the Page Scanner extension, Node.js 24 or newer, and either the @page-scanner/mcp server or the @page-scanner/cli command.
metadata:
  author: sbd530
  version: '1.0'
  homepage: https://docs.pagescanner.app/mcp
---

# DESIGN.md from a live site

Page Scanner measures a site's design as the browser draws it, in the user's own Chrome, and
writes it into files. This skill turns those files into a `DESIGN.md` a person can read and an
agent can build from. Page Scanner does the measuring and decides nothing a model would have to;
you do the writing: the names, the prose, the rules. The `page-scanner` skill covers setting
Page Scanner up and connecting Chrome; read it first if nothing is connected.

## 1. Gather

Read several kinds of page, not one: a home page has no pricing table and a docs page no hero.

- MCP: `extract_design` with `crawl` set to the site's home page and `components: true`. If the
  tool is not listed, the server is an older version; use the command instead. When you know the
  pages, give `urls` instead of `crawl`. `maxPages` defaults to 10.
- Command: `page-scanner design --crawl <home> --components --out <dir> --json`.

A page behind a login is read only from a tab the user has open (`tabId`, or `--tab`). A crawl
takes about 15 seconds a page with components; tell the user before a long one.

## 2. Read the files

All of them are in the one directory the tool returned. Read them in this order.

- **`tokens.json`** (W3C Design Tokens format). Each token has `$value` and, under
  `$extensions["app.pagescanner"]`: `source` (`declared`: the site's own CSS custom property, named
  by it; `inferred`: a value the pages use often, numbered like `color-3`), `uses` (how many
  elements or text runs), `usedAs` for colors (`text`, `background`, `border`), and `dark` when the
  dark scheme differs. Declared names are the site's vocabulary: keep them. Declared colors nothing
  is drawn in were left out; the tool's result (`unusedDeclared`) and the first lines of
  `tokens.css` count them. Do not reintroduce them. Several declared names often share one value
  (`--foreground` and `--card-foreground`), or two colors nobody can tell apart; the token
  carries the most used name and the uses of all of them, and the others are in `aliases` under
  `$extensions["app.pagescanner"]`. List them beside it rather than choosing for the site.
- **`audit.md`**: values nearly equal to a more used one, probably meant as one, and values used
  on one page only. These are slips or special cases, not tokens: report them, never promote them.
- **`contrast.md`**: every text color over its background with its WCAG 2 ratio. Pairs below AA
  go under Do and don't. Pairs listed over a gradient or a picture were not measured; say they
  need a look by eye.
- **`components.json`** (with `components: true`): each component with its instance count, the
  pages it is on and its variants, each with its look, its `hover` and `focus` changes and how
  many are disabled. `components.md` is the same as a table. `catalog.pdf` shows every variant;
  point the user to it, you need not read it.
- **`extract.json`**: the raw counts per page, as the browser wrote them. Only for a detail the
  others do not answer, and to check a count that looks wrong.

In `tokens.json`, `components.json` and the reports, colors are hex; an eight-digit one has alpha
(`#03030315` is 8 % of `#030303`). `extract.json` keeps the browser's own notation (`rgb()`,
`lab()`, `oklch()`). The look of a component was measured in the color scheme Chrome was in during
its scan; `tokens.json`'s `dark` fields have the other scheme for tokens. When the two schemes
draw the same (no `dark` anywhere), one value column is enough. When they differ, a declared token
without `dark` is the same in both, but an inferred one without `dark` only had no twin the tool
could be sure of: write its dark value as unknown rather than pairing it yourself.

Read the counts with judgment:

- An inferred token needs two uses by default (`minUses`, `--min-uses`).
- A declared length counts only the property its name says: `--radius-*` radii, `--space-*` and
  `--gap-*` spacing, `--text-*` font sizes, `--leading-*` line heights, `--tracking-*` letter
  spacing. One whose name says none (`--blur-sm`) has 0 uses because nothing measured draws it,
  not because it is unused; describe it from its name. A `rem` value was read at 16 px, and a
  radius of `9999px` is a pill.
- Spacing includes layout results: a large or fractional value (`128px`, `531.25px`) is usually an
  auto margin or a centered column, not a step of the scale.
- A component's empty Hover or Focus cell means no change was measured; states are forced on the
  first instance of each shape only, so say "not seen" rather than "none".
- Text pairs have a size and a weight but no family or line height. Typography rows that combine
  them with the font-family and line-height tokens are an inference; say so.

## 3. Write DESIGN.md

Use the outline in `references/template.md`: Provenance, Color, Typography, Shape and space,
Components, Do and don't, and Gaps.

- **Every value from a file.** Each hex, size and duration in DESIGN.md is in `tokens.json`,
  `components.json` or `extract.json`. If you want a value no file has, write the gap down under
  Gaps instead.
- **Name the inferred tokens by role, and keep the number.** `color-1` used as text 400 times is
  `text-primary`; a background used by every card is `surface-card`. The Provenance table maps
  each name you gave to the number it had, so the reader can tell your names from the site's.
  Never present a name you gave as the site's own.
- **Keep declared names exactly**, with `--` and all, and give each a role from `usedAs`.
- **Order by use.** A value used twice is not a design decision; say "used on one page only"
  rather than giving it a row of its own.
- **Components**: one section per component the user would recognize (buttons, fields, cards,
  tabs, navigation), with a variant table: look, hover, focus, disabled. Name variants by what they
  look like (filled, outlined, text only), not by guesses about intent. Merge two components only
  when their tables are the same.
- **Do and don't** comes from the evidence: the contrast failures, the nearly-equal pairs, a
  component with too many variants. Each rule says which file shows it.
- Keep it short. A DESIGN.md is read before every change; a page of tables beats a page of prose.

## 4. Optional: Tailwind and components

When the user asks for code, start from `tailwind.preset.js`, which already maps every token, and
rename its keys to the names in DESIGN.md. For React components, one per component section, with
its variants as props and its hover and focus from the tables. Say plainly that these are a
starting point from measured values, not the site's source code.

## What not to do

- Do not copy the site's logo, wordmark, icons, images, fonts or text into anything you write.
  Name a font family; do not bundle it. The design system is measured; the identity stays theirs.
- Do not claim a value, a state or a component no file has, or read values off a screenshot.
- Do not present the audit's slips as tokens, or an unmeasured contrast pair as a pass.
- Do not rewrite a declared token's name or value because another would look tidier.

## If something is missing

- **`components.json` is not there**: `components` was not asked for; run again with it.
- **Few tokens and "0 declared"**: the site declares no custom properties, so every token is
  inferred and numbered; your naming carries more weight, and Provenance should say so.
- **A page answered 403 or 404** and was left out: a site can refuse a browser it takes for
  automated. Read the pages from tabs the user opens instead.
