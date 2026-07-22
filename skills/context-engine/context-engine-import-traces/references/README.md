# Context Engine — reference notes

Background material for the `context-engine-import-traces` skill. **Skeleton** — expand as the
import mechanism is confirmed.

## What the Context Engine is

Project Meridian (the Context Engine) is a governed, shared, self-improving knowledge
foundation that sits between raw enterprise data and any AI agent. This plugin connects Claude
Code to it: as a first step, by importing Claude Code agentic traces into Elasticsearch.

## Ingest surface (to confirm)

The import builds on the existing **Claude Code** Elastic integration, which receives Claude
Code OTLP telemetry via OpenTelemetry (`otelcol` input) and writes it to Elasticsearch.

- Integration docs: <https://www.elastic.co/docs/reference/integrations/claude_code>
- Package source: `~/Code/elastic/integrations/packages/claude_code`
  - `manifest.yml` — `otelcol` input, `basic` subscription, Kibana `^9.4.0`
  - `data_stream/events` — the events data stream
  - `kibana/dashboard` — prebuilt dashboards (session timeline, tool-call analysis, etc.)

> **TODO:** record the exact data stream / index name(s) the integration writes to, and how
> the Context Engine's `ai_index` relates to them, once verified.