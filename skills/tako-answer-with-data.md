---
name: tako-answer-with-data
description: Use to get a single written, cited answer to a natural-language question about live financial, macroeconomic, or company data — grounded in Tako knowledge cards and web results — and, when needed, download the exact numbers behind a card as CSV.
api: openapi/tako-openapi-original.yml
operations: [answer, search, contents]
method: generated
generated: '2026-07-21'
source: openapi/tako-openapi-original.yml
---

# Answer a data question with Tako

Get one grounded, citation-backed answer to a question about companies, markets,
and the economy, then optionally export the underlying numbers.

## Auth

Every call takes the API key in the `X-API-Key` header. Create a key at
https://tako.com/console/api-keys. A missing or invalid key returns `401`
(`error_type: AUTHENTICATION_ERROR`).

## Steps

1. **Ask for a synthesized answer** — `answer` (`POST /api/v1/answer`) with a
   `SearchRequest` body carrying your natural-language `text`. The response
   (`AnswerResponse`) has:
   - `answer` — synthesized prose,
   - `cards[]` — the `TakoCard`s backing it (`cards[0]` is the lead card to show),
   - `web_results[]`, and
   - `request_id` (echo it in support/tracing).
   Prefer `answer` when you want prose + confidence; use `search`
   (`POST /api/v3/search`) instead when you want the structured cards only, with
   no LLM synthesis.
2. **Handle the failure modes** — `400` (malformed body), `401` (bad key), and
   `408` (`REQUEST_TIMEOUT`, the request exceeded the processing limit — retry
   with backoff). Errors use the `BaseAPIError` envelope
   (`error_message` + `error_type`).
3. **Export the numbers behind a card** — call `contents`
   (`POST /api/v1/contents`) with the card's `webpage_url` (or a web result URL).
   Default `mode: url` returns a short-lived presigned download link; `mode:
   inline` returns CSV in the body. Send `quote_only: true` first to price an
   export for free before fetching (`cost` + `export_pricing`, nothing billed).
   A protected-source card returns `403`.

## Rules

- Render `cards[]` as embeddable Tako cards; never invent numbers on top of them.
- The first 20 rows of a card export are free; rows beyond `free_rows` bill at
  the published `row_cpm_usd` per 1,000 rows up to `max_rows_ceiling` (2,000).
- There is no idempotency key — `answer`/`search` are read-style POSTs; retry
  safely on `408`/`5xx`.
