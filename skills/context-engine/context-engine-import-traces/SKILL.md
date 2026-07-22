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

> **Status: technical preview.** This skill guides manual setup — there is no automation script
> yet. Tracking: [search-team#15425](https://github.com/elastic/search-team/issues/15425).

Connect Claude Code to the Context Engine by shipping its agentic telemetry into Elasticsearch.
Claude Code exports telemetry as OTLP (OpenTelemetry Protocol) **log events** — each record is
one action in an agentic session (tool calls, API requests, user prompts, permission decisions,
MCP connections, hooks, plugins, and session lifecycle). This skill builds on the existing
**Claude Code** Elastic integration, which receives those events and indexes them into the
`logs-claude_code.events.otel-*` data stream.

## Prerequisites

| Requirement | Notes |
| ----------- | ----- |
| Claude Code CLI **2.1.0 or later** | Required for OTLP log export |
| Elastic deployment **9.4.0 or later** with Kibana + Fleet | Target for the imported traces |
| An OTLP ingest path | Managed OTLP endpoint, a Fleet-managed Elastic Agent, or an EDOT Collector (see Step 2) |

## Flow

### Step 1 — Install the Claude Code integration (do this first)

Install the **Claude Code** integration in Kibana → Fleet **before** sending data. Installation
creates the ingest pipeline, field mappings, and dashboards that process incoming events.

- Integration docs: <https://www.elastic.co/docs/reference/integrations/claude_code>
- Integration package source (local): `~/Code/elastic/integrations/packages/claude_code`
  (owner: `elastic/security-service-integrations`; `otelcol` input, `basic` subscription).

### Step 2 — Enable Claude Code OpenTelemetry export

Configure the Claude Code CLI to export OTLP logs. Pick one deployment path and set the
environment variables where Claude Code runs.

**Option A — Managed OTLP (recommended; no agent or collector needed):**

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
export OTEL_EXPORTER_OTLP_ENDPOINT="<your-elastic-cloud-otlp-endpoint>"
export OTEL_RESOURCE_ATTRIBUTES="data_stream.dataset=claude_code.events.otel"
```

**Option B — Elastic Agent OTLP receiver** — the Elastic Agent exposes an OTLP HTTP receiver on
port 4318. Use the same variables as Option A, but point the endpoint at the agent:

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT="http://<agent-host>:4318"
```

**Option C — EDOT Collector** — run the Elastic Distribution of the OpenTelemetry Collector with
an `otlp` receiver and an `elasticsearch` exporter, setting the same `data_stream.dataset`
resource attribute. It routes events to `logs-claude_code.events.otel-*`.

**Optional — verbosity gates** (all off by default; enabling them indexes potentially sensitive
content such as commands, file contents, and prompts):

| Variable | Includes |
| -------- | -------- |
| `OTEL_LOG_USER_PROMPTS` | `prompt` text in `user_prompt` events |
| `OTEL_LOG_TOOL_DETAILS` | `tool_parameters` / `tool_input` in tool events |
| `OTEL_LOG_TOOL_CONTENT` | `tool_result` content in tool events |
| `OTEL_LOG_RAW_API_BODIES` | raw API request/response bodies |

### Step 3 — Verify traces landed in Elasticsearch

Run a short Claude Code session with telemetry enabled, then confirm events arrive in the
`logs-claude_code.events.otel-*` data stream.

- In Kibana **Discover**, filter: `data_stream.dataset: claude_code.events.otel`
- Or count events by type with ES|QL:

```esql
FROM logs-claude_code.events.otel-*
| STATS count = COUNT(*) BY event.action
| SORT count DESC
```

If nothing arrives:

- Verify `CLAUDE_CODE_ENABLE_TELEMETRY=1` is set in the environment where Claude Code runs.
- Check the OTLP endpoint is reachable from the Claude Code host
  (`curl -v http://<agent-host>:4318/v1/logs`).
- Confirm the Elastic Agent is running and the integration policy is assigned.

## References

- Claude Code integration docs: <https://www.elastic.co/docs/reference/integrations/claude_code>
- Integration package source: `~/Code/elastic/integrations/packages/claude_code`
- Tracking issue: [search-team#15425](https://github.com/elastic/search-team/issues/15425)
  (parent [#15383](https://github.com/elastic/search-team/issues/15383)); coordinates with
  Feedback Loop [#15386](https://github.com/elastic/search-team/issues/15386), which owns trace
  collection.
- See `references/README.md` for the event schema and Context Engine background.

## Roadmap

- **Query the Context Engine `ai_index` via ES|QL (future).** Once traces are landing, add a
  skill so Claude Code can query the Context Engine's AI-Index directly (acceptance criterion 3
  of #15425). This skill should **compose with the existing `elasticsearch-esql` skill** (in the
  `elasticsearch` plugin) for ES|QL syntax and query execution rather than reimplementing it —
  its value-add is Context-Engine-specific: the target index, its schema, and canned queries
  over the imported traces. Prerequisite: confirm how the `logs-claude_code.events.otel-*` traces
  feed the AI-Index (a Context-Engine-side detail) before building it.

## Out of scope

- Building Key Insights (KIs) from imported traces — that is downstream trace work, not this
  plugin.