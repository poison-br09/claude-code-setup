---
name: poc-review
description: "Assemble evidence for a POC readiness judgement from the codebase knowledge graph and source. Triggers on: evaluate this POC, is this production ready, review this POC before I sign off, what would it take to productionize, technical debt review."
---

# POC Review — assemble evidence for docs/READINESS.md

**This skill assembles the evidence an engineer needs to make the readiness
judgement. It does not make the judgement.** Every finding must be discoverable
and independently checkable by the human reading it: a symbol, a path, a tool
call they can re-run.

Starting map is `docs/ARCHITECTURE.md` (built and owned by `poc-brief`). This
skill checks the system against that map — it never regenerates it.

## 0. Choose tier and cost

| Use case | Tier | Agent | Cost | What it buys |
|---|---|---|---|---|
| Interim check, still building | Verify | `codebase-memory` | Task-directed graph calls; one pagination pass per query, not forced to exhaustion; `get_code_snippet` only for the claims under review; coverage scopes only where a claim needs one; full pass on the dimensions in scope, not necessarily all six | Backs material claims about what was actually looked at |
| Sign-off review before a decision | Auditor | `codebase-memory-auditor` | Bounded but complete: every relevant page exhausted, both trace directions on touched symbols, a snippet for every material finding, coverage scopes for every exhaustive/negative claim, full pass on all six dimensions | The only tier that may carry absence and exhaustive claims (dead code, "no other caller") — what a sign-off needs |

Wall-clock time scales with repository size and index freshness, not tier
choice alone — the columns above are what actually varies between tiers.
Scout is deliberately excluded from this table: it may never make the
absence or exhaustive claims a readiness review depends on, so it cannot
back a finding here at either tier.

Record which tier ran, in the document header. A Verify-tier review presented as
complete is a false claim — the tier is a promise about coverage, not a label.

## 1. Confirm freshness

`list_projects` / `index_status`. Record the graph generation used — a finding
tied to a stale generation must say so.

## 2. Architecture

- `get_architecture(aspects=["clusters","boundaries","layers"])` — do the Leiden
  clusters line up with the components documented in `docs/ARCHITECTURE.md`?
  Divergence is the finding, not a defect to smooth over.
- `get_architecture(aspects=["cycles"])` — opt-in, scans for circular CALLS SCCs.

## 3. Implementation

- `get_architecture(aspects=["hotspots"])`.
- `search_graph(min_degree=10, direction="inbound")` and `direction="outbound"`
  for high fan-in / fan-out.
- `SIMILAR_TO` edges (via `query_graph`) for duplication.
- `FILE_CHANGES_WITH` edges for coupling the folder layout hides.

## 4. Reliability

- `trace_path` outward from every entry point, both directions, to find failure
  paths. `HTTP_CALLS` / `ASYNC_CALLS` edges are the dependency-failure surface.
- Read the actual handlers with `get_code_snippet` — **error handling cannot be
  judged from the graph alone.** State this in the document.

## 5. Security

- `search_graph(name_pattern=...)` for auth-related names, verify each hit with
  `get_code_snippet`.
- `CONFIGURES` edges for secrets and configuration.
- `get_architecture(aspects=["routes"])` for the external interface.
- **State the limit plainly:** this is orientation, not a security audit. It
  does not replace `/security-review` or a human reviewer.

## 6. Maintainability

- `detect_changes()` against a representative change, then
  `trace_path(direction="both", risk_labels=true)` for blast radius.
- Dead code: `search_graph(max_degree=0, exclude_entry_points=true)`. This is an
  absence claim — it requires Auditor tier, `check_index_coverage` with
  `scopes` over the relevant paths, and a confirming grep. Without all three
  it is not a claim: write "not checked," never "no dead code."

## 7. Production readiness

Frame every item as a question with evidence attached, never as a verdict:
what is missing, what is assumed, what is owed. Feeds the Production gap table
in the template.

## 8. Evidence rules

- One batched `check_index_coverage` over every path cited, with `scopes`
  added for every exhaustive or negative claim (dead code, "no other caller",
  "fully covers").
- `indexed_no_recorded_gap` means no recorded gap, not completeness.
- Partial, skipped, excluded, stale, pending, or unknown coverage: read or
  grep the reported range before writing the finding down.
- Every finding cites a qualified symbol and file path so the reader can
  re-derive it without trusting this document.

## 9. Write docs/READINESS.md

Follow the template's section order exactly. Rank findings by what a human
should look at first. Every finding states what evidence would settle it if
the graph or a quick read left it open.

## 10. What this skill does not do

- No verdict, no automated pass/fail gate.
- No substitute for tests, benchmarks, threat modelling, or an operational
  review.
- No hand-editing `docs/READINESS.md` outside this skill's output —
  regenerate, don't patch.
