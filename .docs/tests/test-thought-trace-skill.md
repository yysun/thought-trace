# Thought Trace Skill Flows

## Evolution Across Hosts

Initial conditions: readable Codex and Claude Code session stores include discussions relevant to `Company Wiki`.

Actions:

1. Invoke `/trace evolution "Company Wiki"`.
2. Follow discovery, candidate selection, focused reading, and synthesis.

Observable outcomes:

- The response links conceptual transitions across both hosts rather than presenting two isolated summaries.
- It identifies supporting decisions, alternatives, evidence, and open questions where present.
- It marks host/session provenance without exposing raw full transcripts by default.

## Partial Source Availability

Initial conditions: Codex history is readable and Claude Code history is inaccessible.

Actions:

1. Invoke `/trace this week`.

Observable outcomes:

- The response reports the unavailable Claude Code source and required access path.
- It produces a trace from available Codex evidence.
- It does not fail the request or infer unobserved Claude Code content.