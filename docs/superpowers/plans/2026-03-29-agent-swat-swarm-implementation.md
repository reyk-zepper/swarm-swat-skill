# Agent-SWAT-Swarm Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a specialized SWAT-Team swarm skill for Claude Code that replaces the generic `agent-swarm` with pre-configured, task-optimized agent teams.

**Architecture:** A single Claude Code skill (`agent-swat-swarm`) with a lean dispatcher (SKILL.md) that selects and orchestrates specialized teams defined in `references/teams/*.md`. Team definitions are loaded on-demand to minimize context usage. Quality gate with team-specific Devil's Advocate criteria runs after every swarm.

**Tech Stack:** Claude Code Skills (Markdown), Agent Tool, SendMessage, Read Tool

**Design Document:** `docs/plans/2026-03-29-agent-swat-swarm-design.md`

---

## File Structure

```
/Users/reykz/repositorys/swarm-swat-skill/     (repo root = skill root for GitHub)
├── SKILL.md                                     ← Dispatcher + orchestration
├── references/
│   ├── teams/
│   │   ├── security-audit.md                    ← Security Audit SWAT Team
│   │   ├── frontend-feature.md                  ← Frontend Feature SWAT Team
│   │   ├── backend-api.md                       ← Backend API SWAT Team
│   │   ├── migration-refactor.md                ← Migration/Refactor SWAT Team
│   │   ├── full-stack-feature.md                ← Full-Stack Feature SWAT Team
│   │   └── team-builder.md                      ← SWAT Team Builder (Meta-Team)
│   ├── quality-gate-protocol.md                 ← QG spec with team-specific DA
│   └── generic-swarm.md                         ← Fallback for unmatched tasks
├── docs/
│   ├── plans/
│   │   └── 2026-03-29-agent-swat-swarm-design.md
│   └── superpowers/
│       └── plans/
│           └── 2026-03-29-agent-swat-swarm-implementation.md (this file)
```

**Installation:** Clone repo, then symlink or copy to `~/.claude/skills/agent-swat-swarm/`:
```bash
ln -s /Users/reykz/repositorys/swarm-swat-skill ~/.claude/skills/agent-swat-swarm
```

---

### Task 1: Directory Structure + Git Init

**Files:**
- Create: directory tree as shown above

- [ ] **Step 1: Create directories**

```bash
mkdir -p /Users/reykz/repositorys/swarm-swat-skill/references/teams
```

- [ ] **Step 2: Initialize git repo**

```bash
cd /Users/reykz/repositorys/swarm-swat-skill
git init
```

- [ ] **Step 3: Create .gitignore**

Write to `/Users/reykz/repositorys/swarm-swat-skill/.gitignore`:

```
.DS_Store
*.swp
*.swo
*~
```

- [ ] **Step 4: Commit**

```bash
cd /Users/reykz/repositorys/swarm-swat-skill
git add .gitignore
git commit -m "chore: init repo with directory structure"
```

---

### Task 2: SKILL.md (Dispatcher)

The core of the system. Must be lean — only dispatcher logic, team index, and orchestration protocol. No team details.

**Files:**
- Create: `SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/SKILL.md`:

```markdown
---
name: agent-swat-swarm
description: Spezialisierte SWAT-Team Swarm-Orchestrierung. Bildet aufgabenspezifische Agent-Teams mit voroptimierte Prompts, Output-Contracts und Workflow-Graphs. Triggers proaktiv bei komplexen Implementierungen, Multi-File-Aenderungen, Security Audits, Frontend Features, Backend APIs, Migrationen, Full-Stack Features, oder jeder Aufgabe die von Parallelisierung und Spezialisierung profitiert. User Override: /agent-swat-swarm <team-name> oder 'nutze [Team-Name] Team'.
---

# Agent SWAT Swarm

Spezialisierte Agent-Teams fuer spezifische Aufgabentypen. Jedes SWAT-Team hat vorkonfigurierte Rollen mit optimierten Prompts, Output-Contracts und einem aufgabenspezifischen Workflow.

## Core Principle

Statt bei jeder komplexen Aufgabe von Grund auf zu dekomponieren, wird ein spezialisiertes SWAT-Team deployed das fuer genau diesen Aufgabentyp optimiert ist. Vorkonfigurierte Prompts und Workflows eliminieren Improvisations-Overhead und erhoehen Output-Qualitaet.

## Model Assignment (Non-Negotiable)

| Role | Model | Rationale |
|------|-------|-----------|
| **Dispatcher (Lead)** | `opus` (Hauptinstanz) | Strategische Entscheidungen, Synthese, Integration |
| **Worker Agents** | `sonnet` | Schnelle parallele Ausfuehrung, kosteneffizient |
| **Quality Gate** | `opus` | Kritische Bewertung, Fehler-Erkennung |
| **Devil's Advocate** | `opus` (standard: im QG integriert, rigorous: separater Agent) | Domaenenspezifische Challenges |

## When to Form a SWAT Swarm

### Automatic Triggers (Form Without Asking)

Form a SWAT swarm proactively when ANY of these conditions are met:

1. **Multi-file implementation** — Task touches 4+ files or 3+ distinct modules
2. **Research + implement** — Task requires both deep research AND implementation
3. **Multi-domain task** — Task spans frontend + backend, or code + infrastructure
4. **Large refactor** — Structural changes across a codebase
5. **Comprehensive review** — Full codebase audit, security review, architecture assessment
6. **Parallel-independent subtasks** — Task naturally decomposes into 3+ independent work streams
7. **Quality-critical deliverable** — User explicitly needs production-grade or error-free output

### Do NOT Swarm When

- Task is simple, single-file, or trivially sequential
- User explicitly asks for a quick/simple approach
- Task requires tight iterative feedback with the user
- Coordination overhead exceeds the benefit (fewer than 3 independent sub-tasks)

## Team Index

| Team | Datei | Einsatz |
|------|-------|---------|
| **Security Audit** | `references/teams/security-audit.md` | Vulnerability-Scan, OWASP, Penetration-Review, Dependency-Audit |
| **Frontend Feature** | `references/teams/frontend-feature.md` | UI-Komponenten, Design-System, A11y, Responsive, Performance |
| **Backend API** | `references/teams/backend-api.md` | Endpoints, DB-Schema, Business-Logic, Test-Coverage |
| **Migration/Refactor** | `references/teams/migration-refactor.md` | Version-Upgrades, Pattern-Migration, Breaking Changes, Deprecations |
| **Full-Stack Feature** | `references/teams/full-stack-feature.md` | End-to-end Features ueber Frontend + Backend + Tests |
| **SWAT Team Builder** | `references/teams/team-builder.md` | Analysiert Use Cases und generiert neue Team-Definitionen |
| **Generic Fallback** | `references/generic-swarm.md` | Alle Tasks die kein spezialisiertes Team matchen |

## Team-Selektion (Hybrid)

### 1. User-Override pruefen

- **Slash-Command**: `/agent-swat-swarm <team-name>` → Team erzwungen
  - Gueltige Namen: `security-audit`, `frontend-feature`, `backend-api`, `migration-refactor`, `full-stack-feature`, `team-builder`
- **Keyword im Prompt**: "nutze Security Team", "mit Frontend SWAT", "deploy Backend Team" → Team erzwungen
- Bei Override: direkt zu Phase 1 des Orchestrierungsprotokolls

### 2. Automatisches Matching (kein Override)

Lies die User-Aufgabe und entscheide welches Team am besten passt:

- Verstehe den **Aufgabentyp**: Build, Review, Migrate, Fix, Audit, Analyse
- Beruecksichtige **betroffene Dateien/Domaenen**: Frontend-Dateien (.tsx, .css, Komponenten), Backend-Dateien (.ts routes, DB, API), Security-relevante Dateien (auth, middleware, .env), Config/Dependency-Dateien (package.json, tsconfig)
- Beruecksichtige **Signalwoerter** in der Aufgabe und gleiche gegen die Team-Trigger-Keywords in den Team-Definitionen ab
- Schaetze ob Spezialisierung **Mehrwert** bringt gegenueber dem Generic Fallback

**Entscheidung:**
- Eindeutiger Match → Team deployen
- Unklarer Match (2 Teams passen aehnlich gut) → User kurz fragen (1 Multiple-Choice-Frage: "Aufgabe X — passt Team A oder Team B besser?")
- Kein Match / zu einfach fuer Spezialisierung → Generic Fallback deployen

### 3. Empfehlung anzeigen

Nach Team-Wahl, VOR Ausfuehrung anzeigen:

```
SWAT Team: [Name]
Grund: [1 Satz warum dieses Team passt]
Rollen: [Rollennamen aus Team-Definition auflisten]
Override: /agent-swat-swarm <anderes-team>
```

## Orchestrierungsprotokoll

Nach Team-Wahl, folge diesem Protokoll exakt:

### Phase 1: Vorbereitung

1. **Team-Definition laden**: `Read` die gematchte Team-Definition aus `references/teams/<name>.md`
2. **Task-Kontext extrahieren**:
   - User-Aufgabe (praezise Zusammenfassung)
   - Betroffene Dateien/Module (per Glob/Grep oder aus User-Angabe)
   - Relevanter Codebase-State (existierende Patterns, Frameworks, Conventions)
3. **Prompt-Templates instanziieren**: `{task_context}` mit extrahiertem Kontext ersetzen

### Phase 2: Parallele Ausfuehrung

4. **Parallele Rollen spawnen**: Alle Rollen OHNE `consumes`-Abhaengigkeit in einem einzigen Message-Block spawnen
   - Exakte Agent-Configs aus Team-Definition verwenden (model, subagent_type, mode, isolation)
   - Jedem Worker einen beschreibenden `name` geben (z.B. "scanner", "exploit-analyst")
   - `run_in_background: true` nur wenn genuinely unabhaengige Arbeit parallel moeglich
5. **Auf alle parallelen Agents warten**

### Phase 3: Output-Validation + Sequentielle Ausfuehrung

6. **Output-Validation** fuer jeden abgeschlossenen Worker:
   - Pruefe: Passt der Output zum definierten Output-Contract? (Struktur, Vollstaendigkeit, keine offensichtlichen Fehler)
   - Bei Format-Problemen: einmalig via `SendMessage` an den Worker korrigieren lassen
   - Bei verbosen Outputs (deutlich ueber dem was der naechste Agent braucht): Lead fasst die Kernpunkte zusammen bevor Weitergabe
7. **Sequentielle Rollen spawnen** (die `consumes` haben):
   - `{Rollenname.output}` Platzhalter mit tatsaechlichen (ggf. zusammengefassten) Outputs ersetzen
   - Output-Validation fuer jede sequentielle Rolle wiederholen

### Phase 4: Integration + Quality Gate

8. **Outputs integrieren**: Alle finalen Outputs einsammeln, auf Konflikte/Widersprueche/Luecken pruefen
9. **Quality Gate spawnen**: Siehe `references/quality-gate-protocol.md`
   - `model: "opus"`, team-spezifische Erfolgskriterien + DA Focus
   - Quality Mode wie in Team-Definition (standard oder rigorous)
10. **Ergebnis verarbeiten**:
    - **PASS** → Ergebnis synthetisiert praesentieren
    - **PASS_WITH_NOTES** → Ergebnis praesentieren + Notes hervorheben
    - **FAIL** → Rework-Protokoll ausfuehren

### Rework-Protokoll (max 2 Zyklen)

11. REWORK_NEEDED Items aus Quality Gate parsen
12. Targeted Fix-Agents spawnen:
    - Kleiner Fix → `SendMessage` an bestehenden Worker (bewahrt Kontext)
    - Groesserer Fix → Neuer Sonnet-Agent mit fokussiertem Prompt
13. Quality Gate erneut ausfuehren (gleicher Modus)
14. Nach 2 Zyklen ohne PASS: bestes Ergebnis mit verbleibenden QG-Notes an User praesentieren

## Ergebnis-Praesentation

- **Knappe Synthese** — niemals raw Agent-Outputs dumpen
- **Key Decisions** hervorheben die waehrend der Dekomposition getroffen wurden
- **PASS_WITH_NOTES Items** falls vorhanden auflisten
- **Eingesetztes Team + Rollen** nennen
- **Quality Gate Verdict** zusammenfassen (1-2 Saetze)

## Worker Isolation mit Worktrees

Fuer Rollen mit `isolation: worktree` in der Team-Definition:
- Jeder Worker bekommt eine isolierte Kopie des Repos
- Eliminiert Merge-Konflikte zwischen parallelen Code-Aenderungen
- Nach Abschluss: Lead integriert die Worker-Branches
- Worktrees werden automatisch aufgeraeumt wenn der Worker keine Aenderungen macht

**Wann Worktrees verwenden:**
- Mehrere Worker aendern Code in ueberlappenden Verzeichnissen
- Worker brauchen einen kompilierbaren/lauffaehigen State waehrend der Arbeit

**Wann NICHT:**
- Worker lesen nur Code (Research/Review)
- Worker schreiben in komplett getrennte Dateien ohne shared Dependencies

## Swarm Sizing

| Task-Komplexitaet | Workers | Quality Mode |
|-------------------|---------|--------------|
| Medium (4-6 Files) | 2-3 | standard |
| Gross (7-15 Files) | 3-5 | standard oder rigorous |
| Sehr Gross (15+ Files) | 5-8 | rigorous |
| Produktionskritisch | beliebig | **rigorous** |

Max 8 Workers. Koordinations-Overhead waechst superlinear darueber.

## Anti-Patterns

1. **Over-swarming** — Triviale Tasks nicht swarmen. 5-Zeilen-Fix braucht kein SWAT-Team.
2. **Vage Worker-Prompts** — Prompts muessen praezise, selbsterklaerend und mit klaren Grenzen sein.
3. **Quality Gate skippen** — Nie. Das QG ist nicht verhandelbar.
4. **Sequentiell wenn parallel moeglich** — Unabhaengige Rollen immer in einem Message-Block parallelisieren.
5. **Garbage weiterreichen** — Lead validiert jeden Output bevor Weitergabe an sequentielle Agents.
6. **Rework ignorieren** — FAIL bedeutet Rework, nicht "trotzdem praesentieren".
7. **Neue Agents fuer kleine Fixes** — `SendMessage` an bestehende Worker statt neuen Agent spawnen.
8. **Giant Single-Worker Tasks** — Wenn eine Worker-Aufgabe zu gross ist, weiter dekomponieren.

## Reference Documentation

- **`references/teams/*.md`** — SWAT-Team-Definitionen mit Rollen, Prompts, Workflow-Graphs
- **`references/quality-gate-protocol.md`** — QG-Spec mit DA-Integration, Rework-Protokoll
- **`references/generic-swarm.md`** — Fallback fuer Tasks ohne Team-Match
```

- [ ] **Step 2: Verify SKILL.md**

Read the file back, verify:
- Frontmatter has correct `name` and `description`
- Team index lists all 7 teams (6 SWAT + 1 Generic Fallback)
- Orchestration protocol has all 4 phases
- No references to files that don't exist yet (they will be created in later tasks)

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "feat: add SWAT dispatcher with team selection and orchestration protocol"
```

---

### Task 3: Quality Gate Protocol

Adapted from old `agent-swarm` quality gate with team-specific DA integration.

**Files:**
- Create: `references/quality-gate-protocol.md`

- [ ] **Step 1: Write quality-gate-protocol.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/references/quality-gate-protocol.md`:

```markdown
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
```

- [ ] **Step 2: Verify quality-gate-protocol.md**

Read back, verify:
- Both standard and rigorous mode templates are complete
- Placeholder names (`{original_user_request}`, `{team_da_focus_questions}`, etc.) are consistent with SKILL.md orchestration protocol
- Rework protocol references `SendMessage` for small fixes
- Decision matrix covers all QA/DA combinations

- [ ] **Step 3: Commit**

```bash
git add references/quality-gate-protocol.md
git commit -m "feat: add quality gate protocol with team-specific DA integration"
```

---

### Task 4: Generic Swarm Fallback

The safety net — deployed when no SWAT team matches. Preserves old `agent-swarm` logic as a team definition.

**Files:**
- Create: `references/generic-swarm.md`

- [ ] **Step 1: Write generic-swarm.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/references/generic-swarm.md`:

```markdown
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
```

- [ ] **Step 2: Verify generic-swarm.md**

Read back, verify:
- Follows the team definition format (frontmatter with triggers, quality_mode)
- Empty triggers (this is a fallback, not pattern-matched)
- Decomposition protocol is clear and actionable

- [ ] **Step 3: Commit**

```bash
git add references/generic-swarm.md
git commit -m "feat: add generic swarm fallback for unmatched tasks"
```

---

### Task 5: Security Audit SWAT Team

**Files:**
- Create: `references/teams/security-audit.md`

- [ ] **Step 1: Write security-audit.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/references/teams/security-audit.md`:

```markdown
---
name: Security Audit
triggers:
  keywords: [security, vulnerability, CVE, OWASP, penetration, audit, exploit, injection, XSS, CSRF, auth bypass, secrets, credential, data leak, attack surface, threat model]
  file_patterns: ["**/auth/**", "**/middleware/**", "**/*.env*", "**/security/**", "**/crypto/**", "**/session/**", "**/token/**", "**/permission/**"]
  task_patterns: ["review.*secur", "check.*vulnerab", "audit", "penetration", "find.*issue", "security.*scan"]
quality_mode: rigorous
max_workers: 4
---

# Security Audit SWAT Team

Systematische Sicherheitsanalyse mit Vulnerability-Scan, Exploit-Bewertung, konkreten Fixes und ausfuehrlichem Report.

## Rollen

### 1. Scanner (parallel)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- output_contract: |
    Strukturierte Liste aller gefundenen Issues:

    ## Findings

    ### [CRITICAL/HIGH/MEDIUM/LOW] — [Kurztitel]
    - **Datei:** `pfad/zur/datei.ts:zeile`
    - **Kategorie:** [OWASP-Kategorie, z.B. A03:2021 Injection]
    - **Beschreibung:** [Was ist das Problem, 2-3 Saetze]
    - **Betroffener Code:** [relevantes Code-Snippet, max 10 Zeilen]

    Gruppiert nach Severity (CRITICAL zuerst). Fokus auf Praezision, nicht Vollstaendigkeit der Beschreibung.
- prompt: |
    Du bist ein Security-Scanner in einem SWAT-Team. Deine einzige Aufgabe: systematisch Sicherheitslucken finden.

    ## Aufgabe
    {task_context}

    ## Pruef-Systematik
    Gehe JEDE der folgenden Kategorien durch. Fuer jede Kategorie: pruefe den relevanten Code, dokumentiere Findings oder notiere explizit "Keine Issues gefunden".

    1. **Injection** (SQL, Command, Template, LDAP, XPath, Header)
       - Suche nach String-Concatenation in Queries/Commands
       - Pruefe ob User-Input sanitized wird bevor Verwendung
       - Pruefe Template-Engines auf SSTI

    2. **Authentication & Session**
       - Hardcoded Credentials, Default-Passwords
       - Session-Token Entropy und Expiration
       - Fehlende Rate-Limiting bei Login
       - JWT-Validierung (Algorithm, Expiry, Signature)

    3. **Authorization**
       - IDOR (Insecure Direct Object References)
       - Fehlende Berechtigungspruefungen in Endpoints
       - Privilege Escalation Pfade

    4. **Data Exposure**
       - Secrets in Code oder Configs (.env, API Keys, Tokens)
       - Sensitive Daten in Logs oder Error-Messages
       - PII ohne Encryption
       - Verbose Error Messages an Client

    5. **XSS & Client-Side**
       - Reflected, Stored, DOM-based XSS
       - Fehlende CSP Headers
       - Unsichere innerHTML/dangerouslySetInnerHTML

    6. **CSRF & Request Forgery**
       - Fehlende CSRF-Tokens bei State-Changing Requests
       - SSRF bei URL-Parametern

    7. **Dependencies**
       - Bekannte CVEs in Dependencies (pruefe package.json / requirements.txt)
       - Outdated Packages mit bekannten Issues

    8. **Cryptography**
       - Schwache Algorithmen (MD5, SHA1 fuer Security-Zwecke)
       - Hardcoded IVs/Salts
       - Fehlende TLS-Enforcement

    9. **Configuration**
       - Debug-Mode in Production
       - Fehlende Security-Headers (HSTS, X-Frame-Options, etc.)
       - Offene CORS-Policies

    ## Output-Format
    Exakt wie im Output-Contract definiert. Jedes Finding mit Datei:Zeile, OWASP-Kategorie, Beschreibung, und betroffenem Code-Snippet.

### 2. Exploit-Analyst (parallel mit Scanner)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- output_contract: |
    Pro gefundenem Angriffsvektor:

    ### [Vektor-Name]
    - **Attack-Szenario:** [Wie ein Angreifer das ausnutzen wuerde, 3 Saetze max]
    - **Impact (CIA):** Confidentiality: [H/M/L] | Integrity: [H/M/L] | Availability: [H/M/L]
    - **Exploit-Schwierigkeit:** [trivial / moderat / komplex]
    - **Voraussetzungen:** [Was braucht der Angreifer? Auth? Netzwerkzugang?]
    - **CVSS-Schaetzung:** [0.0-10.0]

    Sortiert nach CVSS absteigend.
- prompt: |
    Du bist ein Exploit-Analyst in einem SWAT-Team. Deine einzige Aufgabe: Angriffsvektoren identifizieren und deren Exploitability bewerten.

    ## Aufgabe
    {task_context}

    ## Analyse-Systematik

    1. **Attack Surface Mapping**
       - Identifiziere alle Eingangspunkte (API-Endpoints, Forms, File-Uploads, WebSockets)
       - Identifiziere Trust Boundaries (auth/unauth, user/admin, internal/external)
       - Identifiziere Datenfluesse (wo geht User-Input hin?)

    2. **Threat Modeling (STRIDE)**
       Fuer jeden Eingangspunkt:
       - **Spoofing**: Kann ein Angreifer sich als jemand anderes ausgeben?
       - **Tampering**: Kann ein Angreifer Daten manipulieren?
       - **Repudiation**: Kann ein Angreifer Aktionen leugnen?
       - **Information Disclosure**: Kann ein Angreifer an vertrauliche Daten kommen?
       - **Denial of Service**: Kann ein Angreifer den Service lahmlegen?
       - **Elevation of Privilege**: Kann ein Angreifer hoehere Rechte erlangen?

    3. **Exploit-Bewertung**
       Pro Vektor: Wie realistisch ist der Angriff? Was sind die Voraussetzungen?
       Nutze das CIA-Triad fuer Impact-Bewertung.

    ## Output-Format
    Exakt wie im Output-Contract definiert. Fokus auf realistische, ausfuehrbare Angriffe — keine theoretischen Szenarien.

### 3. Fix-Proposer (sequentiell, nach Scanner + Exploit-Analyst)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- consumes: [Scanner.output, Exploit-Analyst.output]
- output_contract: |
    Pro Issue einen konkreten Fix:

    ### Fix fuer: [Issue-Titel]
    - **Severity:** [CRITICAL/HIGH/MEDIUM/LOW]
    - **Datei:** `pfad/zur/datei.ts`
    - **Aenderung:** [Diff oder vollstaendiger Code-Block]
    - **Erklaerung:** [Warum dieser Fix das Problem loest, 1-2 Saetze]
    - **Risiko:** [Koennte der Fix etwas anderes brechen? Wenn ja, was?]

    CRITICAL und HIGH Issues zuerst. Kein Fix darf neue Vulnerabilities einfuehren.
- prompt: |
    Du bist ein Fix-Proposer in einem SWAT-Team. Du bekommst die Ergebnisse vom Scanner und Exploit-Analyst und schreibst konkrete Fixes.

    ## Aufgabe
    {task_context}

    ## Scanner-Ergebnisse
    {Scanner.output}

    ## Exploit-Analyse
    {Exploit-Analyst.output}

    ## Fix-Regeln

    1. **Priorisierung**: CRITICAL zuerst, dann HIGH, dann MEDIUM. LOW nur wenn trivial.
    2. **Konkrete Fixes**: Jeder Fix ist ein konkreter Code-Change (Diff oder vollstaendiger Code-Block). Keine vagen Empfehlungen wie "sanitize input" — zeige den exakten Code.
    3. **Defense in Depth**: Bevorzuge mehrschichtige Fixes (Input Validation + Parameterized Queries + Output Encoding) statt einzelner Massnahmen.
    4. **Keine neuen Probleme**: Pruefe ob dein Fix neue Attack Surfaces eroeffnet.
    5. **Bestehende Patterns**: Folge den Patterns die im Codebase bereits existieren (z.B. gleiche Validation Library, gleiche Auth Middleware).
    6. **Minimale Aenderungen**: Fix nur das Sicherheitsproblem. Kein Refactoring, keine Feature-Aenderungen.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 4. Report-Writer (sequentiell, nach Fix-Proposer)
- model: sonnet
- subagent_type: general-purpose
- consumes: [Scanner.output, Exploit-Analyst.output, Fix-Proposer.output]
- output_contract: |
    Vollstaendiger Security-Report als Markdown:

    # Security Audit Report

    ## Executive Summary
    [3-5 Saetze: Gesamtbewertung, kritischste Findings, Handlungsempfehlung]

    ## Risk Score
    [CRITICAL / HIGH / MEDIUM / LOW basierend auf hoechster Severity]

    ## Findings Overview
    | # | Severity | Titel | OWASP | Fix verfuegbar |
    |---|----------|-------|-------|----------------|

    ## Detailed Findings
    [Pro Finding: Beschreibung, Impact, Exploit-Szenario, Fix, Residual Risk]

    ## Remediation Plan
    [Priorisierte Liste der Fixes mit geschaetztem Aufwand]

    ## Residual Risk
    [Was bleibt auch nach allen Fixes als Risiko?]
- prompt: |
    Du bist ein Security-Report-Writer in einem SWAT-Team. Du kompilierst die Ergebnisse aller vorherigen Agents in einen ausfuehrlichen, actionable Security Report.

    ## Aufgabe
    {task_context}

    ## Scanner-Ergebnisse
    {Scanner.output}

    ## Exploit-Analyse
    {Exploit-Analyst.output}

    ## Vorgeschlagene Fixes
    {Fix-Proposer.output}

    ## Report-Regeln

    1. **Executive Summary**: Fuer Management lesbar. Keine technischen Details, nur Impact und Handlungsbedarf.
    2. **Findings**: Vollstaendig — jedes Finding vom Scanner muss im Report erscheinen.
    3. **Keine Erfindungen**: Nur Findings dokumentieren die tatsaechlich gefunden wurden. Keine spekulativen Issues hinzufuegen.
    4. **Remediation Plan**: Priorisiert nach Risk (CRITICAL zuerst), mit Aufwandschaetzung (S/M/L).
    5. **Residual Risk**: Ehrlich dokumentieren was auch nach Fixes bleibt. Kein "alles ist sicher nach den Fixes".

    ## Output-Format
    Exakt wie im Output-Contract definiert.

## Workflow-Graph

```
Scanner ──────────┐
                  ├──→ Fix-Proposer ──→ Report-Writer
Exploit-Analyst ──┘
```

## Devil's Advocate Focus

- Welche Angriffsvektoren wurden NICHT geprueft? (z.B. Business Logic Flaws, Race Conditions, Timing Attacks)
- Sind die vorgeschlagenen Fixes wirklich sicher oder nur Symptombekaempfung?
- Gibt es transitive Dependency-Vulnerabilities die uebersehen wurden?
- Was passiert wenn der Angreifer bereits authentifiziert ist (Insider Threat)?
- Sind die CVSS-Schaetzungen realistisch oder inflated/deflated?
- Fehlen Security-Headers oder CSP-Konfigurationen?

## Erfolgskriterien (Quality Gate)

- Alle CRITICAL und HIGH Issues haben einen konkreten, implementierbaren Fix
- Kein vorgeschlagener Fix fuehrt neue Vulnerabilities ein
- Report ist vollstaendig (jedes Scanner-Finding erscheint im Report)
- Exploit-Bewertungen sind realistisch (keine theoretischen Szenarien ohne praktische Relevanz)
- Remediation Plan ist priorisiert und actionable
```

- [ ] **Step 2: Verify security-audit.md**

Read back, verify:
- 4 roles with full prompts and output contracts
- Workflow graph matches role dependencies (Scanner + Exploit parallel → Fix-Proposer → Report-Writer)
- `consumes` fields reference correct role names
- DA focus questions are security-domain-specific

- [ ] **Step 3: Commit**

```bash
git add references/teams/security-audit.md
git commit -m "feat: add Security Audit SWAT team definition"
```

---

### Task 6: Frontend Feature SWAT Team

**Files:**
- Create: `references/teams/frontend-feature.md`

- [ ] **Step 1: Write frontend-feature.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/references/teams/frontend-feature.md`:

```markdown
---
name: Frontend Feature
triggers:
  keywords: [component, UI, frontend, design system, responsive, accessibility, a11y, CSS, Tailwind, React, layout, form, modal, dialog, animation, theme, dark mode, mobile]
  file_patterns: ["**/*.tsx", "**/*.jsx", "**/*.css", "**/components/**", "**/pages/**", "**/app/**/*.tsx", "**/styles/**", "**/hooks/**"]
  task_patterns: ["build.*component", "create.*UI", "implement.*design", "add.*page", "frontend.*feature", "responsive", "accessib"]
quality_mode: standard
max_workers: 4
---

# Frontend Feature SWAT Team

End-to-end Frontend-Feature-Implementierung von Architektur ueber Komponenten-Bau bis Accessibility-Review.

## Rollen

### 1. UI-Architekt (parallel)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- output_contract: |
    ## Komponenten-Architektur

    ### Komponenten-Baum
    [Hierarchie der Komponenten als ASCII-Tree]

    ### Pro Komponente
    - **Name:** `ComponentName`
    - **Datei:** `pfad/zur/datei.tsx`
    - **Typ:** Server Component | Client Component
    - **Props:** [TypeScript Interface]
    - **State:** [Welcher State, wo verwaltet]
    - **Verantwortung:** [1 Satz]

    ### Datenfluss
    [Wie fliessen Daten durch den Baum? Props, Context, Server Actions?]

    ### Styling-Strategie
    [Tailwind Classes, CSS Modules, oder beides? Design Tokens?]
- prompt: |
    Du bist ein UI-Architekt in einem SWAT-Team. Deine Aufgabe: die Komponenten-Architektur fuer ein Frontend-Feature designen.

    ## Aufgabe
    {task_context}

    ## Design-Prinzipien

    1. **Server Components Default**: Nur `'use client'` wo Interaktivitaet oder Browser-APIs noetig sind. Push Client-Boundaries so weit runter wie moeglich.
    2. **Composition over Inheritance**: Kleine, fokussierte Komponenten die zusammengesteckt werden.
    3. **Bestehende Patterns folgen**: Schaue dir die existierenden Komponenten im Projekt an und folge deren Patterns (Naming, Struktur, State-Management).
    4. **Props Interface zuerst**: Definiere die TypeScript Interfaces bevor du ueber Implementierung nachdenkst.
    5. **Data Fetching**: Server Components fetchen Daten direkt. Client Components nutzen Server Actions oder Hooks.
    6. **Styling**: Nutze das bestehende Styling-System des Projekts (Tailwind, CSS Modules, styled-components — was auch immer schon da ist).

    ## Output-Format
    Exakt wie im Output-Contract definiert. Jede Komponente mit Datei, Typ, Props, State, Verantwortung.

### 2. Component-Builder (parallel mit UI-Architekt)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- output_contract: |
    Pro Komponente die gebaut wurde:

    ### `ComponentName`
    - **Datei:** `pfad/zur/datei.tsx`
    - **Code:** [Vollstaendiger Komponenten-Code]
    - **Tests:** [Test-Code falls applicable]

    Alle Komponenten muessen compilieren und korrekte TypeScript-Typen haben.
- prompt: |
    Du bist ein Component-Builder in einem SWAT-Team. Deine Aufgabe: die UI-Komponenten implementieren.

    ## Aufgabe
    {task_context}

    ## Implementierungs-Regeln

    1. **TypeScript Strict**: Alle Props typisiert, keine `any`. Exportiere Props-Interfaces.
    2. **Bestehende Design-System nutzen**: Wenn das Projekt shadcn/ui, Radix, oder ein anderes Component-System hat — nutze es. Baue keine Primitives von Scratch.
    3. **Responsive**: Mobile-first mit Tailwind Breakpoints (sm, md, lg). Teste mental fuer 320px, 768px, 1280px.
    4. **Accessibility Basics**: Semantisches HTML, ARIA-Labels wo noetig, Keyboard-Navigation, Fokus-Management.
    5. **Error/Loading/Empty States**: Jede Komponente die Daten anzeigt braucht alle drei States.
    6. **Performance**: Keine unnecessary Re-Renders. `useMemo`/`useCallback` nur wo messbar noetig, nicht praventiv.
    7. **Naming**: Folge dem bestehenden Naming-Schema des Projekts.

    ## Output-Format
    Exakt wie im Output-Contract definiert. Vollstaendiger Code pro Komponente.

### 3. Integrator (sequentiell, nach UI-Architekt + Component-Builder)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- consumes: [UI-Architekt.output, Component-Builder.output]
- output_contract: |
    ## Integrations-Ergebnis

    ### Angepasste Dateien
    Pro Datei die geaendert oder erstellt wurde:
    - **Datei:** `pfad/datei.tsx`
    - **Aenderung:** [Was wurde geaendert/hinzugefuegt]
    - **Code:** [Vollstaendiger Code der geaenderten Datei]

    ### Routing
    [Welche Routes wurden hinzugefuegt/geaendert?]

    ### Data Flow Verification
    [Bestaetigung dass Datenfluss wie vom Architekten geplant funktioniert]
- prompt: |
    Du bist ein Integrator in einem SWAT-Team. Du verbindest die Architektur-Entscheidungen des UI-Architekten mit den gebauten Komponenten des Component-Builders.

    ## Aufgabe
    {task_context}

    ## Architektur-Vorgabe
    {UI-Architekt.output}

    ## Gebaute Komponenten
    {Component-Builder.output}

    ## Integrations-Aufgaben

    1. **Komponenten verdrahten**: Stelle sicher dass der Komponenten-Baum wie vom Architekten geplant zusammengesetzt ist.
    2. **Datenfluss implementieren**: Props, Context, Server Actions — wie im Architektur-Plan.
    3. **Routing einrichten**: Neue Pages/Routes erstellen falls noetig.
    4. **Imports aufloesen**: Alle Imports korrekt, keine zirkulaeren Dependencies.
    5. **Anpassungen**: Falls die Architektur und die Implementation nicht exakt zusammenpassen (z.B. Props-Interface Unterschiede), passe an und dokumentiere die Abweichung.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 4. A11y-Reviewer (sequentiell, nach Integrator)
- model: sonnet
- subagent_type: general-purpose
- consumes: [Integrator.output]
- output_contract: |
    ## Accessibility Review

    ### Findings
    Pro Issue:
    - **Severity:** [CRITICAL/MAJOR/MINOR]
    - **WCAG-Kriterium:** [z.B. 2.1.1 Keyboard]
    - **Datei:** `pfad/datei.tsx:zeile`
    - **Problem:** [Was ist das A11y-Problem]
    - **Fix:** [Konkreter Code-Fix]

    ### Checklist
    - [ ] Semantisches HTML (keine div-Suppe)
    - [ ] ARIA-Labels vollstaendig
    - [ ] Keyboard-Navigation funktioniert
    - [ ] Fokus-Reihenfolge logisch
    - [ ] Color Contrast ausreichend (4.5:1 Text, 3:1 UI)
    - [ ] Screen Reader verstaendlich
    - [ ] Responsive bis 320px
    - [ ] Reduzierte Bewegung respektiert (prefers-reduced-motion)
- prompt: |
    Du bist ein Accessibility-Reviewer in einem SWAT-Team. Deine einzige Aufgabe: die implementierten Komponenten auf Barrierefreiheit pruefen.

    ## Aufgabe
    {task_context}

    ## Zu pruefender Code
    {Integrator.output}

    ## Pruef-Systematik (WCAG 2.1 AA)

    1. **Perceivable**
       - Text-Alternativen fuer Bilder (alt-Attribute)
       - Color Contrast (4.5:1 fuer normalen Text, 3:1 fuer grossen Text und UI-Elemente)
       - Content ohne Farbe verstaendlich
       - Responsive bis 320px ohne horizontales Scrollen

    2. **Operable**
       - Alle interaktiven Elemente per Keyboard erreichbar (Tab, Enter, Space, Arrow Keys)
       - Fokus-Reihenfolge entspricht visueller Reihenfolge
       - Fokus ist sichtbar (Focus-Ring)
       - Keine Keyboard-Traps
       - Skip-Links wo noetig

    3. **Understandable**
       - Labels fuer alle Form-Felder
       - Error Messages klar und assoziiert
       - Konsistente Navigation
       - Language-Attribut auf HTML gesetzt

    4. **Robust**
       - Valides HTML (keine fehlenden closing Tags, korrekte Nesting)
       - ARIA korrekt verwendet (roles, states, properties)
       - Keine ARIA wo natives HTML reicht (button statt div role="button")

    ## Output-Format
    Exakt wie im Output-Contract definiert. Jedes Finding mit konkretem Fix-Code.

## Workflow-Graph

```
UI-Architekt ──────┐
                   ├──→ Integrator ──→ A11y-Reviewer
Component-Builder ─┘
```

## Devil's Advocate Focus

- Wie verhaelt sich das Feature bei Screen-Readern (VoiceOver, NVDA)?
- Was passiert bei langsamer Verbindung (3G)? Sind Loading States vorhanden?
- Funktioniert es bei 320px Viewport-Breite ohne horizontales Scrollen?
- Sind alle interaktiven Elemente per Keyboard erreichbar und bedienbar?
- Folgt das Styling dem bestehenden Design-System oder weicht es ab?
- Gibt es unnecessary Client Components die Server Components sein koennten?

## Erfolgskriterien (Quality Gate)

- Alle Komponenten compilieren fehlerfrei mit TypeScript strict
- Kein Component hat fehlende Props oder falsche Typen
- WCAG 2.1 AA wird eingehalten (keine CRITICAL A11y Issues)
- Responsive Layout funktioniert bei 320px, 768px, 1280px
- Datenfluss entspricht der Architektur-Vorgabe
- Bestehende Design-System Patterns werden eingehalten
```

- [ ] **Step 2: Verify frontend-feature.md**

Read back, verify:
- 4 roles with full prompts and output contracts
- Workflow matches dependencies (Architekt + Builder parallel → Integrator → A11y-Reviewer)
- `consumes` fields reference correct role names

- [ ] **Step 3: Commit**

```bash
git add references/teams/frontend-feature.md
git commit -m "feat: add Frontend Feature SWAT team definition"
```

---

### Task 7: Backend API SWAT Team

**Files:**
- Create: `references/teams/backend-api.md`

- [ ] **Step 1: Write backend-api.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/references/teams/backend-api.md`:

```markdown
---
name: Backend API
triggers:
  keywords: [API, endpoint, route, REST, GraphQL, database, schema, migration, query, ORM, Prisma, Drizzle, controller, service, middleware, CRUD, webhook, server action]
  file_patterns: ["**/api/**", "**/routes/**", "**/server/**", "**/db/**", "**/schema/**", "**/models/**", "**/services/**", "**/prisma/**", "**/drizzle/**"]
  task_patterns: ["build.*api", "create.*endpoint", "implement.*backend", "add.*route", "database.*schema", "server.*action"]
quality_mode: standard
max_workers: 4
---

# Backend API SWAT Team

End-to-end Backend-Implementierung von Schema-Design ueber Endpoint-Bau bis Test-Coverage.

## Rollen

### 1. Schema-Designer (parallel)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- output_contract: |
    ## Datenmodell

    ### Schema-Aenderungen
    [Vollstaendiges Schema als Code (Prisma, Drizzle, SQL — was im Projekt verwendet wird)]

    ### Migrations
    [Migration-Dateien falls noetig]

    ### Relationen
    [Entity-Relationship Beschreibung als ASCII]

    ### Indices
    [Welche Indices fuer Performance, mit Begruendung]

    ### Validierung
    [Welche Constraints auf DB-Level (NOT NULL, UNIQUE, CHECK)]
- prompt: |
    Du bist ein Schema-Designer in einem SWAT-Team. Deine Aufgabe: das Datenmodell fuer ein Backend-Feature designen.

    ## Aufgabe
    {task_context}

    ## Design-Prinzipien

    1. **Bestehendes ORM verwenden**: Schaue welches ORM/Query-Tool im Projekt verwendet wird (Prisma, Drizzle, Knex, raw SQL) und nutze es.
    2. **Normalisierung**: 3NF als Default. Denormalisiere nur mit expliziter Performance-Begruendung.
    3. **Indices**: Erstelle Indices fuer alle Foreign Keys und haeufig gefilterte/sortierte Spalten.
    4. **Constraints**: NOT NULL wo moeglich. UNIQUE wo Business-Logik es verlangt. CHECK fuer Wertebereich-Validierung.
    5. **Naming**: Folge dem bestehenden Schema-Naming (snake_case vs camelCase, Singular vs Plural).
    6. **Migrations**: Erstelle Migrations die vorwaerts UND rueckwaerts laufen koennen.
    7. **Soft Deletes**: Nutze wenn im Projekt etabliert, sonst Hard Deletes.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 2. Endpoint-Builder (parallel mit Schema-Designer)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- output_contract: |
    Pro Endpoint:

    ### [METHOD] /api/pfad
    - **Datei:** `pfad/zur/route.ts`
    - **Auth:** [Erforderlich/Oeffentlich, welche Rolle]
    - **Input:** [Request Body / Query Params mit Types]
    - **Output:** [Response Body mit Types]
    - **Error Codes:** [400/401/403/404/500 mit Beschreibung]
    - **Code:** [Vollstaendiger Endpoint-Code]
- prompt: |
    Du bist ein Endpoint-Builder in einem SWAT-Team. Deine Aufgabe: die API-Endpoints implementieren.

    ## Aufgabe
    {task_context}

    ## Implementierungs-Regeln

    1. **Bestehendes Framework verwenden**: Schaue ob das Projekt Next.js Route Handlers, Express, Hono, oder ein anderes Framework nutzt — und folge dessen Patterns.
    2. **Input Validation**: Validiere JEDEN Input an der Systemgrenze. Nutze das bestehende Validation-Tool (Zod, Joi, etc.) oder definiere Zod-Schemas.
    3. **Error Handling**: Konsistente Error-Responses. Nutze bestehende Error-Handler falls vorhanden.
    4. **Auth**: Nutze die bestehende Auth-Middleware/Guard. Implementiere keine eigene Auth-Logik.
    5. **Response Types**: TypeScript-Typen fuer alle Responses. Keine `any`.
    6. **HTTP Semantik**: Korrekter Method (GET fuer Reads, POST fuer Creates, PUT/PATCH fuer Updates, DELETE fuer Deletes). Korrekte Status Codes.
    7. **Kein Business-Logic in Endpoints**: Endpoints sind duenn — sie validieren Input, rufen Services auf, und formatieren Output. Business-Logik lebt in Service-Funktionen.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 3. Test-Writer (sequentiell, nach Schema-Designer + Endpoint-Builder)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- consumes: [Schema-Designer.output, Endpoint-Builder.output]
- output_contract: |
    ## Test Suite

    Pro Endpoint:
    ### Tests fuer [METHOD] /api/pfad
    - **Datei:** `tests/pfad/test-datei.test.ts`
    - **Code:** [Vollstaendiger Test-Code]
    - **Coverage:** [Welche Szenarien werden abgedeckt]

    Minimum: Happy Path + Auth Failure + Validation Error + Not Found pro Endpoint.
- prompt: |
    Du bist ein Test-Writer in einem SWAT-Team. Du schreibst Tests fuer die implementierten Endpoints.

    ## Aufgabe
    {task_context}

    ## Schema
    {Schema-Designer.output}

    ## Endpoints
    {Endpoint-Builder.output}

    ## Test-Regeln

    1. **Test-Framework**: Nutze das bestehende Test-Setup (Jest, Vitest, etc.).
    2. **Pro Endpoint mindestens:**
       - Happy Path (200/201 mit korrektem Response)
       - Auth Failure (401/403 ohne/mit falschem Token)
       - Validation Error (400 mit ungueltigem Input)
       - Not Found (404 fuer nicht-existierende Ressource)
       - Edge Cases (leere Listen, Grenzwerte, Unicode)
    3. **Arrange-Act-Assert**: Klare Trennung in jedem Test.
    4. **Keine Mocks fuer DB**: Nutze die echte DB (Test-DB oder In-Memory) wenn moeglich. Mocks nur fuer externe Services.
    5. **Beschreibende Test-Namen**: `it('returns 400 when email is missing')` nicht `it('test 1')`.
    6. **Cleanup**: Tests hinterlassen keine Daten. Setup/Teardown pro Test.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 4. Integration-Tester (sequentiell, nach Test-Writer)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- consumes: [Schema-Designer.output, Endpoint-Builder.output, Test-Writer.output]
- output_contract: |
    ## Integration Test Report

    ### Test-Ergebnisse
    [Zusammenfassung: X passed, Y failed, Z skipped]

    ### Probleme
    Pro Problem:
    - **Test:** [Test-Name]
    - **Fehler:** [Error Message]
    - **Ursache:** [Root Cause Analyse]
    - **Fix:** [Konkreter Fix-Vorschlag]

    ### API-Contract Verification
    [Stimmen Response-Typen mit den deklarierten Typen ueberein?]
    [Sind alle Error Codes korrekt?]
- prompt: |
    Du bist ein Integration-Tester in einem SWAT-Team. Du pruefst ob Schema, Endpoints, und Tests zusammen funktionieren.

    ## Aufgabe
    {task_context}

    ## Schema
    {Schema-Designer.output}

    ## Endpoints
    {Endpoint-Builder.output}

    ## Test Suite
    {Test-Writer.output}

    ## Pruef-Aufgaben

    1. **Schema-Endpoint Alignment**: Nutzen die Endpoints die korrekten Schema-Felder? Stimmen die Typen ueberein?
    2. **Test-Endpoint Alignment**: Testen die Tests die richtigen Endpoints mit den richtigen Payloads?
    3. **Contract Verification**: Stimmen die Response-Typen der Endpoints mit dem was die Tests erwarten ueberein?
    4. **Missing Coverage**: Gibt es Endpoints oder Pfade die nicht getestet sind?
    5. **Integration Issues**: Wuerde das als Ganzes funktionieren? Fehlende Imports, falsche Pfade, inkompatible Typen?

    ## Output-Format
    Exakt wie im Output-Contract definiert.

## Workflow-Graph

```
Schema-Designer ──────┐
                      ├──→ Test-Writer ──→ Integration-Tester
Endpoint-Builder ─────┘
```

## Devil's Advocate Focus

- Wie skaliert die API bei 100x der erwarteten Last? Gibt es N+1 Queries?
- Was passiert bei Race Conditions (gleichzeitige Writes auf dieselbe Ressource)?
- Sind alle Endpoints korrekt autorisiert? Kann ein User Daten eines anderen Users lesen/aendern?
- Ist die Error-Handling konsistent ueber alle Endpoints?
- Gibt es fehlende Indices die bei grosser Datenmenge zu Slow Queries fuehren?
- Sind die Responses zu gross? Braucht es Pagination?

## Erfolgskriterien (Quality Gate)

- Alle Endpoints haben Input-Validation
- Alle Endpoints haben korrekte Auth-Checks
- Schema-Migrationen sind reversibel
- Tests decken Happy Path + Error Cases ab
- Keine N+1 Queries in der Implementierung
- Response-Typen sind konsistent mit Schema-Typen
```

- [ ] **Step 2: Verify backend-api.md**

Read back, verify same criteria as previous teams.

- [ ] **Step 3: Commit**

```bash
git add references/teams/backend-api.md
git commit -m "feat: add Backend API SWAT team definition"
```

---

### Task 8: Migration/Refactor SWAT Team

**Files:**
- Create: `references/teams/migration-refactor.md`

- [ ] **Step 1: Write migration-refactor.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/references/teams/migration-refactor.md`:

```markdown
---
name: Migration/Refactor
triggers:
  keywords: [migrate, migration, upgrade, refactor, deprecat, breaking change, version, codemod, legacy, modernize, rename, restructure, move, extract, consolidate]
  file_patterns: ["**/package.json", "**/tsconfig.json", "**/*.config.*", "**/next.config.*", "**/migration*/**"]
  task_patterns: ["migrat", "upgrad", "refactor", "move.*to", "extract.*from", "consolidat", "modern"]
quality_mode: rigorous
max_workers: 4
---

# Migration/Refactor SWAT Team

Sichere Migrationen und Refactorings mit Impact-Analyse, Planung, Ausfuehrung und Regression-Testing.

## Rollen

### 1. Impact-Analyst (parallel)
- model: sonnet
- subagent_type: Explore
- output_contract: |
    ## Impact-Analyse

    ### Betroffene Dateien
    | Datei | Aenderungstyp | Risiko | Abhaengigkeiten |
    |-------|--------------|--------|-----------------|

    ### Dependency-Graph
    [Welche Module haengen von den zu aendernden Modulen ab?]

    ### Breaking Changes
    [Liste aller Breaking Changes mit Beschreibung]

    ### Risiko-Bewertung
    - **Gesamt-Risiko:** [LOW/MEDIUM/HIGH/CRITICAL]
    - **Hoechstes Einzelrisiko:** [Beschreibung]
    - **Rollback-Strategie:** [Wie kann die Aenderung rueckgaengig gemacht werden?]
- prompt: |
    Du bist ein Impact-Analyst in einem SWAT-Team. Deine Aufgabe: analysiere die Auswirkungen einer geplanten Migration oder eines Refactorings.

    ## Aufgabe
    {task_context}

    ## Analyse-Systematik

    1. **Direkte Abhaengigkeiten**: Welche Dateien importieren/nutzen das zu aendernde Modul?
    2. **Transitive Abhaengigkeiten**: Welche Module haengen indirekt davon ab?
    3. **Runtime-Abhaengigkeiten**: Gibt es dynamische Imports, Reflection, oder String-basierte Referenzen die statische Analyse nicht findet?
    4. **Test-Abhaengigkeiten**: Welche Tests muessen angepasst werden?
    5. **Config-Abhaengigkeiten**: Aenderungen in tsconfig, package.json, build configs?
    6. **Breaking Changes**: Was bricht fuer Consumer (andere Module, externe APIs, CLI)?
    7. **Rollback**: Wie kann die Aenderung sicher rueckgaengig gemacht werden?

    Nutze `Grep` und `Glob` extensiv um alle Referenzen zu finden. Verlasse dich nicht auf Annahmen.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 2. Migration-Planner (parallel mit Impact-Analyst)
- model: sonnet
- subagent_type: general-purpose
- output_contract: |
    ## Migrations-Plan

    ### Schritt-fuer-Schritt
    Pro Schritt:
    1. **[Schritt-Name]**
       - Dateien: [betroffene Dateien]
       - Aenderung: [Was wird geaendert]
       - Pruefung: [Wie wird verifiziert dass der Schritt funktioniert hat]
       - Rollback: [Wie wird der Schritt rueckgaengig gemacht]

    ### Reihenfolge
    [Warum diese Reihenfolge? Welche Schritte muessen vor anderen kommen?]

    ### Codemods
    [Gibt es automatisierbare Aenderungen? Welche Tools?]
- prompt: |
    Du bist ein Migration-Planner in einem SWAT-Team. Deine Aufgabe: einen sicheren, schrittweisen Migrations-Plan erstellen.

    ## Aufgabe
    {task_context}

    ## Planungs-Prinzipien

    1. **Inkrementell**: Jeder Schritt ist fuer sich lauffaehig. Kein "Big Bang".
    2. **Testbar**: Nach jedem Schritt kann verifiziert werden dass nichts kaputt ist.
    3. **Reversibel**: Jeder Schritt hat eine Rollback-Strategie.
    4. **Codemods zuerst**: Wenn automatisierbare Aenderungen moeglich sind (z.B. via jscodeshift, ts-morph), nutze sie statt manueller Edits.
    5. **Backward Compatibility**: Wenn moeglich, erst den neuen Code einfuehren, dann den alten entfernen (Expand-Contract Pattern).
    6. **Dokumentation**: Wenn die Migration Breaking Changes fuer externe Consumer hat, dokumentiere sie.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 3. Executor (sequentiell, nach Impact-Analyst + Migration-Planner)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- consumes: [Impact-Analyst.output, Migration-Planner.output]
- output_contract: |
    ## Durchgefuehrte Aenderungen

    Pro Schritt aus dem Plan:
    ### Schritt N: [Name]
    - **Status:** [DONE / SKIPPED mit Begruendung]
    - **Geaenderte Dateien:** [Liste mit kurzer Beschreibung der Aenderung]
    - **Code:** [Vollstaendiger Code der geaenderten Dateien]
    - **Verifikation:** [Ergebnis der Pruefung]
    - **Abweichungen:** [Falls vom Plan abgewichen, warum?]
- prompt: |
    Du bist der Executor in einem SWAT-Team. Du fuehrst den Migrations-Plan schrittweise aus.

    ## Aufgabe
    {task_context}

    ## Impact-Analyse
    {Impact-Analyst.output}

    ## Migrations-Plan
    {Migration-Planner.output}

    ## Ausfuehrungs-Regeln

    1. **Plan folgen**: Fuehre die Schritte in der geplanten Reihenfolge aus.
    2. **Verifizieren**: Nach jedem Schritt pruefe ob der Code noch konsistent ist.
    3. **Abweichungen dokumentieren**: Wenn du vom Plan abweichen musst, dokumentiere warum.
    4. **Nicht mehr als noetig**: Fuehre NUR die geplanten Aenderungen durch. Kein "nebenbei aufraeuemen".
    5. **Impact beachten**: Pruefe die Impact-Analyse — aendere nicht nur die direkten Dateien, sondern auch alle betroffenen Referenzen.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 4. Regression-Tester (sequentiell, nach Executor)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- consumes: [Impact-Analyst.output, Executor.output]
- output_contract: |
    ## Regressions-Test Report

    ### Test-Ergebnisse
    [Bestehende Tests: X passed, Y failed, Z skipped]

    ### Regressionen
    Pro Regression:
    - **Test:** [Test-Name]
    - **Fehler:** [Error Message]
    - **Ursache:** [Durch welchen Migrationsschritt verursacht]
    - **Fix:** [Konkreter Fix]

    ### Nicht-getestete Bereiche
    [Betroffene Dateien/Module ohne Test-Coverage]

    ### Manuelle Pruefung empfohlen
    [Bereiche die nur manuell verifiziert werden koennen]
- prompt: |
    Du bist der Regression-Tester in einem SWAT-Team. Deine Aufgabe: pruefen ob die Migration nichts kaputt gemacht hat.

    ## Aufgabe
    {task_context}

    ## Impact-Analyse (was betroffen ist)
    {Impact-Analyst.output}

    ## Durchgefuehrte Aenderungen
    {Executor.output}

    ## Test-Strategie

    1. **Bestehende Tests ausfuehren**: Laufe die bestehende Test-Suite. Dokumentiere jedes Failure.
    2. **Impact-basiert testen**: Fuer jede betroffene Datei aus der Impact-Analyse pruefe ob zugehoerige Tests existieren und passen.
    3. **Regressions-Analyse**: Fuer jedes Test-Failure analysiere ob es durch die Migration verursacht wurde und schlage einen Fix vor.
    4. **Coverage-Luecken**: Identifiziere betroffene Bereiche ohne Tests und empfehle manuelle Pruefung.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

## Workflow-Graph

```
Impact-Analyst ──────┐
                     ├──→ Executor ──→ Regression-Tester
Migration-Planner ───┘
```

## Devil's Advocate Focus

- Welche Breaking Changes wurden uebersehen? (besonders transitive, runtime, dynamische Referenzen)
- Gibt es versteckte Runtime-Abhaengigkeiten die statische Analyse nicht findet?
- Ist die Rollback-Strategie realistisch oder nur theoretisch?
- Was passiert mit laufenden Prozessen/Deployments waehrend der Migration?
- Sind alle Codemods korrekt oder gibt es Edge Cases die sie falsch transformieren?
- Gibt es External Consumer (APIs, Packages) die von den Breaking Changes betroffen sind?

## Erfolgskriterien (Quality Gate)

- Alle bestehenden Tests passen nach der Migration
- Jeder Migrationsschritt ist dokumentiert und reversibel
- Keine unbehandelten Breaking Changes fuer Consumer
- Impact-Analyse deckt sich mit den tatsaechlichen Aenderungen
- Keine vergessenen Referenzen (Grep-Verification fuer alte Patterns)
```

- [ ] **Step 2: Verify migration-refactor.md**

Read back, verify same criteria as previous teams.

- [ ] **Step 3: Commit**

```bash
git add references/teams/migration-refactor.md
git commit -m "feat: add Migration/Refactor SWAT team definition"
```

---

### Task 9: Full-Stack Feature SWAT Team

**Files:**
- Create: `references/teams/full-stack-feature.md`

- [ ] **Step 1: Write full-stack-feature.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/references/teams/full-stack-feature.md`:

```markdown
---
name: Full-Stack Feature
triggers:
  keywords: [full-stack, fullstack, end-to-end, e2e feature, frontend AND backend, page with API, form with server, dashboard, CRUD feature, user flow]
  file_patterns: []
  task_patterns: ["build.*feature", "implement.*end.to.end", "create.*with.*api", "full.stack"]
quality_mode: standard
max_workers: 5
---

# Full-Stack Feature SWAT Team

End-to-end Feature-Implementierung ueber Frontend, Backend und Tests in einem koordinierten Swarm.

## Rollen

### 1. Schema-Designer (parallel)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- output_contract: |
    ## Datenmodell

    ### Schema
    [Vollstaendiges Schema als Code (Prisma/Drizzle/SQL)]

    ### API-Contract
    [TypeScript Types fuer Request/Response die Frontend UND Backend nutzen]

    ```typescript
    // shared/types.ts
    export interface CreateItemRequest { ... }
    export interface CreateItemResponse { ... }
    export interface GetItemsResponse { items: Item[]; total: number; }
    ```

    ### Validierung
    [Zod-Schemas fuer Input-Validation, teilbar zwischen Client und Server]
- prompt: |
    Du bist der Schema-Designer in einem Full-Stack SWAT-Team. Du definierst das Datenmodell UND die API-Contracts die Frontend und Backend teilen.

    ## Aufgabe
    {task_context}

    ## Design-Prinzipien

    1. **Shared Types**: Definiere TypeScript Interfaces die sowohl Frontend als auch Backend nutzen. Eine Source of Truth.
    2. **Zod-Schemas**: Definiere Zod-Schemas fuer Input-Validation die auf Client UND Server laufen.
    3. **Schema folgt den Projekt-Conventions**: Nutze das bestehende ORM und dessen Patterns.
    4. **API-Contract zuerst**: Frontend-Builder und Backend-Builder arbeiten parallel — der Contract ist ihr Vertrag.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 2. Frontend-Builder (parallel)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- output_contract: |
    Pro Komponente/Page:

    ### `ComponentName`
    - **Datei:** `pfad/datei.tsx`
    - **Typ:** Server Component | Client Component
    - **Code:** [Vollstaendiger Code]

    Verwendet die API-Contract Types aus Schema-Designer (importiert aus shared/types).
    Alle Data-Fetching nutzt die definierten Endpoints (noch nicht implementiert — Typ-korrekte Calls).
- prompt: |
    Du bist der Frontend-Builder in einem Full-Stack SWAT-Team. Du baust die UI-Seite des Features.

    ## Aufgabe
    {task_context}

    ## WICHTIG: API-Contract
    Der Schema-Designer definiert parallel die API-Contracts. Du kennst die Types noch nicht im Detail — verwende Platzhalter-Types die der Integrator spaeter mit den echten Types ersetzt. Strukturiere deinen Code so dass die API-Aufrufe leicht austauschbar sind.

    ## Implementierungs-Regeln

    1. **Server Components Default**: Nur `'use client'` wo noetig.
    2. **Data Fetching vorbereiten**: Erstelle Fetch-Funktionen/Hooks die die API aufrufen. Types werden vom Integrator finalisiert.
    3. **Server Actions fuer Mutations**: Nutze Server Actions fuer Form Submissions und Mutations (nicht Route Handler).
    4. **Bestehende UI-Komponenten nutzen**: shadcn/ui, Design-System des Projekts.
    5. **Error/Loading/Empty States**: Alle drei pro Daten-Komponente.
    6. **Accessibility**: Semantisches HTML, ARIA wo noetig, Keyboard-Nav.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 3. Backend-Builder (parallel)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- output_contract: |
    Pro Endpoint/Action:

    ### [METHOD] /api/pfad (oder Server Action)
    - **Datei:** `pfad/route.ts`
    - **Input:** [Request mit Types]
    - **Output:** [Response mit Types]
    - **Code:** [Vollstaendiger Code]

    Verwendet die Schema-Types. Alle Endpoints haben Input-Validation und Error-Handling.
- prompt: |
    Du bist der Backend-Builder in einem Full-Stack SWAT-Team. Du baust die API/Server-Seite des Features.

    ## Aufgabe
    {task_context}

    ## WICHTIG: API-Contract
    Der Schema-Designer definiert parallel die API-Contracts. Du kennst die Types noch nicht im Detail — verwende Platzhalter-Types die der Integrator spaeter mit den echten Types ersetzt.

    ## Implementierungs-Regeln

    1. **Input Validation**: Zod-Schemas an der Systemgrenze.
    2. **Auth-Checks**: Bestehende Auth-Middleware nutzen.
    3. **Error-Handling**: Konsistente Error-Responses.
    4. **Thin Handlers**: Business-Logik in Service-Funktionen, nicht im Handler.
    5. **Types exportieren**: Response-Types exportieren fuer Frontend-Konsumption.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 4. Integrator (sequentiell, nach allen parallelen Rollen)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- isolation: worktree
- consumes: [Schema-Designer.output, Frontend-Builder.output, Backend-Builder.output]
- output_contract: |
    ## Integrations-Ergebnis

    ### Shared Types
    [Finalisierte shared/types.ts basierend auf Schema-Designer]

    ### Frontend-Anpassungen
    [Welche Frontend-Dateien wurden angepasst um die echten Types zu nutzen]

    ### Backend-Anpassungen
    [Welche Backend-Dateien wurden angepasst]

    ### Vollstaendige Dateiliste
    [Alle Dateien die erstellt/geaendert wurden, mit finalem Code]
- prompt: |
    Du bist der Integrator in einem Full-Stack SWAT-Team. Du verbindest Frontend, Backend, und Schema zu einem funktionierenden Ganzen.

    ## Aufgabe
    {task_context}

    ## Schema + API-Contract
    {Schema-Designer.output}

    ## Frontend
    {Frontend-Builder.output}

    ## Backend
    {Backend-Builder.output}

    ## Integrations-Aufgaben

    1. **Shared Types finalisieren**: Erstelle die finale `shared/types.ts` basierend auf dem Schema-Designer Output.
    2. **Frontend verdrahten**: Ersetze Platzhalter-Types im Frontend mit den echten Types. Verbinde Data-Fetching mit den echten Endpoints.
    3. **Backend verdrahten**: Stelle sicher dass Backend-Endpoints die Schema-Types korrekt nutzen.
    4. **Imports aufloesen**: Alle Cross-Module Imports korrekt setzen.
    5. **Inkonsistenzen fixen**: Falls Frontend und Backend unterschiedliche Annahmen gemacht haben, harmonisiere sie.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 5. E2E-Tester (sequentiell, nach Integrator)
- model: sonnet
- subagent_type: general-purpose
- mode: bypassPermissions
- consumes: [Integrator.output]
- output_contract: |
    ## E2E Test Report

    ### Test-Dateien
    Pro Test-Datei:
    - **Datei:** `tests/pfad/test.test.ts`
    - **Code:** [Vollstaendiger Test-Code]

    ### User Flows getestet
    [Liste der getesteten End-to-End Flows]

    ### Integration Issues
    [Probleme die beim Testen aufgefallen sind, mit Fixes]
- prompt: |
    Du bist der E2E-Tester in einem Full-Stack SWAT-Team. Du pruefst ob das Feature end-to-end funktioniert.

    ## Aufgabe
    {task_context}

    ## Integrierter Code
    {Integrator.output}

    ## Test-Strategie

    1. **Happy Path**: Der primaere User-Flow funktioniert von UI bis DB und zurueck.
    2. **Error Cases**: Ungueltige Inputs, Netzwerk-Fehler, Auth-Failures.
    3. **Edge Cases**: Leere Listen, Grenzwerte, Sonderzeichen, gleichzeitige Requests.
    4. **API-Tests**: Direkte Endpoint-Tests (nicht nur ueber UI).
    5. **Type-Safety**: Stimmen die Runtime-Daten mit den TypeScript-Types ueberein?

    Nutze das bestehende Test-Framework. Schreibe vollstaendige Tests, keine Skizzen.

    ## Output-Format
    Exakt wie im Output-Contract definiert.

## Workflow-Graph

```
Schema-Designer ──────┐
                      │
Frontend-Builder ─────┼──→ Integrator ──→ E2E-Tester
                      │
Backend-Builder ──────┘
```

## Devil's Advocate Focus

- Stimmen die API-Contracts zwischen Frontend und Backend ueberein? (Typen, Feldnamen, Nullable)
- Was passiert bei Netzwerk-Fehlern zwischen Client und Server? Gibt es Retry-Logik?
- Ist die Aufgabenteilung sauber oder gibt es Business-Logik die auf beiden Seiten dupliziert ist?
- Sind die Server Actions korrekt abgesichert (Auth-Check, Input-Validation)?
- Was passiert bei gleichzeitigen Writes (Optimistic Updates vs. Pessimistic Locking)?
- Ist das Feature auch ohne JavaScript funktional (Progressive Enhancement)?

## Erfolgskriterien (Quality Gate)

- Shared Types werden konsistent in Frontend UND Backend verwendet
- Alle Endpoints haben Input-Validation und Auth-Checks
- Frontend hat Error/Loading/Empty States fuer alle Daten-Komponenten
- E2E Happy Path funktioniert
- Keine TypeScript Errors (strict mode)
- API-Contract ist konsistent (Frontend-Calls matchen Backend-Responses)
```

- [ ] **Step 2: Verify full-stack-feature.md**

Read back, verify:
- 5 roles with full prompts and output contracts
- 3 parallel (Schema + Frontend + Backend) → Integrator → E2E-Tester
- consumes fields are correct

- [ ] **Step 3: Commit**

```bash
git add references/teams/full-stack-feature.md
git commit -m "feat: add Full-Stack Feature SWAT team definition"
```

---

### Task 10: SWAT Team Builder (Meta-Team)

**Files:**
- Create: `references/teams/team-builder.md`

- [ ] **Step 1: Write team-builder.md**

Write to `/Users/reykz/repositorys/swarm-swat-skill/references/teams/team-builder.md`:

```markdown
---
name: SWAT Team Builder
triggers:
  keywords: [create team, new team, build team, generate team, team definition, new swat, custom team]
  file_patterns: []
  task_patterns: ["erstell.*team", "create.*swat", "build.*team.*defini", "new.*team"]
quality_mode: rigorous
max_workers: 3
---

# SWAT Team Builder (Meta-Team)

Generiert neue SWAT-Team-Definitionen fuer Use Cases die nicht von den bestehenden Teams abgedeckt werden.

## Wann verwenden

- User sagt explizit "Erstelle ein neues SWAT-Team fuer X"
- Dispatcher erkennt ein wiederkehrendes Muster das kein bestehendes Team abdeckt
- User will eine bestehende Team-Definition anpassen/verbessern

## Rollen

### 1. Research-Agent (parallel)
- model: sonnet
- subagent_type: Explore
- output_contract: |
    ## Use Case Analyse

    ### Domaene
    [Welche Domaene deckt das Team ab?]

    ### Typische Aufgaben
    [3-5 konkrete Aufgaben die dieses Team loesen soll]

    ### Bestehende Team-Referenzen
    [Welche bestehenden Teams sind aehnlich? Was kann uebernommen werden?]

    ### Rollen-Vorschlag
    [Welche Rollen braucht das Team? Name + Beschreibung + Abhaengigkeiten]

    ### Workflow-Vorschlag
    [Welche Rollen parallel, welche sequentiell?]

    ### Domain-spezifische Qualitaetskriterien
    [Woran misst man Erfolg in dieser Domaene?]
- prompt: |
    Du bist ein Research-Agent in einem SWAT Team Builder. Deine Aufgabe: den Use Case analysieren und Grundlagen fuer eine neue Team-Definition erarbeiten.

    ## Aufgabe
    Erstelle eine Analyse fuer folgendes neue SWAT-Team:
    {task_context}

    ## Analyse-Schritte

    1. **Bestehende Teams lesen**: Lies die Team-Definitionen in `references/teams/` um die bestehende Struktur und das Format zu verstehen.
    2. **Domaene verstehen**: Was sind die typischen Aufgaben, Challenges, und Qualitaetskriterien in dieser Domaene?
    3. **Rollen identifizieren**: Welche spezialisierten Rollen braucht ein Team fuer diese Domaene? (Minimum 2, Maximum 5)
    4. **Workflow designen**: Welche Rollen koennen parallel arbeiten? Welche brauchen Output von anderen?
    5. **Quality Criteria**: Was sind die domaenenspezifischen Erfolgskriterien?
    6. **DA Focus**: Welche Challenge-Fragen sind in dieser Domaene am wertvollsten?

    ## Output-Format
    Exakt wie im Output-Contract definiert.

### 2. Prompt-Engineer (sequentiell, nach Research-Agent)
- model: sonnet
- subagent_type: general-purpose
- consumes: [Research-Agent.output]
- output_contract: |
    ## Team-Definition (Draft)

    Vollstaendige Team-Definition im Standard-Format:

    ```markdown
    ---
    name: [Team-Name]
    triggers:
      keywords: [...]
      file_patterns: [...]
      task_patterns: [...]
    quality_mode: [standard/rigorous]
    max_workers: [N]
    ---

    # [Team-Name] SWAT Team

    [Beschreibung]

    ## Rollen
    [Vollstaendige Rollen mit model, subagent_type, mode, isolation, consumes, output_contract, prompt]

    ## Workflow-Graph
    [ASCII DAG]

    ## Devil's Advocate Focus
    [5-6 domaenenspezifische Challenge-Fragen]

    ## Erfolgskriterien (Quality Gate)
    [5-6 messbare Kriterien]
    ```

    Jede Rolle muss einen vollstaendigen, optimierten Prompt haben.
    Jede Rolle muss einen praezisen Output-Contract haben.
- prompt: |
    Du bist ein Prompt-Engineer in einem SWAT Team Builder. Du schreibst die vollstaendige Team-Definition basierend auf der Research-Analyse.

    ## Aufgabe
    Erstelle eine Team-Definition fuer:
    {task_context}

    ## Research-Ergebnisse
    {Research-Agent.output}

    ## Qualitaets-Regeln fuer Prompts

    1. **Spezifisch, nicht generisch**: Jeder Prompt muss domaenenspezifische Anweisungen enthalten. "Pruefe auf Fehler" ist schlecht. "Pruefe ob alle SQL-Queries parameterisiert sind und keine String-Concatenation enthalten" ist gut.
    2. **Output-Contract zuerst**: Definiere zuerst was der Agent produzieren soll (Output-Contract), dann schreibe den Prompt so dass er diesen Output produziert.
    3. **Self-contained**: Jeder Prompt muss alles enthalten was der Agent braucht. Kein Verweis auf "die allgemeinen Regeln" — alles inline.
    4. **{task_context} Platzhalter**: Jeder Prompt bekommt den Task-Kontext injiziert.
    5. **{Rollenname.output} Platzhalter**: Sequentielle Rollen bekommen die Outputs der vorherigen Rollen.
    6. **Bestehende Teams als Vorlage**: Orientiere dich am Format und der Qualitaet der bestehenden Team-Definitionen (besonders security-audit.md als Gold Standard).

    ## Format
    Die Team-Definition muss exakt dem Standard-Format folgen (siehe bestehende Teams in references/teams/).

    ## Output-Format
    Exakt wie im Output-Contract definiert — eine vollstaendige, einsatzbereite Team-Definition.

### 3. Validator (Quality Gate fuer die Team-Definition)
- model: opus
- subagent_type: general-purpose
- consumes: [Research-Agent.output, Prompt-Engineer.output]
- output_contract: |
    ## Validation Report

    ### Format-Check
    - [ ] Frontmatter vollstaendig (name, triggers, quality_mode, max_workers)
    - [ ] Alle Rollen haben: model, subagent_type, output_contract, prompt
    - [ ] Sequentielle Rollen haben: consumes
    - [ ] Workflow-Graph ist konsistent mit Rollen-Abhaengigkeiten
    - [ ] DA Focus hat mindestens 5 domaenenspezifische Fragen
    - [ ] Erfolgskriterien hat mindestens 5 messbare Kriterien

    ### Prompt-Qualitaet
    - [ ] Prompts sind spezifisch, nicht generisch
    - [ ] Output-Contracts sind praezise und konsumierbar
    - [ ] {task_context} Platzhalter in jedem Prompt vorhanden
    - [ ] {Rollenname.output} Platzhalter korrekt in sequentiellen Rollen

    ### Architektur
    - [ ] Rollen-Aufteilung ist sinnvoll (keine Redundanz, keine Luecken)
    - [ ] Workflow-Graph ist effizient (maximale Parallelisierung)
    - [ ] max_workers passt zur Rollen-Anzahl

    ### Verdict
    - **PASS** / **FAIL** mit spezifischen Issues und Fixes
- prompt: |
    Du bist ein Team-Definition Validator. Deine Aufgabe: pruefe ob die neue Team-Definition den Qualitaetsstandard der bestehenden Teams erfuellt.

    ## Research-Grundlage
    {Research-Agent.output}

    ## Team-Definition (Draft)
    {Prompt-Engineer.output}

    ## Validation-Checklist

    Pruefe JEDEN der folgenden Punkte. Markiere als bestanden oder gefailed mit spezifischem Issue und Fix-Vorschlag.

    ### Format
    1. Hat die Definition korrektes YAML-Frontmatter? (name, triggers, quality_mode, max_workers)
    2. Hat JEDE Rolle: model, subagent_type, output_contract, prompt?
    3. Haben sequentielle Rollen (die nach anderen kommen) ein `consumes` Feld?
    4. Stimmt der Workflow-Graph mit den tatsaechlichen Abhaengigkeiten ueberein?

    ### Prompt-Qualitaet
    5. Sind die Prompts SPEZIFISCH fuer die Domaene? (Nicht: "Pruefe auf Fehler". Sondern: spezifische Fehler-Kategorien fuer diese Domaene)
    6. Sind die Output-Contracts praezise genug dass der naechste Agent sie konsumieren kann?
    7. Hat jeder Prompt einen `{task_context}` Platzhalter?
    8. Haben sequentielle Prompts korrekte `{Rollenname.output}` Platzhalter?

    ### Architektur
    9. Ist die Rollen-Aufteilung sinnvoll? (Keine zwei Rollen die das Gleiche tun, keine fehlende Rolle)
    10. Ist der Workflow maximal parallelisiert? (Alles was parallel sein kann IST parallel)
    11. Sind die DA Focus Fragen wirklich domaenenspezifisch und nicht generisch?
    12. Sind die Erfolgskriterien messbar und spezifisch?

    ## Output-Format
    Exakt wie im Output-Contract definiert.

## Workflow-Graph

```
Research-Agent ──→ Prompt-Engineer ──→ Validator
```

## Nach erfolgreicher Validation

1. Team-Definition wird geschrieben nach `references/teams/<name>.md`
2. Team-Index im SKILL.md wird um den neuen Eintrag erweitert
3. User wird informiert: "Neues SWAT-Team '[Name]' erstellt und einsatzbereit."

## Devil's Advocate Focus

- Braucht es wirklich ein neues Team oder deckt ein bestehendes Team den Use Case ab?
- Sind die Prompts spezifisch genug um bessere Ergebnisse zu liefern als der Generic Fallback?
- Ist die Rollen-Aufteilung die richtige oder gibt es eine einfachere Dekomposition?
- Werden die Output-Contracts tatsaechlich vom naechsten Agent in der Kette konsumierbar sein?
- Hat das Team zu viele oder zu wenige Rollen?

## Erfolgskriterien (Quality Gate)

- Team-Definition folgt exakt dem Standard-Format
- Alle Prompts sind domaenenspezifisch (nicht copy-paste von generischen Anweisungen)
- Output-Contracts sind praezise und von nachfolgenden Agenten konsumierbar
- Workflow-Graph ist korrekt und effizient
- DA Focus und Erfolgskriterien sind domaenenspezifisch
- Die neue Definition ist mindestens so detailliert wie `security-audit.md` (Gold Standard)
```

- [ ] **Step 2: Verify team-builder.md**

Read back, verify:
- 3 roles: Research → Prompt-Engineer → Validator
- Validator is Opus (quality gate for the definition itself)
- Post-validation actions described (write file, update index)

- [ ] **Step 3: Commit**

```bash
git add references/teams/team-builder.md
git commit -m "feat: add SWAT Team Builder meta-team definition"
```

---

### Task 11: Migration from old agent-swarm

**Files:**
- Modify: `~/.claude/skills/` (symlink)
- Archive: `~/.claude/skills/agent-swarm/` (backup)

- [ ] **Step 1: Archive old agent-swarm skill**

```bash
cp -r ~/.claude/skills/agent-swarm ~/.claude/skills/agent-swarm.bak
```

- [ ] **Step 2: Remove old agent-swarm skill**

```bash
rm -rf ~/.claude/skills/agent-swarm
```

- [ ] **Step 3: Symlink new skill**

```bash
ln -s /Users/reykz/repositorys/swarm-swat-skill ~/.claude/skills/agent-swat-swarm
```

- [ ] **Step 4: Verify symlink**

```bash
ls -la ~/.claude/skills/agent-swat-swarm/SKILL.md
```

Expected: symlink pointing to repo, file exists and is readable.

- [ ] **Step 5: Verify old skill removed from active skills**

```bash
ls ~/.claude/skills/ | grep agent
```

Expected: `agent-swat-swarm` (symlink) and `agent-swarm.bak` (archive). No `agent-swarm` directory.

---

### Task 12: Validation

- [ ] **Step 1: Verify all files exist**

```bash
ls -la /Users/reykz/repositorys/swarm-swat-skill/SKILL.md
ls -la /Users/reykz/repositorys/swarm-swat-skill/references/quality-gate-protocol.md
ls -la /Users/reykz/repositorys/swarm-swat-skill/references/generic-swarm.md
ls -la /Users/reykz/repositorys/swarm-swat-skill/references/teams/security-audit.md
ls -la /Users/reykz/repositorys/swarm-swat-skill/references/teams/frontend-feature.md
ls -la /Users/reykz/repositorys/swarm-swat-skill/references/teams/backend-api.md
ls -la /Users/reykz/repositorys/swarm-swat-skill/references/teams/migration-refactor.md
ls -la /Users/reykz/repositorys/swarm-swat-skill/references/teams/full-stack-feature.md
ls -la /Users/reykz/repositorys/swarm-swat-skill/references/teams/team-builder.md
```

- [ ] **Step 2: Verify SKILL.md team index matches actual files**

Check that all 7 entries in the Team Index table correspond to actual files in `references/teams/` and `references/`.

- [ ] **Step 3: Verify cross-references**

Check consistency:
- Role names in `consumes` fields match actual role headings
- `{Rollenname.output}` placeholders in prompts reference roles that exist
- Workflow graphs match the parallel/sequential structure defined by `consumes`

- [ ] **Step 4: Verify symlink works**

```bash
cat ~/.claude/skills/agent-swat-swarm/SKILL.md | head -5
```

Expected: Shows the frontmatter of the new SKILL.md.

- [ ] **Step 5: Final commit with all files**

```bash
cd /Users/reykz/repositorys/swarm-swat-skill
git add -A
git status
git commit -m "feat: complete agent-swat-swarm skill with 6 SWAT teams + generic fallback"
```
