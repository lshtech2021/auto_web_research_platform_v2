# Auto Research & Strategy Kit Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship a thin skill-and-template kit that turns a seed into Markdown research + strategy + ops/monetization packs via Claude Code / Codex / Cursor and MCP research tools.

**Architecture:** No app server. Agents follow project skills; free MCP (Keywords-MCP) plus optional paid DataForSEO enrichment; artifacts written under `projects/<slug>/`. Human checkpoints after research and after planning. Auto-build is out of scope.

**Tech Stack:** Markdown skills (`SKILL.md`), Markdown templates, JSON (`meta.json`, `keywords.json`), MCP (Keywords-MCP; optional OpenSEO/UpgradeSEO + DataForSEO), agent hosts (Claude Code, Codex, Cursor).

**Spec:** `docs/superpowers/specs/2026-10-03-auto-research-strategy-kit-design.md`

## Global Constraints

- Solo indie hacker, CLI/agent workflow — no dashboard, auth, or billing in v1.
- Deliverables are Markdown (+ JSON data) only — never auto-build or deploy a site/app.
- Free research by default; paid enrichment only when `use_paid: true`.
- Never invent `volume` / `cpc` / `difficulty`; use `null` in JSON and `n/a` in Markdown; always set `confidence`.
- Project artifact filenames are fixed: `meta.json`, `01-research-pack.md`, `02-strategy.md`, `03-ops-monetization.md`, `data/keywords.json`.
- Modes: `unknown | seo | saas | hybrid`. Ambiguous classify → `hybrid` + ask user before planning.
- Skills live under repo-root `skills/` (spec). Cursor users: copy or symlink into `.cursor/skills/` if the host only loads that path — document in README, do not fork skill content.

---

## File map (create)

| Path | Responsibility |
|------|----------------|
| `README.md` | What this is, quickstart, how to invoke skills on each host |
| `.gitignore` | Ignore `projects/*` contents except `.gitkeep` |
| `projects/.gitkeep` | Keep empty projects tree in git |
| `templates/research-pack.md` | Skeleton for `01-research-pack.md` |
| `templates/strategy-seo.md` | Skeleton copied to `02-strategy.md` in SEO mode |
| `templates/strategy-saas.md` | Skeleton for SaaS/hybrid product-leaning plans |
| `templates/ops-monetization-seo.md` | Skeleton for `03-ops-monetization.md` (SEO) |
| `templates/ops-monetization-saas.md` | Skeleton for SaaS ops/monetization |
| `skills/research/SKILL.md` | Bootstrap project, MCP research, score, classify, write pack + JSON |
| `skills/research/scoring.md` | Scoring rubric reference |
| `skills/plan-seo/SKILL.md` | Fill SEO strategy + ops from research pack |
| `skills/plan-saas/SKILL.md` | Fill SaaS strategy + ops from research pack |
| `docs/setup-mcp.md` | Wire Keywords-MCP + optional paid MCP |
| `docs/qa/fixtures.md` | Two fixture seeds |
| `docs/qa/checklist-research.md` | Golden outline for research pack |
| `docs/qa/checklist-plan-seo.md` | Golden outline for SEO plan pair |
| `docs/qa/checklist-plan-saas.md` | Golden outline for SaaS plan pair |
| `docs/qa/runbook.md` | Manual E2E verification steps |

---

### Task 1: Repo scaffolding

**Files:**
- Create: `README.md`
- Create: `.gitignore`
- Create: `projects/.gitkeep`

**Interfaces:**
- Consumes: nothing
- Produces: runnable repo skeleton; `projects/` convention documented

- [ ] **Step 1: Write `.gitignore`**

```gitignore
# Opportunity artifacts (regenerable / private)
projects/*
!projects/.gitkeep

# Local env / secrets
.env
.env.*
*.pem

# OS / editor
.DS_Store
.idea/
.vscode/
```

- [ ] **Step 2: Write `projects/.gitkeep`**

Empty file (no contents).

- [ ] **Step 3: Write `README.md`**

Use this content verbatim as the starting README:

```markdown
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
```

- [ ] **Step 4: Verify scaffolding**

Run:

```bash
test -f README.md && test -f .gitignore && test -f projects/.gitkeep && echo OK
```

Expected: `OK`

- [ ] **Step 5: Commit**

```bash
git add README.md .gitignore projects/.gitkeep
git commit -m "$(cat <<'EOF'
chore: scaffold research strategy kit repo

EOF
)"
```

---

### Task 2: Markdown templates

**Files:**
- Create: `templates/research-pack.md`
- Create: `templates/strategy-seo.md`
- Create: `templates/strategy-saas.md`
- Create: `templates/ops-monetization-seo.md`
- Create: `templates/ops-monetization-saas.md`

**Interfaces:**
- Consumes: section names from spec quality bars
- Produces: skeletons that skills copy into `projects/<slug>/` and fill (never leave `{{placeholders}}` in final artifacts — replace with content or `n/a`)

- [ ] **Step 1: Write `templates/research-pack.md`**

```markdown
# Research Pack — {{slug}}

- Seed: {{seed}}
- Locale: {{locale}}
- use_paid: {{use_paid}}
- Generated: {{generated_at}}
- Data sources used: {{sources}}
- Data sources missing/failed: {{sources_missing}}
- Overall confidence: {{overall_confidence}}

## Executive summary

{{executive_summary}}

## Ranked opportunities

### 1. {{term}}

- Suggested mode: {{suggested_mode}}
- Confidence: {{confidence}}
- Value hypothesis: {{value_hypothesis}}
- Evidence: {{evidence}}
- Signals: trend={{trend}}; question_demand={{question_demand}}; volume={{volume}}; cpc={{cpc}}; difficulty={{difficulty}}
- Risks: {{risks}}
- Why ranked here: {{rank_reason}}

<!-- Repeat ### N. for each item (target 5–15). -->

## Kill criteria

{{kill_criteria}}

## Recommended mode

- Mode: {{mode}}
- Rationale: {{mode_rationale}}
- Needs user confirm before plan: {{needs_confirm}}

## Next step

Human checkpoint: approve shortlist + mode, then run `plan-seo` or `plan-saas`.
```

- [ ] **Step 2: Write `templates/strategy-seo.md`**

```markdown
# Strategy — {{slug}} (SEO)

- Seed: {{seed}}
- Primary opportunity: {{primary_term}}
- Locale: {{locale}}

## One-sentence bet

{{bet}}

## Positioning

{{positioning}}

## Audience

{{audience}}

## MVP scope

### In

{{mvp_in}}

### Out

{{mvp_out}}

## Content / site outline

{{content_outline}}

## Build stages (no code)

1. {{stage_1}}
2. {{stage_2}}
3. {{stage_3}}

## Risks & assumptions

{{risks_assumptions}}

## Success metric (first)

{{first_metric}}
```

- [ ] **Step 3: Write `templates/strategy-saas.md`**

```markdown
# Strategy — {{slug}} (SaaS)

- Seed: {{seed}}
- Primary opportunity: {{primary_term}}
- Locale: {{locale}}

## One-sentence bet

{{bet}}

## Problem

{{problem}}

## Target user

{{target_user}}

## Solution sketch

{{solution}}

## Positioning

{{positioning}}

## MVP scope

### In

{{mvp_in}}

### Out

{{mvp_out}}

## Build stages (no code)

1. {{stage_1}}
2. {{stage_2}}
3. {{stage_3}}

## Risks & assumptions

{{risks_assumptions}}

## Success metric (first)

{{first_metric}}
```

- [ ] **Step 4: Write `templates/ops-monetization-seo.md`**

```markdown
# Ops & Monetization — {{slug}} (SEO)

## Primary revenue model

{{revenue_model}}

## Channels (2–3)

1. {{channel_1}}
2. {{channel_2}}
3. {{channel_3}}

## Weekly ops loop

{{weekly_ops}}

## 30 / 60 / 90

- 30 days: {{d30}}
- 60 days: {{d60}}
- 90 days: {{d90}}

## First metric to watch

{{first_metric}}

## Cost notes

{{cost_notes}}
```

- [ ] **Step 5: Write `templates/ops-monetization-saas.md`**

```markdown
# Ops & Monetization — {{slug}} (SaaS)

## Primary revenue model

{{revenue_model}}

## Pricing sketch

{{pricing}}

## Channels (2–3)

1. {{channel_1}}
2. {{channel_2}}
3. {{channel_3}}

## Weekly ops loop

{{weekly_ops}}

## 30 / 60 / 90

- 30 days: {{d30}}
- 60 days: {{d60}}
- 90 days: {{d90}}

## First metric to watch

{{first_metric}}

## Cost notes

{{cost_notes}}
```

- [ ] **Step 6: Verify templates exist**

Run:

```bash
ls templates/research-pack.md templates/strategy-seo.md templates/strategy-saas.md \
  templates/ops-monetization-seo.md templates/ops-monetization-saas.md
```

Expected: all five paths listed.

- [ ] **Step 7: Commit**

```bash
git add templates/
git commit -m "$(cat <<'EOF'
feat: add research and plan markdown templates

EOF
)"
```

---

### Task 3: Research skill + scoring rubric

**Files:**
- Create: `skills/research/SKILL.md`
- Create: `skills/research/scoring.md`

**Interfaces:**
- Consumes: templates in `templates/research-pack.md`; MCP keyword tools; user inputs `seed`, `locale`, `use_paid`, optional `mode_hint`, optional `slug`
- Produces: `projects/<slug>/meta.json`, `01-research-pack.md`, `data/keywords.json` matching spec schema; updates `meta.mode` after classify

- [ ] **Step 1: Write `skills/research/scoring.md`**

```markdown
# Scoring rubric (v1)

## Rules

1. Never invent volume, CPC, or difficulty.
2. Free-only runs may rank using trend + question_demand + qualitative notes only.
3. Every item must have `confidence: low | med | high`.
4. Target 5–15 ranked items; if fewer, explain why in the executive summary.

## Free signals

| Signal | How to set | Notes |
|--------|------------|-------|
| trend | up / flat / down / unknown | From Trends MCP or mark unknown |
| question_demand | low / med / high / unknown | From question/long-tail density |
| competition_notes | free text in `notes` | SERP impressions if available; else "unverified" |

## Paid signals (only if use_paid and MCP succeeds)

| Signal | JSON field | On failure |
|--------|------------|------------|
| volume | `signals.volume` | `null` |
| cpc | `signals.cpc` | `null` |
| difficulty | `signals.difficulty` | `null` |

## Confidence heuristic

- **high:** paid metrics present for the item OR multiple free signals agree and sources listed
- **med:** free signals only, sources partially available
- **low:** MCP failed or mostly LLM reasoning; must list missing sources

## Rank order

Prefer: clearer demand signals, higher confidence, sharper mode fit, lower obvious competition (when noted). Demote: vague terms, branded-only, regulated/YMYL without expertise path.
```

- [ ] **Step 2: Write `skills/research/SKILL.md`**

```markdown
---
name: research
description: >
  Bootstrap an opportunity project and produce 01-research-pack.md plus
  data/keywords.json from a seed. Use when the user wants keyword/idea
  research for SEO or SaaS opportunities. Free MCP first; optional paid enrichment.
---

# Research skill

## When to use

User provides a seed niche, keyword, or product hunch and wants a ranked research pack before planning or building.

## Inputs (ask if missing)

- `seed` (required)
- `locale` (default `en-US`)
- `use_paid` (default `false`)
- `mode_hint` (optional: `seo` | `saas` | `hybrid`)
- `slug` (optional; default slugify seed, lowercase kebab-case)

## Steps

1. **Bootstrap** `projects/<slug>/` and `data/`.
2. Write `meta.json`:

```json
{
  "slug": "<slug>",
  "seed": "<seed>",
  "locale": "<locale>",
  "mode": "unknown",
  "use_paid": false,
  "created_at": "<ISO-8601>"
}
```

3. **Free MCP first.** Prefer Keywords-MCP tools when available:
   - suggestions / autocomplete
   - trends
   - question keywords
   - long-tail
   - full research report if one tool covers the above
4. If `use_paid` is true, enrich **top candidates only** via DataForSEO-backed MCP (OpenSEO / UpgradeSEO). On failure: continue free-only; record skipped enrichment in the pack.
5. If MCP is unavailable entirely: continue with seed + reasoning; set overall confidence `low`; list missing sources.
6. Score and rank using `skills/research/scoring.md`.
7. **Classify** suggested mode (`seo` | `saas` | `hybrid`). If ambiguous, set `hybrid` and `needs_confirm: true`. Update `meta.json` `mode` field to the suggestion (user may override later).
8. Copy `templates/research-pack.md` → `projects/<slug>/01-research-pack.md` and fill every section. Replace placeholders; use `n/a` for missing metrics.
9. Write `projects/<slug>/data/keywords.json` with this shape (paid fields `null` when absent):

```json
{
  "seed": "string",
  "locale": "en-US",
  "use_paid": false,
  "generated_at": "ISO-8601",
  "items": [
    {
      "term": "string",
      "suggested_mode": "seo",
      "signals": {
        "trend": "unknown",
        "question_demand": "unknown",
        "volume": null,
        "cpc": null,
        "difficulty": null
      },
      "confidence": "low",
      "notes": "string"
    }
  ]
}
```

10. **Stop for human checkpoint.** Do not run plan skills until the user approves the shortlist and mode.

## Done when

- `meta.json`, `01-research-pack.md`, `data/keywords.json` exist
- Shortlist has evidence, risks, kill criteria, recommended mode
- No fabricated volume/CPC/difficulty

## Out of scope

Writing `02-strategy.md` / `03-ops-monetization.md`, generating code, deploying.
```

- [ ] **Step 3: Verify skill files**

Run:

```bash
test -f skills/research/SKILL.md && test -f skills/research/scoring.md && grep -q "Never invent" skills/research/scoring.md && echo OK
```

Expected: `OK`

- [ ] **Step 4: Commit**

```bash
git add skills/research/
git commit -m "$(cat <<'EOF'
feat: add research skill and scoring rubric

EOF
)"
```

---

### Task 4: Plan-SEO skill

**Files:**
- Create: `skills/plan-seo/SKILL.md`

**Interfaces:**
- Consumes: `projects/<slug>/01-research-pack.md`, `meta.json` with mode `seo` or user override; templates `strategy-seo.md`, `ops-monetization-seo.md`
- Produces: `02-strategy.md`, `03-ops-monetization.md` (SEO-shaped content, fixed filenames)

- [ ] **Step 1: Write `skills/plan-seo/SKILL.md`**

```markdown
---
name: plan-seo
description: >
  From an approved research pack, write SEO/content strategy and
  ops-monetization Markdown into a project folder. Use after research
  when mode is seo (or user chooses SEO planning).
---

# Plan SEO skill

## Preconditions

- `projects/<slug>/01-research-pack.md` exists
- User approved shortlist (human checkpoint)
- Mode is `seo`, or user explicitly asked for SEO planning on a hybrid pack

## Inputs

- `slug` (required)
- Optional: `primary_term` (default: #1 ranked opportunity)

## Steps

1. Read `meta.json` and `01-research-pack.md`.
2. If `meta.mode` is `saas` and user did not override, stop and ask whether to use `plan-saas` instead.
3. Set `meta.mode` to `seo` if user confirmed SEO plan.
4. Copy `templates/strategy-seo.md` → `projects/<slug>/02-strategy.md` and fill all sections from the research pack. No code. No invented traffic numbers — use `n/a` and qualitative language when metrics missing.
5. Copy `templates/ops-monetization-seo.md` → `projects/<slug>/03-ops-monetization.md` and fill all sections. Include primary revenue model (e.g. affiliate, ads, lead-gen, digital product), 2–3 channels, weekly ops loop, 30/60/90, first metric.
6. **Stop for human checkpoint.** Do not start implementation/build.

## Done when

- Both plan files exist and match `docs/qa/checklist-plan-seo.md` section list
- Strategy has one-sentence bet, MVP in/out, build stages
- Ops has revenue model, channels, weekly loop, first metric

## Out of scope

Writing application code, scaffolding a site, running deploy.
```

- [ ] **Step 2: Verify**

Run:

```bash
test -f skills/plan-seo/SKILL.md && grep -q "02-strategy.md" skills/plan-seo/SKILL.md && echo OK
```

Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add skills/plan-seo/
git commit -m "$(cat <<'EOF'
feat: add plan-seo skill

EOF
)"
```

---

### Task 5: Plan-SaaS skill

**Files:**
- Create: `skills/plan-saas/SKILL.md`

**Interfaces:**
- Consumes: `projects/<slug>/01-research-pack.md`, `meta.json`; templates `strategy-saas.md`, `ops-monetization-saas.md`
- Produces: `02-strategy.md`, `03-ops-monetization.md` (SaaS-shaped content, fixed filenames)

- [ ] **Step 1: Write `skills/plan-saas/SKILL.md`**

```markdown
---
name: plan-saas
description: >
  From an approved research pack, write SaaS/product strategy and
  ops-monetization Markdown into a project folder. Use after research
  when mode is saas or hybrid leaning product.
---

# Plan SaaS skill

## Preconditions

- `projects/<slug>/01-research-pack.md` exists
- User approved shortlist (human checkpoint)
- Mode is `saas` or `hybrid`, or user explicitly asked for SaaS planning

## Inputs

- `slug` (required)
- Optional: `primary_term` (default: #1 ranked opportunity)

## Steps

1. Read `meta.json` and `01-research-pack.md`.
2. If `meta.mode` is `seo` and user did not override, stop and ask whether to use `plan-seo` instead.
3. Set `meta.mode` to `saas` or keep `hybrid` if user says hybrid but wants SaaS-shaped plan (record choice in strategy positioning).
4. Copy `templates/strategy-saas.md` → `projects/<slug>/02-strategy.md` and fill all sections. No code. No fake ARR/TAM figures — label estimates as assumptions.
5. Copy `templates/ops-monetization-saas.md` → `projects/<slug>/03-ops-monetization.md` and fill all sections (pricing sketch, channels, weekly ops, 30/60/90, first metric).
6. **Stop for human checkpoint.** Do not start implementation/build.

## Done when

- Both plan files exist and match `docs/qa/checklist-plan-saas.md` section list
- Strategy has one-sentence bet, problem/user, MVP in/out, build stages
- Ops has revenue model, pricing sketch, channels, weekly loop, first metric

## Out of scope

Writing application code, scaffolding an app, running deploy.
```

- [ ] **Step 2: Verify**

Run:

```bash
test -f skills/plan-saas/SKILL.md && grep -q "plan-seo" skills/plan-saas/SKILL.md && echo OK
```

Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add skills/plan-saas/
git commit -m "$(cat <<'EOF'
feat: add plan-saas skill

EOF
)"
```

---

### Task 6: MCP setup documentation

**Files:**
- Create: `docs/setup-mcp.md`

**Interfaces:**
- Consumes: external projects Keywords-MCP, OpenSEO, UpgradeSEO
- Produces: host-agnostic setup instructions so research skill can call tools

- [ ] **Step 1: Write `docs/setup-mcp.md`**

```markdown
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
```

- [ ] **Step 2: Verify**

Run:

```bash
test -f docs/setup-mcp.md && grep -q "Keywords-MCP" docs/setup-mcp.md && echo OK
```

Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add docs/setup-mcp.md
git commit -m "$(cat <<'EOF'
docs: add MCP setup guide for research skill

EOF
)"
```

---

### Task 7: QA fixtures, checklists, runbook

**Files:**
- Create: `docs/qa/fixtures.md`
- Create: `docs/qa/checklist-research.md`
- Create: `docs/qa/checklist-plan-seo.md`
- Create: `docs/qa/checklist-plan-saas.md`
- Create: `docs/qa/runbook.md`

**Interfaces:**
- Consumes: quality bars from spec; template section headings
- Produces: manual verification materials for Task 8

- [ ] **Step 1: Write `docs/qa/fixtures.md`**

```markdown
# Fixture seeds

## Fixture A — SEO-ish

- seed: `best portable espresso for camping`
- locale: `en-US`
- use_paid: `false`
- expected mode lean: `seo`
- slug suggestion: `portable-espresso-camping`

## Fixture B — SaaS-ish

- seed: `scheduling tool for independent physios`
- locale: `en-US`
- use_paid: `false`
- expected mode lean: `saas` or `hybrid`
- slug suggestion: `physio-scheduling`
```

- [ ] **Step 2: Write `docs/qa/checklist-research.md`**

```markdown
# Checklist — research pack

Pass only if all are true for `projects/<slug>/`:

- [ ] `meta.json` exists with slug, seed, locale, mode, use_paid, created_at
- [ ] `01-research-pack.md` exists
- [ ] `data/keywords.json` exists and parses as JSON
- [ ] Pack has: Executive summary, Ranked opportunities, Kill criteria, Recommended mode, Next step
- [ ] Each ranked item has: suggested mode, confidence, value hypothesis, evidence, signals, risks
- [ ] No invented numeric volume/CPC/difficulty (null/n/a when unknown)
- [ ] Missing/failed data sources listed when MCP incomplete
- [ ] Shortlist size 5–15 OR fewer with explanation in executive summary
```

- [ ] **Step 3: Write `docs/qa/checklist-plan-seo.md`**

```markdown
# Checklist — SEO plan pair

- [ ] `02-strategy.md` exists with: One-sentence bet, Positioning, Audience, MVP In/Out, Content/site outline, Build stages, Risks & assumptions, Success metric
- [ ] `03-ops-monetization.md` exists with: Primary revenue model, Channels (2–3), Weekly ops loop, 30/60/90, First metric, Cost notes
- [ ] No application source code generated as part of this skill run
```

- [ ] **Step 4: Write `docs/qa/checklist-plan-saas.md`**

```markdown
# Checklist — SaaS plan pair

- [ ] `02-strategy.md` exists with: One-sentence bet, Problem, Target user, Solution sketch, Positioning, MVP In/Out, Build stages, Risks & assumptions, Success metric
- [ ] `03-ops-monetization.md` exists with: Primary revenue model, Pricing sketch, Channels (2–3), Weekly ops loop, 30/60/90, First metric, Cost notes
- [ ] No application source code generated as part of this skill run
```

- [ ] **Step 5: Write `docs/qa/runbook.md`**

```markdown
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

Both fixtures pass their checklists. Paid path optional: one extra run with `use_paid: true` if keys configured; verify paid fields non-null for at least one item **or** explicit skipped-enrichment note.
```

- [ ] **Step 6: Verify QA docs**

Run:

```bash
ls docs/qa/fixtures.md docs/qa/checklist-research.md docs/qa/checklist-plan-seo.md \
  docs/qa/checklist-plan-saas.md docs/qa/runbook.md
```

Expected: all five paths listed.

- [ ] **Step 7: Commit**

```bash
git add docs/qa/
git commit -m "$(cat <<'EOF'
docs: add QA fixtures, checklists, and runbook

EOF
)"
```

---

### Task 8: Spec status + dry-run verification

**Files:**
- Modify: `docs/superpowers/specs/2026-10-03-auto-research-strategy-kit-design.md` (status line only)
- Verify: full tree against file map

**Interfaces:**
- Consumes: Tasks 1–7 outputs; `docs/qa/runbook.md`
- Produces: confirmed v1 kit ready for agent use

- [ ] **Step 1: Update spec status**

Change the status line from `Draft for review` to `Approved — implementing` (or `Approved` if implementation of Tasks 1–7 is already merged in the same session).

- [ ] **Step 2: Tree check**

Run:

```bash
find skills templates docs/qa docs/setup-mcp.md README.md -type f | sort
```

Expected: includes research/plan skills, five templates, five QA files, setup-mcp, README.

- [ ] **Step 3: Agent dry-run**

Follow `docs/qa/runbook.md` Fixture A (minimum). If MCP is not wired in this environment, simulate by having the agent run the research skill with explicit "MCP unavailable" path and confirm checklist still passes with `confidence: low` and missing sources listed.

- [ ] **Step 4: Commit status + any dry-run doc fixes**

```bash
git add docs/superpowers/specs/2026-10-03-auto-research-strategy-kit-design.md
git commit -m "$(cat <<'EOF'
docs: mark research strategy kit spec approved

EOF
)"
```

---

## Self-review (plan vs spec)

| Spec requirement | Task |
|------------------|------|
| skills/research + scoring + classify | Task 3 |
| plan-seo / plan-saas | Tasks 4–5 |
| templates | Task 2 |
| projects/<slug> artifacts + meta.json + keywords.json schema | Tasks 1, 3 |
| Free + optional paid MCP | Tasks 3, 6 |
| Human checkpoints | Tasks 3–5 skill steps |
| Error/degradation rules | Task 3 skill + scoring |
| QA fixtures + checklists + runbook | Tasks 7–8 |
| No auto-build | Global constraints + skill out-of-scope |
| README / host notes | Tasks 1, 6 |

No TBD placeholders. Filenames consistent across tasks (`01-research-pack.md`, `02-strategy.md`, `03-ops-monetization.md`, `keywords.json` fields).
