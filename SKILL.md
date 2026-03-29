---
name: agent-swat-swarm
description: Parallel-Agent SWAT-Teams mit voroptimierte Prompts, Output-Contracts und Workflow-Graphs. PROAKTIV AKTIVIEREN wenn eine Aufgabe 4+ Dateien betrifft, mehrere Domaenen (Frontend+Backend) umspannt, ein Security Audit oder Review erfordert, eine Migration/Refactoring ueber viele Files durchfuehrt, oder 3+ unabhaengige Subtasks hat. NICHT aktivieren bei Single-File-Tasks, schnellen Fixes, oder wenn der User explizit einen einfachen Ansatz will.
---

# Agent SWAT Swarm

Spezialisierte Agent-Teams fuer komplexe Aufgaben. Jedes SWAT-Team hat vorkonfigurierte Rollen mit optimierten Prompts, Output-Contracts und Workflow-Graphs.

**Core Principle:** Statt bei jeder komplexen Aufgabe von Grund auf zu dekomponieren, wird ein spezialisiertes SWAT-Team deployed das fuer genau diesen Aufgabentyp optimiert ist.

## Model Assignment (Non-Negotiable)

| Role | Model | Rationale |
|------|-------|-----------|
| **Dispatcher (Lead)** | `opus` (Hauptinstanz) | Strategische Entscheidungen, Synthese, Integration |
| **Worker Agents** | `sonnet` | Schnelle parallele Ausfuehrung, kosteneffizient |
| **Quality Gate** | `opus` | Kritische Bewertung, Fehler-Erkennung |
| **Devil's Advocate** | `opus` | Domaenenspezifische Challenges (standard: im QG integriert, rigorous: separater Agent) |

## When to Swarm

**Do swarm** when: Multi-file implementation (4+ files), research + implement, multi-domain (frontend + backend), large refactor, comprehensive review, 3+ parallel-independent subtasks, quality-critical deliverable.

**Do NOT swarm** when: Single-file task, user wants quick/simple approach, tight iterative feedback needed, fewer than 3 independent sub-tasks.

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

## Team-Selektion

### 1. User-Override pruefen

- **Slash-Command**: `/agent-swat-swarm <team-name>` → Team erzwungen
  - Gueltige Namen: `security-audit`, `frontend-feature`, `backend-api`, `migration-refactor`, `full-stack-feature`, `team-builder`, `team-audit`
- **Keyword im Prompt**: "nutze Security Team", "mit Frontend SWAT", "deploy Backend Team" → Team erzwungen

### 2. Automatisches Matching (kein Override)

- Verstehe den **Aufgabentyp**: Build, Review, Migrate, Fix, Audit, Analyse
- Beruecksichtige **betroffene Dateien/Domaenen** und gleiche gegen Team-Trigger-Keywords in den Team-Definitionen ab
- Eindeutiger Match → Team deployen
- Unklarer Match → User kurz fragen (1 Multiple-Choice-Frage)
- Kein Match → `Read references/trainer-protocol.md` Teil 1: Prüfe ob Auto-Creation sinnvoll ist (wiederkehrender Task-Typ + distinct + Spezialisierung bringt Mehrwert). Falls ja: Team Builder auto-deployen. Falls nein: Generic Fallback.

### 3. Empfehlung anzeigen (VOR Ausfuehrung)

```
SWAT Team: [Name]
Grund: [1 Satz warum dieses Team passt]
Rollen: [Rollennamen aus Team-Definition]
Agents: [N]x Sonnet (Worker) + [1-2]x Opus (Quality Gate) = [Total] Agents
Quality Mode: [standard / rigorous]
Override: /agent-swat-swarm <anderes-team>
```

## Ausfuehrung

**Pfad-Konvention:** Alle Pfade in diesem Skill sind relativ zum Skill-Root-Verzeichnis. Wenn der Skill unter `~/.claude/skills/agent-swat-swarm/` installiert ist, loese `references/teams/security-audit.md` als `~/.claude/skills/agent-swat-swarm/references/teams/security-audit.md` auf. Falls das Verzeichnis nicht ermittelt werden kann, nutze Glob mit Pattern `**/agent-swat-swarm/references/teams/*.md` um die Dateien zu finden.

Nach Team-Wahl:

1. `Read references/orchestration-protocol.md` — folge dem Protokoll exakt
2. Falls Read fehlschlaegt (Pfad nicht aufloesbar): Glob `**/agent-swat-swarm/references/**` um das Skill-Verzeichnis zu finden, dann erneut lesen
3. Falls auch das fehlschlaegt: Generic Fallback verwenden (dynamische Dekomposition ohne Team-Definition)

## References

- **`references/orchestration-protocol.md`** — Vollstaendiges Ausfuehrungsprotokoll (Phasen 1-5, Rework, Sizing, Anti-Patterns)
- **`references/trainer-protocol.md`** — Auto-Creation, Post-Swarm Learning, Team Fitness Audit
- **`references/teams/*.md`** — SWAT-Team-Definitionen mit Rollen, Prompts, Workflow-Graphs
- **`references/quality-gate-protocol.md`** — QG-Spec mit DA-Integration, Rework-Protokoll
- **`references/generic-swarm.md`** — Fallback fuer Tasks ohne Team-Match
