# Auto Research & Strategy Kit — Design Spec

**Date:** 2026-10-03  
**Status:** Approved
**Repo:** `auto_web_research_platform_v2`

## Problem

Solo indie hackers need a repeatable way to go from a vague seed (niche, keyword, or product hunch) to a ranked opportunity pack and a concrete strategy + ops/monetization plan — without building a full product platform or auto-coding a site in v1.

## Goals (v1)

- Research high-value keywords **or** SaaS/product ideas (same pipeline, mode-aware).
- Produce Markdown artifacts: research pack, implementation strategy, ops & monetization plan.
- Run via Claude Code / Codex / Cursor using skills + MCP tools.
- Free data by default; optional paid enrichment (DataForSEO or equivalent).

## Non-goals (v1)

- Auto-building or deploying websites/apps.
- Web dashboard, auth, multi-user, billing.
- Guaranteed Semrush-quality metrics without paid APIs.
- Inventing volume/CPC/difficulty when data is missing.

## Users

Primary: solo indie hacker (CLI / agent workflow, minimal UI).

## Approach

**Skill-and-template kit** (thin repo). No app server. Agents execute skills; MCP provides research tools; artifacts land in `projects/<slug>/`.

Deferred: custom CLI orchestrator, local web UI, OpenHands/Aider auto-build stage.

## Architecture

```
seed / niche
    ↓
[research skill]  →  MCP (Keywords-MCP free; DataForSEO optional)
    ↓
01-research-pack.md  (+ data/keywords.json)
    ↓  human checkpoint
[classify]  →  seo | saas | hybrid
    ↓
[plan skill]  →  02-strategy.md + 03-ops-monetization.md
    ↓
human builds later with Claude Code / Codex (out of v1)
```

## Repo layout

```
skills/
  research/          # research + scoring rubric + classify
  plan-seo/          # SEO/content strategy + ops/monetization
  plan-saas/         # SaaS/product strategy + ops/monetization
templates/
  research-pack.md
  strategy-seo.md
  strategy-saas.md
  ops-monetization-seo.md
  ops-monetization-saas.md
projects/            # gitignored or sparse; one folder per opportunity
  <slug>/
    meta.json
    01-research-pack.md
    02-strategy.md
    03-ops-monetization.md
    data/
      keywords.json
docs/
  setup-mcp.md       # wire Keywords-MCP, optional DataForSEO / OpenSEO / UpgradeSEO
  qa/                # fixture seeds + golden outline checklists
docs/superpowers/specs/  # this design
```

Optional later (not v1): `scripts/` for batch scoring or `keywords.json` schema validation.

## Components

### 1. Project bootstrap

Create `projects/<slug>/` with:

```json
{
  "slug": "example",
  "seed": "user-provided seed",
  "locale": "en-US",
  "mode": "unknown",
  "use_paid": false,
  "created_at": "ISO-8601"
}
```

`mode` becomes `seo | saas | hybrid` after classify (user may override).

### 2. Research skill

**Inputs:** seed, locale/market, optional mode hint, `use_paid` flag.

**Behavior:**
1. Call free MCP tools first (suggestions, trends, questions, long-tail).
2. If `use_paid`, enrich top candidates with volume / CPC / difficulty via DataForSEO-backed MCP (UpgradeSEO / OpenSEO or direct).
3. Rank opportunities; attach evidence, risks, suggested mode.
4. Write `01-research-pack.md` and `data/keywords.json`.

**`keywords.json` shape (v1):**

```json
{
  "seed": "string",
  "locale": "en-US",
  "use_paid": false,
  "generated_at": "ISO-8601",
  "items": [
    {
      "term": "string",
      "suggested_mode": "seo|saas|hybrid",
      "signals": {
        "trend": "up|flat|down|unknown",
        "question_demand": "low|med|high|unknown",
        "volume": null,
        "cpc": null,
        "difficulty": null
      },
      "confidence": "low|med|high",
      "notes": "string"
    }
  ]
}
```

Paid fields stay `null` when free-only or enrichment skipped.

**Scoring (v1, simple):** free signals (trend direction, question demand, qualitative competition notes) plus optional paid metrics when present. Every item includes `confidence: low | med | high`. Missing metrics are `null` / `n/a` — never fabricated.

### 3. Classify

Lightweight step (same skill or one-shot prompt): set mode from the pack; update `meta.json`. Default to `hybrid` if ambiguous and ask the user to confirm before planning.

### 4. Plan skills

**Inputs:** research pack + confirmed mode.

**Outputs:**
- `02-strategy.md` — one-sentence bet, positioning, MVP in/out, staged build outline (no code).
- `03-ops-monetization.md` — revenue model, 2–3 channels, weekly ops loop, 30/60/90, first metric to watch.

SEO and SaaS templates differ in structure (content/affiliate/ads vs pricing/GTM/retention) but share the same two filenames for a consistent project folder.

### 5. Human checkpoints

1. After research — kill weak ideas before planning.  
2. After plan — approve before any separate build session (out of v1).

## External tools (leverage, don’t rebuild)

| Stage | Tools |
|-------|--------|
| Free research | Keywords-MCP; UpgradeSEO free paths |
| Paid enrichment | DataForSEO via OpenSEO / UpgradeSEO MCP (optional) |
| Execution host | Claude Code, Codex, Cursor |
| Future build (v2+) | OpenHands, Aider, OpenPage, AgentSite — out of v1 |

## Error handling

| Failure | Behavior |
|---------|----------|
| MCP/API down | Continue with seed + LLM reasoning; mark `confidence: low`; list missing sources |
| Paid enrichment fails | Fall back to free-only; note skipped enrichment |
| Ambiguous mode | Default `hybrid`; confirm with user before writing plan files |
| Missing metrics | `n/a` + confidence; never invent numbers |

## Quality bars

**Research pack is done when:**
- Ranked shortlist of 5–15 opportunities (or fewer if seed is narrow, with explanation)
- Each item: value hypothesis, evidence, risks, suggested mode
- Explicit kill criteria

**Plan docs are done when:**
- Strategy has one-sentence bet, MVP in/out, build stages
- Ops/monetization has primary revenue model, 2–3 channels, weekly ops loop, first metric

## Testing / QA (lightweight)

- Fixture seeds in `docs/qa/`: one SEO-ish, one SaaS-ish.
- Golden outline checklists (required Markdown sections).
- Manual runbook: wire MCP → research → plan → files exist and sections filled.
- No automated app test suite in v1.

## Success criteria

A user with Claude Code/Codex and free MCP wired can, for a new seed, produce a project folder with research + strategy + ops/monetization Markdown in one session, with optional paid enrichment when configured.

## Open decisions (resolved for v1)

| Decision | Choice |
|----------|--------|
| Primary user | Solo indie hacker |
| v1 deliverable | Research + strategy Markdown only |
| Opportunity types | SEO and SaaS (mode switch) |
| Data | Free default + optional paid |
| Architecture | Skills + MCP + templates |

## Next step

After this spec is approved: write an implementation plan (`writing-plans` skill) covering skills, templates, MCP setup docs, and QA fixtures — still no auto-build.
