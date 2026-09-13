# Thought Trace / 思考轨迹
**PRD v0.2**

## 1. Overview

**Thought Trace（思考轨迹）** is an agent skill that reconstructs how the user's thinking evolves across AI work sessions.

It reads native session history from supported AI hosts such as **Codex** and **Claude Code**, then identifies:

- Questions being explored
- Evolution of ideas
- Alternatives considered
- Decisions made
- Approaches changed or rejected
- Outcomes
- Unresolved questions

The goal is not to archive conversations.

The goal is to recover the **trajectory of thinking behind the work**.

---

## 2. Problem

AI-assisted work is increasingly distributed across many sessions and tools.

Important context quickly becomes difficult to recover:

- Why was an architecture chosen?
- When did an idea first appear?
- What alternatives were considered?
- Why was an earlier approach abandoned?
- How has thinking about a project changed?
- What questions remain unresolved?
- Which decisions came from Codex versus Claude sessions?

Raw session history contains much of this information, but manually reviewing it is impractical.

---

## 3. Goal

Turn native AI session history into a searchable and understandable **thinking history**.

Example:

```text
RAG
 ↓
LLM Wiki
 ↓
Company Wiki
 ↓
Company Library Index
 ↓
Personal Wiki
 ↓
Progressive Expansion
```

Thought Trace should explain **how and why the transitions happened**, not merely list when they occurred.

---

## 4. Product Principle

> **Sessions are evidence. The product is the evolution of thought.**

Thought Trace should answer:

> **How did I get from what I thought then to what I think now?**

---

## 5. Host-Native Architecture

Thought Trace should use the agent host's existing filesystem and shell capabilities whenever possible.

It should not require users to manually export their sessions.

Typical native sources:

```text
Codex
~/.codex/sessions/

Claude Code
~/.claude/projects/
```

Architecture:

```text
             Thought Trace
                  │
        ┌─────────┴─────────┐
        │                   │
   Codex Source        Claude Source
        │                   │
~/.codex/sessions/   ~/.claude/projects/
        │                   │
        └─────────┬─────────┘
                  ↓
          Session Discovery
                  ↓
          Candidate Selection
                  ↓
           Selective Reading
                  ↓
             Normalize
                  ↓
       Reconstruct Thought Trace
```

The host may use built-in capabilities such as:

- Read
- Glob
- Grep
- filesystem search
- shell commands
- `find`
- `rg`
- `jq`

No dedicated exporter is required for normal operation.

---

## 6. Session Sources

Thought Trace uses a lightweight logical abstraction called **SessionSource**.

Conceptually:

```text
SessionSource
├── codex
└── claude
```

Each source is responsible for:

1. Detecting whether the session store exists
2. Discovering available sessions
3. Reading session metadata
4. Selecting sessions by date/project/topic
5. Reading relevant conversation content
6. Mapping source-specific records into a common representation

A source does not need to copy or permanently import the original session.

### Codex Source

Default location:

```text
~/.codex/sessions/
```

Typical responsibilities:

- Discover session JSONL files
- Extract timestamps
- Identify working directory/repository
- Extract user and assistant conversation
- Ignore irrelevant tool events by default

### Claude Source

Default location:

```text
~/.claude/projects/
```

Typical responsibilities:

- Discover project/session files
- Determine project context
- Extract user and assistant conversation
- Ignore irrelevant tool events by default

Additional sources may be added later without changing Thought Trace's core analysis model.

---

## 7. Core Workflow

```text
User Request
    ↓
Determine Scope
    ↓
Discover Candidate Sessions
    ↓
Select Relevant Sessions
    ↓
Read Relevant Content
    ↓
Normalize
    ↓
Identify Questions / Decisions / Changes
    ↓
Reconstruct Evolution
    ↓
Generate Thought Trace
```

Example:

```text
/trace evolution "LLM Wiki"
```

Thought Trace should first find sessions likely related to `LLM Wiki`, rather than loading every available session.

---

## 8. Selective Reading

Session files may contain large amounts of:

- Tool calls
- Terminal output
- Build logs
- patches
- screenshots
- generated content
- repeated context

Thought Trace should therefore follow **progressive disclosure**.

### Phase 1 — Discovery

Read only enough metadata to identify likely sessions:

```text
timestamp
source
working directory
repository
session identifier
basic topic signals
```

### Phase 2 — Candidate Selection

Select sessions relevant to:

- Date range
- Project
- Topic
- Keyword
- Decision
- User question

### Phase 3 — Focused Reading

Read user prompts and relevant assistant responses.

Tool output should only be included if it materially explains:

- why a decision changed
- what experiment failed
- what implementation result affected the thinking
- what evidence led to a conclusion

### Principle

> **Do not load everything when identifying the right evidence first is possible.**

---

## 9. Common Session Model

Different session formats should map into a minimal common representation.

Example:

```json
{
  "source": "codex",
  "session_id": "...",
  "started_at": "...",
  "updated_at": "...",
  "working_directory": "~/Documents/Projects/company-wiki",
  "project": "company-wiki",
  "messages": [
    {
      "timestamp": "...",
      "role": "user",
      "content": "..."
    },
    {
      "timestamp": "...",
      "role": "assistant",
      "content": "..."
    }
  ]
}
```

The common model should remain intentionally small.

Source-specific details can remain in the original session files.

---

## 10. Core Output

A Thought Trace may contain:

```markdown
# Topic / Project

## Questions

What the user was trying to understand or solve.

## Thought Evolution

How the idea changed over time.

## Decisions

Important conclusions, product decisions, or architecture decisions.

## Alternatives Considered

Other approaches that were explored.

## Changed / Rejected

Ideas that were abandoned, corrected, or replaced.

## Outcomes

What was eventually implemented, documented, or decided.

## Open Questions

Questions that remain unresolved.
```

Not every section is required.

The structure should follow the evidence rather than mechanically filling every section.

---

## 11. Analysis Principles

Thought Trace should prioritize:

1. User questions and hypotheses
2. Changes in direction
3. Explicit decisions
4. Reasons behind decisions
5. Alternatives considered
6. Contradictions or reversals
7. Evidence affecting a decision
8. Unresolved questions
9. Resulting implementation or artifact

It should reconstruct **relationships between thoughts**, rather than merely summarize individual sessions.

---

## 12. Chronology vs Thought Evolution

Thought Trace is not primarily a timeline generator.

Avoid:

```text
10:03 — Asked about RAG.
10:17 — Discussed graph structure.
11:02 — Changed a file.
13:15 — Asked about LLM Wiki.
```

Prefer:

```text
The initial design treated retrieval as a conventional RAG problem.

The discussion then shifted toward representing knowledge as linked,
LLM-readable documents.

This led to the LLM Wiki concept.

A later concern about discovering documents outside a user's existing
wiki introduced the Company Library Index, separating organization-wide
discovery from user-specific knowledge organization.
```

Dates and sessions are supporting evidence.

The **conceptual transition** is the primary artifact.

---

## 13. Commands

Primary skill:

```text
thought-trace
```

Suggested aliases:

```text
/trace
/思考轨迹
```

Examples:

```text
/trace today
/trace this week
/trace project company-wiki
/trace topic "agent orchestration"
/trace evolution "LLM Wiki"
/trace decisions
/trace open-questions
```

Natural-language requests should work equally well:

```text
How has my thinking about Company Wiki evolved?

What architectural decisions did I make about Agent World this month?

Why did I move from traditional RAG toward LLM Wiki?

What questions about Team World remain unresolved?

Compare what I was thinking about this project last month with now.
```

---

## 14. Project Detection

Thought Trace should infer project context from signals including:

- Working directory
- Repository name
- Session metadata
- Files discussed
- User prompts
- Repeated concepts

For example:

```text
~/Documents/Projects/company-wiki
```

strongly suggests:

```text
project: company-wiki
```

Project inference should not depend solely on keywords in conversation.

---

## 15. Topic Detection

A session may contain multiple topics.

Example:

```text
Project:
company-wiki

Topics:
- information architecture
- progressive disclosure
- cloud document access
- personal wiki
- retrieval accuracy
```

Topics should normally be inferred dynamically.

Thought Trace does not require users to manually tag sessions.

---

## 16. Cross-Session Reasoning

The main value of Thought Trace comes from connecting multiple sessions.

It should detect patterns such as:

### Evolution

```text
A → B → C
```

### Refinement

```text
A
├── A1
├── A2
└── A3
```

### Reversal

```text
Initially: A

Later evidence: X

Decision: reject A, adopt B
```

### Recurring concern

```text
Question X appeared in four sessions but remains unresolved.
```

### Convergence

```text
Several separate discussions eventually led to the same architectural principle.
```

---

## 17. Cross-Host Reasoning

Codex and Claude sessions should form one thinking history.

Thought Trace should not treat host boundaries as conceptual boundaries.

Example:

```text
Codex — initial architecture exploration
         ↓
Claude — challenged assumption
         ↓
Codex — implementation experiment
         ↓
Claude — reviewed result
         ↓
Final decision
```

The source should remain available as provenance, but the resulting Thought Trace should describe the unified evolution of the work.

---

## 18. Optional Index

Thought Trace v0.1 should work without requiring a dedicated database or indexing service.

For small histories:

```text
Discover → Filter → Read
```

may be sufficient.

If session volume becomes large, Thought Trace may maintain a lightweight derived index.

Example:

```text
~/.thought-trace/
└── index/
    └── sessions.jsonl
```

Possible record:

```json
{
  "source": "codex",
  "path": "~/.codex/sessions/...",
  "session_id": "...",
  "started_at": "...",
  "updated_at": "...",
  "project": "company-wiki",
  "working_directory": "...",
  "topics": [
    "LLM Wiki",
    "progressive disclosure"
  ]
}
```

The index is an optimization.

It is **not the source of truth**.

---

## 19. Storage

Original sessions should remain where their host stores them.

Thought Trace should not copy raw session history by default.

Optional derived data may live under:

```text
~/.thought-trace/

index/
cache/
traces/
```

For example:

```text
~/.thought-trace/
├── index/
│   └── sessions.jsonl
├── cache/
│   └── normalized/
└── traces/
    ├── projects/
    ├── topics/
    ├── daily/
    └── weekly/
```

All derived data should be disposable and reconstructable from original sessions.

---

## 20. Permissions

Thought Trace depends on the host being allowed to read the native session directories.

Required access may include:

```text
~/.codex/sessions/
~/.claude/projects/
```

If a host cannot access another host's session directory, Thought Trace should:

1. Report the inaccessible source
2. Continue using available sources
3. Explain what permission or directory access is required

It should not fail the whole trace because one source is unavailable.

---

## 21. Exporters

Export is **not part of the core architecture**.

External exporters may still be useful for:

- Portability
- Backups
- Moving history between machines
- Offline archival
- Importing sessions from unsupported hosts

But Thought Trace should not require:

```text
Session → Export → Import → Analyze
```

when it can simply use:

```text
Session → Read → Analyze
```

Exporter support should therefore be deferred until there is a demonstrated need.

---

## 22. Implementation Philosophy

Thought Trace should initially be **LLM-first rather than script-first**.

The host should perform as much of the workflow as practical using its existing capabilities:

```text
Glob
Read
Grep
Shell
Reasoning
```

Avoid introducing custom code solely to reproduce functions already provided by the agent host.

Small scripts may later be introduced for:

- Faster indexing
- Session format normalization
- Very large histories
- Incremental scanning
- Cross-machine synchronization

These are performance optimizations, not prerequisites.

---

## 23. Non-Goals

Thought Trace v0.2 is not:

- A complete chat-history viewer
- A session backup system
- A transcript exporter
- A project management system
- A replacement for Git
- A general enterprise knowledge base
- A monitoring system for employee activity
- A mechanism for exposing hidden model chain-of-thought

It reconstructs the **observable evolution of ideas and work** using session history as evidence.

---

## 24. MVP

Version 0.1 implementation should support:

- Direct reading of Codex sessions
- Direct reading of Claude Code sessions
- Session discovery
- Date filtering
- Project detection
- Topic detection
- Selective session reading
- Common lightweight session model
- Tool-noise filtering
- Cross-session reasoning
- Cross-host reasoning
- Thought evolution reconstruction
- Decision extraction
- Alternative/rejected approach extraction
- Open-question extraction
- Markdown output

No exporter, database, daemon, or background service is required.

---

## 25. Future Capabilities

Possible future capabilities include:

- Incremental session index
- Semantic search across past thinking
- Timeline visualization
- Thought graph visualization
- Compare two time periods
- Detect contradictions and reversals
- Detect recurring unresolved questions
- Link decisions directly to source sessions
- Generate ADRs from decision history
- Generate project retrospectives
- Automatically maintain decision logs
- Connect thoughts to resulting Git commits or files
- Build a personal idea graph
- Cross-device session history
- Integration with Company Wiki or other knowledge systems

---

## 26. Design Principle

Thought Trace should stay lightweight.

The desired user experience is:

```text
Install skill

/trace evolution "Company Wiki"

→ Thought Trace discovers relevant Codex + Claude sessions
→ reads only relevant evidence
→ reconstructs how the idea developed
→ returns the thinking trajectory
```

Not:

```text
Configure database
→ install exporter
→ export sessions
→ import sessions
→ run parser
→ maintain another service
→ finally ask the question
```

**The agent host is already the runt**