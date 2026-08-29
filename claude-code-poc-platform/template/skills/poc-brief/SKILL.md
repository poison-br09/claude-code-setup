---
name: poc-brief
description: "Build or refresh docs/ARCHITECTURE.md from the codebase knowledge graph. Triggers on: generate architecture doc, refresh the architecture, document this POC, onboard me to this repo, what does this POC do."
---

# POC Brief — build docs/ARCHITECTURE.md from the graph

Produces the human interface to the codebase: architecture -> component -> source.
Source is the graph, verified against code before writing — never a fresh read-through
of the repo.

## Procedure

1. `list_projects` — confirm indexed; `index_status` — confirm current. Not indexed or
   stale is a gap to report, not to silently fix: `index_repository` is deliberately
   absent from this repo's permission allow list, so re-indexing needs a human.
2. `get_architecture(aspects=["overview"])`, then `structure`, `packages`,
   `entry_points`, `layers`, `boundaries` — orient.
3. `get_architecture(aspects=["clusters"])` — find the de-facto modules. Leiden
   communities are the real seams, often cutting across the folder layout: describe
   the system as it behaves, not as it is filed.
4. `get_architecture(aspects=["hotspots"])`, plus
   `search_graph(min_degree=10, relationship="CALLS", direction="inbound")` for fan-in
   and `direction="outbound"` for fan-out — find the load-bearing code.
5. `get_architecture(aspects=["routes"])` for inbound; `query_graph` Cypher over
   `HTTP_CALLS` for outbound integrations, e.g.:
   ```
   MATCH (a)-[r:HTTP_CALLS]->(b) RETURN a.name, b.name, r.url_path, r.confidence LIMIT 20
   ```
6. `trace_path(direction="both")` from each entry point found in step 2, bounded
   `depth` — the main path for the Data Flow section.
7. `CONFIGURES` edges, via `query_graph` — what sets what, and what consumes it.
8. **Verify before writing.** One batched `check_index_coverage` call over every path
   the document is about to cite, plus `scopes` for any section that claims
   completeness (e.g. "all inbound routes"). For anything reported partial, skipped,
   excluded, stale, pending, or unknown, read or grep that range directly before
   relying on it. Confirm each component's headline claim with one
   `get_code_snippet(qualified_name=...)`.
9. Write, following `docs/ARCHITECTURE.md`'s section order exactly. Every Components
   row carries its source path — the architecture-to-source bridge; keep it, do not
   drop it for prose.
10. Record what could not be verified in the document's own Coverage and Confidence
    section, naming the tool call and the reason (partial, skipped, stale, unknown). A
    gap written down is a working document; a gap hidden is a broken one.

## When to re-run

A component boundary, interface, dependency, or integration changed. Use
`detect_changes()` against the current diff first — if nothing affected shows up,
re-running is unlikely to change the doc.

## What this skill does not do

- It does not judge production readiness. That is `/poc-review`, which writes
  `docs/READINESS.md`.
- It does not invent rationale for a decision. Decisions and their reasoning belong
  in `manage_adr`, referenced from the Design Decisions section, not authored here.
