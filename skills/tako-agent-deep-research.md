---
name: tako-agent-deep-research
description: Use for token-efficient deep research over Tako's live data — dispatch a long-running answer or retrieval agent run, then poll (or stream via SSE) until it completes, and read the cited result.
api: openapi/tako-openapi-original.yml
operations: [createAnswerAgentRun, getAnswerAgentRun, listAnswerAgentRuns, createRetrievalAgentRun, getRetrievalAgentRun, listRetrievalAgentRuns]
method: generated
generated: '2026-07-21'
source: openapi/tako-openapi-original.yml
---

# Run a Tako research agent

Dispatch an asynchronous agent run for deep, multi-step research, then poll or
stream to completion. Two products share one shape: the **Answer Agent**
(prose + cards, top-level citations) and the **Retrieval Agent**.

## Auth

All agent endpoints take the API key in the `X-API-Key` header
(https://tako.com/console/api-keys). Anonymous → `401`.

## Steps (Answer Agent — same shape for Retrieval under `/v1/agent/retrieval/...`)

1. **Dispatch** — `createAnswerAgentRun` (`POST /v1/agent/answer/runs`) with an
   `AnswerAgentRunRequest`: `query` (required), optional `thread_id` to continue
   a thread, `source_indexes` (`data`, `web`, or both), `locale`, `timezone`.
   Returns `202` with an `AnswerAgentRun` (`run_id`, `status`).
   - `402` (`PAYMENT_REQUIRED`) means insufficient credit balance — top up.
   - `409` is a conflict: the thread already has a run in flight, a product
     mismatch, or changed pinned `source_indexes`.
2. **Poll** — `getAnswerAgentRun` (`GET /v1/agent/answer/runs/{run_id}`) until
   `status` is `completed` or `failed`. On `completed`, read `result`
   (`AnswerAgentResult`: `answer` markdown with `[n]` markers, `cards[]`,
   `citations[]`, `metadata`). `403` if the run is not owned by the caller;
   `404` if the run id is unknown.
3. **Stream (optional)** — send `Accept: text/event-stream` on dispatch or poll
   to receive an SSE stream of `AnswerAgentStreamEnvelope` events; the stream
   ends at `stream_done`. Resume with `starting_after` / `Last-Event-ID`. If the
   stream ends without an `agent_result` event, poll for the terminal status.
4. **List history** — `listAnswerAgentRuns` (`GET /v1/agent/answer/runs`) for a
   cursor-paginated page of run summaries (newest first; `next_cursor`,
   `has_more`).

## Rules

- Agent runs are asynchronous — never block on dispatch; poll or stream.
- Citations join to inline `[n]` markers; render sources, never fabricate.
- Follow-ups reuse `thread_id`; do not change pinned `source_indexes` mid-thread
  (that returns `409`).
- Errors on agent endpoints use the `ErrorObject` envelope (`code` + `message`).
