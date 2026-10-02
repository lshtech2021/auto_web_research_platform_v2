# MCP setup

## Goal

Give the research skill access to keyword tools. Free path is enough for v1; paid enrichment is optional.

## A. Free — Keywords-MCP (required for best results)

Repo: https://github.com/seovimalraj/keywords-mcp

1. Clone or install per upstream README.
2. Register the MCP server in your agent host (Claude Code / Codex / Cursor MCP config).
3. Confirm tools appear: suggestions, trends, questions, long-tail, and/or full research.
4. Re-run a trivial query (e.g. seed `coffee`) to verify connectivity.

If Keywords-MCP is not available, the research skill still runs at `confidence: low` using reasoning only — document missing sources in the pack.

## B. Optional paid — DataForSEO via OpenSEO or UpgradeSEO

- OpenSEO: https://github.com/every-app/open-seo (DataForSEO key required)
- UpgradeSEO: https://github.com/upgrade-ventures/upgradeseo (free sources + optional paid)

1. Obtain a DataForSEO API key only if you want volume/CPC/difficulty.
2. Configure the chosen MCP per upstream docs; store keys in host secret/env — never commit `.env`.
3. Set `use_paid: true` when invoking the research skill.
4. If paid calls fail, research continues free-only and notes skipped enrichment.

## C. Agent host notes

### Claude Code

Add MCP servers in Claude Code MCP settings; ensure project `skills/` are loadable (or install skills per Claude Code docs).

### Codex

Register MCP servers in Codex config; point the session at this repo so `skills/` and `templates/` resolve.

### Cursor

Add MCP in Cursor MCP settings. Symlink skills into `.cursor/skills/` as described in the root README if needed.

## Smoke check

After wiring, you should be able to ask the agent: "Using the research skill, research seed 'test kitchen timers' with use_paid false" and see MCP tool calls in the trace (or an explicit missing-sources note).
