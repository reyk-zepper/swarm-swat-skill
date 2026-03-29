# Agent-SWAT-Swarm Design

## Summary

Spezialisierte Agent-Swarm-Teams ("SWAT Teams") fuer Claude Code, die auf spezifische Aufgabentypen voroptimiert sind. Ersetzt den generischen `agent-swarm` Skill durch ein System mit vorkonfigurierten Team-Definitionen, einem automatischen Dispatcher und einem Quality Gate mit team-spezifischem Devil's Advocate.

## Entscheidungen

| Frage | Entscheidung | Begruendung |
|-------|-------------|-------------|
| Team-Selektion | Hybrid: automatische Empfehlung, User kann ueberstimmen | Volle Autonomie im Normalfall, Kontrolle bei Bedarf |
| Verhaeltnis zu agent-swarm | Ersetzt komplett, Generic-Fallback eingebaut | Ein System, ein Entscheidungspunkt, keine Komplexitaet durch Split |
| Definitions-Ort | Ein Skill + Team-Dateien unter references/teams/ | MAX_SKILLS Cap (3) macht separate Skills unpraktikabel. 1 Skill-Slot, Teams on-demand per Read |
| Team-Struktur | Prompt-zentriert mit Output-Contracts + Workflow-Graph | Prompt-Qualitaet ist der einzige Hebel. Vorkonfigurierte Prompts > improvisierte |
| Override-Mechanismus | Slash-Command + Keyword-Erkennung im Prompt | Power-User: /agent-swat-swarm <team>. Alle anderen: natuerliche Sprache |
| Skill-Name | agent-swat-swarm | Neues System, neuer Name |
| Matching-Logik | Natuerlichsprachlich (kein Scoring) | Claude versteht Kontext besser als gewichtete Formeln |
| Devil's Advocate | Integriert, team-spezifische Challenge-Fragen | Domänenspezifische Challenges > generische Fragen |

## Architektur

### Dateistruktur

```
~/.claude/skills/agent-swat-swarm/
├── SKILL.md                          # Dispatcher: Entscheidungsbaum + Orchestrierungsprotokoll
├── references/
│   ├── teams/                        # SWAT-Team-Definitionen
│   │   ├── security-audit.md
│   │   ├── frontend-feature.md
│   │   ├── backend-api.md
│   │   ├── migration-refactor.md
│   │   ├── full-stack-feature.md
│   │   └── team-builder.md
│   ├── quality-gate-protocol.md      # QG-Spec mit DA-Integration
│   └── generic-swarm.md              # Fallback fuer Tasks ohne Team-Match
```

### Kontext-Effizienz

- SKILL.md enthaelt NUR Dispatcher-Logik + Team-Index (~200 Zeilen)
- Team-Definitionen werden on-demand per `Read` geladen — nur das gematchte Team belastet das Kontext-Budget
- Belegt 1 Skill-Slot statt bis zu N bei separaten Skills

## Dispatcher-Logik (SKILL.md)

### Inhalt

1. Trigger-Description (wann wird der Skill geladen)
2. Team-Index (Name + Kurzbeschreibung pro Team, keine Details)
3. Matching-Protokoll (natuerlichsprachlich)
4. Override-Erkennung
5. Orchestrierungsprotokoll
6. Generic-Fallback-Regeln

### Matching-Protokoll

```
1. User-Override pruefen
   - Slash-Command: /agent-swat-swarm <team-name> → Team erzwungen
   - Keyword im Prompt: "nutze Security Team", "mit Frontend SWAT" → Team erzwungen

2. Kein Override → Automatisches Matching
   - User-Aufgabe lesen und verstehen
   - Team-Index (Name + Beschreibung + Trigger-Hints) gegen Aufgabe abgleichen
   - Entscheidung per Verstaendnis, nicht per Scoring-Formel
   - Beruecksichtige: Aufgabentyp, betroffene Dateien, Domaene, Komplexitaet

3. Ergebnis
   - Eindeutiger Match: Team deployen
   - Unklarer Match: User kurz fragen (max 1 Frage)
   - Kein Match: Generic-Fallback deployen

4. Empfehlung anzeigen
   "SWAT Team: [Name] — [Grund fuer Match]"
   "Override: /agent-swat-swarm <anderes-team> oder 'nutze [Team-Name]'"
```

### Orchestrierungsprotokoll

```
 1. Read: references/teams/<matched-team>.md
 2. Task-Kontext extrahieren (Codebase-State, User-Aufgabe, betroffene Files)
 3. Prompt-Templates instanziieren ({task_context} befuellen)
 4. Parallele Rollen spawnen (alle ohne consumes-Abhaengigkeit)
 5. Auf parallele Agents warten
 6. Output-Validation: Lead prueft ob Output zum Contract passt
    - Format ok → weiter
    - Format broken → einmalig korrigieren via SendMessage
 7. Bei verbose Outputs: Lead fasst zusammen bevor Weitergabe
 8. Sequentielle Rollen spawnen ({Agent.output} Platzhalter befuellen)
 9. Schritte 6-7 fuer sequentielle Outputs wiederholen
10. Alle Outputs einsammeln + integrieren
11. Quality Gate spawnen (Opus, team-spezifische Kriterien + DA Focus)
12. PASS → Ergebnis praesentieren
    FAIL → Rework (max 2 Zyklen, targeted Sonnet-Fixes, dann QG erneut)
```

## Team-Definitions-Format

Jede Team-Definition unter `references/teams/<name>.md`:

```markdown
---
name: [Team-Name]
triggers:
  keywords: [relevante Keywords als Matching-Hints]
  file_patterns: [Glob-Patterns fuer betroffene Dateien]
  task_patterns: [Regex-Patterns fuer Aufgabenbeschreibungen]
quality_mode: standard | rigorous
max_workers: [Zahl]
---

# [Team-Name] SWAT Team

## Rollen

### 1. [Rollenname] (parallel | sequentiell, nach [Abhaengigkeit])
- model: sonnet | opus
- subagent_type: [general-purpose | Explore | code-reviewer | ...]
- mode: [default | bypassPermissions]
- isolation: [worktree] (optional, nur bei Code-Aenderungen in ueberlappenden Bereichen)
- consumes: [Liste von Rollen deren Output benoetigt wird] (optional)
- output_contract: |
    Exakte Beschreibung des erwarteten Outputs.
    Format und Struktur klar definiert.
    Emphasis auf Praezision und Kuerzee.
- prompt: |
    Vollstaendiger, voroptimierter Prompt fuer diese Rolle.
    Enthaelt {task_context} Platzhalter fuer Aufgabenkontext.
    Enthaelt {Rollenname.output} Platzhalter fuer konsumierte Outputs.

### 2. [Naechste Rolle] ...

## Workflow-Graph

    [Visuelle DAG-Darstellung]
    Agent A ──────┐
                  ├──→ Agent C ──→ Agent D
    Agent B ──────┘

## Devil's Advocate Focus

- Team-spezifische Challenge-Frage 1
- Team-spezifische Challenge-Frage 2
- Team-spezifische Challenge-Frage 3
- Team-spezifische Challenge-Frage 4

## Erfolgskriterien (Quality Gate)

- Kriterium 1
- Kriterium 2
- Kriterium 3
```

## Model Assignment

| Rolle | Model | Begruendung |
|-------|-------|-------------|
| Dispatcher (Lead) | opus (Hauptinstanz) | Strategische Entscheidungen, Task-Dekomposition, Synthese |
| Worker Agents | sonnet | Schnelle parallele Ausfuehrung, kosteneffizient |
| Quality Gate Agent | opus | Kritische Bewertung, Fehler-Erkennung |
| Devil's Advocate | opus (im QG integriert: standard, separat: rigorous) | Domänenspezifische Challenges |

## Quality Gate

### Modi

| Modus | Wann | Agents | Kosten |
|-------|------|--------|--------|
| standard (default) | Die meisten Tasks | 1x Opus: QA + DA kombiniert | 1 Opus Call |
| rigorous | Produktionskritisch, hohe Fehlerkosten | 2x Opus parallel: dedizierter QA + dedizierter DA | 2 Opus Calls |

### Devil's Advocate Integration

Jedes Team definiert eigene DA-Focus-Fragen. Beispiele:

| Team | DA fragt |
|------|----------|
| Security Audit | Welche Angriffsvektoren nicht geprueft? Sind Fixes sicher oder Pflaster? |
| Frontend Feature | Wie verhaelt es sich bei Screen-Readern? Bei 3G? |
| Migration | Welche Breaking Changes uebersehen? Versteckte Runtime-Abhaengigkeiten? |
| Backend API | Wie skaliert es bei 100x Load? Race Conditions? |

### Rework-Protokoll

1. QG-Ergebnis parsen: PASS / PASS_WITH_NOTES / FAIL
2. FAIL: Rework-Items extrahieren
3. Targeted Sonnet-Fix-Agents spawnen (oder via SendMessage an bestehende Worker)
4. Quality Gate erneut ausfuehren (gleicher Modus)
5. Maximum 2 Rework-Zyklen — danach bestes Ergebnis mit QG-Notes an User

## Starter-Teams

### 1. Security Audit
- Scanner + Exploit-Analyst (parallel) → Fix-Proposer → Report-Writer
- Quality Mode: rigorous
- Focus: OWASP Top 10, Injection, Auth Bypass, Data Exposure

### 2. Frontend Feature
- UI-Architekt + Component-Builder (parallel) → Integrator → A11y-Reviewer
- Quality Mode: standard
- Focus: Komponenten-Architektur, Responsive, Accessibility, Performance

### 3. Backend API
- Schema-Designer + Endpoint-Builder (parallel) → Test-Writer → Integration-Tester
- Quality Mode: standard
- Focus: API-Design, DB-Schema, Business-Logic, Test-Coverage

### 4. Migration/Refactor
- Impact-Analyst + Migration-Planner (parallel) → Executor → Regression-Tester
- Quality Mode: rigorous
- Focus: Breaking Changes, Dependency-Graph, Backward Compatibility

### 5. Full-Stack Feature
- Frontend-Builder + Backend-Builder + Schema-Designer (parallel) → Integrator → E2E-Tester
- Quality Mode: standard (rigorous wenn produktionskritisch)
- Focus: End-to-end Konsistenz, API-Contract, UI-Backend-Alignment

### 6. SWAT Team Builder (Meta-Team)
- Research-Agent → Prompt-Engineer → Validator (Opus QG)
- Generiert neue Team-Definitionen im Standard-Format
- Schreibt nach references/teams/<name>.md
- Aktualisiert Team-Index im SKILL.md

## Generic-Fallback

Lebt in `references/generic-swarm.md`. Entspricht der heutigen agent-swarm Logik:

- Lead dekomponiert dynamisch (keine vordefinierten Rollen)
- Worker-Rollen werden ad-hoc bestimmt
- Generische Prompt-Templates
- Quality Gate im Standard-Modus
- Generische DA-Fragen

Wird deployed wenn kein SWAT-Team matcht.

## Anti-Patterns

1. Over-swarming: Triviale Tasks nicht swarmen. 5-Zeilen-Fix braucht kein SWAT-Team
2. Vage Prompts: Worker-Prompts muessen praezise und selbsterklaerend sein
3. Quality Gate skippen: Nie. Das QG macht Swarms zuverlaessig
4. Sequentiell wenn parallel moeglich: Unabhaengige Rollen immer parallelisieren
5. Garbage weiterreichen: Lead validiert Outputs bevor Weitergabe
6. Rework ignorieren: FAIL bedeutet Rework, nicht "trotzdem praesentieren"
7. Neue Agents fuer kleine Fixes: SendMessage an bestehende Worker nutzen

## Migration von agent-swarm

1. Alte agent-swarm Dateien archivieren (Backup)
2. Neues agent-swat-swarm Skill erstellen
3. Altes agent-swarm Skill loeschen
4. Vault-Referenzen aktualisieren (agent-swarm-skill Decision Note)
5. Skill-Description in Claude's Skill-Registry testen

## Offene Punkte (Phase 2)

- Team-Chaining fuer Multi-Team Tasks (bei bewiesenem Bedarf)
- Feedback-Loop: Post-Mortem nach SWAT-Einsatz, Erkenntnisse in Team-Definitionen
- Team-Komposition: Sub-Teams als Bausteine fuer groessere Teams
- Prompt-Evolution: A/B-Testing von Prompt-Varianten
