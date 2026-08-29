# Readiness Review — <PROJECT_NAME>

| | |
|---|---|
| Reviewed | <REPO_PATH_OR_NAME> |
| Graph generation | <GENERATION_ID_OR_TIMESTAMP> |
| Tier | <Verify \| Auditor> — `<codebase-memory \| codebase-memory-auditor>` |
| Date | <YYYY-MM-DD> |
| Reviewer | <NAME> |

## Read this first

- <ONE-LINE STATE OF THE SYSTEM TODAY>
- <THE SINGLE BIGGEST OPEN RISK>
- <WHAT TIER THIS REVIEW RAN AT, AND WHAT THAT TIER DOES NOT COVER>
- <WHETHER THE GRAPH GENERATION ABOVE IS CURRENT>

## Findings

Ranked by what a human should look at first.

| # | Area | Finding | Evidence (symbol + path) | What would settle it | Effort |
|---|---|---|---|---|---|
| 1 | <ARCHITECTURE\|IMPLEMENTATION\|RELIABILITY\|SECURITY\|MAINTAINABILITY> | <FINDING> | `<SYMBOL>` — `<PATH>` | <TOOL CALL OR READ THAT WOULD RESOLVE IT> | <S\|M\|L> |
| 2 | <ARCHITECTURE\|IMPLEMENTATION\|RELIABILITY\|SECURITY\|MAINTAINABILITY> | <FINDING> | `<SYMBOL>` — `<PATH>` | <TOOL CALL OR READ THAT WOULD RESOLVE IT> | <S\|M\|L> |
| 3 | <ARCHITECTURE\|IMPLEMENTATION\|RELIABILITY\|SECURITY\|MAINTAINABILITY> | <FINDING> | `<SYMBOL>` — `<PATH>` | <TOOL CALL OR READ THAT WOULD RESOLVE IT> | <S\|M\|L> |

## Architecture

- Checked: <ASPECTS/TOOL CALLS RUN>
- Evidence shows: <CLUSTER/BOUNDARY/LAYER RESULT VS docs/ARCHITECTURE.md>
- Open: <WHAT REMAINS UNRESOLVED>

## Implementation

- Checked: <HOTSPOTS / FAN-IN-OUT / SIMILAR_TO / FILE_CHANGES_WITH RUN>
- Evidence shows: <RESULT>
- Open: <WHAT REMAINS UNRESOLVED>

## Reliability

- Checked: <ENTRY POINTS TRACED; HTTP_CALLS/ASYNC_CALLS INSPECTED; HANDLERS READ>
- Evidence shows: <RESULT>
- Open: error handling is not fully judgeable from the graph — <WHAT WAS OR WAS NOT READ FROM SOURCE>

## Security

- Checked: <AUTH NAME PATTERNS VERIFIED VIA SNIPPETS; CONFIGURES EDGES; ROUTES>
- Evidence shows: <RESULT>
- Open: this is orientation, not a security audit — <WHAT A REAL AUDIT OR /security-review WOULD STILL NEED TO COVER>

## Maintainability

- Checked: <detect_changes + trace_path ON A REPRESENTATIVE CHANGE; DEAD-CODE SCAN>
- Evidence shows: <BLAST RADIUS RESULT>
- Open: <DEAD-CODE CLAIM STATUS — CHECKED AT AUDITOR TIER WITH SCOPES + GREP, OR "NOT CHECKED">

## Production gap

| Capability | Present? | Evidence | What is required |
|---|---|---|---|
| Tests | <Y\|N\|PARTIAL> | <PATH OR "NONE FOUND"> | <GAP> |
| Error handling | <Y\|N\|PARTIAL> | <PATH OR "NONE FOUND"> | <GAP> |
| Observability (logs/metrics/traces) | <Y\|N\|PARTIAL> | <PATH OR "NONE FOUND"> | <GAP> |
| Configuration and secrets | <Y\|N\|PARTIAL> | <PATH OR "NONE FOUND"> | <GAP> |
| Deployment | <Y\|N\|PARTIAL> | <PATH OR "NONE FOUND"> | <GAP> |
| Data migration | <Y\|N\|PARTIAL> | <PATH OR "NONE FOUND"> | <GAP> |
| Access control | <Y\|N\|PARTIAL> | <PATH OR "NONE FOUND"> | <GAP> |

## Assumptions to validate

| Assumption | Who can validate it |
|---|---|
| <ASSUMPTION> | <ROLE OR PERSON> |

## Known POC-quality areas

| Area | Shortcut taken | What upgrading it takes |
|---|---|---|
| <AREA> | <SHORTCUT> | <UPGRADE PATH> |

## Coverage and confidence

- Graph covered: <PATHS/SCOPES CHECKED; check_index_coverage RESULT>
- Source-verified: <WHAT WAS READ DIRECTLY, AND WHY>
- Not checked at all: <EXPLICIT GAPS — NAME THE TOOL CALL OR CHECK NOT RUN>

---

This document is evidence for an engineer's judgement. It is not a
production-readiness verdict.
