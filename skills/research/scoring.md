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
