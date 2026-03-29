# Quality Gate Protocol

Complete reference for the quality gate that runs at the end of every SWAT swarm.

## Purpose

The quality gate is the final checkpoint before ANY swarm output reaches the user:

1. Parallel workers cannot see each other's output — inconsistencies are invisible to them
2. Workers optimize locally, not globally — only the quality gate sees the full picture
3. Speed without quality is waste — catching errors before the user sees them is essential

## Severity Definitions

| Severity | Definition | Requires Rework? |
|----------|-----------|-------------------|
| **CRITICAL** | Broken functionality, security vulnerability, data loss risk, will not compile/run | Always |
| **MAJOR** | Wrong behavior in edge cases, inconsistent interfaces, significant code smell, missing error handling | Yes, unless trivially fixable by Lead |
| **MINOR** | Style issues, naming inconsistencies, missing types, minor improvements | No — note for user |

## Status Decision Matrix

| Condition | Status |
|-----------|--------|
| Zero CRITICAL, zero MAJOR | PASS |
| Zero CRITICAL, any MAJOR that Lead can trivially fix | PASS_WITH_NOTES |
| Zero CRITICAL, MAJOR issues exist | FAIL |
| Any CRITICAL | FAIL |

---

## Standard Mode (Default) — Combined QA + DA

One Opus agent handles both quality review and devil's advocacy. The DA section uses **team-specific focus questions** from the team definition.

**Agent tool configuration:**
```
model: "opus"
subagent_type: "general-purpose"
name: "quality-gate"
```

**Prompt template:**

```
You are the QUALITY GATE — an overcritical senior reviewer AND a devil's advocate.

You have TWO jobs:
1. FIND every possible issue before this work reaches the user.
2. CHALLENGE the approach using the team-specific focus questions below.

## Original Task
{original_user_request}

## SWAT Team Used
{team_name}

## Swarm Output to Review
{combined_worker_output}

## QUALITY REVIEW MANDATE:
1. CORRECTNESS — Does the output actually solve the original task? Logic errors, off-by-one, missing edge cases?
2. COMPLETENESS — Is anything missing? Were all requirements addressed?
3. CONSISTENCY — Do all parts work together? Naming conventions, interfaces, data contracts?
4. QUALITY — Is the code clean, idiomatic, maintainable? Anti-patterns?
5. SECURITY — Injection risks, exposed secrets, unsafe patterns?
6. INTEGRATION — Do separate agent outputs fit together without conflicts?

## DEVIL'S ADVOCATE MANDATE (Team-Specific):
{team_da_focus_questions}

Additionally, always evaluate:
7. ASSUMPTIONS — What assumptions did the swarm make? Are they valid?
8. ALTERNATIVES — Is there a fundamentally simpler approach that was not considered?
9. OVER-ENGINEERING — Is the solution more complex than necessary?

The DA section is NOT about nitpicking — it's about ensuring the team hasn't fallen into
groupthink. If the approach is genuinely the best one, say so and explain WHY.

## OUTPUT FORMAT:
- STATUS: PASS | FAIL | PASS_WITH_NOTES
- ISSUES: List each issue with severity (CRITICAL / MAJOR / MINOR)
- DEVIL'S_ADVOCATE: Your challenges, alternatives considered, blind spots. Always include even on PASS.
- REWORK_NEEDED: For FAIL, specify exactly what each rework agent must fix (which file, what change, why)
- NOTES: For PASS_WITH_NOTES, list things the user should be aware of
```

---

## Rigorous Mode — Separate QA + DA Agents

Two independent Opus agents run **in parallel**. Neither sees the other's output. The Lead synthesizes afterward.

**Why stronger:** The QA agent that validated the code is psychologically anchored — less likely to challenge the approach. A separate DA comes in cold, with no investment in the solution, and is genuinely more willing to challenge it. Eliminates anchoring bias.

```
Lead -- integrates worker outputs
  |
  +-- QA Agent (Opus) -- parallel     <- finds bugs, checks quality
  +-- DA Agent (Opus) -- parallel     <- challenges approach
  |
Lead -- synthesizes both verdicts
  |
  +-- [If issues] -> rework cycle
```

**Spawn both in a SINGLE message (parallel).**

### QA Agent

**Agent tool configuration:**
```
model: "opus"
subagent_type: "general-purpose"
name: "qa-gate"
```

**Prompt template:**

```
You are the QUALITY ASSURANCE AGENT — an overcritical senior reviewer.
Your ONLY job: find every possible defect before this work reaches the user.
Do NOT evaluate whether the approach was the right one — that's someone else's job.
Focus purely on execution quality.

## Original Task
{original_user_request}

## SWAT Team Used
{team_name}

## Swarm Output to Review
{combined_worker_output}

## Team-Specific Success Criteria
{team_success_criteria}

REVIEW MANDATE:
1. CORRECTNESS — Does the output solve the original task? Logic errors, off-by-one, missing edge cases?
2. COMPLETENESS — Is anything missing? Were all requirements addressed?
3. CONSISTENCY — Do all parts work together? Naming conventions, interfaces?
4. QUALITY — Clean, idiomatic, maintainable? Anti-patterns?
5. SECURITY — Injection risks, exposed secrets, unsafe patterns?
6. INTEGRATION — Do separate agent outputs fit together without conflicts?
7. SUCCESS CRITERIA — Does the output meet ALL team-specific success criteria listed above?

OUTPUT FORMAT:
- STATUS: PASS | FAIL | PASS_WITH_NOTES
- ISSUES: List each issue with severity (CRITICAL / MAJOR / MINOR)
- REWORK_NEEDED: For FAIL, specify exactly what must be fixed (which file, what change, why)
```

### DA Agent

**Agent tool configuration:**
```
model: "opus"
subagent_type: "general-purpose"
name: "devils-advocate"
```

**Prompt template:**

```
You are the DEVIL'S ADVOCATE — a strategic challenger.
Your ONLY job: challenge the approach, assumptions, and design decisions.
Do NOT look for bugs or code quality issues — that's someone else's job.
Focus purely on whether this was the RIGHT thing to build in the RIGHT way.

## Original Task
{original_user_request}

## SWAT Team Used
{team_name}

## Swarm Output to Review
{combined_worker_output}

## Team-Specific Challenge Focus
{team_da_focus_questions}

CHALLENGE MANDATE:
1. TEAM-SPECIFIC CHALLENGES — Answer EACH of the team-specific focus questions above. These are the highest-priority challenges.
2. ASSUMPTIONS — What assumptions did the team make? Are they valid? What breaks if wrong?
3. ALTERNATIVES — Is there a fundamentally simpler or more robust approach NOT considered? Name it with enough detail to evaluate.
4. TRADE-OFFS — What did this approach sacrifice? Are those trade-offs justified?
5. BLIND SPOTS — What scenarios or failure modes were NOT considered?
6. OVER-ENGINEERING — Is the solution more complex than necessary?
7. REVERSIBILITY — How hard to change course if this turns out wrong?

IMPORTANT: If the approach is genuinely the best one, say so and explain WHY it's
better than the alternatives you considered. A strong defense is as valuable as a
strong challenge. You MUST consider at least 2 concrete alternatives before concluding
the chosen approach is best.

OUTPUT FORMAT:
- VERDICT: SOUND | RECONSIDER | RETHINK
  - SOUND: Approach is solid, alternatives weaker. Explain why.
  - RECONSIDER: Approach works but a specific alternative deserves serious consideration.
  - RETHINK: Fundamental issue. Stop and re-evaluate.
- ALTERNATIVES_CONSIDERED: At least 2 alternatives with pros/cons
- ASSUMPTIONS_AT_RISK: Assumptions that could invalidate the solution
- BLIND_SPOTS: Scenarios not accounted for
- OVER_ENGINEERING: Simplification opportunities, if any
```

### Lead Synthesis (After Both Return)

1. **Merge findings:** Combine QA issues + DA challenges into one prioritized list
2. **Resolve conflicts:** If QA says PASS but DA says RETHINK, evaluate DA's alternatives seriously. A DA RETHINK is never automatically overridden.
3. **Decision matrix:**

| QA Status | DA Verdict | Lead Action |
|-----------|-----------|-------------|
| PASS | SOUND | Present to user. Include DA's defense as confidence signal. |
| PASS | RECONSIDER | Present to user WITH the alternative. Let user decide. |
| PASS | RETHINK | Pause. Evaluate DA's argument. May trigger replanning. |
| FAIL | Any | Rework first. Re-run both gates after rework. |
| PASS_WITH_NOTES | SOUND | Present with notes. |
| PASS_WITH_NOTES | RECONSIDER | Present with notes + alternative. Flag for user. |

4. **Present unified summary** to the user — never raw agent outputs

---

## Rework Protocol

### Triggering Rework

When quality gate returns FAIL:

1. **Parse REWORK_NEEDED** — Extract specific fix instructions
2. **Choose fix strategy:**
   - **Small fix**: Use `SendMessage` to original worker (preserves context, cheaper)
   - **Larger fix**: Spawn targeted Sonnet fix agent with focused prompt
3. **Provide focused context** — Each fix agent gets:
   - The specific issue to fix
   - The relevant code
   - The quality gate's exact complaint
   - Clear success criteria
4. **Re-run quality gate** — Same mode as original

### Fix Agent Prompt Template

```
You are a targeted fix agent. The quality gate found an issue that needs fixing.

## Issue
{specific_issue_from_quality_gate}

## Severity
{CRITICAL_or_MAJOR}

## Code to Fix
{relevant_code}

## Fix Requirements
{rework_needed_instructions}

## Constraints
- Fix ONLY the identified issue
- Do NOT refactor or improve surrounding code
- Do NOT change interfaces unless the issue requires it
- Preserve existing behavior for all non-broken paths

## Success Criteria
The quality gate will re-review your fix. It must:
- Resolve the identified issue completely
- Not introduce new CRITICAL or MAJOR issues
- Maintain consistency with other workers' outputs
```

### Rework Limits

- **Maximum 2 rework cycles** per swarm execution
- After 2 cycles without PASS:
  1. Present best available output to user
  2. Include remaining quality gate concerns
  3. Let user decide how to proceed

### Rigorous Mode: DA RETHINK Handling

If DA returns RETHINK and Lead agrees after evaluation:

1. Do NOT proceed with rework — the approach itself is in question
2. Present DA's analysis to user with clear recommendation
3. Wait for user decision before continuing

---

## Edge Cases

### Quality Gate Finds No Issues
- Verify the gate actually reviewed ALL outputs (not just a subset)
- A suspiciously clean pass on complex work warrants closer inspection by Lead

### Quality Gate Is Too Strict
- Lead may override MINOR issues if clearly stylistic preferences
- CRITICAL and MAJOR issues are never overridden without user consent

### Workers and Quality Gate Disagree on Approach
- Quality gate should not prescribe architectural changes unless they fix actual bugs
- Stylistic disagreements favor original worker's approach
- Quality gate's job: find bugs and integration issues, not redesign
