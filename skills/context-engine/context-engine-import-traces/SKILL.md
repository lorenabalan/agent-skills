---
name: context-engine-import-traces
description: >
  Import Claude Code agentic traces into Elasticsearch so they can feed the Elastic Context
  Engine. Use when setting up Claude Code telemetry export, installing the Claude Code
  integration, or verifying that Claude Code session traces are landing in Elasticsearch.
metadata:
  author: elastic
  version: 0.1.0
---

# Import Claude Code Traces into Elasticsearch

> **Status: skeleton (technical preview).** This skill lays out the intended flow for
> importing Claude Code traces into Elasticsearch as part of the Context Engine plugin
> ([search-team#15425](https://github.com/elastic/search-team/issues/15425)). Steps marked
> **TODO** are not yet verified against the integration and must be confirmed before this
> skill is used for real ingestion.

Connect Claude Code to the Context Engine by shipping its agentic telemetry into
Elasticsearch. This builds on the existing **Claude Code** Elastic integration, which
receives Claude Code OTLP telemetry exported by the Claude Code CLI via OpenTelemetry.

## Prerequisites

| Requirement | Notes |
| ----------- | ----- |
| Elastic deployment (Cloud, Serverless, or self-managed) with Kibana + Fleet | Target for the imported traces |
| Claude Code CLI | The source of the traces |
| Fleet-managed Elastic Agent (or an OTel collector endpoint) | Receives the OTLP export |

Set connection details before running any verification step:

| Variable | Required | Description |
| -------- | -------- | ----------- |
| `ELASTICSEARCH_URL` | Yes | Elasticsearch base URL |
| `ELASTICSEARCH_API_KEY` | No | API key for authentication (preferred) |
| `KIBANA_URL` | No | Kibana base URL (for installing the integration via Fleet) |

## Flow

### Step 1 — Enable Claude Code OpenTelemetry export

Configure the Claude Code CLI to export telemetry over OTLP to your Elastic Agent / OTel
collector endpoint.

> **TODO:** confirm the exact environment variables and values against the integration docs
> (e.g. enabling telemetry and pointing the OTLP exporter at the collector endpoint). Do not
> hard-code specific variable names here until verified — see
> <https://www.elastic.co/docs/reference/integrations/claude_code>.

### Step 2 — Install the Claude Code integration

Install the **Claude Code** integration in Kibana → Fleet so the Elastic Agent receives the
Claude Code OTLP telemetry and writes it to Elasticsearch.

- Integration docs: <https://www.elastic.co/docs/reference/integrations/claude_code>
- Integration package source (local): `~/Code/elastic/integrations/packages/claude_code`
  (owner: `elastic/security-service-integrations`; collects Claude Code events via the
  `otelcol` input as OTLP logs).

> **TODO:** document the concrete Fleet policy / input configuration this plugin expects.

### Step 3 — Verify traces landed in Elasticsearch

Confirm Claude Code session traces are being indexed.

> **TODO:** confirm the data stream / index the integration writes to, then add a concrete
> verification query (e.g. an `_search` or ES|QL count over the Claude Code events data
> stream) here.

## References

- Existing Claude Code logs integration: <https://www.elastic.co/docs/reference/integrations/claude_code>
- Integration package source: `~/Code/elastic/integrations/packages/claude_code`
- Tracking issue: [search-team#15425](https://github.com/elastic/search-team/issues/15425)
  (parent [#15383](https://github.com/elastic/search-team/issues/15383));
  coordinates with Feedback Loop [#15386](https://github.com/elastic/search-team/issues/15386),
  which owns trace collection.
- See `references/README.md` for background on the Context Engine and the ingest surface.

## Roadmap

- **Query the `ai_index` via ES|QL (future).** Once traces are landing, add an ES|QL skill so
  Claude Code can query the `ai_index` directly (acceptance criterion 3 of #15425). This will
  ship as a separate skill in this plugin — see the elasticsearch-esql skill for ES|QL syntax.

## Out of scope

- Building Key Insights (KIs) from imported traces — that is downstream trace work, not this
  plugin.