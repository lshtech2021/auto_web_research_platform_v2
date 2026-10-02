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
