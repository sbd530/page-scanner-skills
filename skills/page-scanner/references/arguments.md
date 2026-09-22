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

| Argument        | Meaning                                                                                                                                |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `browserId`     | Which browser. Optional when only one is connected.                                                                                    |
| `tabId`         | A tab from `list_tabs`. Give this or `url`, not both.                                                                                  |
| `url`           | Opens a background tab there, captures it, closes it again.                                                                            |
| `windowId`      | Which window to open `url` in. Ignored with `tabId`.                                                                                   |
| `format`        | `pdf` (default), `png`, `jpeg`.                                                                                                        |
| `pageSize`      | PDF only. `a4` (default) and `letter` slice onto printable sheets with a half-inch margin; `auto` is one page the size of the capture. |
| `quality`       | JPEG only, 0.1 to 1.                                                                                                                   |
| `videoHandling` | `frame` keeps a video's paused frame, `blank` leaves its area empty.                                                                   |
| `colorScheme`   | Which of a page's two themes to capture: `auto` (default, whatever the browser shows), `light`, `dark`.                                |
| `captureWidth`  | Lay the page out at a sheet's width first, so the PDF prints at 1:1: `window` (default), `a4`, `letter`.                               |
| `openEditor`    | Also leave the capture open in a Page Scanner editor tab.                                                                              |
| `outputPath`    | A file, or a directory to keep the suggested name. Defaults to the cwd.                                                                |
| `waitSeconds`   | How long to wait for a browser to connect. 0 fails immediately.                                                                        |

Returns the absolute `path`, `width` and `height` in CSS pixels, `mode` (`vector` or `raster`),
`selectableText`, and `truncated` (`null` when whole; otherwise what the page measured, what was
captured, and a sentence naming the gap).

## Command line

```
page-scanner pair     [--port <n>] [--rotate] [--wait <s>=120] [--no-wait] [--json]
page-scanner status   [--json]
page-scanner browsers [--json]
page-scanner tabs     [--browser <id|label>] [--wait <s>=30] [--json]
page-scanner scan     (--url <u> | --tab <id>) [--window <id>] [--browser <id|label>]
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
