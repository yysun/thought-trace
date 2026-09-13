# Plan: Thought Trace Skill

## Outcome

Create a compact, self-contained agent skill that operationalizes the PRD as a repeatable host-native workflow.

## Story Base

`af22748bde2b6eb3a9a42ca891ba59037b82b254`

## Boundaries

- Add only the Thought Trace skill and RPD story artifacts.
- Do not add runtime code, dependencies, storage, or external integrations.
- The skill must describe an evidence-grounded procedure that works with partial source availability.

## Decisions

- Use `skills/thought-trace/SKILL.md` to honor the requested repository layout, with standard skill frontmatter.
- Make the workflow LLM-first: native file discovery, metadata filtering, and focused reading precede synthesis.
- Include explicit evidence and uncertainty rules to prevent fabricated transitions or hidden-reasoning claims.

## Tasks

- [x] Author `skills/thought-trace/SKILL.md` with discovery metadata, invocation examples, source handling, selective-reading procedure, normalization, analysis rules, and Markdown output format.
- [x] Validate the skill frontmatter and manually inspect the workflow against the acceptance criteria.

## Validation

- Parse YAML frontmatter and confirm the required `name` and meaningful `description` fields.
- Use the E2E scenarios in `.docs/tests/test-thought-trace-skill.md` as a procedure walkthrough.
- Run `git diff --check` to detect whitespace errors.

## Risks

- Session formats and permissions differ by host. The skill mitigates this by treating source adapters as logical procedures, reporting inaccessible sources, and continuing with available evidence.