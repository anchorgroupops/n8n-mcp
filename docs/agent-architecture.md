# n8n-mcp Agent Architecture Reference

> Full detail for the two-phase orchestration system. Referenced by CLAUDE.md via `@docs/agent-architecture.md`.

## The Two-Phase Workflow

### Phase 1: Architecture (n8n-workflow-architect)
First agent to invoke. Designs the complete blueprint:
- Performs all discovery and research
- Selects templates and patterns from the 2,598-template library
- Creates incremental build plan with milestones
- Defines validation checkpoints (every 3–5 nodes)
- Hands off ready-to-build instructions

### Phase 2: Implementation (n8n-workflow-builder)
Second agent to invoke. Builds from architect's blueprint:
- Implements exactly as specified
- Validates every 3–5 nodes (Validation Sandwich: validate_minimal → changes → validate_full)
- Uses `update_partial_workflow` for surgical updates — NEVER full replacement for connection fixes
- Reports progress after each milestone
- Never makes architectural decisions

### Supporting Agents

**n8n-webhook-tester**: Automated webhook testing after webhook-trigger workflows. Creates bash test script in `/tmp/test_webhook_[timestamp].sh`, handles JWT, executes, retrieves n8n execution data, cleans up.

**n8n-workflow-debugger**: Root cause analysis. Analyzes execution history, data flow, error patterns. Does NOT modify — diagnoses only.

## Systems Thinking Framework

Before invoking any agent:
1. **Data Layer** — all sources and destinations
2. **Processing Layer** — transformations, conditions, logic
3. **Control Layer** — error handling, retry strategies

## Requirements Gathering

### New Workflows
- Trigger mechanism and frequency
- All data sources (group by type)
- Processing stages
- All destinations
- Error resilience and retry strategies
- Validation checkpoints

### Debugging
- Exact error + node location
- Last successful execution timestamp
- Execution history pattern (consistent/intermittent)
- Can auto-fix (validate_minimal) resolve it?

### Enhancements
- Which parts must be preserved?
- New capability clusters
- Integration points
- Rollback strategy

## Anti-Patterns to Avoid

| Category | Anti-Pattern |
|---|---|
| Delegation | Rush (before understanding), Vague Request, Direct Call (bypass orchestrator) |
| Building | Big Bang (no incremental), Full Replace, Skip Validation |
| Debugging | Manual Fix, Guess, Ignore Autofix |

## MCP Tool Pattern
1. **Find** — `search_nodes` / `list_nodes`
2. **Configure** — `get_node_essentials` / `get_node_info`
3. **Validate** — `validate_workflow` / `validate_node_operation`

## Key Notes
- When Architect finishes, save the blueprint as an `.md` in `projects/` — present summary to user before invoking Builder
- After each conversation compact, call `tools_documentation` for tool overview
- `ALWAYS` call `tool_documentation` on a specific tool before using it if not in recent memory
- Webhook flows: user must run one test execution and activate workflow before you can test
- Pinned data: test-mode only — does not affect production executions
