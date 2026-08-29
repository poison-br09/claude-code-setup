# Claude Code Development Environment — Architecture

How I run agentic development: a retrieval architecture that keeps model context precise, an agent topology that keeps investigation cheap, and a review discipline that keeps the output small.

**→ [CLAUDE_CODE_SETUP.md](CLAUDE_CODE_SETUP.md)**

## The thesis

The default agentic coding loop wastes context reconstructing structure it should have been handed, and ships redundant code because it does not know what already exists. Those are the same problem. An agent with precise knowledge of a codebase retrieves less *and* writes less — so this environment treats context engineering and output discipline as one system.

## What's inside

| Section | |
|---|---|
| 1–3 | Executive summary, design principles, architecture — the inner and outer loops |
| 4 | **The context engineering method** — question routing, the evidence ladder, claim binding, context isolation, durable knowledge |
| 5–9 | Codebase Memory MCP, the tiered investigation agents, the query skill, lifecycle hooks, Graphify |
| 10 | **CAO** — multi-agent orchestration, scoped durable memory, the learning loop |
| 11 | **Output discipline** — the enforcement chain from intent to durable lesson |
| 12–15 | Skills and plugins, end-to-end workflows, component ownership, security posture |
| 16–18 | Reproducing the setup, repository layout, source map |
| 19–20 | Roadmap and summary |

Every component was verified against the live configuration: settings, hook scripts, agent and skill definitions, plugin manifests, MCP tool surfaces, and executable CLI output.

Paths are portable (`~/.claude/...`). No credentials, identities, machine details, or repository names appear anywhere.
