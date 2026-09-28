# Arguments

The MCP tools and the command line take the same choices under two spellings. This file is the
whole of both; `SKILL.md` has the workflow.

## MCP tools

### `install`

Sets up Page Scanner's helper for every Chromium browser on the machine and returns which, and
what the user does next. Returns no secret. An unpacked build loaded in developer mode is found
and allowed by itself, so never ask the user for an extension id. One optional argument,
`extensionIds`: more ids to allow by hand. Shell:
`npx @page-scanner/cli install [--extension-id <id>]... [--browser-dir <dir>]... [--node <path>]`
(`--node` names the Node the helper runs on; `status` says when that Node has gone, and
`install` again fixes it), and
`uninstall` to take it out again.

### `pair`

The older setup. Writes the pairing and returns the port and token to paste into Chrome. One
optional argument, `rotate`, which issues a new token and invalidates the old one in every browser
paired by hand. The token comes back to the agent, so prefer `install`, or having the user run
`npx @page-scanner/cli pair` in a terminal.

### `list_browsers`

No arguments. The Chrome profiles connected right now, with the name the user gave each
one and how long it has been connected.

### `list_tabs`

| Argument      | Meaning                                                         |
| ------------- | --------------------------------------------------------------- |
| `browserId`   | Which browser. Optional when only one is connected.             |
| `waitSeconds` | How long to wait for a browser to connect. 0 fails immediately. |

Tabs carry the `windowId` they belong to, and windows say which is focused.

### `scan_page`

| Argument        | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `browserId`     | Which browser. Optional when only one is connected.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `tabId`         | A tab from `list_tabs`. Give one of `tabId`, `url` or `urls`.                                                                                                                                                                                                                                                                                                                                                                                                          |
| `url`           | Opens a background tab there, captures it, closes it again.                                                                                                                                                                                                                                                                                                                                                                                                            |
| `urls`          | Up to 50 addresses, captured one at a time into `outputPath`, which is then a directory. A page that fails is reported and the rest are captured.                                                                                                                                                                                                                                                                                                                      |
| `windowId`      | Which window to open `url` in. Ignored with `tabId`.                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `format`        | `pdf` (default), `png`, `jpeg`.                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `pageSize`      | PDF only. `a4` (default) and `letter` slice onto printable sheets with a half-inch margin; `phone` cuts it into phone screens, 390 × 844 px, with no margin; `auto` is one page the size of the capture.                                                                                                                                                                                                                                                               |
| `quality`       | JPEG only, 0.1 to 1.                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `videoHandling` | `frame` keeps a video's paused frame, `blank` leaves its area empty.                                                                                                                                                                                                                                                                                                                                                                                                   |
| `colorScheme`   | Which of a page's two themes to capture: `auto` (default, whatever the browser shows), `light`, `dark`.                                                                                                                                                                                                                                                                                                                                                                |
| `captureWidth`  | Lay the page out at a sheet's width first, so the PDF prints at 1:1: `window` (default), `a4`, `letter`; or `phone`, 390 px, the page as a phone shows it.                                                                                                                                                                                                                                                                                                             |
| `openEditor`    | Also leave the capture open in a Page Scanner editor tab.                                                                                                                                                                                                                                                                                                                                                                                                              |
| `outputPath`    | A file, or a directory to keep the suggested name. A leading `~` is home. Defaults to the cwd, or `~/Downloads` when that is `/` or read-only, as under Claude Desktop.                                                                                                                                                                                                                                                                                                |
| `hide`          | Clutter to hide before the capture and put back after: any of `ads`, `consent`, `chat`, `overlays`; `[]` hides nothing. Left out, the extension's own settings decide, and they hide all four unless the user changed them. The result counts what was hidden as `hidden`.                                                                                                                                                                                             |
| `slices`        | Also write the page as PNG slices top to bottom for looking at it with a vision model: `true` for at most 1568 px a side (past which Claude shrinks a picture), or a side in pixels from 256 to 4096 for another model. Each repeats the last 48 px of the one before. The result lists `slices[]` with each `path` and the band (`y`, `height`, CSS px) it shows, in a folder beside the file.                                                                        |
| `pictureFiles`  | Also write the page's pictures and canvases (at least 120 by 40 px), cut from the capture where they were drawn, as PNGs in a folder beside the file. The result lists `pictures[]` with each `path`, `kind` (`image` or `canvas`) and box.                                                                                                                                                                                                                            |
| `highlight`     | Passages you quoted from this page, highlighted in the file where the page has them as written (as `check_quotes` finds them), each listed under a "Quoted passages" bookmark in a PDF. The result's `highlighted` says which were found. One page only, not with `urls`.                                                                                                                                                                                              |
| `markdown`      | The page as Markdown, read from the page rather than the PDF: `inline` returns it in the result, `beside` writes a `.md` next to the file, `only` writes the `.md` and no file. Adds `page`: title, address, capture time, language, headings. Pictures are left out.                                                                                                                                                                                                  |
| `textOptions`   | How `markdown` is written, to fit a context window: `scope` (`page`, or `main` for the main content only), `maxChars` (200 or more, cut at a block boundary), `links` (`inline`, `references`, or `text` for no addresses), `pictures` (`omit`, or `alt` for a placeholder), `redact` (`true` replaces what Sensitive Text's rules and the user's patterns find with placeholders such as `⟦EMAIL 1⟧`; the Markdown only, not a file beside it). Only with `markdown`. |
| `fileName`      | The file name inside `outputPath`, as a template: `{n}` (place in `urls`), `{host}`, `{name}` (the suggested name), `{date}`, `{time}`, `{ext}` (added when left out). A `/` makes a subdirectory.                                                                                                                                                                                                                                                                     |
| `waitSeconds`   | How long to wait for a browser to connect. 0 fails immediately.                                                                                                                                                                                                                                                                                                                                                                                                        |

Returns the absolute `path`, `width` and `height` in CSS pixels, `mode` (`vector` or `raster`),
`selectableText`, and `truncated` (`null` when whole; otherwise what the page measured, what was
captured, and a sentence naming the gap). With `urls`: `results`, one per page in order, each
`ok: true` with that page's result and `url`, or `ok: false` with `error.code` and `error.message`;
and `written` and `failed` counts.

`citation`, when the page declares how to cite it, is what it declared (authors, date, journal,
DOI and the rest), unchecked. `translated`, when the browser had machine-translated the page
before the scan, is `{ "by": "chrome" }` or `"edge"`: the text is then a translation.

With `markdown`, the result also has `page`: `title`, `url`, `capturedAt` (ISO 8601), `language`
(the page's own `lang`, or `null`) and `headings` (`level` and `text`, in order), plus
`markdownPath` where the `.md` was written, or `markdown` with the text itself for `inline`. For
`only`, `path` is the `.md`. An extension older than this feature sends no text, and the call
fails with a sentence saying to update it.

With `textOptions.redact`, `page.redacted` has `count`, the values replaced, and `kinds`, how many
distinct ones of each kind (`email`, `phone`, `card`, `iban`, `nid`, `ip`, `token`, `pattern`).

With `textOptions`, `page.scope` says whether the main content was found and written, `page.cut`
(when `maxChars` left blocks out) gives the whole text's length and how many blocks went, and each
heading in `page.headings` has `offset` and `chars`: where its section starts in the text and how
long it is. With an extension older than this, the call fails with a sentence saying to update it.

### `diff_captures`

| Argument     | Meaning                                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| `oldPath`    | The older capture: a `.md` from `markdown` `beside` or `only`, a PNG, or a PDF with its `.md` beside it.     |
| `newPath`    | The newer capture of the same page, of the same kind.                                                        |
| `outputPath` | For two PNGs, where to write the newer one with the changed regions outlined. Defaults to `<name>.diff.png`. |
| `section`    | For Markdown, compare only the section under this heading, or a path of headings (`"Pricing > Pro"`).        |

Returns `kind` (`text` or `visual`) and `changed`. For text: `added` and `removed` line counts,
`hunks`, the same as a `unified` diff, each side's `source` and `captured` from its front matter,
and `sameAddress`; with `section`, `section` says where each side has it (`line`, `heading`), or `null` for a side without it, which counts as a change. For pictures: `regions` (`x`, `y`, `width`, `height` in pixels of the newer
capture), `changedFraction`, both sides' sizes, and `path`, the outlined picture, or `null` when
nothing changed. No browser is needed: both captures are already on disk.

### `check_quotes`

| Argument   | Meaning                                                                                                   |
| ---------- | --------------------------------------------------------------------------------------------------------- |
| `path`     | The capture's `.md`, as `scan_page` wrote it with `markdown` `beside` or `only`. Give this or `markdown`. |
| `markdown` | The capture's Markdown as text, as `scan_page` returned it with `markdown` `inline`.                      |
| `quotes`   | The quotes and values to look for, as they will appear in the answer. 1 to 200.                           |
| `loose`    | `true` to accept a quote that matches only once curly quotes, dashes, the ellipsis and case are folded.   |

Returns `checked`, `found`, `missing` and `allFound`, and per quote `match` (`exact`, `loose`,
`none`), `count` and `occurrences` (`line`, `section`). A missing quote has `near`: `agreed`, the
longest start of it the capture has, where, and `capture` and `quote`, how each goes on, or
`inside: true` when it is there only inside a longer word or number. The Markdown's markup is
undone and line wrapping ignored first; a quote never spans two blocks.

### `extract_design`

| Argument      | Meaning                                                                                                                                                                                                                                                                                      |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `browserId`   | Which browser. Optional when only one is connected.                                                                                                                                                                                                                                          |
| `tabId`       | A tab from `list_tabs`, read as it stands. Give one of `tabId`, `url`, `urls` or `crawl`.                                                                                                                                                                                                    |
| `url`         | Opens a background tab there, reloads it under each color scheme, reads it and closes it again.                                                                                                                                                                                              |
| `urls`        | Up to 20 pages of one site, read one after another and written as one design; `audit.md` lists what only one page uses.                                                                                                                                                                      |
| `crawl`       | A start address: its same-origin links are followed, one page per kind, honoring `robots.txt` and `nofollow`, never a log-out or delete address; a page that lands on another site or a page already read is left out.                                                                       |
| `maxPages`    | With `crawl`, the most pages to read. Default 10, at most 50.                                                                                                                                                                                                                                |
| `depth`       | With `crawl`, how many links from the start. Default 2, at most 5.                                                                                                                                                                                                                           |
| `outputPath`  | The directory. Defaults to `design-<host>` in the working directory, or in `~/Downloads` as for `scan_page`.                                                                                                                                                                                 |
| `minUses`     | How many elements must use a value before it becomes an inferred token. Default 2.                                                                                                                                                                                                           |
| `components`  | `true` to find the components as well: repeated structures and controls, their variants and hover and focus states, written to `components.json` and `components.md`, and `catalog.pdf` with each variant cut from the page as vector artwork. Each page is scanned too, so it takes longer. |
| `waitSeconds` | How long to wait for a browser to connect. 0 fails immediately.                                                                                                                                                                                                                              |

Returns `directory`, `files` (each file's path), `tokens` (counts by group), `declared`,
`unusedDeclared` (declared colors nothing was drawn in, left out), `contrast` (`pairs`, `failing`
below WCAG AA, `overImage` left to the eye), `nearlyEqual`, `onePage` (values only one page uses)
and `darkDiffers`. `pages` has each page's `ok`, and `message` when it was not read; in a crawl,
`landed` and `left` (`elsewhere` or `duplicate`) say where a page went and why it was left out,
and `crawl` has `robots` and the links `skipped`, by reason. With `components`, `components` has
the counts of components and variants.

## Command line

```
page-scanner install  [--extension-id <id>]... [--browser-dir <dir>]... [--node <path>] [--json]
page-scanner uninstall [--browser-dir <dir>]... [--json]
page-scanner pair     [--port <n>] [--rotate] [--wait <s>=120] [--no-wait] [--json]
page-scanner status   [--json]
page-scanner browsers [--json]
page-scanner tabs     [--browser <id|label>] [--wait <s>=30] [--json]
page-scanner scan     (--url <u>... | --urls <file|-> | --tab <id>) [--window <id>]
                      [--name <template>] [--markdown beside|only]
                      [--hide <kinds>|all|none]
                      [--browser <id|label>]
                      [--format pdf|png|jpeg=pdf] [--page-size auto|a4|letter|phone=a4]
                      [--quality <0-1>] [--video frame|blank]
                      [--scheme auto|light|dark]
                      [--page-width window|a4|letter|phone] [--open-editor]
                      [--out <file|dir>] [--wait <s>=30] [--timeout <s>=120] [--json]
page-scanner diff     <old> <new> [--section <heading>] [--out <file>] [--json]
page-scanner verify   <file> [--record <file.integrity.json>] [--json]
page-scanner check    <file.md> [<quote>...] [--from <file|->] [--loose] [--json]
page-scanner design   (--url <u>... | --urls <file|-> | --tab <id> |
                       --crawl <u> [--max-pages <n>=10] [--depth <n>=2])
                      [--components] [--window <id>]
                      [--out <dir>] [--min-uses <n>=2] [--browser <id|label>]
                      [--wait <s>=30] [--timeout <s>=120] [--json]
page-scanner serve    [--daemon] [--idle <min>]
page-scanner stop     [--json]
```

Run it as `npx @page-scanner/cli <command>`, or install it with `npm install -g @page-scanner/cli`.

`--out` is a file when it ends in a 2 to 5 character extension and is not an existing directory,
and a directory otherwise. A relative path is relative to the working directory. `--page-width` is
the command line's `captureWidth`, `--scheme` its `colorScheme`, `--video` its `videoHandling`.

`--json` puts exactly one JSON document on stdout, for success and for failure alike:

```json
{
  "ok": true,
  "path": "/Users/you/pdf.pdf",
  "width": 1280,
  "height": 4200,
  "mode": "vector",
  "selectableText": true,
  "truncated": null,
  "browserId": "b-9f2c41",
  "fileName": "PDF - Wikipedia.pdf"
}
```

```json
{ "ok": false, "code": 3, "error": "NO_BROWSER", "message": "No browser is connected." }
```

| Exit | Meaning                                                                       |
| ---- | ----------------------------------------------------------------------------- |
| 0    | success                                                                       |
| 1    | the browser was reached and the work failed                                   |
| 2    | the arguments were wrong                                                      |
| 3    | no usable browser: none connected, several connected, or the one named is not |
| 4    | not set up: no pairing on this machine (`install` makes one)                  |
| 5    | the daemon would not start                                                    |

Everything the pairing writes lives in `~/.page-scanner` (`config.json`, `daemon.json`,
`daemon.log`, and the helper in `native-host/`), or under `$PAGE_SCANNER_HOME`.
