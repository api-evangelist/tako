---
name: tako-graph-agent
description: Use when building agentic flows over Tako's live financial, macro, and company data — find out what data Tako has for an entity or a metric, resolve a name to a graph node, compose a grounded /v3/search query, or report what Tako does and does NOT have for a question. Also use when Tako searches return vague or irrelevant cards.
api: openapi/tako-openapi-original.yml
operations: [graphSearch, graphRelated, graphNode, search]
source: https://docs.tako.com/skills/tako-graph-agent.md
method: searched
generated: '2026-07-21'
---

# Graph-grounded Tako search

Tako serves live financial / macro / company data as embeddable knowledge-card
charts through `POST https://tako.com/api/v3/search`. The graph endpoints
(`GET /beta/graph/search`, `GET /beta/graph/related`) tell you what data Tako
actually has BEFORE you search — so you compose queries that hit, pin the exact
nodes you resolved, and honestly report what's missing.

Every endpoint takes the API key in the `X-API-Key` header (create one at
https://tako.com/console/api-keys) — an anonymous call returns 401. Key
namespaces are per-environment: a key only authenticates on the host it was
issued for. Graph calls consume no credits (rate limits: 180/min, 10,000/day —
back off on 429 with jittered exponential retry).
Full parameter/response reference:
https://docs.tako.com/documentation/integrating-tako/data-graph/for-coding-agent.md

A runnable reference implementation — the resolution loop, the compose guards,
and the cohort-enumeration loop, with WHY-comments — is in
[resolve-example.ts](resolve-example.ts) alongside this skill.

## The loop

1. Break the question into a few entities and/or metrics. Decide per lookup: a
   thing → `types=entity`; a measure → `types=metric` (never mix them in one
   lookup).
2. Resolve each with `GET /beta/graph/search?q=<name>&types=…`. Labels are
   auto-inferred from `q`; pass `label=` (PERSON, ORG, GPE, LOC, PRODUCT,
   EVENT, LANGUAGE, MONEY, METRIC, STOCK_TICKER, WEBSITE — sports teams are
   ORG) only to force disambiguation. A label is a ranking BOOST, not a filter
   — off-label nodes still return.
3. Pick nodes by reading each result's `subtype`, `label`, and `description`.
4. Read relations: `GET /beta/graph/related?node_id=…` returns an overview
   `relations[]` — named semantic edges (`rel:competes_with`,
   `rel:in_industry`, …) first, then `part_of`/`members`, `metrics`/`entities`,
   `siblings`. Drill one group with `relation=<key>&q=<topic>`. Always pass `q`
   on big entities; items are ranked by popularity, blended with how strongly
   each matches `q` when a filter is set — read the top few. Drill
   items are in `relation.items`, NOT `results`. `total_capped: true` means
   render "total+". For cohorts ("Nvidia's competitors", "the Magnificent
   Seven"), drill the named `rel:*` edge or `members` — members arrive as full
   nodes; never use LLM-recalled member lists. An unknown relation key returns
   200 with empty items, so a typo reads as "no relation."
5. Compose short, data-shaped `/v3/search` queries (subject + measure + time)
   from the resolved names AND aliases — never analytical/causal phrasing
   ("how has X affected Y" retrieves nothing). For broad asks with no single
   metric, query the resolved entity and let search surface the rankings and
   overviews the graph doesn't expose as nodes. Pin the resolved ids in the
   request body under `sources.data.node_ids` (max 20) — pinned nodes get a
   strong retrieval boost (verified live: pinning an entity's id floats its
   cards up); a malformed id → 400, a stale id is silently skipped. `strict:
   true` is documented to return only cards matching a pinned node, but live
   testing has seen non-matching cards still return — rely on the pin's boost,
   not on `strict`, for hard exclusion. Enforce grounding in code
   (implementations in [resolve-example.ts](resolve-example.ts)): drop composed
   queries that don't cite a resolved name/alias verbatim; one entity per
   query; fall back to mechanical "{entity} {metric-fragment}" pairs if
   everything was dropped.
6. Fetch searches concurrently; render each card's `embed_url` as an iframe
   (cards post their height via a `tako::resize` message — don't hard-code
   heights); report gaps explicitly: "Tako has X and Y, but not Z." Never add a
   synthesis paragraph inventing numbers over the cards.

## Honesty caveats

- A node's related metrics are table-level evidence, not entity-level proof —
  `/v3/search` is the final validator.
- The graph is not the whole index: stock/share-price and market-quote data
  usually is NOT a graph metric, yet search almost always has it (same for
  rankings and overviews). A thin graph result for a well-covered entity means
  run an entity-level search, not declare a gap.

## Scope

Skip discovery when you already know the exact chart (fully-qualified
metric + entity → `POST /v3/search` directly). Decline
advice/opinion/prediction asks ("should I buy X") — serve the factual
sub-question and decline the rest.
