# Makini agent plugins

Plugins that connect an AI agent to a company's business systems through [Makini](https://makini.io).
Each plugin pairs one Makini MCP server, `https://mcp.makini.io`, with a skill for one system.

| Plugin | System |
|---|---|
| [`makini-netsuite`](plugins/makini-netsuite) | Oracle NetSuite |
| [`makini-fishbowl-advanced`](plugins/makini-fishbowl-advanced) | Fishbowl Advanced |

Every plugin points at the same server, so an agent with several of them installed still sees one set of tools.
The first sign-in creates a Makini account if you don't have one yet.

## Install

Claude Code:

```
claude plugin marketplace add makinihq/agent-plugins
claude plugin install makini-netsuite@makini
```

## Layout

The content is vendor-neutral, and each agent vendor adds only its own manifest files next to it.

- `plugins/<plugin>/skills/<skill>/SKILL.md` follows the [Agent Skills](https://agentskills.io) standard.
- `plugins/<plugin>/.claude-plugin/plugin.json` and `plugins/<plugin>/.mcp.json` are Claude's manifests.
- `.claude-plugin/marketplace.json` is Claude's catalogue.

Rules for skills:

- Say "the agent", never a product name.
- Name only the Makini MCP tools (`list_platforms`, `search_platform_tools`, `verify_platform_authentication`,
  `execute_read_tool`, `execute_write_tool`, `request_tool_access`). No client-specific prefixes, slash commands or hooks.
- The skill folder name equals its frontmatter `name`. No dotfiles and no binaries inside `skills/`.
- Never share one MCP config file across vendors; each vendor's format differs.

## Security

See [SECURITY.md](SECURITY.md). Makini's security and compliance information: https://makini.io.
