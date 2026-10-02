# Auto Research & Strategy Kit

Turn a seed (niche, keyword, or product hunch) into:

1. `01-research-pack.md` + `data/keywords.json`
2. `02-strategy.md`
3. `03-ops-monetization.md`

Built for solo indie hackers using Claude Code, Codex, or Cursor. No auto-build in v1.

## Quickstart

1. Follow [docs/setup-mcp.md](docs/setup-mcp.md) (Keywords-MCP required; DataForSEO optional).
2. In your agent, invoke the **research** skill with a seed, e.g. `seed: "meal prep for shift workers"`, `locale: "en-US"`, `use_paid: false`.
3. Review `projects/<slug>/01-research-pack.md`. Kill weak ideas or confirm mode.
4. Invoke **plan-seo** or **plan-saas** (or follow classify recommendation).
5. Review `02-strategy.md` and `03-ops-monetization.md` before any separate build session.

## Layout

- `skills/` — agent skills (`research`, `plan-seo`, `plan-saas`)
- `templates/` — Markdown skeletons skills fill
- `projects/<slug>/` — one folder per opportunity (gitignored)
- `docs/setup-mcp.md` — MCP wiring
- `docs/qa/` — fixtures and checklists

## Cursor note

If your Cursor build only auto-loads `.cursor/skills/`, symlink:

```bash
mkdir -p .cursor/skills
ln -s ../../skills/research .cursor/skills/research
ln -s ../../skills/plan-seo .cursor/skills/plan-seo
ln -s ../../skills/plan-saas .cursor/skills/plan-saas
```

## Spec

See `docs/superpowers/specs/2026-10-03-auto-research-strategy-kit-design.md`.
