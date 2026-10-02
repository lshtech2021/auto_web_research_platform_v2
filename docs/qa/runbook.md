# Manual E2E runbook

## Prep

1. Complete `docs/setup-mcp.md` (free path minimum).
2. Open this repo in Claude Code, Codex, or Cursor.

## Run Fixture A

1. Invoke research skill with Fixture A from `fixtures.md`.
2. Complete `checklist-research.md`.
3. Confirm mode `seo` (or override).
4. Invoke plan-seo for that slug.
5. Complete `checklist-plan-seo.md`.

## Run Fixture B

1. Invoke research skill with Fixture B.
2. Complete `checklist-research.md`.
3. Confirm mode `saas` or `hybrid`.
4. Invoke plan-saas for that slug.
5. Complete `checklist-plan-saas.md`.

## Pass criteria

**v1 merge bar:** Fixture A must pass `checklist-research.md` and `checklist-plan-seo.md` (minimum to merge). Fixture B is recommended before relying on the SaaS planning path in production workflows. **Full runbook pass:** both Fixture A and Fixture B pass their checklists end-to-end.

Paid path optional: one extra run with `use_paid: true` if keys configured; verify paid fields non-null for at least one item **or** explicit skipped-enrichment note.
