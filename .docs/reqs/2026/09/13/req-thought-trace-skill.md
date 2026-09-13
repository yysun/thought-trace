# Requirements: Thought Trace Skill

## Problem

AI work sessions contain the observable evidence of a user's evolving reasoning, but native session histories are difficult to use as a unified, selective record of questions, decisions, alternatives, and unresolved issues.

## Outcome

Provide an installable `thought-trace` agent skill that guides a host through discovering Codex and Claude Code sessions, selectively reading relevant evidence, and returning a Markdown reconstruction of the user's thinking evolution.

## Acceptance Criteria

- [ ] A skill exists at `skills/thought-trace/SKILL.md` with valid, discoverable YAML frontmatter whose `name` matches its folder.
- [ ] The skill supports `/trace`-style and natural-language requests scoped by date, project, topic, evolution, decisions, or open questions.
- [ ] The workflow discovers candidates from native Codex and Claude Code stores before reading full session content and continues when either source is unavailable.
- [ ] The skill maps relevant messages into a small common session model and filters tool noise unless it explains a changed decision or outcome.
- [ ] The output prioritizes conceptual transitions, decisions, alternatives, rejected approaches, outcomes, open questions, and source provenance over a raw chronology.
- [ ] The skill requires no exporter, database, daemon, or copying of raw transcripts for normal use.

## Constraints

- Preserve the PRD's LLM-first and progressive-disclosure approach.
- Treat session history as evidence; do not claim access to hidden reasoning.
- Use host filesystem and shell capabilities where available.
- Original session files remain the source of truth.

## Non-Goals

- Implementing a session parser, indexer, exporter, database, daemon, or visual UI.
- Building a chat-history viewer or employee-monitoring system.

## Blocking Questions

None.