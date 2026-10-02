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
