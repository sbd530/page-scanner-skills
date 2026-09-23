---
name: page-scanner
description: Capture a web page from the user's own signed-in Chrome as a PDF whose text stays selectable and searchable, as a PNG or JPEG, or as Markdown text, through the Page Scanner extension's MCP tools (list_browsers, list_tabs, scan_page, diff_captures) or the page-scanner command. Use when the user asks to save, scan, capture, archive or print a web page or an open tab to PDF, wants a full-page screenshot, needs a document from a page behind a login, wants a list of pages captured, once or on a schedule, or wants to know what changed on a page since an earlier capture.
license: Apache-2.0
compatibility: Google Chrome with the Page Scanner extension, Node.js 24 or newer, and either the @page-scanner/mcp server registered with the agent or the @page-scanner/cli command (npx works for both). Local machine only; the connection is to 127.0.0.1.
metadata:
  author: sbd530
  version: '1.0'
  homepage: https://docs.pagescanner.app/mcp
---

# Page Scanner

Page Scanner captures a whole web page, top to bottom, in the Chrome the user is already signed in
to, and writes a file: a PDF whose text is real text, or a PNG or JPEG. The capture goes through
Chrome's own print pipeline, so a page behind a login is captured as the user sees it, and no
second browser is started. You reach it through five MCP tools, or the `page-scanner` command when
you only have a shell. Both use the same pairing and the same background daemon.

## 1. Check the connection first

Call `list_browsers` (shell: `npx @page-scanner/cli browsers --json`). It lists the Chrome
profiles that are paired and connected right now, each with the name the user gave it.

- **One browser listed:** go ahead; you never need `browserId`.
- **Several listed:** every later call takes `browserId`. Ask which profile if the task does not
  say; a work profile and a personal profile are logged into different things.
- **None listed:** the user has to pair, which is a deliberate act on their side. Tell them to run
  `npx @page-scanner/cli pair` in a terminal, then open the extension's settings page (the gear in
  the editor toolbar, or `chrome://extensions`, Details, Extension options), and under **Local
  agents** name the browser, paste the port and token, and press **Connect**. Prefer that over the
  `pair` tool: the tool returns the token to you, which puts a secret in the transcript. Use the
  tool only when the user asks you to.
- **Empty right after a quiet spell:** Chrome retires the extension's service worker after about
  thirty seconds of silence and it dials back in when something wakes it. `list_tabs` and
  `scan_page` wait up to `waitSeconds` (default 30) for that; do not report "not connected" from
  one empty `list_browsers` if the user just paired.

Never run `pair` with `rotate` on your own. It invalidates the token in every browser and the user
has to paste a new one everywhere.

## 2. Pick the page

**A tab the user has open** (the common case, and the only way to get a page in the state they are
looking at: scrolled, filtered, logged in): call `list_tabs`, match the title or URL, and pass its
`tabId` to `scan_page`. A tab you were given is never closed.

**An address:** pass `url` to `scan_page`. It opens a background tab, captures it, and closes it
again, including when the capture fails. It still runs in the user's Chrome, so a URL behind a login
works if they are logged in there. Use `windowId` from `list_tabs` if it matters which window it
opens in.

**Several addresses:** pass `urls` (up to 50) and an `outputPath` directory. They are captured one
after another, and `fileName` names each file from a template, such as `"{n}-{host}"`. For a list
the user wants captured on a schedule, give them the `page-scanner scan --urls <file>` command and
point them at the documentation's scheduling section rather than running it yourself each time.

Give exactly one of `tabId`, `url` or `urls`.

## 3. Scan

`scan_page` with the target and, as the task needs them:

| Want                                      | Set                                                                            |
| ----------------------------------------- | ------------------------------------------------------------------------------ |
| A PDF to read, search or copy from        | nothing; `format` defaults to `pdf`                                            |
| A PDF that prints at full size            | `captureWidth: "a4"` (or `"letter"`) with the same `pageSize`                  |
| One long page instead of sheets           | `pageSize: "auto"`                                                             |
| A picture                                 | `format: "png"`, or `"jpeg"` with `quality` 0.1 to 1                           |
| The dark theme of a page that has two     | `colorScheme: "dark"` (or `"light"`)                                           |
| A video's area empty rather than a frame  | `videoHandling: "blank"`                                                       |
| The user to crop or mark it up afterwards | `openEditor: true`; you cannot crop, the editor can                            |
| To read the page's text yourself          | `markdown: "inline"`, far more reliable than reading the PDF                   |
| The text as a `.md` file for the user     | `markdown: "beside"`, or `"only"` for no PDF                                   |
| The file somewhere specific               | `outputPath`: a file path, or a directory to keep the browser's suggested name |

`outputPath` defaults to the current working directory. When the user named a place, pass an
absolute path. `captureWidth` matters for printing: a page captured at a 1280 px window is scaled
to 55 % on A4, which puts 16 px body text at 6.5 pt; laid out at A4 width first it lands at 12 pt.

## 4. Read the result before you report

For `urls`, read each entry of `results`: report the pages that failed by address and reason, not
just the count, since a page that failed may need the user to log in or the address fixed.

The result carries the absolute `path`, `width` and `height` in CSS pixels, `mode`,
`selectableText` and `truncated`.

- `selectableText: true` is the only basis for telling the user the text is real. A PNG or JPEG is
  false by nature.
- `truncated` is `null` when the page was captured whole. Otherwise it says what the page measured,
  what was captured and what is missing: a page over 60,000 CSS pixels on a side is captured up to
  there and no further, because an infinite-scroll page has no bottom (the extension's settings
  page can raise the height's limit to 120,000 or 240,000). Say so; do not present the file as
  complete.
- Quote the path back. The file is on disk; nothing is returned inline.

## 5. What changed since last time

To tell the user what changed on a page since an earlier capture, capture it again the same way,
with `markdown: "beside"` (or `"only"`), and call `diff_captures` with the older and the newer
`.md`. Report `changed`, and when it is true, the passages from `unified` in plain words, not the
diff syntax. Two PNGs work too, and give a picture with the changed regions outlined, but content
that moved shows as changed below the move, so prefer the text. When the user wants this on a
schedule, give them `page-scanner scan --urls` with `--markdown beside` and `page-scanner diff`,
which exits 1 when something changed.

## What fails, and what to say

- **No browser, or not paired** (CLI exit 3 or 4): section 1.
- **DevTools is open on that tab:** only one debugger can attach, so the capture fails. Ask the
  user to close DevTools on it, or capture by `url` instead.
- **`chrome://` pages, the Chrome Web Store and other extensions' pages** cannot be captured by
  any extension. Say so rather than retrying.
- **There is no fallback.** A click in the extension can fall back to stitching screenshots; an
  agent's scan cannot, so a page the vector capture cannot handle fails loudly. Report the error.
- **The "Page Scanner started debugging this browser" bar** that appears in Chrome during a scan is
  Chrome's own notice and cannot be hidden. It is expected; mention it if the user asks.
- **A background tab gets 30 seconds to load.** A page that never settles fails on time; a `tabId`
  of a tab that has finished loading avoids that.
- **A JPEG or PNG of a very tall page** is one bitmap; for a long page the PDF is the smaller,
  searchable file.

Each scan is one debugger session on the user's browser. Scan what the task needs, not every tab.

## The command line

The same operations, for an agent with a shell and no MCP. `--json` puts one JSON document on
stdout for success and failure alike; without it, `scan` prints the path and nothing else.

```bash
npx @page-scanner/cli browsers --json
npx @page-scanner/cli tabs --json
npx @page-scanner/cli scan --tab <id> --out ~/Desktop/ --json
npx @page-scanner/cli scan --url https://example.com/doc --page-width a4 --out ./doc.pdf --json
```

Exit codes: 0 success, 1 the browser was reached and the work failed, 2 wrong arguments, 3 no
usable browser, 4 not paired, 5 the daemon would not start. The full argument tables for both are
in [references/arguments.md](references/arguments.md); the product documentation is at
https://docs.pagescanner.app/mcp.
