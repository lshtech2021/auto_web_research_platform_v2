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
- `meta.mode` is not `unknown` (if it is, stop and ask the user to confirm mode or re-run research classify before planning)

## Inputs

- `slug` (required)
- Optional: `primary_term` (default: #1 ranked opportunity)

## Steps

1. Read `meta.json` and `01-research-pack.md`.
2. If `meta.mode` is `unknown`, stop and ask the user to confirm mode (or re-run research classify) before planning.
3. If `meta.mode` is `seo` and user did not override, stop and ask whether to use `plan-seo` instead.
4. Set `meta.mode` to `saas` or keep `hybrid` if user says hybrid but wants SaaS-shaped plan (record choice in strategy positioning).
5. Copy `templates/strategy-saas.md` → `projects/<slug>/02-strategy.md` and fill all sections. No code. No fake ARR/TAM figures — label estimates as assumptions.
6. Copy `templates/ops-monetization-saas.md` → `projects/<slug>/03-ops-monetization.md` and fill all sections (pricing sketch, channels, weekly ops, 30/60/90, first metric).
7. **Stop for human checkpoint.** Do not start implementation/build.

## Done when

- Both plan files exist and match `docs/qa/checklist-plan-saas.md` section list
- Strategy has one-sentence bet, problem/user, MVP in/out, build stages
- Ops has revenue model, pricing sketch, channels, weekly loop, first metric

## Out of scope

Writing application code, scaffolding an app, running deploy.
