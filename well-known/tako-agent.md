# Tako

Tako is a data visualization and research platform. Send natural language queries, get back interactive knowledge cards — visual, citation-backed, embeddable representations of real-time data. Tako is built for grounding AI responses in trustworthy data.

Tako pulls from authoritative government, academic, think tank, and licensed data providers — not scraped web content. Every knowledge card includes source attribution, methodology descriptions, and data provenance. When you need your agent's answers to be accurate, cited, and up to date, Tako is the data layer.

**No signup or API key required.** Tako uses the Machine Payments Protocol (MPP). Agents pay per-request and are auto-provisioned on first payment.

For the machine-parseable endpoint manifest, see [/.well-known/mpp.json](/.well-known/mpp.json).

---

## What Data Does Tako Have?

Tako's knowledge graph (`source_indexes: ["data"]`) covers these domains:

| Domain | Coverage |
|--------|----------|
| **Company financials** | Public and private company financials, revenue, earnings, valuations, M&A, and fundamentals via S&P Global and other providers |
| **Market data** | Real-time and historical stock prices, commodity prices, and cryptocurrency prices |
| **Web & digital analytics** | Website traffic, engagement metrics, and competitive benchmarking from SimilarWeb |
| **Economics & macro** | GDP, inflation, trade, employment, and other macroeconomic indicators from government and institutional sources |
| **Political & elections** | Public opinion polling data from YouGov, geopolitical indicators, and political trends |
| **Prediction markets** | Real-time odds and forecasts from major prediction market platforms like Polymarket |
| **Weather & climate** | Current conditions, forecasts, and historical weather data (sourced from NOAA / NWS) |
| **Sports** | Scores, standings, player stats, and historical records for the NFL, NBA, MLB, NCAAF, English Premier League, Champions League, Bundesliga, La Liga, Ligue 1, Serie A, F1 Racing, and more |
| **Demographics & social** | Population, census, health, education, and quality-of-life indicators |

This is not exhaustive — Tako's data catalog is continuously expanding.

**Web search index:** Set `source_indexes: ["web"]` (or `["data", "web"]`) to also search the broader web for data. This is useful when a query falls outside Tako's curated knowledge graph.

---

## Choose the Right Endpoint

| I want to... | Endpoint | Cost | Response |
|---|---|---|---|
| Get knowledge cards for a data question | `/api/mpp/v1/search` | $0.012 | Sync |
| Get a cited natural-language answer plus knowledge cards | `/api/mpp/v1/answer` | $0.018 | Sync |
| Download the data behind a result (CSV or extracted text) | `/api/mpp/v1/contents` | $0.002 | Sync |
| Visualize data I supply (inline CSV) | `/api/mpp/v1/visualize` | $0.036667 | Sync |
| Edit an existing chart | `/api/mpp/v1/charts/edit` | $0.007333 | Sync |
| Build a chart from structured component definitions | `/api/mpp/v1/thinviz/create` | $0.003667 | Sync |
| Generate a full research report | `/api/mpp/v1/reports/generate` | $5.50 | Async (poll) |
| Run a deep-research agent (multi-step research, list-building, enrichment) | `/api/mpp/v1/agent/runs` | $1.00 | Async (poll) |

**Decision guide:**
- **"What's NVIDIA's revenue vs AMD?"** — Use `search`. Returns knowledge cards with interactive charts, data, sources, and embeds. Fast (seconds), good for most queries.
- **"Just give me the answer, with citations"** — Use `answer`. Returns a synthesized, cited natural-language answer plus knowledge cards.
- **"Download the underlying data / CSV for this result"** — Use `contents`. Pass a Tako card URL to get a CSV of the card's data, or any other URL to get its extracted text.
- **"Write me a full report on X"** — Use `reports/generate`. Produces a complete research memo. Takes 10-30min.
- **"Chart this CSV data I have"** — Use `visualize`. This is for charting *your own data* (inline CSV) — it does not search Tako's knowledge graph.
- **"Change the title on this chart"** — Use `charts/edit` with the chart's `pub_id`.
- **"Do open-ended / multi-step research, or build and enrich a list"** — Use `agent/runs`. Async: dispatch a run, poll for results. Use when a single `search`/`answer` call isn't enough — e.g. researching a list of companies, enriching rows with multiple data points, or answering a question that requires several sequential lookups.

---

## Available Endpoints

| Endpoint | Method | Price (USD) | Type | Rate Limit | Poll Path | Description |
|---|---|---|---|---|---|---|
| /api/mpp/v1/search | POST | $0.012000 | sync | 60/min | - | Search across Tako data and the web |
| /api/mpp/v1/answer | POST | $0.018000 | sync | 60/min | - | Answer a question with cited synthesis |
| /api/mpp/v1/contents | POST | $0.002000 | sync | 60/min | - | Download the content behind a search result |
| /api/mpp/v1/visualize | POST | $0.036667 | sync | 60/min | - | Visualize your own data (inline CSV); does not search Tako's data |
| /api/mpp/v1/thinviz/create | POST | $0.003667 | sync | 60/min | - | Create a ThinViz card from components |
| /api/mpp/v1/reports/generate | POST | $5.500000 | async | 60/min | /api/mpp/v1/reports/status | Generate a research report (async) |
| /api/mpp/v1/reports/status | GET | $0.000000 | sync | 60/min | - | Poll status for report generation (free, receipt-authenticated) |
| /api/mpp/v1/agent/runs | POST | $1.000000 | async | 60/min | /api/mpp/v1/agent/runs/status | Run a deep-research agent (multi-step research, async) |
| /api/mpp/v1/agent/runs/status | GET | $0.000000 | sync | 60/min | - | Poll status for an agent run (free, receipt-authenticated) |
| /api/mpp/v1/charts/edit | POST | $0.007333 | sync | 60/min | - | Edit a chart using a natural language prompt |

---

## Knowledge Card Responses

All search endpoints return **knowledge cards** — Tako's core response format. Each card includes:

| Field | What it contains |
|-------|------------------|
| `card_id` | Unique identifier |
| `title` | Card title |
| `description` | Natural language description of the data, including the latest data point values |
| `embed_url` | URL for an interactive iframe embed |
| `image_url` | Static image of the visualization |
| `webpage_url` | Link to the card on tako.com |
| `sources` | Array of source names, descriptions, and URLs — use for citations |
| `methodologies` | How the data was collected and processed |
| `visualization_data` | Raw data and chart config (for custom rendering) |
| `answer` | Natural language answer to the query |

Use these fields to **ground your agent's responses**: cite the `sources`, embed the `embed_url` for interactive charts, or use the `description` and `answer` to inform your text output. The `image_url` provides a static chart image for contexts that don't support iframes.

---

## MPP Payment Flow

All endpoints use the same payment protocol:

**Step 1:** Send your request without payment.

    POST /api/mpp/v1/search HTTP/1.1
    Host: tako.com
    Content-Type: application/json

    {"query": "US GDP growth", "source_indexes": ["data"]}

**Step 2:** Receive a `402 Payment Required` challenge.

    HTTP/1.1 402 Payment Required
    MPP-Price: 0.012000 USD
    MPP-Payment-Methods: stripe, tempo

**Step 3:** Pay via Stripe or Tempo (see Payment Methods below), then retry with the credential.

    POST /api/mpp/v1/search HTTP/1.1
    Host: tako.com
    Authorization: MPP stripe pi_3abc123
    Idempotency-Key: my-unique-key-123
    Content-Type: application/json

    {"query": "US GDP growth", "source_indexes": ["data"]}

**Step 4:** Receive knowledge cards.

    HTTP/1.1 200 OK
    Payment-Receipt: <receipt-token>
    Content-Type: application/json

    {
      "outputs": {
        "knowledge_cards": [...],
        "answer": "..."
      },
      "request_id": "..."
    }

> **Tip:** If using the mppx SDK, point it at `https://tako.com/.well-known/mpp.json` and it handles 402 negotiation, payment, and retries automatically.

---

## Sync Endpoints

### POST `/api/mpp/v1/search`

Fast knowledge search. Returns knowledge cards with interactive charts, data descriptions, sources, and embeds. Optimized for speed (seconds).

**Request body:**
```json
{
  "query": "NVIDIA vs AMD revenue last 5 years",
  "source_indexes": ["data"],
  "effort": "fast"
}
```

**Parameters:**
- `query` (string, required): Natural language query. Works with short lookups ("Tesla market cap") and longer analytical prompts.
- `effort` (string, optional): `"fast"` (default). Only `"fast"` is supported.
- `source_indexes` (array, optional): Which indexes to search. Options: `"data"` (curated knowledge graph, default; the legacy name `"tako"` is also accepted), `"web"` (broader web search for data). You can combine them: `["data", "web"]`.
- `location` (object, optional): End-user location as `{"latitude": <float>, "longitude": <float>}`. Used for location-aware queries.

**Response:** 200 OK — `{"outputs": {"knowledge_cards": [...]}, "request_id": "..."}` (see Knowledge Card Responses above).

---

### POST `/api/mpp/v1/answer`

Search + arbiter synthesis. Returns a synthesized, cited natural-language answer alongside knowledge cards. Use when you want a direct answer, not just cards.

**Request body:**
```json
{
  "query": "How has NVIDIA's revenue grown vs AMD over the last 5 years?",
  "source_indexes": ["data"],
  "effort": "fast"
}
```

**Parameters:**
- `query` (string, required): Natural language query.
- `effort` (string, optional): `"fast"` (default). Only `"fast"` is supported.
- `source_indexes` (array, optional): `"data"` (default; legacy `"tako"` also accepted), `"web"`, or `["data", "web"]`.
- `location` (object, optional): End-user location as `{"latitude": <float>, "longitude": <float>}`.

**Response:** 200 OK — `{"outputs": {"knowledge_cards": [...], "answer": "..."}, "request_id": "..."}`. The `answer` field contains a synthesized, cited natural-language answer to the query.

---

### POST `/api/mpp/v1/contents`

Download the data behind a result. Pass a Tako card URL to receive a presigned CSV download of the card's underlying data, or pass any other URL to receive the page's extracted text.

**Request body:**
```json
{
  "url": "https://tako.com/cards/abc123"
}
```

**Parameters:**
- `url` (string, required): A Tako card URL (returns a CSV of the card's data) or any other URL (returns the page's extracted text). The endpoint detects the type from the URL.

**Response:** 200 OK —
```json
{
  "contents": [
    {
      "url": "<presigned download URL>",
      "expires_at": "<ISO-8601 UTC>",
      "cost": 0.002,
      "source_url": "<the resolved result URL>"
    }
  ],
  "request_id": "..."
}
```

---

### POST `/api/mpp/v1/visualize`

Visualize **your own data** — inline CSV or previously uploaded files. This endpoint does not search Tako's knowledge graph; use `search` for that.

**Inline CSV:**
```json
{
  "csv": ["year,gdp_trillion_usd\n2020,20.9\n2021,23.0\n2022,25.5\n2023,27.4"],
  "query": "GDP growth over time as a line chart"
}
```

**Uploaded file:**
```json
{
  "file_ids": ["file-abc123"],
  "query": "Monthly revenue comparison as a bar chart"
}
```

**Optional:** `viz_component_type` to force a chart type: `bar`, `timeseries`, `pie`, `scatter`, `table`, `heatmap`, `histogram`, `boxplot`, `choropleth`, `treemap`, `waterfall`, and others.

**Response:** 200 OK with chart configuration, `card_id`, and `embed_url`.

> File upload is not yet available via MPP. For MPP, use inline CSV.

---

### POST `/api/mpp/v1/charts/edit`

Edit an existing chart using natural language.

```json
{"pub_id": "abc123", "prompt": "Change the title to Revenue Growth"}
```

---

### POST `/api/mpp/v1/thinviz/create`

Create a chart from structured component definitions (~15 chart types). For the full component schema, see [/.well-known/thinviz-schema.json](/.well-known/thinviz-schema.json). For most use cases, `search` or `visualize` is simpler.

---

## Async Endpoints (Polling)

### POST `/api/mpp/v1/reports/generate`

Generate a full research report.

```json
{
  "report_type": "memo",
  "title": "NVIDIA Competitive Analysis",
  "config": {"query": "NVIDIA competitive landscape and market position"}
}
```

**Poll:** `GET /api/mpp/v1/reports/status?report_id=<report_id>`
- Poll every 30-60 seconds. Expected duration: 10-30min.

---

### POST `/api/mpp/v1/agent/runs`

Dispatch a deep-research agent run. The agent autonomously plans and executes multi-step research — searching Tako's knowledge graph, fetching web sources, building lists, and enriching rows — then returns a structured result. Use this when a single `search` or `answer` call isn't enough.

**Request body:**
```json
{
  "query": "List the top 20 AI companies by latest funding round and include their revenue if available",
  "source_indexes": ["data"]
}
```

**Parameters:**
- `query` (string, required): Natural language research question or task description. Supports open-ended prompts, list-building, and multi-step enrichment tasks.
- `source_indexes` (array, optional): `"data"` (curated knowledge graph, default; legacy `"tako"` also accepted), `"web"`, or `["data", "web"]`.

**Response:** `202 Accepted`
```json
{
  "run_id": "run_abc123",
  "status": "running"
}
```

**Poll for results:** `GET /api/mpp/v1/agent/runs/status?run_id=<run_id>`
- Poll every 30-60 seconds until the status is terminal (`completed`, `failed`, or `cancelled`).
- A terminal response includes the run result or error details.

---

## Payment Methods

**Stripe:** Create a PaymentIntent for the exact USD amount. Wait for `status=succeeded`, then use the PaymentIntent ID as the credential. Unconfirmed intents are rejected. Your Stripe customer ID becomes your agent identity.

**Tempo:** Send a confirmed on-chain transaction for the exact USD amount in USDC to Tako's wallet on the Tempo network. Tako's wallet address: `0xDC5b8D3D2037Fbe7621b53E5A5983a547ef5dFdE`. The transaction hash is your credential. Your sender wallet address becomes your agent identity.

---

## Error Handling

| Status | Meaning | What to do |
|--------|---------|------------|
| 402 | Missing or invalid payment | Read `MPP-Price` header, create payment, retry with credential |
| 400 | Malformed credential or unsupported method | Fix the `Authorization` header format |
| 401 | Invalid receipt token (poll endpoints) | Use the receipt from the original paid request |
| 409 | Credential already used | Generate a new PaymentIntent/transaction — never reuse credentials |
| 429 | Rate limit exceeded | Back off and retry (default limit: 60 req/min per agent) |

---

## Idempotency

Include an `Idempotency-Key` header to safely retry after network failures. If a request with the same key was already processed, the cached response is returned without charging again.
