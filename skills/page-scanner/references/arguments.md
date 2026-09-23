# Arguments

The MCP tools and the command line take the same choices under two spellings. This file is the
whole of both; `SKILL.md` has the workflow.

## MCP tools

### `pair`

Writes the pairing and returns the port and token to paste into Chrome. One optional argument,
`rotate`, which issues a new token and invalidates the old one in every browser. The token comes
back to the agent, so prefer having the user run `npx @page-scanner/cli pair` in a terminal.

### `list_browsers`

No arguments. The Chrome profiles paired and connected right now, with the name the user gave each
one and how long it has been connected.

### `list_tabs`

| Argument      | Meaning                                                         |
| ------------- | --------------------------------------------------------------- |
| `browserId`   | Which browser. Optional when only one is connected.             |
| `waitSeconds` | How long to wait for a browser to connect. 0 fails immediately. |

Tabs carry the `windowId` they belong to, and windows say which is focused.

### `scan_page`

| Argument        | Meaning                                                                                                                                                                                                                                                               |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `browserId`     | Which browser. Optional when only one is connected.                                                                                                                                                                                                                   |
| `tabId`         | A tab from `list_tabs`. Give one of `tabId`, `url` or `urls`.                                                                                                                                                                                                         |
| `url`           | Opens a background tab there, captures it, closes it again.                                                                                                                                                                                                           |
| `urls`          | Up to 50 addresses, captured one at a time into `outputPath`, which is then a directory. A page that fails is reported and the rest are captured.                                                                                                                     |
| `windowId`      | Which window to open `url` in. Ignored with `tabId`.                                                                                                                                                                                                                  |
| `format`        | `pdf` (default), `png`, `jpeg`.                                                                                                                                                                                                                                       |
| `pageSize`      | PDF only. `a4` (default) and `letter` slice onto printable sheets with a half-inch margin; `auto` is one page the size of the capture.                                                                                                                                |
| `quality`       | JPEG only, 0.1 to 1.                                                                                                                                                                                                                                                  |
| `videoHandling` | `frame` keeps a video's paused frame, `blank` leaves its area empty.                                                                                                                                                                                                  |
| `colorScheme`   | Which of a page's two themes to capture: `auto` (default, whatever the browser shows), `light`, `dark`.                                                                                                                                                               |
| `captureWidth`  | Lay the page out at a sheet's width first, so the PDF prints at 1:1: `window` (default), `a4`, `letter`.                                                                                                                                                              |
| `openEditor`    | Also leave the capture open in a Page Scanner editor tab.                                                                                                                                                                                                             |
| `outputPath`    | A file, or a directory to keep the suggested name. A leading `~` is home. Defaults to the cwd, or `~/Downloads` when that is `/` or read-only, as under Claude Desktop.                                                                                               |
| `markdown`      | The page as Markdown, read from the page rather than the PDF: `inline` returns it in the result, `beside` writes a `.md` next to the file, `only` writes the `.md` and no file. Adds `page`: title, address, capture time, language, headings. Pictures are left out. |
| `fileName`      | The file name inside `outputPath`, as a template: `{n}` (place in `urls`), `{host}`, `{name}` (the suggested name), `{date}`, `{time}`, `{ext}` (added when left out). A `/` makes a subdirectory.                                                                    |
| `waitSeconds`   | How long to wait for a browser to connect. 0 fails immediately.                                                                                                                                                                                                       |

Returns the absolute `path`, `width` and `height` in CSS pixels, `mode` (`vector` or `raster`),
`selectableText`, and `truncated` (`null` when whole; otherwise what the page measured, what was
captured, and a sentence naming the gap). With `urls`: `results`, one per page in order, each
`ok: true` with that page's result and `url`, or `ok: false` with `error.code` and `error.message`;
and `written` and `failed` counts.

With `markdown`, the result also has `page`: `title`, `url`, `capturedAt` (ISO 8601), `language`
(the page's own `lang`, or `null`) and `headings` (`level` and `text`, in order), plus
`markdownPath` where the `.md` was written, or `markdown` with the text itself for `inline`. For
`only`, `path` is the `.md`. An extension older than this feature sends no text, and the call
fails with a sentence saying to update it.

### `diff_captures`

| Argument     | Meaning                                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| `oldPath`    | The older capture: a `.md` from `markdown` `beside` or `only`, a PNG, or a PDF with its `.md` beside it.     |
| `newPath`    | The newer capture of the same page, of the same kind.                                                        |
| `outputPath` | For two PNGs, where to write the newer one with the changed regions outlined. Defaults to `<name>.diff.png`. |

Returns `kind` (`text` or `visual`) and `changed`. For text: `added` and `removed` line counts,
`hunks`, the same as a `unified` diff, each side's `source` and `captured` from its front matter,
and `sameAddress`. For pictures: `regions` (`x`, `y`, `width`, `height` in pixels of the newer
capture), `changedFraction`, both sides' sizes, and `path`, the outlined picture, or `null` when
nothing changed. No browser is needed: both captures are already on disk.

## Command line

```
page-scanner pair     [--port <n>] [--rotate] [--wait <s>=120] [--no-wait] [--json]
page-scanner status   [--json]
page-scanner browsers [--json]
page-scanner tabs     [--browser <id|label>] [--wait <s>=30] [--json]
page-scanner scan     (--url <u>... | --urls <file|-> | --tab <id>) [--window <id>]
                      [--name <template>] [--markdown beside|only]
                      [--browser <id|label>]
                      [--format pdf|png|jpeg=pdf] [--page-size auto|a4|letter=a4]
                      [--quality <0-1>] [--video frame|blank]
                      [--scheme auto|light|dark]
                      [--page-width window|a4|letter] [--open-editor]
                      [--out <file|dir>] [--wait <s>=30] [--timeout <s>=120] [--json]
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
| 4    | not paired                                                                    |
| 5    | the daemon would not start                                                    |

Everything the pairing writes lives in `~/.page-scanner` (`config.json`, `daemon.json`,
`daemon.log`), or under `$PAGE_SCANNER_HOME`.
