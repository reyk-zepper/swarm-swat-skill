# Agent SWAT Swarm

**Spezialisierte Agent-Teams fuer Claude Code** — vorkonfigurierte SWAT-Teams mit optimierten Prompts, Output-Contracts und Workflow-Graphs fuer spezifische Aufgabentypen.

**[English version below](#english)**

> Statt bei jeder komplexen Aufgabe von Grund auf zu improvisieren, wird ein spezialisiertes SWAT-Team deployed — wie eine Spezialeinheit, die genau fuer diesen Einsatz trainiert ist.

---

## Das Problem

Claude Code's Agent-System ist maechtig: parallele Agents, Worktrees, Quality Gates. Aber bei jedem komplexen Task muss der Lead-Agent von Null dekomponieren:

- Welche Worker-Rollen brauche ich?
- Welche Prompts gebe ich ihnen?
- In welcher Reihenfolge laufen sie?
- Wie uebergebe ich Outputs zwischen Agents?
- Woran messe ich die Qualitaet?

Das kostet Zeit, produziert inkonsistente Ergebnisse, und die Prompt-Qualitaet haengt davon ab, wie gut der Lead gerade improvisiert.

## Die Loesung: SWAT-Teams

Agent SWAT Swarm ersetzt generische Ad-hoc-Swarms durch **vorkonfigurierte Spezialeinheiten**:

- **Feste Rollen** mit voroptimierte Prompts — jeder Agent weiss exakt was er tun soll
- **Output-Contracts** — jeder Agent produziert Output in einem definierten Format, das der naechste Agent konsumieren kann
- **Workflow-Graphs** — klare Abhaengigkeiten: was parallel, was sequentiell
- **Team-spezifische Quality Gates** — mit domaenenspezifischen Devil's Advocate Fragen
- **Hybrid-Selektion** — automatische Team-Empfehlung oder expliziter User-Override

Der Unterschied ist wie zwischen einer zusammengewuerfelten Gruppe und einem eingespielten SWAT-Team: gleiche Werkzeuge, aber vorab koordinierte Rollen, Ablaeufe und Qualitaetskriterien.

## Wie es funktioniert

```
User-Aufgabe
    |
    v
[Dispatcher] Analysiert die Aufgabe
    |
    +-- Match gefunden? --> Laedt Team-Definition
    |                       Instanziiert Prompts
    |                       Spawnt Worker-Agents
    |
    +-- Kein Match? ------> Wiederkehrender Task-Typ?
    |                       |-- Ja --> Auto-Team-Builder (erstellt + deployed sofort)
    |                       +-- Nein -> Generic Fallback (Ad-hoc-Dekomposition)
    |
    v
[Parallele Worker] Arbeiten gleichzeitig an unabhaengigen Teilaufgaben
    |
    v
[Output-Validation] Lead prueft Outputs gegen Contracts
    |
    v
[Sequentielle Worker] Konsumieren Outputs der parallelen Phase
    |
    v
[Quality Gate] Opus-Agent prueft alles + Devil's Advocate
    |
    +-- PASS -----------> Ergebnis an User
    +-- FAIL -----------> Rework (max 2 Zyklen)
    |
    v
[Post-Swarm Learning] (nur nach Rework)
    Analysiert QG-Failures, schlaegt Prompt-Verbesserungen vor
    Verbessert Team-Definition auf Disk --> naechste Session profitiert
```

### Team-Selektion (Hybrid)

1. **Automatisch**: Der Dispatcher analysiert die Aufgabe und waehlt das beste Team per natuerlichsprachlichem Matching — kein starres Keyword-Scoring, sondern echtes Verstaendnis des Aufgabentyps.

2. **User-Override**: Jederzeit ueberstimmbar:
   - Slash-Command: `/agent-swat-swarm security-audit`
   - Natuerliche Sprache: "nutze das Frontend Team"

3. **Fallback**: Wenn kein Team passt, wird der Generic Swarm deployed — dynamische Dekomposition wie bisher.

## SWAT-Teams

### Security Audit

```
Scanner ──────────┐
                  ├──> Fix-Proposer ──> Report-Writer
Exploit-Analyst ──┘
```

Systematische Sicherheitsanalyse: Scanner prueft 9 OWASP-Kategorien, Exploit-Analyst bewertet Angriffsvektoren per STRIDE + CVSS, Fix-Proposer schreibt konkrete Code-Fixes, Report-Writer kompiliert einen Executive-tauglichen Security Report.

**Quality Mode:** Rigorous (separater QA + Devil's Advocate)

### Frontend Feature

```
UI-Architekt ──────┐
                   ├──> Integrator ──> A11y-Reviewer
Component-Builder ─┘
```

Von Architektur bis Accessibility: UI-Architekt designed den Komponenten-Baum (RSC-first), Component-Builder implementiert, Integrator verdrahtet alles, A11y-Reviewer prueft WCAG 2.1 AA Compliance.

**Quality Mode:** Standard

### Backend API

```
Schema-Designer ──────┐
                      ├──> Test-Writer ──> Integration-Tester
Endpoint-Builder ─────┘
```

Schema-Design + API-Endpoints parallel, dann Test-Suite und Integration-Verification. Jeder Endpoint bekommt Input-Validation, Auth-Checks, und die Test-Suite deckt Happy Path + Error Cases ab.

**Quality Mode:** Standard

### Migration/Refactor

```
Impact-Analyst ──────┐
                     ├──> Executor ──> Regression-Tester
Migration-Planner ───┘
```

Sichere Migrationen: Impact-Analyse (direkte + transitive Abhaengigkeiten), schrittweiser Plan (jeder Schritt reversibel), Ausfuehrung, und Regression-Testing gegen die Impact-Analyse.

**Quality Mode:** Rigorous

### Full-Stack Feature

```
Schema-Designer ──────┐
                      |
Frontend-Builder ─────┼──> Integrator ──> E2E-Tester
                      |
Backend-Builder ──────┘
```

End-to-end Features: Schema-Designer definiert shared Types (eine Source of Truth), Frontend und Backend arbeiten parallel gegen Platzhalter-Types, Integrator harmonisiert alles, E2E-Tester verifiziert den kompletten Flow.

**Quality Mode:** Standard (rigorous bei produktionskritischen Features)

### SWAT Team Builder (Meta-Team)

```
Research-Agent ──> Prompt-Engineer ──> Validator
```

Das selbst-erweiternde Team: Analysiert einen Use Case, schreibt eine vollstaendige Team-Definition mit optimierten Prompts, und validiert sie gegen den Qualitaetsstandard der bestehenden Teams. Neue SWAT-Teams koennen jederzeit generiert werden.

**Quality Mode:** Rigorous (Opus Validator als QG fuer die Definition selbst)

## Anatomie eines SWAT-Teams

Jede Team-Definition ist eine Markdown-Datei mit dieser Struktur:

```yaml
---
name: Team-Name
triggers:
  keywords: [relevante, signalwoerter]
  file_patterns: ["**/relevante/**", "**/*.dateien"]
  task_patterns: ["regex.*fuer.*aufgaben"]
quality_mode: standard | rigorous
max_workers: N
---
```

Gefolgt von:

- **Rollen** — Jede mit `model`, `subagent_type`, `mode`, `isolation`, `consumes` (Abhaengigkeiten), einem vollstaendigen **Prompt-Template** und einem praezisen **Output-Contract**
- **Workflow-Graph** — ASCII-DAG der Ausfuehrungsreihenfolge
- **Devil's Advocate Focus** — 5-6 domaenenspezifische Challenge-Fragen
- **Erfolgskriterien** — 5-6 messbare Kriterien fuer die Quality Gate

### Warum Prompt-Templates der Schluessel sind

Der gesamte Wert eines SWAT-Teams liegt in den voroptimierte Prompts. Ein generischer Swarm gibt einem Agent "mach Security-Review". Ein SWAT-Team gibt ihm:

- 9 spezifische OWASP-Kategorien zum systematischen Durchpruefen
- Exakte Patterns wonach gesucht werden soll (String-Concatenation in Queries, hardcoded Credentials, fehlende Rate-Limiting...)
- Ein praezises Output-Format das der naechste Agent direkt konsumieren kann
- Klare Grenzen was dieser Agent verantwortet und was nicht

Das ist der Unterschied zwischen "pruef mal auf Sicherheit" und einem professionellen Penetration-Test-Protokoll.

### Output-Contracts: Die Kette die funktioniert

Output-Contracts loesen ein fundamentales Problem paralleler Agent-Systeme: Wie uebergibt Agent A seinen Output an Agent B, ohne dass B raten muss was er bekommt?

```
Scanner produziert:
  ### [CRITICAL] SQL Injection in user.ts:42
  - Datei: src/api/user.ts:42
  - Kategorie: A03:2021 Injection
  - Beschreibung: String-Concatenation in SQL Query
  - Code: `db.query("SELECT * FROM users WHERE id = " + userId)`

Fix-Proposer konsumiert exakt dieses Format und produziert:
  ### Fix fuer: SQL Injection in user.ts:42
  - Severity: CRITICAL
  - Datei: src/api/user.ts
  - Aenderung: [Parameterized Query Code]
```

Jeder Agent weiss was er bekommt und was er liefern muss.

## Quality Gate

Jeder SWAT-Einsatz endet mit einer Quality Gate — nie optional:

| Modus | Wann | Agents | Kosten |
|-------|------|--------|--------|
| **Standard** | Die meisten Tasks | 1x Opus (QA + DA kombiniert) | 1 Opus Call |
| **Rigorous** | Produktionskritisch, hohe Fehlerkosten | 2x Opus parallel (QA + DA separat) | 2 Opus Calls |

### Devil's Advocate

Jedes Team definiert eigene DA-Challenge-Fragen. Kein generisches "sieht das gut aus?", sondern:

| Team | Der DA fragt |
|------|-------------|
| Security Audit | Welche Angriffsvektoren wurden NICHT geprueft? Sind die Fixes Symptombekaempfung? |
| Frontend Feature | Funktioniert es bei 320px? Sind alle Elemente per Keyboard erreichbar? |
| Migration | Welche Breaking Changes wurden uebersehen? Ist Rollback realistisch? |
| Backend API | Wie skaliert das bei 100x Load? Race Conditions? |

### Rework-Protokoll

Wenn die Quality Gate FAIL zurueckgibt:

1. Issues werden geparst
2. Targeted Fix-Agents werden gespawnt (oder bestehende Worker per `SendMessage` fortgesetzt)
3. Quality Gate laeuft erneut
4. Maximum 2 Rework-Zyklen — danach bestes Ergebnis mit QG-Notes an User

## Trainer (Selbstlernendes System)

Der Trainer ist der Coach aller SWAT-Teams — er erstellt, verbessert und auditiert Teams automatisch:

### Auto-Creation

Wenn kein Team zu einer Aufgabe passt und der Task-Typ wiederkehrend ist, erstellt der Trainer automatisch ein neues SWAT-Team:

1. Dispatcher erkennt: kein Match, aber wiederkehrender Task-Typ
2. User wird gefragt: "Soll ich ein neues Team erstellen?"
3. Team Builder laeuft (Research → Prompt-Engineer → Validator)
4. Neues Team wird auf Disk geschrieben und **sofort** fuer den aktuellen Task deployed
5. Ab naechster Session ist das Team permanent verfuegbar

### Post-Swarm Learning

Nach jedem Einsatz mit Rework (QG hat Issues gefunden) analysiert der Trainer:

- **Was** hat der QG bemaengelt?
- **Welche Prompt-Schwaeche** hat das verursacht?
- **Welcher Fix** behebt das Problem? (max 5 Zeilen, targeted)

Der User muss jede Aenderung bestaetigen. Aenderungen werden direkt in die Team-Definition geschrieben — der Skill wird mit jedem Einsatz besser.

### Team Fitness Audit

On-demand per `/agent-swat-swarm team-audit`:

- Strukturelle Integritaet aller Teams pruefen
- Prompt-Qualitaet bewerten
- Changelog reviewen (wiederkehrende Probleme?)
- Coverage Gaps identifizieren (fehlende Teams?)

## Installation

### Claude Code Skill

```bash
# Repository klonen
git clone https://github.com/reyk-zepper/swarm-swat-skill.git

# Als Claude Code Skill installieren (Symlink)
ln -s /pfad/zu/swarm-swat-skill ~/.claude/skills/agent-swat-swarm
```

Der Skill wird automatisch geladen wenn Claude eine komplexe Aufgabe erkennt die von einem SWAT-Team profitieren wuerde.

### Voraussetzungen

- Claude Code (CLI, Desktop App, oder IDE Extension)
- Claude Opus + Sonnet Zugang (Opus fuer Lead/QG, Sonnet fuer Worker)

## Nutzung

### Automatisch (empfohlen)

Einfach arbeiten. Der Dispatcher erkennt automatisch wann ein SWAT-Team Mehrwert bringt:

```
> Baue eine User-Verwaltung mit Login, Profil-Seite und API-Endpoints

SWAT Team: Full-Stack Feature
Grund: End-to-end Feature ueber Frontend + Backend + Tests
Rollen: Schema-Designer, Frontend-Builder, Backend-Builder, Integrator, E2E-Tester
Agents: 5x Sonnet (Worker) + 1x Opus (Quality Gate) = 6 Agents
Quality Mode: standard
Override: /agent-swat-swarm <anderes-team>
```

### Expliziter Override

```
> /agent-swat-swarm security-audit
```

Oder natuerlichsprachlich:

```
> Nutze das Security Team um die Auth-Middleware zu pruefen
```

### Neues Team erstellen

```
> Erstelle ein SWAT-Team fuer Performance-Optimierung
```

Der Team Builder analysiert den Use Case, schreibt Prompts und Output-Contracts, und validiert die neue Definition.

## Architektur-Entscheidungen

### Warum ein Skill statt viele?

Claude Code's Hook-System hat ein `MAX_SKILLS` Cap von 3 pro Invocation. Bei separaten Skills pro Team wuerden SWAT-Teams untereinander und mit anderen Skills um diese 3 Slots konkurrieren. Mit einem einzigen Dispatcher-Skill wird nur 1 Slot belegt — Team-Definitionen werden on-demand per `Read` geladen.

### Warum natuerlichsprachliches Matching statt Scoring?

Eine fruehe Version hatte eine gewichtete Scoring-Formel (`keyword_hits * 2 + file_hits * 3`). Das ist Pseudo-Praezision. Claude ist ein LLM — es versteht Kontext besser als jede Formel. "Refactor the auth middleware" ist ein Migration-Task, keine Security-Aufgabe, auch wenn "auth" ein Security-Keyword ist. Natuerlichsprachliches Matching trifft die richtige Entscheidung.

### Warum Output-Contracts statt freie Textausgabe?

Ohne Contracts muss der Lead bei jedem sequentiellen Uebergang den Output des vorherigen Agents interpretieren und zusammenfassen. Mit Contracts ist das Format vorab definiert — der naechste Agent kann direkt konsumieren. Das eliminiert Interpretations-Overhead und Informationsverlust.

### Warum Sonnet fuer Worker, Opus fuer Quality Gate?

Sonnet ist schnell und kosteneffizient — perfekt fuer gut-scopte Teilaufgaben mit klaren Prompts. Opus ist langsamer aber deutlich besser bei kritischer Bewertung, Muster-Erkennung ueber grosse Kontexte, und strategischem Denken. Die Kombination maximiert Speed-to-Quality Ratio: schnelle Ausfuehrung, gruendliche Pruefung.

## Dateistruktur

```
agent-swat-swarm/
|-- SKILL.md                              # Dispatcher (Team-Selektion + Routing)
|-- references/
|   |-- teams/
|   |   |-- security-audit.md             # Security Audit SWAT Team
|   |   |-- frontend-feature.md           # Frontend Feature SWAT Team
|   |   |-- backend-api.md                # Backend API SWAT Team
|   |   |-- migration-refactor.md         # Migration/Refactor SWAT Team
|   |   |-- full-stack-feature.md         # Full-Stack Feature SWAT Team
|   |   +-- team-builder.md              # SWAT Team Builder (Meta-Team)
|   |-- orchestration-protocol.md         # Ausfuehrungsprotokoll (Phasen 1-5, Rework)
|   |-- trainer-protocol.md              # Auto-Creation, Post-Swarm Learning, Audit
|   |-- quality-gate-protocol.md          # QG-Spec mit DA-Integration
|   +-- generic-swarm.md                 # Fallback fuer ungematchte Tasks
+-- docs/
    |-- plans/
    |   +-- 2026-03-29-agent-swat-swarm-design.md
    +-- superpowers/
        +-- plans/
            +-- 2026-03-29-agent-swat-swarm-implementation.md
```

**~4.500 Zeilen** ueber 12 Dateien, davon **24 Agent-Rollen** mit vollstaendigen Prompt-Templates und Output-Contracts.

## Eigene Teams erstellen

Zwei Wege:

### 1. Team Builder (empfohlen)

Sag Claude einfach was du brauchst:

```
> Erstelle ein SWAT-Team fuer Datenbank-Performance-Optimierung
```

Der Team Builder (Research-Agent -> Prompt-Engineer -> Opus-Validator) generiert eine vollstaendige Definition im Standard-Format.

### 2. Manuell

Erstelle eine Markdown-Datei in `references/teams/` nach dem Standard-Format (siehe bestehende Teams als Vorlage). Fuege einen Eintrag im Team-Index in `SKILL.md` hinzu.

## Konzeptioneller Hintergrund

### Von generisch zu spezialisiert

Die Evolution von Agent-Swarms in Claude Code:

1. **Einzelner Agent** — Claude bearbeitet alles sequentiell. Funktioniert, aber langsam bei komplexen Tasks.

2. **Generischer Swarm** — Parallele Agents mit dynamischer Dekomposition. Schneller, aber der Lead muss bei jedem Einsatz improvisieren. Prompt-Qualitaet variiert.

3. **SWAT-Teams** (dieses Projekt) — Vorkonfigurierte Spezialeinheiten. Der Lead waehlt und deployed, statt zu improvisieren. Konsistente, hohe Prompt-Qualitaet. Optimierte Workflows pro Domaene.

### Die SWAT-Metapher

Eine Polizei-Spezialeinheit (SWAT) unterscheidet sich von einer Streife nicht durch bessere Ausruestung, sondern durch:

- **Vordefinierte Rollen** — Jedes Mitglied weiss seine Position
- **Eingespielte Ablaeufe** — Kein Improvisieren unter Druck
- **Domaenen-Expertise** — Training fuer spezifische Szenarien
- **Qualitaetskontrolle** — Nachbesprechung nach jedem Einsatz

Genau das uebertraegt dieses Projekt auf Claude Code's Agent-System.

## Limitierungen

- **Kein Team-Chaining** — Aktuell kann nur ein Team pro Task deployed werden. Multi-Team Tasks (z.B. "Build Feature + Security Audit") erfordern separate Aufrufe.
- **Team Builder nutzt Sonnet** — Die Quality Gate (Opus Validator) faengt Format-Issues ab, aber tiefe Domaenen-Expertise in generierten Prompts ist limitiert. Upgrade auf Opus fuer den Prompt-Engineer ist empfohlen.
- **Post-Swarm Learning ist konservativ** — Max 1 Edit pro Swarm, max 5 Zeilen. Groessere Prompt-Ueberarbeitungen erfordern manuellen Team Builder Einsatz.

## Roadmap (Phase 2)

- [ ] Team-Chaining fuer Multi-Team Tasks
- [ ] Prompt-Evolution: A/B-Testing von Prompt-Varianten
- [ ] Team-Komposition: Sub-Teams als Bausteine
- [ ] Performance-Metriken: Tracking von Swarm-Dauer, Rework-Rate, QG-Pass-Rate

## Lizenz

MIT

---

Gebaut mit Claude Code. Das erste SWAT-Team wurde von seinem eigenen Vorgaenger (dem generischen `agent-swarm`) deployed — 6 parallele Sonnet-Worker, Opus Quality Gate, 1 Rework-Zyklus.

---

<a name="english"></a>

# Agent SWAT Swarm (English)

**Specialized agent teams for Claude Code** — pre-configured SWAT teams with optimized prompts, output contracts, and workflow graphs for specific task types.

> Instead of improvising from scratch on every complex task, deploy a specialized SWAT team — like a special forces unit trained for exactly this type of mission.

---

## The Problem

Claude Code's agent system is powerful: parallel agents, worktrees, quality gates. But on every complex task, the lead agent must decompose from zero:

- Which worker roles do I need?
- What prompts do I give them?
- In what order do they run?
- How do I pass outputs between agents?
- How do I measure quality?

This costs time, produces inconsistent results, and prompt quality depends on how well the lead happens to improvise.

## The Solution: SWAT Teams

Agent SWAT Swarm replaces generic ad-hoc swarms with **pre-configured specialist units**:

- **Fixed roles** with pre-optimized prompts — every agent knows exactly what to do
- **Output contracts** — every agent produces output in a defined format that the next agent can consume directly
- **Workflow graphs** — clear dependencies: what runs in parallel, what runs sequentially
- **Team-specific quality gates** — with domain-specific Devil's Advocate questions
- **Hybrid selection** — automatic team recommendation or explicit user override

The difference is like a thrown-together group vs. a well-drilled SWAT team: same tools, but pre-coordinated roles, workflows, and quality criteria.

## How It Works

```
User Task
    |
    v
[Dispatcher] Analyzes the task
    |
    +-- Match found? -----> Loads team definition
    |                       Instantiates prompts
    |                       Spawns worker agents
    |
    +-- No match? --------> Recurring task type?
    |                       |-- Yes --> Auto-Team-Builder (creates + deploys immediately)
    |                       +-- No ---> Generic fallback (ad-hoc decomposition)
    |
    v
[Parallel Workers] Work simultaneously on independent subtasks
    |
    v
[Output Validation] Lead checks outputs against contracts
    |
    v
[Sequential Workers] Consume outputs from the parallel phase
    |
    v
[Quality Gate] Opus agent reviews everything + Devil's Advocate
    |
    +-- PASS ------------> Result to user
    +-- FAIL ------------> Rework (max 2 cycles)
    |
    v
[Post-Swarm Learning] (only after rework)
    Analyzes QG failures, proposes prompt improvements
    Improves team definition on disk --> next session benefits
```

### Team Selection (Hybrid)

1. **Automatic**: The dispatcher analyzes the task and selects the best team via natural language matching — no rigid keyword scoring, but genuine understanding of the task type.

2. **User override**: Override at any time:
   - Slash command: `/agent-swat-swarm security-audit`
   - Natural language: "use the Frontend team"

3. **Fallback**: If no team fits, the generic swarm is deployed — dynamic decomposition as before.

## SWAT Teams

### Security Audit

```
Scanner ──────────┐
                  ├──> Fix-Proposer ──> Report-Writer
Exploit-Analyst ──┘
```

Systematic security analysis: Scanner checks 9 OWASP categories, Exploit-Analyst evaluates attack vectors via STRIDE + CVSS, Fix-Proposer writes concrete code fixes, Report-Writer compiles an executive-ready security report.

**Quality Mode:** Rigorous (separate QA + Devil's Advocate)

### Frontend Feature

```
UI-Architect ──────┐
                   ├──> Integrator ──> A11y-Reviewer
Component-Builder ─┘
```

From architecture to accessibility: UI-Architect designs the component tree (RSC-first), Component-Builder implements, Integrator wires everything together, A11y-Reviewer checks WCAG 2.1 AA compliance.

**Quality Mode:** Standard

### Backend API

```
Schema-Designer ──────┐
                      ├──> Test-Writer ──> Integration-Tester
Endpoint-Builder ─────┘
```

Schema design + API endpoints in parallel, then test suite and integration verification. Every endpoint gets input validation, auth checks, and the test suite covers happy path + error cases.

**Quality Mode:** Standard

### Migration/Refactor

```
Impact-Analyst ──────┐
                     ├──> Executor ──> Regression-Tester
Migration-Planner ───┘
```

Safe migrations: impact analysis (direct + transitive dependencies), step-by-step plan (every step reversible), execution, and regression testing against the impact analysis.

**Quality Mode:** Rigorous

### Full-Stack Feature

```
Schema-Designer ──────┐
                      |
Frontend-Builder ─────┼──> Integrator ──> E2E-Tester
                      |
Backend-Builder ──────┘
```

End-to-end features: Schema-Designer defines shared types (single source of truth), frontend and backend work in parallel against placeholder types, Integrator harmonizes everything, E2E-Tester verifies the complete flow.

**Quality Mode:** Standard (rigorous for production-critical features)

### SWAT Team Builder (Meta-Team)

```
Research-Agent ──> Prompt-Engineer ──> Validator
```

The self-extending team: analyzes a use case, writes a complete team definition with optimized prompts, and validates it against the quality standard of existing teams. New SWAT teams can be generated at any time.

**Quality Mode:** Rigorous (Opus Validator as QG for the definition itself)

## Anatomy of a SWAT Team

Each team definition is a Markdown file with this structure:

```yaml
---
name: Team-Name
triggers:
  keywords: [relevant, signal, words]
  file_patterns: ["**/relevant/**", "**/*.files"]
  task_patterns: ["regex.*for.*tasks"]
quality_mode: standard | rigorous
max_workers: N
---
```

Followed by:

- **Roles** — Each with `model`, `subagent_type`, `mode`, `isolation`, `consumes` (dependencies), a complete **prompt template**, and a precise **output contract**
- **Workflow graph** — ASCII DAG of execution order
- **Devil's Advocate focus** — 5-6 domain-specific challenge questions
- **Success criteria** — 5-6 measurable criteria for the quality gate

### Why Prompt Templates Are the Key

The entire value of a SWAT team lies in the pre-optimized prompts. A generic swarm tells an agent "do a security review". A SWAT team gives it:

- 9 specific OWASP categories to systematically check
- Exact patterns to search for (string concatenation in queries, hardcoded credentials, missing rate limiting...)
- A precise output format that the next agent can consume directly
- Clear boundaries of what this agent is responsible for and what it is not

That's the difference between "check for security stuff" and a professional penetration testing protocol.

### Output Contracts: The Chain That Works

Output contracts solve a fundamental problem of parallel agent systems: How does Agent A pass its output to Agent B without B having to guess what it's getting?

```
Scanner produces:
  ### [CRITICAL] SQL Injection in user.ts:42
  - File: src/api/user.ts:42
  - Category: A03:2021 Injection
  - Description: String concatenation in SQL query
  - Code: `db.query("SELECT * FROM users WHERE id = " + userId)`

Fix-Proposer consumes exactly this format and produces:
  ### Fix for: SQL Injection in user.ts:42
  - Severity: CRITICAL
  - File: src/api/user.ts
  - Change: [Parameterized Query Code]
```

Every agent knows what it receives and what it must deliver.

## Quality Gate

Every SWAT deployment ends with a quality gate — never optional:

| Mode | When | Agents | Cost |
|------|------|--------|------|
| **Standard** | Most tasks | 1x Opus (QA + DA combined) | 1 Opus call |
| **Rigorous** | Production-critical, high error cost | 2x Opus in parallel (QA + DA separate) | 2 Opus calls |

### Devil's Advocate

Each team defines its own DA challenge questions. No generic "does this look good?", but instead:

| Team | The DA asks |
|------|------------|
| Security Audit | Which attack vectors were NOT checked? Are the fixes just symptom treatment? |
| Frontend Feature | Does it work at 320px? Are all elements keyboard-accessible? |
| Migration | Which breaking changes were overlooked? Is rollback realistic? |
| Backend API | How does it scale at 100x load? Race conditions? |

### Rework Protocol

When the quality gate returns FAIL:

1. Issues are parsed
2. Targeted fix agents are spawned (or existing workers continued via `SendMessage`)
3. Quality gate runs again
4. Maximum 2 rework cycles — then best result with QG notes presented to user

## Trainer (Self-Learning System)

The Trainer is the coach of all SWAT teams — it creates, improves, and audits teams automatically:

### Auto-Creation

When no team matches a task and the task type is recurring, the Trainer automatically creates a new SWAT team:

1. Dispatcher detects: no match, but recurring task type
2. User is asked: "Should I create a new team?"
3. Team Builder runs (Research → Prompt-Engineer → Validator)
4. New team is written to disk and **immediately** deployed for the current task
5. From the next session onwards, the team is permanently available

### Post-Swarm Learning

After every deployment with rework (QG found issues), the Trainer analyzes:

- **What** did the QG flag?
- **Which prompt weakness** caused it?
- **What fix** resolves the problem? (max 5 lines, targeted)

The user must approve every change. Changes are written directly to the team definition file — the skill gets better with every deployment.

### Team Fitness Audit

On-demand via `/agent-swat-swarm team-audit`:

- Check structural integrity of all teams
- Evaluate prompt quality
- Review changelog (recurring problems?)
- Identify coverage gaps (missing teams?)

## Installation

### Claude Code Skill

```bash
# Clone the repository
git clone https://github.com/reyk-zepper/swarm-swat-skill.git

# Install as Claude Code skill (symlink)
ln -s /path/to/swarm-swat-skill ~/.claude/skills/agent-swat-swarm
```

The skill is automatically loaded when Claude detects a complex task that would benefit from a SWAT team.

### Prerequisites

- Claude Code (CLI, Desktop App, or IDE Extension)
- Claude Opus + Sonnet access (Opus for Lead/QG, Sonnet for Workers)

## Usage

### Automatic (recommended)

Just work. The dispatcher automatically detects when a SWAT team adds value:

```
> Build a user management system with login, profile page, and API endpoints

SWAT Team: Full-Stack Feature
Reason: End-to-end feature across frontend + backend + tests
Roles: Schema-Designer, Frontend-Builder, Backend-Builder, Integrator, E2E-Tester
Agents: 5x Sonnet (Worker) + 1x Opus (Quality Gate) = 6 Agents
Quality Mode: standard
Override: /agent-swat-swarm <other-team>
```

### Explicit Override

```
> /agent-swat-swarm security-audit
```

Or in natural language:

```
> Use the Security team to review the auth middleware
```

### Create a New Team

```
> Create a SWAT team for performance optimization
```

The Team Builder analyzes the use case, writes prompts and output contracts, and validates the new definition.

## Architecture Decisions

### Why one skill instead of many?

Claude Code's hook system has a `MAX_SKILLS` cap of 3 per invocation. With separate skills per team, SWAT teams would compete with each other and other skills for these 3 slots. With a single dispatcher skill, only 1 slot is used — team definitions are loaded on-demand via `Read`.

### Why natural language matching instead of scoring?

An early version had a weighted scoring formula (`keyword_hits * 2 + file_hits * 3`). That's pseudo-precision. Claude is an LLM — it understands context better than any formula. "Refactor the auth middleware" is a migration task, not a security task, even though "auth" is a security keyword. Natural language matching makes the right call.

### Why output contracts instead of free text output?

Without contracts, the lead must interpret and summarize the output of the previous agent at every sequential handoff. With contracts, the format is pre-defined — the next agent can consume directly. This eliminates interpretation overhead and information loss.

### Why Sonnet for workers, Opus for quality gate?

Sonnet is fast and cost-efficient — perfect for well-scoped subtasks with clear prompts. Opus is slower but significantly better at critical evaluation, pattern recognition across large contexts, and strategic thinking. The combination maximizes the speed-to-quality ratio: fast execution, thorough review.

## File Structure

```
agent-swat-swarm/
|-- SKILL.md                              # Dispatcher (team selection + routing)
|-- references/
|   |-- teams/
|   |   |-- security-audit.md             # Security Audit SWAT Team
|   |   |-- frontend-feature.md           # Frontend Feature SWAT Team
|   |   |-- backend-api.md                # Backend API SWAT Team
|   |   |-- migration-refactor.md         # Migration/Refactor SWAT Team
|   |   |-- full-stack-feature.md         # Full-Stack Feature SWAT Team
|   |   +-- team-builder.md              # SWAT Team Builder (Meta-Team)
|   |-- orchestration-protocol.md         # Execution protocol (phases 1-5, rework)
|   |-- trainer-protocol.md              # Auto-creation, post-swarm learning, audit
|   |-- quality-gate-protocol.md          # QG spec with DA integration
|   +-- generic-swarm.md                 # Fallback for unmatched tasks
+-- docs/
    |-- plans/
    |   +-- 2026-03-29-agent-swat-swarm-design.md
    +-- superpowers/
        +-- plans/
            +-- 2026-03-29-agent-swat-swarm-implementation.md
```

**~4,500 lines** across 12 files, including **24 agent roles** with complete prompt templates and output contracts.

## Creating Your Own Teams

Two ways:

### 1. Team Builder (recommended)

Just tell Claude what you need:

```
> Create a SWAT team for database performance optimization
```

The Team Builder (Research-Agent -> Prompt-Engineer -> Opus-Validator) generates a complete definition in the standard format.

### 2. Manual

Create a Markdown file in `references/teams/` following the standard format (see existing teams as templates). Add an entry to the team index in `SKILL.md`.

## Conceptual Background

### From Generic to Specialized

The evolution of agent swarms in Claude Code:

1. **Single agent** — Claude handles everything sequentially. Works, but slow for complex tasks.

2. **Generic swarm** — Parallel agents with dynamic decomposition. Faster, but the lead must improvise on every deployment. Prompt quality varies.

3. **SWAT teams** (this project) — Pre-configured specialist units. The lead selects and deploys instead of improvising. Consistent, high prompt quality. Optimized workflows per domain.

### The SWAT Metaphor

A police special forces unit (SWAT) differs from a patrol not by having better equipment, but by having:

- **Pre-defined roles** — Every member knows their position
- **Rehearsed procedures** — No improvising under pressure
- **Domain expertise** — Training for specific scenarios
- **Quality control** — Debrief after every deployment

This project transfers exactly this concept to Claude Code's agent system.

## Limitations

- **No team chaining** — Currently only one team can be deployed per task. Multi-team tasks (e.g., "build feature + security audit") require separate invocations.
- **Team Builder uses Sonnet** — The quality gate (Opus Validator) catches format issues, but deep domain expertise in generated prompts is limited. Upgrading the Prompt-Engineer to Opus is recommended.
- **Post-Swarm Learning is conservative** — Max 1 edit per swarm, max 5 lines. Larger prompt overhauls require manual Team Builder invocation.

## Roadmap (Phase 2)

- [ ] Team chaining for multi-team tasks
- [ ] Prompt evolution: A/B testing of prompt variants
- [ ] Team composition: sub-teams as building blocks
- [ ] Performance metrics: tracking swarm duration, rework rate, QG pass rate

## License

MIT

---

Built with Claude Code. The first SWAT team was deployed by its own predecessor (the generic `agent-swarm`) — 6 parallel Sonnet workers, Opus quality gate, 1 rework cycle.
