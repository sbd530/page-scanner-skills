---
name: research
description: Research from web pages the way a careful person does, with Page Scanner - capture each page from the user's own Chrome as a PDF plus its Markdown, cite from what the page itself declares, quote only words that are in the capture, write the notes in a file labeled as written by an agent beside the copies, and check every quote against the capture before handing the notes over. Use when the user asks to research a topic from pages, to collect sources, to write notes, a summary or a literature list from pages with citations and quotes, or to keep a record of what a page said.
license: Apache-2.0
compatibility: Google Chrome with the Page Scanner extension, Node.js 24 or newer, and either the @page-scanner/mcp server (scan_page, check_quotes) or the @page-scanner/cli command (scan, check).
metadata:
  author: sbd530
  version: '1.0'
  homepage: https://docs.pagescanner.app/mcp/skill
---

# Research with a copy of every source

The user wants notes they can trust and go back to. So every source is kept as a file, every
citation comes from what the page declares about itself, every quote is words the page has, and
what you wrote is kept apart from what the pages said. Page Scanner runs no model: it captures and
it checks, and the reading and the writing are yours. The `page-scanner` skill covers setting Page
Scanner up and connecting Chrome; read it first if `list_browsers` lists nothing.

## 1. Where the work goes

Make one folder for the research, named for the question, where the user asked or in the current
directory: `research/<topic>/`. Inside it, `sources/` holds the captures and `notes.md` your
notes. Tell the user the path at the start.

## 2. Capture each source

One `scan_page` per page, with `outputPath` set to `sources/`, `markdown: "beside"` (a PDF for the
user and a `.md` for you, with the same name), and `fileName: "{n}-{host}"` when you capture a
list with `urls`. A page the user is signed in to, or has filtered or scrolled, is captured from
their tab (`tabId` from `list_tabs`), never reopened by address. Shell:
`page-scanner scan --url <u> --out sources/ --markdown beside --json`.

Read each result before going on:

- `truncated` not null: the copy stops short of the page. Say so in the notes next to that source.
- `citation`: what the page declares about itself (authors, date, publication, DOI, canonical
  address), copied from its markup and never checked. Keep it for the citation.
- `markdownPath`: the text you read and quote from.
- `translated`: the user's browser had machine-translated the page. Its quotes are a
  translation; mark them "(machine translation)" in the notes, and never attribute them to the
  author as written.

Read the `.md`, not the PDF, and not the live page again: the `.md` is the copy the quotes will be
checked against, and a page can change between two reads.

For a long page, `textOptions: { scope: "main", links: "text" }` keeps navigation and addresses
out of what you read; it changes the `.md` only, never the PDF.

## 3. Cite from what the page declares

Build each citation from the result's `citation`, and write down where each field came from:

- A field the page declares: use it as given ("the page lists Jane Doe as the author").
- A field it does not declare: leave it out and say it is missing ("no date given"). Do not fill
  it in from the text, the address or what you know; a guessed date is worse than none.
- Always add the address the page was captured from (`page.url`) and when (`page.capturedAt`),
  and the file name in `sources/`.

## 4. Quote only what the capture has

A quote is the page's words, character for character, from its `.md`: copy it, do not retype
it from memory. Keep quotes short, a sentence or a figure, and put everything else in your own
words, marked as yours. A number, a price, a date or a name that the note presents as the page's
is a quote too and goes through the same check.

## 5. Write the notes as yours

`notes.md` starts with a line that says what it is, and keeps the two voices apart:

```markdown
> Written by an AI agent from the captures in `sources/`, on <date>. Quotes are the pages' words,
> checked against the captures; everything else is the agent's summary and may be wrong.

## <Source title>

- Captured: `sources/1-example.org.pdf` (and `.md`), from <url>, <capturedAt>
- Cited as: <authors as declared>, "<title>", <publication>, <date as declared or "no date given">. DOI <doi>.

<your summary, in your words>

> "<quote>" (section: <section from check_quotes>)
```

Never put your summary inside the capture, its `.md` or the PDF's metadata, and never call a
capture proof, evidence or verified: it is a copy of what the page showed when it was captured.

## 6. Check every quote before handing the notes over

For each source, call `check_quotes` with `path` set to its `.md` and `quotes` set to every quote
and value `notes.md` attributes to it, exactly as written in the notes. Shell:
`page-scanner check sources/<file>.md "<quote>" "<quote>" --json` (exit 1 when one is missing).

- `found: true`: keep it, and add the `section` the check names.
- `match: "loose"`: the page has it with other quote marks, dashes or case. Copy the page's form
  from `near` or the `.md` and check again.
- `match: "none"`: fix it from `near` (where the capture parts from your words) and check again,
  or take it out. Never leave a quote the check did not find presented as the page's words.

Run the check again after the last edit, and tell the user `allFound` for each source: "all 12
quotes found in the captures", or which ones you took out and why.

If the user wants the quotes marked in the copies, scan each source again with `highlight` set to
its quotes that were found (`scan_page` with the same `url` and `outputPath`, or
`page-scanner scan --url <url> --highlight "<quote>" ... --out sources/<file>.pdf`). The PDF then
has each passage highlighted and listed under a "Quoted passages" bookmark. It is a new capture:
if the page changed since the first, `highlighted` says which quotes it no longer has, and those
stay unmarked.

## 7. What to tell the user at the end

The folder, how many sources were captured, any copy that is cut short (`truncated`), citation
fields the pages did not declare, and the result of the quote check. If a page failed to capture,
name it and the reason; do not quote it from memory instead.
