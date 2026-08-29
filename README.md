# Claude Code Solutions

Two engineering documents on running Claude Code as a development platform rather than as an
autocomplete: one describing an environment that exists, one proposing a setup a team can adopt.

| Folder | What it is | Start at |
|---|---|---|
| [`claude-code-setup/`](claude-code-setup/) | Architecture of a working Claude Code environment — retrieval architecture, agent topology, context engineering method, output discipline. A case study, verified against the live configuration. | [`CLAUDE_CODE_SETUP.md`](claude-code-setup/CLAUDE_CODE_SETUP.md) |
| [`claude-code-poc-platform/`](claude-code-poc-platform/) | A reusable setup that makes an AI-built POC readable and evaluable by a second engineer. A proposal plus the templates a team copies into a repo. | [`README.md`](claude-code-poc-platform/README.md) |

## The shared thesis

An agent working in an unfamiliar repository re-discovers the same structure every session,
and the engineer reviewing what it built has no interface into the result except reading all
of it. Both problems have one cause and one fix: a structured, persistent representation of
the codebase, queried selectively and always verified against source.

    structured representation -> targeted retrieval -> source verification

The graph is a retrieval mechanism, not ground truth. Coverage is best-effort, staleness is
real, and neither document claims otherwise — the second one exists precisely so a human can
check the evidence rather than trust the tooling.

## Reading order

Start with [`claude-code-poc-platform/README.md`](claude-code-poc-platform/README.md) if you
want something to adopt: it opens with the problem, the layered setup, worked examples, a
rollout plan, and the costs and limits. Hand
[`ADOPT.md`](claude-code-poc-platform/ADOPT.md) to whoever sets it up.

Read [`claude-code-setup/CLAUDE_CODE_SETUP.md`](claude-code-setup/CLAUDE_CODE_SETUP.md) if you
want the deeper argument for why the environment is shaped this way.
