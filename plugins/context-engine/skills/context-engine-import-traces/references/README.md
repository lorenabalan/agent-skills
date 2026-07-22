# Context Engine — reference notes

Background material for the `context-engine-import-traces` skill.

## What the Context Engine is

Project Meridian (the Context Engine) is a governed, shared, self-improving knowledge foundation
that sits between raw enterprise data and any AI agent. This plugin connects Claude Code to it —
as a first step, by importing Claude Code agentic traces into Elasticsearch.

## Ingest surface

Import builds on the existing **Claude Code** Elastic integration, which receives Claude Code
OTLP **log events** via OpenTelemetry (the Elastic Agent's built-in OTLP HTTP receiver on port
4318), applies an ingest pipeline, and indexes the result.

- **Data stream:** `logs-claude_code.events.otel-*` (type `logs`, dataset `claude_code.events.otel`)
- **Compatibility:** Claude Code CLI 2.1.0+, Elastic 9.4.0+
- Integration docs: <https://www.elastic.co/docs/reference/integrations/claude_code>
- Package source: `~/Code/elastic/integrations/packages/claude_code`
  - `manifest.yml` — `otelcol` input, `basic` subscription, Kibana `^9.4.0`
  - `data_stream/events` — the events data stream and field mappings
  - `kibana/dashboard` — prebuilt dashboards (session timeline, tool-call analysis, permission
    decisions, MCP server access, cost & usage, security overview)

## Event types

Each OTLP log record carries an event name (mapped to `event.action`) and event-specific
attributes namespaced under `claude_code.events.*`:

| `event.action` | Description | ECS category |
| -------------- | ----------- | ------------ |
| `tool_result` | Tool execution outcome (success/failure, duration, parameters) | `process` |
| `tool_decision` | Permission decision for a tool call (accept/reject, source) | `iam` |
| `api_request` | API call to Anthropic (model, cost, tokens, duration) | `api` |
| `user_prompt` | User prompt submission (length, command, optionally text) | — |
| `api_error` | API request failure (error, status code, retry) | `api` |
| `api_refusal` | Content-safety refusal from the model | `api` |
| `permission_mode_changed` | Permission mode change (from/to, trigger) | `configuration` |
| `mcp_server_connection` | MCP server connection attempt (status, transport) | `network` |
| `hook_registered` / `hook_execution_start` / `hook_execution_complete` | Hook lifecycle | `configuration` / `process` |
| `plugin_loaded` | Plugin loaded (name, scope, paths) | `library` |
| `skill_activated` | Skill activation (name, source, trigger) | — |

Common fields include `claude_code.events.session.id`, `event.action`, `gen_ai.request.model`,
`claude_code.events.cost_usd`, `gen_ai.usage.input_tokens` / `output_tokens`, and `user.email`.
See the integration's "Exported fields" reference for the full list.

## Relationship to the Context Engine `ai_index`

**To confirm.** The traces land in `logs-claude_code.events.otel-*`. How that data feeds the
Context Engine's AI-Index — the surface the planned ES|QL skill will query (acceptance criterion
3 of #15425) — is a Context-Engine-side concern and is not yet documented here. Resolve this
before building the ES|QL skill; do not assume the trace data stream and the AI-Index are the
same index.