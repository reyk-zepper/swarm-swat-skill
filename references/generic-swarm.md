---
name: Generic Swarm
triggers:
  keywords: []
  file_patterns: []
  task_patterns: []
quality_mode: standard
max_workers: 8
---

# Generic Swarm (Fallback)

Deployed when no specialized SWAT team matches the task. Uses dynamic decomposition instead of pre-defined roles.

## When This Fires

- No SWAT team has a clear match for the task
- Task is complex enough to swarm but doesn't fit a specialization
- Unusual or novel task types

## Decomposition Protocol

Unlike SWAT teams, the Generic Swarm has no pre-defined roles. The Lead (Opus, main instance) decomposes dynamically:

1. **Analyze the task** — Identify sub-tasks and their dependencies
2. **Identify parallelism** — Which sub-tasks are independent?
3. **Define contracts** — What does each worker produce? Format?
4. **Choose parallelization strategy:**
   - **File-based**: Workers own specific files
   - **Layer-based**: Workers own architectural layers (data, logic, presentation)
   - **Feature-based**: Workers own feature slices (all layers per feature)
   - **Perspective-based**: Workers analyze from different angles (for research/review)
5. **Scope worker prompts** — Each prompt must be self-contained:

```
## Objective
[Clear, specific description of what to produce]

## Context
[Relevant code, file paths, architectural decisions, original user request]

## Boundaries
[What this worker IS responsible for]
[What this worker is NOT responsible for]

## Constraints
[Technical constraints, style, patterns]

## Expected Output
[Format, where to write, what to return]

## Quality Criteria
[How the quality gate will evaluate this output]
```

## Worker Configuration

All workers use:
- `model: "sonnet"`
- `subagent_type: "general-purpose"` (or `"Explore"` for research, `"code-reviewer"` for review)
- `mode: "bypassPermissions"` for implementation workers
- `isolation: "worktree"` when workers modify overlapping directories

## Quality Gate

- Mode: **standard** (default). Escalate to **rigorous** when:
  - Output is production code going to users
  - Architecture decisions that are hard to reverse
  - Multiple workers touched the same domain
  - User explicitly asks for thorough review

## Devil's Advocate Focus (Generic)

- What assumptions did the swarm make? Are they valid?
- Is there a fundamentally simpler approach not considered?
- What edge cases or failure modes were not thought about?
- Is the solution more complex than necessary?
- What happens under 10x load / in 6 months / when requirements change?

## Success Criteria (Generic)

- All parts of the original task are addressed
- Worker outputs integrate without conflicts
- Code compiles/runs (for implementation tasks)
- No obvious security issues or anti-patterns
- Output is consistent in style and naming
