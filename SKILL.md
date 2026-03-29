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

**`max_workers` Definition:** Der `max_workers` Wert in Team-Definitionen gibt die **Gesamtzahl der Rollen** im Team an (nicht die Peak-Parallelitaet). Die tatsaechliche Parallelitaet ergibt sich aus dem Workflow-Graph (Rollen ohne `consumes` laufen parallel).

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
