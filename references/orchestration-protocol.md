# Orchestrierungsprotokoll

Exaktes Protokoll fuer die Ausfuehrung eines SWAT-Swarms nach Team-Wahl.

## Phase 1: Vorbereitung

1. **Team-Definition laden**: `Read` die gematchte Team-Definition aus `references/teams/<name>.md`
2. **Task-Kontext extrahieren**:
   - User-Aufgabe (praezise Zusammenfassung)
   - Betroffene Dateien/Module (per Glob/Grep oder aus User-Angabe)
   - Relevanter Codebase-State (existierende Patterns, Frameworks, Conventions)
3. **Prompt-Templates instanziieren**: `{task_context}` mit extrahiertem Kontext ersetzen

## Phase 2: Parallele Ausfuehrung

4. **Parallele Rollen spawnen**: Alle Rollen OHNE `consumes`-Abhaengigkeit in einem einzigen Message-Block spawnen
   - Exakte Agent-Configs aus Team-Definition verwenden (model, subagent_type, mode, isolation)
   - Jedem Worker einen beschreibenden `name` geben (z.B. "scanner", "exploit-analyst")
   - `run_in_background: true` nur wenn genuinely unabhaengige Arbeit parallel moeglich
5. **Auf alle parallelen Agents warten**

## Phase 3: Output-Validation + Sequentielle Ausfuehrung

6. **Output-Validation** fuer jeden abgeschlossenen Worker:
   - Pruefe: Passt der Output zum definierten Output-Contract? (Struktur, Vollstaendigkeit, keine offensichtlichen Fehler)
   - Bei Format-Problemen: einmalig via `SendMessage` an den Worker korrigieren lassen
   - Bei verbosen Outputs: Lead fasst die Kernpunkte zusammen bevor Weitergabe
7. **Sequentielle Rollen spawnen** (die `consumes` haben):
   - `{Rollenname.output}` Platzhalter mit tatsaechlichen (ggf. zusammengefassten) Outputs ersetzen
   - Output-Validation fuer jede sequentielle Rolle wiederholen

## Phase 4: Integration + Quality Gate

8. **Outputs integrieren**: Alle finalen Outputs einsammeln, auf Konflikte/Widersprueche/Luecken pruefen
9. **Quality Gate spawnen**: Siehe `references/quality-gate-protocol.md`
   - `model: "opus"`, team-spezifische Erfolgskriterien + DA Focus
   - Quality Mode wie in Team-Definition (standard oder rigorous)
10. **Ergebnis verarbeiten**:
    - **PASS** → Ergebnis synthetisiert praesentieren
    - **PASS_WITH_NOTES** → Ergebnis praesentieren + Notes hervorheben
    - **FAIL** → Rework-Protokoll ausfuehren

## Rework-Protokoll (max 2 Zyklen)

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

**Wann Worktrees:** Mehrere Worker aendern Code in ueberlappenden Verzeichnissen oder brauchen kompilierbaren State.
**Wann NICHT:** Worker lesen nur Code oder schreiben in komplett getrennte Dateien.

## Swarm Sizing

| Task-Komplexitaet | Workers | Quality Mode |
|-------------------|---------|--------------|
| Medium (4-6 Files) | 2-3 | standard |
| Gross (7-15 Files) | 3-5 | standard oder rigorous |
| Sehr Gross (15+ Files) | 5-8 | rigorous |
| Produktionskritisch | beliebig | **rigorous** |

Max 8 Workers. `max_workers` in Team-Definitionen = Gesamtzahl der Rollen (nicht Peak-Parallelitaet).

## Anti-Patterns

1. **Over-swarming** — Triviale Tasks nicht swarmen. 5-Zeilen-Fix braucht kein SWAT-Team.
2. **Vage Worker-Prompts** — Prompts muessen praezise, selbsterklaerend und mit klaren Grenzen sein.
3. **Quality Gate skippen** — Nie. Das QG ist nicht verhandelbar.
4. **Sequentiell wenn parallel moeglich** — Unabhaengige Rollen immer in einem Message-Block parallelisieren.
5. **Garbage weiterreichen** — Lead validiert jeden Output bevor Weitergabe an sequentielle Agents.
6. **Rework ignorieren** — FAIL bedeutet Rework, nicht "trotzdem praesentieren".
7. **Neue Agents fuer kleine Fixes** — `SendMessage` an bestehende Worker statt neuen Agent spawnen.
