# Page Scanner for AI agents

The agent skill and the Claude Code plugin for [Page Scanner](https://pagescanner.app), the
Chrome extension that captures a whole web page from your own signed-in Chrome as a PDF whose text
stays selectable and searchable, or as a PNG or JPEG.

The skill is a `SKILL.md` in the [Agent Skills](https://agentskills.io) format. It tells an agent
when to reach for Page Scanner and how: check the pairing first, capture the tab you are looking at
rather than reopen a page you are logged into, which options print well, what `truncated` means,
and what Chrome's debugger bar is. The same file works in every agent that reads the standard:
Claude Code, Codex, Cursor, GitHub Copilot, Gemini CLI and the rest.

## Install

**Claude Code, as a plugin.** The plugin carries the skill and registers the MCP server, so this
is the whole setup on the agent's side:

```bash
claude plugin marketplace add sbd530/page-scanner-skills
claude plugin install page-scanner@page-scanner-skills
```

**Any agent, the skill on its own:**

```bash
npx skills add sbd530/page-scanner-skills
```

That installs it for the agents it finds in the current project (`-g` for your user directory).
An agent that speaks MCP still needs the server registered, which is one line per client on the
[install page](https://docs.pagescanner.app/mcp/install); an agent with only a shell uses the
`page-scanner` command, and the skill covers both.

Then [pair Chrome](https://docs.pagescanner.app/mcp/pairing), once per profile: the extension
ships with local agents off, and turning them on means pasting a token into its settings page.

## What is here

| Path                              | What                                                                  |
| --------------------------------- | --------------------------------------------------------------------- |
| `skills/page-scanner/SKILL.md`    | The skill: the workflow, the options, what fails and what to say      |
| `skills/page-scanner/references/` | The full argument tables for the MCP tools and the command line       |
| `.claude-plugin/plugin.json`      | The Claude Code plugin manifest: this tree is the plugin              |
| `.mcp.json`                       | The MCP server the plugin registers, `npx -y @page-scanner/mcp serve` |
| `.claude-plugin/marketplace.json` | The marketplace that lists the plugin, so `marketplace add` works     |

## Where it comes from

This repository is published from the Page Scanner product repository, which is private; every
commit here is one sync from it. Issues are welcome here, and a fix lands through the product
repository and comes back in the next sync. The documentation is at
[docs.pagescanner.app](https://docs.pagescanner.app/mcp); the server and the command are the
`@page-scanner/mcp` and `@page-scanner/cli` packages on npm.

Apache-2.0, see [LICENSE](LICENSE).
