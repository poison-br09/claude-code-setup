# ADOPT — team runbook

Follow in order. Each step is verifiable before you move to the next.

## 1. Prerequisites

- Claude Code CLI installed and on `PATH`.
- The Codebase Memory MCP binary at `~/.local/bin/codebase-memory-mcp`, installed
  and registered as MCP server **`codebase-memory-mcp`** (Step 1 does this).

If Step 1 has already run on your machine (check with `claude mcp list`), skip to
Step 2 — every step after that is unchanged either way.

## Step 1 — install the shared layer (once per machine)

One command installs everything. Preview first, then accept:

```bash
codebase-memory-mcp install --dry-run   # prints exactly what it would touch
codebase-memory-mcp install -y          # accept; add --force to overwrite an existing install
```

It detects Claude Code and writes:

| Item | What lands |
|---|---|
| Agents | `~/.claude/agents/codebase-memory-scout.md`, `codebase-memory.md`, `codebase-memory-auditor.md` |
| Skill | `~/.claude/skills/codebase-memory/SKILL.md` |
| MCP registration | server `codebase-memory-mcp` registered in `~/.claude.json` |
| Hooks | PreToolUse Grep/Glob search augmentation + PostToolUse Read coverage (non-blocking); SessionStart (MCP usage reminder on startup/resume/clear/compact); SubagentStart (MCP usage reminder for subagents) |

No team hand-copies agent, skill or hook files, and no one hand-edits settings to
register the hooks — the installer owns all of it. Every hook it registers is
non-blocking: it never fails a tool call.

Defaults (`codebase-memory-mcp config list`): `auto_index=false` — nothing is
indexed until you run Step 2; `auto_watch=true` — once indexed, a project
refreshes in the background; `ui_enabled=true` on `ui_port=9749`, loopback only,
not exposed beyond localhost. Indexes live under
`~/.cache/codebase-memory-mcp/<slug>.db`.

Confirm registration:

```bash
claude mcp list   # confirm codebase-memory-mcp is listed
```

## Step 2 — index the POC repo

Inside a Claude Code session opened at the repo root, ask the agent to call the
`index_repository` MCP tool:

Tool: `index_repository`
```json
{ "repo_path": "/absolute/path/to/poc-repo", "name": "poc-repo", "mode": "moderate", "persistence": true }
```

Set `name` explicitly. Without it, the project defaults to a path-derived slug
(e.g. a repo at `/home/<user>/Projects/bot_platform` becomes
`home-<user>-Projects-bot_platform`), which is easy to guess wrong on your first
`index_status` call. Use this same value for `<GRAPH_PROJECT_NAME>` in
`CLAUDE.md` in Step 3.

Modes trade completeness for time: `fast` is a shallow pass for quick iteration,
`moderate` is the default balance, `full` is deepest and slowest, and
`cross-repo-intelligence` additionally builds `CROSS_HTTP_CALLS`,
`CROSS_ASYNC_CALLS`, `CROSS_CHANNEL` edges when the POC spans repos. Actual
runtime depends on repo size — there is no fixed number.

`persistence: true` writes `.codebase-memory/graph.db.zst`. **Commit this file.**
The tool's own message says to, and it writes a `graph.db.zst binary merge=ours`
entry to `.gitattributes` — that gitattribute is what makes committing safe: the
blob can never produce a merge conflict. The cost is that the blob grows in git
history on every refresh, so refresh on a cadence — per release, or when the
graph has drifted enough to matter — not on every commit.

**A teammate joining an already-indexed repo:** clone, then run the same
`index_repository` call. The committed artifact is used to bootstrap instead of a
full extraction, so it costs a fraction of the first index. There is no consume
command — the mechanism is automatic. Confirm with `index_status`.

Read the response's `skipped` and `parse_partial` lists — do not assume full
coverage just because indexing finished. Verify with:

Tool: `index_status` — `{ "project": "poc-repo" }`
Tool: `list_projects` — `{}`

Confirm the project shows a ready state and appears in the project list before
moving on. The same tools also run from a shell or CI, without a conversation:
`codebase-memory-mcp cli [--progress] [--json] <tool> [args]` — this is what
makes the verification checklist below scriptable.

## Step 3 — drop `template/` into the repo

```bash
cp claude-code-poc-platform/template/CLAUDE.md         ./CLAUDE.md
mkdir -p .claude/skills docs
cp claude-code-poc-platform/template/settings.json      .claude/settings.json
cp -r claude-code-poc-platform/template/skills/*        .claude/skills/
cp -r claude-code-poc-platform/template/docs/*          docs/
```

`template/settings.json` registers no hooks: the Step 1 hooks are already
user-level and firing for every session, and duplicating them here would fire
each reminder twice.

Its allow list is 11 MCP tools, deliberately read-only — not the full set the
repo skills touch. The four writing or destructive tools — `index_repository`,
`delete_project`, `manage_adr`, `ingest_traces` — are excluded on purpose so each
surfaces a permission prompt instead of running unattended; the skills still use
`manage_adr`, with a human in the loop. The three investigation agents are
read-only by tool allow-list plus `permissionMode: plan`, not by instruction. For
a narrower surface than the full 15, run the MCP server with `--tool-profile=scout`.

Fill in `CLAUDE.md`'s placeholders by hand (project name, repo path, owning
team, and `<GRAPH_PROJECT_NAME>` — the same `name` you gave `index_repository`
in Step 2). Leave `docs/ARCHITECTURE.md` and `docs/READINESS.md` alone: they
are generated by `/poc-brief` and `/poc-review` in Step 4, and
`template/CLAUDE.md` says never to hand-edit them.

## Step 4 — first run

```
/poc-brief
```
Builds `docs/ARCHITECTURE.md` from the graph (architecture → component →
source). Good output cites specific paths and functions, not generalities, and
flags anything `check_index_coverage` couldn't confirm. Small repos: a few
minutes. Scales with repo size and the mode chosen in Step 2.

```
/poc-review
```
Runs the evaluation workflow into `docs/READINESS.md`. Good output is evidence
for a human decision, not a verdict — this platform never declares a POC
production-ready.

## Verification checklist

Every row below can also be checked from a shell or CI via
`codebase-memory-mcp cli [--progress] [--json] <tool> [args]`.

| Check | How to confirm | Pass looks like |
|---|---|---|
| MCP tools reachable | `claude mcp list`, or ask the agent to call `list_projects` | server shown connected; tool call returns |
| Project ready | `index_status` for the project | status field shows the repo as ready, not indexing/error |
| Hooks fire at session start | start a new session, watch for the reminder | session-start reminder appears once |
| An agent tier answers | ask a structural question in chat (routes to `codebase-memory`) | reply cites graph evidence + coverage check, or falls back to source |
| Docs generated | `docs/ARCHITECTURE.md`, `docs/READINESS.md` exist and are non-empty | both files have repo-specific content, not placeholders |
| Coverage check returns | ask the agent to call `check_index_coverage` on a doc's cited paths | response returned, read even if it says `parse_partial` |

## Keeping it honest

A stale graph still answers — fluently and wrongly. That is the failure mode to
design against, not an edge case.

- `auto_watch=true` means an indexed project refreshes in the background as
  files change — the baseline freshness mechanism, not a substitute for judgment.
- Re-index explicitly (Step 2) after any significant merge, and after
  upgrading `codebase-memory-mcp` (`update` picks up that version's extraction
  improvements — background watching will not backfill those on its own).
- Before starting new work, call `detect_changes` to see which symbols the
  current working diff touches, so you know what the graph has not seen yet.
- Any claim the graph makes about code changed since the last index is
  re-verified against source before it goes in a doc or a decision.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| MCP tools absent from the agent | server not registered or Claude Code not restarted | re-run `codebase-memory-mcp install --force`, restart the session |
| Project not in `list_projects` | `index_repository` never completed or targeted the wrong path | re-run Step 2 with the correct `repo_path` |
| Empty graph for a language | parser has no/partial support for that language | check `skipped`/`parse_partial` in the index response; read that code directly |
| Coverage reports `parse_partial` | files exist but couldn't be fully parsed | read or grep the reported paths before trusting graph claims there |
| Agent claims something is absent or exhaustive | wrong tier for that claim — only Auditor may claim absence | escalate to `codebase-memory-auditor`, or read source directly |
| Hooks not firing | install was skipped, partial, or later overwritten | `codebase-memory-mcp install --dry-run` to see what's missing, then `install --force` |

## Day-2 habits

- Quick provisional question -> Scout. Anything that ends up in a doc or a
  decision -> Verify (default). Negative or exhaustive claims -> Auditor only.
- Escalate to Auditor when Verify's coverage check comes back partial/skipped/
  stale and the claim matters.
- Re-run `/poc-brief` after any significant merge, before trusting
  `docs/ARCHITECTURE.md` again.
- ADRs live with the graph — read or update via the `manage_adr` tool, not as
  loose files in `docs/`.
- Source always wins over the graph. The graph is retrieval, not ground truth.
