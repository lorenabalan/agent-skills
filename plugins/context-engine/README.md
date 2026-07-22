# Context Engine plugin

Connect Claude Code to the Elastic **Context Engine** (Project Meridian).

> **Status: technical preview.** Scaffolded for
> [search-team#15425](https://github.com/elastic/search-team/issues/15425). The trace-import
> flow is documented (manual setup via the Claude Code integration); there is no automation
> script yet, and the ES|QL `ai_index` skill is planned.

## What it does

- **Import Claude Code traces into Elasticsearch** — ships Claude Code agentic telemetry into
  Elasticsearch (via the existing [Claude Code integration](https://www.elastic.co/docs/reference/integrations/claude_code))
  so it can feed the Context Engine.
- **Query the `ai_index` via ES|QL** — *planned*; a future skill so Claude Code can query the
  Context Engine's `ai_index` directly.

## Install

Add the marketplace, then install this plugin:

```sh
claude plugin marketplace add https://github.com/elastic/agent-skills
claude plugin install context-engine@elastic-agent-skills
```

Restart your Claude Code session if newly installed skills don't appear immediately.

## Use

After installing, the `context-engine-import-traces` skill guides you through:

1. Enabling Claude Code OpenTelemetry export.
2. Installing the Claude Code integration in Kibana → Fleet.
3. Verifying traces are landing in Elasticsearch.

See [`skills/context-engine-import-traces/SKILL.md`](skills/context-engine-import-traces/SKILL.md)
for the full flow.

## Skills

| Skill | Description |
| ----- | ----------- |
| [`context-engine-import-traces`](skills/context-engine-import-traces/SKILL.md) | Import Claude Code agentic traces into Elasticsearch |

## Roadmap

- ES|QL skill to query the Context Engine `ai_index` (acceptance criterion 3 of #15425).

## References

- Tracking issue: [search-team#15425](https://github.com/elastic/search-team/issues/15425)
- Claude Code integration: <https://www.elastic.co/docs/reference/integrations/claude_code>