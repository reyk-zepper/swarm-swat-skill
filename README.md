# Agent SWAT Swarm

**Spezialisierte Agent-Teams fuer Claude Code** — vorkonfigurierte SWAT-Teams mit optimierten Prompts, Output-Contracts und Workflow-Graphs fuer spezifische Aufgabentypen.

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
    +-- Kein Match? ------> Generic Fallback (Ad-hoc-Dekomposition)
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
|-- SKILL.md                              # Dispatcher + Orchestrierungsprotokoll
|-- references/
|   |-- teams/
|   |   |-- security-audit.md             # Security Audit SWAT Team
|   |   |-- frontend-feature.md           # Frontend Feature SWAT Team
|   |   |-- backend-api.md                # Backend API SWAT Team
|   |   |-- migration-refactor.md         # Migration/Refactor SWAT Team
|   |   |-- full-stack-feature.md         # Full-Stack Feature SWAT Team
|   |   +-- team-builder.md              # SWAT Team Builder (Meta-Team)
|   |-- quality-gate-protocol.md          # QG-Spec mit DA-Integration
|   +-- generic-swarm.md                 # Fallback fuer ungematchte Tasks
+-- docs/
    |-- plans/
    |   +-- 2026-03-29-agent-swat-swarm-design.md
    +-- superpowers/
        +-- plans/
            +-- 2026-03-29-agent-swat-swarm-implementation.md
```

**~3.500 Zeilen** ueber 10 Dateien, davon **24 Agent-Rollen** mit vollstaendigen Prompt-Templates und Output-Contracts.

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

- **Prompt-Qualitaet ist statisch** — Die initialen Prompts werden nicht automatisch verbessert. Feedback-Loops und A/B-Testing sind Phase-2 Features.
- **Kein Team-Chaining** — Aktuell kann nur ein Team pro Task deployed werden. Multi-Team Tasks (z.B. "Build Feature + Security Audit") erfordern separate Aufrufe.
- **Team Builder nutzt Sonnet** — Die Quality Gate (Opus Validator) faengt Format-Issues ab, aber tiefe Domaenen-Expertise in generierten Prompts ist limitiert. Upgrade auf Opus fuer den Prompt-Engineer ist empfohlen.

## Roadmap (Phase 2)

- [ ] Feedback-Loop: Post-Mortem nach SWAT-Einsatz, Erkenntnisse in Team-Definitionen
- [ ] Team-Chaining fuer Multi-Team Tasks
- [ ] Prompt-Evolution: A/B-Testing von Prompt-Varianten
- [ ] Team-Komposition: Sub-Teams als Bausteine
- [ ] Performance-Metriken: Tracking von Swarm-Dauer, Rework-Rate, QG-Pass-Rate

## Lizenz

MIT

---

Gebaut mit Claude Code. Das erste SWAT-Team wurde von seinem eigenen Vorgaenger (dem generischen `agent-swarm`) deployed — 6 parallele Sonnet-Worker, Opus Quality Gate, 1 Rework-Zyklus.
