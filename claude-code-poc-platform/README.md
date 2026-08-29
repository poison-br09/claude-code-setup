# claude-code-poc-platform

A reusable Claude Code setup that makes an AI-built POC readable and evaluable by a second
engineer — and cheaper for the agent to work in.

```text
claude-code-poc-platform/
├── README.md                      this file — the proposal, read end to end
├── ADOPT.md                       hand this to the team
└── template/                      copied into each POC repo
    ├── CLAUDE.md                  routing policy, evidence rules, doc contract
    ├── settings.json              project tool permissions
    ├── skills/
    │   ├── poc-brief/SKILL.md     build/refresh docs/ARCHITECTURE.md from the graph
    │   └── poc-review/SKILL.md    the evaluation workflow -> docs/READINESS.md
    └── docs/
        ├── ARCHITECTURE.md        human interface: architecture -> component -> source
        └── READINESS.md           evidence behind the production-readiness judgement
```

## 1. The problem

A POC built by an agent becomes a black box the moment the original author moves on.
The agent re-greps the same files every session and burns context on re-discovery.
The reviewing engineer has no interface into the repo except reading all of it.
Both need the same thing: a structured, persistent representation of the codebase.
One representation serves both: `structured representation -> targeted retrieval -> source verification`.

## 2. What we are proposing

Claude Code used as a developer platform, with one responsibility per mechanism.

```text
Claude Code
     │
     ├── Project Instructions ─── template/CLAUDE.md          (NEW, per repo)
     │       └── routing policy, evidence rules, doc contract
     │
     ├── Skills ───────────────── /poc-brief, /poc-review      (NEW, per repo)
     │       └── named workflows a human invokes
     │
     ├── Agents ───────────────── scout / verify / auditor      (installed, shared)
     │       └── investigation at a chosen depth, isolated context
     │
     ├── Hooks ────────────────── session, discovery, subagent  (installed, shared)
     │       └── lifecycle nudges + search augmentation, fail-open
     │
     ├── MCP ──────────────────── codebase-memory-mcp           (installed, shared)
     │       └── 15 tools for structured retrieval
     │
     ├── Knowledge Graph ──────── .codebase-memory/graph.db.zst (per repo, shared artifact)
     │       └── symbols, calls, imports, data flows, HTTP edges
     │
     └── Documentation ────────── docs/ARCHITECTURE.md, docs/READINESS.md (NEW, per repo)
             └── the human-readable interface to the same graph
```

| Mechanism | Owns | Does NOT own |
|---|---|---|
| `CLAUDE.md` | routing policy, evidence rules, doc contract | workflows, code knowledge |
| Skills | reusable named workflows a human invokes | enforcement |
| Agents | investigation at a chosen depth, isolated context | writing code or docs |
| Hooks | lifecycle nudges, search augmentation, fail-open | freshness — that is `auto_watch` — and blocking work |
| MCP | structured retrieval over the graph | judgement |
| Graph | relationships, cited, coverage-checked | ground truth |
| `docs/` | the human interface to the codebase | duplicating source |

## 3. What already exists vs what we add

Installed once per machine by one command. `codebase-memory-mcp install` (v0.10.8) detects the
coding agents present and writes the three agents, the skill, the MCP registration in
`~/.claude.json` and all three hook registrations — nothing is hand-copied and no settings file
is hand-edited. `install --dry-run` prints exactly what it would touch. Nothing is indexed
without the team asking (`auto_index=false`); once a project is indexed it refreshes in the
background (`auto_watch=true`). All of it is shared by every repo on the machine:

| Piece | Path |
|---|---|
| Codebase Memory MCP (indexer + graph server, 15 tools) | `~/.local/bin/codebase-memory-mcp` |
| Skill: query patterns, tool decision matrix, evidence tiers | `~/.claude/skills/codebase-memory/SKILL.md` |
| Agent Tier 1 — fast provisional lookup | `~/.claude/agents/codebase-memory-scout.md` |
| Agent Tier 2 — task-directed verification (default) | `~/.claude/agents/codebase-memory.md` |
| Agent Tier 3 — bounded-scope full audit | `~/.claude/agents/codebase-memory-auditor.md` |
| Hook: SessionStart (startup, resume, clear, compact) | `~/.claude/hooks/cbm-session-reminder` |
| Hook: PreToolUse `Grep\|Glob`, PostToolUse `Read` | `~/.claude/hooks/cbm-code-discovery-gate` |
| Hook: SubagentStart `*` | `~/.claude/hooks/cbm-subagent-reminder` |

All three agents are read-only, enforced by tool allow-list **and** `permissionMode: plan` —
not by instruction alone. That is what makes it safe to let them roam an unfamiliar repo.

What this proposal adds, per repo:

| New artifact | Owns |
|---|---|
| [`template/CLAUDE.md`](template/CLAUDE.md) | routing policy + evidence rules + the doc contract |
| [`template/settings.json`](template/settings.json) | project tool permissions — the installer owns the hooks |
| [`template/skills/poc-brief/SKILL.md`](template/skills/poc-brief/SKILL.md) | build/refresh `docs/ARCHITECTURE.md` from the graph |
| [`template/skills/poc-review/SKILL.md`](template/skills/poc-review/SKILL.md) | the evaluation workflow -> `docs/READINESS.md` |
| [`template/docs/ARCHITECTURE.md`](template/docs/ARCHITECTURE.md) | architecture -> component -> source |
| [`template/docs/READINESS.md`](template/docs/READINESS.md) | evidence for the readiness judgement |

The installed layer answers the *agent's* questions. It produces no human-readable interface
and no readiness judgement. `template/` is that missing half — most of the machinery is
already there and shared, which is why adoption is one install command per machine, a copy,
and one index run — not a project.

## 4. How it behaves in practice

**"How does authentication work here?"** — Scout tier.
`search_graph` for auth-named symbols, `trace_path` inbound on the handler it finds, one
`get_code_snippet` on the entry point, one batched `check_index_coverage` on those paths.
~3-4 graph calls, no repo-wide grep. The answer is explicitly **provisional and positive only**:
Scout may say "this is where auth happens", never "this is the only place auth happens".
Escalate to Verify if the answer will be acted on.

**"What breaks if I change this API?"** — Verify tier (default).
`detect_changes()` maps the working diff to the affected symbols. `trace_path` runs
`direction=both` on each, paginating fully. Every material claim is backed by an exact
`get_code_snippet`. Then one batched `check_index_coverage` over every cited path, plus the
scopes behind any "nothing else calls this" claim. Partial, skipped, stale or unknown coverage
sends the agent to read or grep the reported ranges before it answers.

**"Is this POC ready to productionize?"** — `/poc-review`, at Verify tier for an interim
check while the POC is still moving, at Auditor tier for a sign-off review.
`get_architecture` with `boundaries`, `layers`, `hotspots` and `clusters` (Leiden communities
expose de-facto modules that cut across the folder layout), `search_graph` with `min_degree`
for high-fan-out components, `trace_path` on failure and external-call paths. Output is
`docs/READINESS.md`: production-shaped, POC-shaped and unverified, each with a citation.
It returns no verdict — an engineer reads the evidence and decides.

## 5. The lifecycle it paves

| Step | What the platform contributes |
|---|---|
| Understand | `docs/ARCHITECTURE.md` plus `get_architecture` overview — orientation without reading the repo |
| Explore | Scout answers cheap structural questions in ~3-4 calls instead of a grep sweep |
| Develop | `CLAUDE.md` routes the agent to the graph first, so context goes to the task, not re-discovery |
| Review | `detect_changes` + `trace_path` both directions turns "what did this touch" into a cited list |
| Evaluate | `/poc-review` assembles the readiness evidence — Verify tier for an interim check, Auditor for sign-off |
| Document | `/poc-brief` regenerates `docs/ARCHITECTURE.md` from the graph, so docs track the code |
| Productionize | `READINESS.md` gives the next team a gap list with source citations, not a blank page |

## 6. Rollout

| Phase | Scope | Time | Exit test |
|---|---|---|---|
| 1 | `codebase-memory-mcp install` on the pilot machine, then one repo: copy `template/`, run `index_repository(persistence=true)`, run `/poc-brief` | Half a day | An engineer new to the repo answers "what does this do and where do I start" from `docs/ARCHITECTURE.md` alone |
| 2 | Two more repos + hold the doc contract in `CLAUDE.md` on real PRs | About a week | `/poc-review` produces a `READINESS.md` a reviewer actually uses in a real review |
| 3 | Standard for new POCs; graph artifact committed and bootstrapped by teammates | Ongoing | A teammate clones and queries without a full re-extraction |

Measure four things, all cheap: time for a new engineer to first useful question; whether
review comments cite `READINESS.md`; how often the agent falls back to repo-wide grep;
and how often the graph is stale at the moment someone asks it something.

## 7. Cost, limits and honest risks

| Risk | Reality |
|---|---|
| Stale graph | **The main failure mode.** A stale graph is worse than no graph: it answers fluently and wrongly. `auto_watch=true` refreshes indexed projects in the background, hooks nudge on session start and `detect_changes` catches the working diff — none of the three is a guarantee. Re-index after large external merges and after a version upgrade. |
| Coverage is not proof | `check_index_coverage` returning `indexed_no_recorded_gap` means *no recorded gap*, not completeness. `index_repository` reports `skipped` and `parse_partial` files. Negative and exhaustive claims always need a source read. |
| Language variance | Graph quality depends on the parser. Expect weaker edges in less-supported languages; the fallback is source verification, which still works, just costs more context. |
| Indexing time and disk | Both scale with repo size and mode (`full`/`moderate`/`fast`). Measure on the pilot repo rather than trusting a number from this document; indexes live under `~/.cache/codebase-memory-mcp/`. |
| Artifact distribution | `persistence=true` writes `.codebase-memory/graph.db.zst`. Commit it — the tool's own message says to, and it writes a `graph.db.zst binary merge=ours` entry to `.gitattributes` so the blob can never produce a merge conflict. The cost is real: the blob grows in git history on every refresh. Mitigate by refreshing on a cadence — per release, or when the graph has drifted enough to matter — not on every commit. A team that will not carry a binary in git can publish it from CI or a shared store instead, at the cost of a pipeline step. |
| Docs drift | `docs/` is regenerated by `/poc-brief`, not hand-maintained. If nobody runs it, it rots like any other doc. The doc contract in `CLAUDE.md` is the only thing holding this. |
| Readiness | **The platform never declares a POC production-ready.** It assembles the evidence for an engineer's judgement. Anything that reads like a verdict is a bug in the output. |

Adoption cost is low, but it is not zero: someone must own the pilot, and the
first `READINESS.md` will need a human edit before it is trustworthy.

**Deliberately out of scope: autonomous and iterative agent workflows.** This proposal
does not add unattended loops. What it is worth is evidence a human can check, and a loop
running unattended over a graph that can be stale multiplies unverified output faster than
anyone reads it. Two things would have to exist first: a freshness guarantee stronger than
`auto_watch`, and a verification gate on each iteration's output that a human still signs.

## 8. What to tell the team

- Copy `template/` into the POC repo and index it once with `persistence=true`.
- Commit the graph artifact so nobody else pays the indexing cost; refresh it on a cadence,
  not on every commit.
- Run `/poc-brief` before you ask anyone to review the POC.
- Run `/poc-review` before you claim any part of it is production-shaped.
- Treat every graph answer as a lead, not a fact — cited or read from source, or say you are unsure.
- The step-by-step is in [`ADOPT.md`](ADOPT.md).
