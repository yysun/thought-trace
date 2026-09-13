---
name: thought-trace
description: 'Reconstruct the observable evolution of a user''s thinking across Codex and Claude Code sessions. Use when asked for a thought trace, thinking history, how an idea evolved, past decisions, alternatives, rejected approaches, or unresolved questions; triggers include /trace, /thought-trace, /思考轨迹, evolution, decisions, and open questions.'
argument-hint: 'today | this week | project <name> | topic <term> | evolution <term> | decisions | open-questions'
user-invocable: true
---

# Thought Trace

**Version:** `1.0.0`
**Repository:** https://github.com/yysun/thought-trace

Reconstruct how a user's observable ideas and work developed across native AI work sessions. Sessions are evidence; the product is an evidence-grounded explanation of the evolution of thought, not a transcript archive or hidden chain-of-thought recovery.

## Use For

- `/trace today`, `/trace this week`, or `/trace project company-wiki`
- `/trace topic "agent orchestration"` or `/trace evolution "LLM Wiki"`
- `/trace decisions` or `/trace open-questions`
- Natural-language requests such as "Why did I move from traditional RAG toward LLM Wiki?"
- Comparisons between a project's earlier and current thinking

## Boundaries

- Read original host session stores directly when permission permits; do not require export, import, a database, an index, a daemon, or a background service.
- Do not copy raw transcripts or create durable derived data unless the user explicitly asks.
- Use only user-visible prompts, assistant responses, and material tool evidence. Never claim to recover hidden model reasoning.
- Treat source availability and session format as fallible. Report unavailable sources and continue with accessible evidence.
- Prefer a concise conclusion with evidence-backed transitions over exhaustive chronology.

## Scope The Request

Determine the narrowest useful scope before reading content:

| Request form | Scope |
| --- | --- |
| `today`, `this week`, date range | Time window |
| `project <name>` | Repository, working directory, and project-name signals |
| `topic <term>` | Topic and related terms |
| `evolution <term>` | Topic plus changes, decisions, and causal evidence |
| `decisions` | Explicit conclusions and their rationale |
| `open-questions` | Unresolved or recurring questions |

If the request is ambiguous, infer scope from the current repository, directory, and wording. Ask a question only when plausible scopes would produce materially different traces.

## Discover Sources

Check each source independently. Typical locations are:

```text
Codex:       ~/.codex/sessions/
Claude Code: ~/.claude/projects/
```

1. Determine whether each directory is readable.
2. Discover likely session files using host file search or shell tools such as `find` and `rg --files`.
3. Read only metadata or small record samples needed to identify timestamps, session IDs, working directories, repository/project signals, and topic terms.
4. Record source status: available, empty, inaccessible, or unsupported format.

Do not fail the trace if a source is missing or inaccessible. State the source and expected path, then continue with available sources.

## Select Candidates Before Reading

Rank sessions using the request scope, in this order:

1. Explicit time range.
2. Working directory, repository, or inferred project name.
3. Exact topic, keyword, or file-name matches.
4. Nearby related concepts, decisions, or recurring questions.

Start with a small candidate set. Expand only when the evidence is insufficient to explain a transition, decision, reversal, or unresolved question. Do not load every session merely because it exists.

## Read And Normalize

For selected sessions, read user prompts and the assistant responses that directly address them. Ignore tool calls, verbose command output, patches, and generated text by default. Include tool evidence only when it materially explains an experiment, failure, outcome, or changed decision.

Represent each relevant session with this minimal logical model:

```json
{
  "source": "codex",
  "session_id": "...",
  "started_at": "...",
  "updated_at": "...",
  "working_directory": "...",
  "project": "...",
  "messages": [
    {"timestamp": "...", "role": "user", "content": "..."},
    {"timestamp": "...", "role": "assistant", "content": "..."}
  ]
}
```

Derive a project from the strongest available signals: working directory, repository name, session metadata, discussed files, and repeated concepts. A session may belong to more than one topic; infer topics dynamically from the evidence rather than requiring manual tags.

## Reconstruct The Thinking

Extract and connect only evidence supported by the selected material:

- **Questions and hypotheses:** What the user was trying to understand, decide, or solve.
- **Decisions:** Explicit conclusions and their stated reasons.
- **Alternatives:** Competing approaches that received genuine consideration.
- **Changes and rejections:** Reversals, refinements, and abandoned approaches, together with the evidence that caused the change.
- **Outcomes:** Implementations, documents, experiments, or conclusions resulting from the discussion.
- **Open questions:** Explicitly unresolved issues or concerns that recur without resolution.

Build conceptual relationships, not a timestamp list. Useful patterns include:

```text
Evolution:   A -> B -> C
Refinement:  A -> A1, A2, A3
Reversal:    A -> evidence X -> reject A -> adopt B
Convergence: separate discussions -> shared principle
```

Cross host boundaries freely: Codex and Claude Code sessions are one thinking history. Preserve each source as provenance, but do not treat host transitions as conceptual transitions without supporting evidence.

## Evidence Discipline

- Distinguish direct statements from careful inferences. Use wording such as "The discussion appears to have shifted" when the link is inferred.
- Do not invent rationale, alternatives, outcomes, or a resolution absent from the evidence.
- When evidence conflicts, describe the contradiction and identify which statement is later or better supported; do not silently flatten it.
- When candidate evidence is too thin, say so and name the scope or source that would improve the trace.
- Include compact provenance after material claims, for example `Codex, 2026-09-13, project/company-wiki session`.

## Produce Markdown

Use only the sections supported by evidence. Favor this shape:

```markdown
# <Topic or Project>

## Thought Evolution

Explain the conceptual transitions and why they happened. Put chronology only where it clarifies the transition.

## Decisions

- **Decision:** rationale and supporting evidence. _(source, date/session)_

## Alternatives Considered

- Alternative, why it was considered, and why it was retained or rejected.

## Outcomes

- Resulting implementation, artifact, or conclusion.

## Open Questions

- Unresolved question, including recurring evidence where relevant.

## Evidence Scope

- Sources searched, date range, selected-session count, and unavailable sources.
```

Omit empty sections. For an evolution request, make **Thought Evolution** the center of the answer. For decisions or open-question requests, lead with that requested evidence and include only enough surrounding evolution to make it understandable.
