# Architecture — `<PROJECT_NAME>`

*This document is generated and refreshed by the `poc-brief` skill from the codebase
knowledge graph, then verified against source. Do not hand-author it from a read-through
of the repo — re-run the skill instead. `<PLACEHOLDER>` marks a value the skill fills in.*

## 1. What this POC does, and the problem it solves

*5 lines max. The problem it solves and the approach taken, in plain language.*

<PROBLEM_STATEMENT>
<APPROACH_SUMMARY>

## 2. How to read this document

*Where a new engineer should start: one file and one symbol, not the whole tree.*

Start at `<ENTRY_FILE_PATH>`, symbol `<ENTRY_SYMBOL>`. Then read section 4 (Components)
top to bottom, following each source path.

## 3. Architecture

*A fenced ASCII component diagram of the de-facto modules from `get_architecture`
(`clusters` aspect) — not the folder layout, unless they coincide.*

```
<ASCII_COMPONENT_DIAGRAM>
```

## 4. Components

*The architecture -> source bridge. Every row must resolve to a real path. This is the
most important table in the document.*

| Component | Responsibility | Source path | Key symbols | Notes |
|---|---|---|---|---|
| `<COMPONENT_NAME>` | `<RESPONSIBILITY>` | `<SOURCE_PATH>` | `<KEY_SYMBOLS>` | `<NOTES>` |

## 5. Interfaces

*Inbound routes and outbound calls, each tied to the symbol that handles it.*

**Inbound**

| Route | Method | Handler symbol | Source path |
|---|---|---|---|
| `<ROUTE_PATH>` | `<METHOD>` | `<HANDLER_SYMBOL>` | `<SOURCE_PATH>` |

**Outbound**

| Target | Called from symbol | Source path |
|---|---|---|
| `<EXTERNAL_TARGET>` | `<CALLER_SYMBOL>` | `<SOURCE_PATH>` |

## 6. Data flow

*The main path traced end to end via `trace_path`, as numbered hops with symbols —
not a narrative paragraph.*

1. `<ENTRY_SYMBOL>` receives `<INPUT>` — `<SOURCE_PATH>`
2. `<SYMBOL_2>` — `<SOURCE_PATH>`
3. `<SYMBOL_N>` produces `<OUTPUT>` — `<SOURCE_PATH>`

## 7. Dependencies

*Internal coupling and external services, each with where it is invoked from.*

| Dependency | Type (internal/external) | Called from | Source path |
|---|---|---|---|
| `<DEPENDENCY_NAME>` | `<TYPE>` | `<CALLER_SYMBOL>` | `<SOURCE_PATH>` |

## 8. Configuration

*Settings that change runtime behavior, sourced from `CONFIGURES` edges, each with what
consumes it and its default.*

| Setting | Controls | Consumed by | Default |
|---|---|---|---|
| `<SETTING_NAME>` | `<WHAT_IT_CONTROLS>` | `<CONSUMING_SYMBOL>` | `<DEFAULT_VALUE>` |

## 9. Design decisions

*A pointer, not a duplicate. Full rationale lives in the ADR record.*

See `manage_adr(project="<GRAPH_PROJECT_NAME>", mode="get")` for the full record, stored
at `.codebase-memory/adr.md`. Index:

| ADR | Decision | Status |
|---|---|---|
| `<ADR_ID>` | `<ONE_LINE_DECISION>` | `<STATUS>` |

## 10. Known limitations and assumptions

*Plainly stated, not softened. What this POC does not handle, and what it assumes
about its environment or inputs.*

- <LIMITATION_OR_ASSUMPTION>

## 11. Coverage and confidence

*Every generated document ends here. State the graph generation used, what was
verified against source via `get_code_snippet`, what coverage reported as partial,
skipped, excluded, stale, or unknown, and what remains unverified as a result. This
section assembles evidence — it does not declare the POC production-ready; that
judgement is the reader's, informed by `docs/READINESS.md` from `/poc-review`.*

- Graph generation: `<GENERATION_ID_OR_TIMESTAMP>`
- Verified against source: `<LIST_OF_SPOT_CHECKED_SYMBOLS>`
- Coverage gaps: `<PATH_OR_SCOPE>` — `<REASON: partial/skipped/stale/unknown>`
- Not verified: `<WHAT_REMAINS_UNCONFIRMED>`
