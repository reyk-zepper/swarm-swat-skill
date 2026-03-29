# Orchestrierungsprotokoll

Exaktes Protokoll fuer die Ausfuehrung eines SWAT-Swarms nach Team-Wahl.

## Progress-Updates (Pflicht)

Der User muss nach jeder Phase einen kurzen Status sehen. Format:

```
[Phase X/4] <Was passiert> — <Ergebnis oder Status>
```

Beispiele:
- `[Phase 1/4] Vorbereitung — Team-Definition geladen, 12 Dateien im Scope`
- `[Phase 2/4] Parallele Ausfuehrung — Scanner + Exploit-Analyst gestartet`
- `[Phase 3/4] Sequentielle Ausfuehrung — Fix-Proposer arbeitet mit 3 CRITICAL + 5 HIGH Findings`
- `[Phase 4/4] Quality Gate — Rigorous Mode, QA + DA laufen parallel`
- `[Rework 1/2] — 2 MAJOR Issues werden gefixed`

## Phase 1: Vorbereitung

1. **Team-Definition laden**: `Read` die gematchte Team-Definition aus `references/teams/<name>.md`
2. **Task-Kontext extrahieren**:
   - User-Aufgabe (praezise Zusammenfassung)
   - Betroffene Dateien/Module (per Glob/Grep oder aus User-Angabe)
   - Relevanter Codebase-State (existierende Patterns, Frameworks, Conventions)
3. **Team-Revalidierung**: Nach Context-Extraction pruefen ob das gewaehlte Team noch passt. Die Codebase-Analyse kann zeigen dass ein anderes Team besser geeignet ist (z.B. User sagt "baue Feature X" → Full-Stack gewaehlt, aber nach Code-Analyse: Frontend existiert bereits, nur API fehlt → Backend API waere richtig). Falls Wechsel noetig: dem User kurz zeigen und Team tauschen BEVOR Worker gespawnt werden.
4. **Prompt-Templates instanziieren**: `{task_context}` mit extrahiertem Kontext ersetzen

## Phase 2: Parallele Ausfuehrung

5. **Parallele Rollen spawnen**: Alle Rollen OHNE `consumes`-Abhaengigkeit in einem einzigen Message-Block spawnen
   - Exakte Agent-Configs aus Team-Definition verwenden (model, subagent_type, mode, isolation)
   - Jedem Worker einen beschreibenden `name` geben (z.B. "scanner", "exploit-analyst")
   - `run_in_background: true` nur wenn genuinely unabhaengige Arbeit parallel moeglich
6. **Auf alle parallelen Agents warten**

## Phase 3: Output-Validation + Sequentielle Ausfuehrung

7. **Output-Validation** fuer jeden abgeschlossenen Worker:
   - Pruefe: Passt der Output zum definierten Output-Contract? (Struktur, Vollstaendigkeit, keine offensichtlichen Fehler)
   - Bei Format-Problemen: einmalig via `SendMessage` an den Worker korrigieren lassen
   - **Output-Kompression**: Wenn die kombinierten Worker-Outputs einer Phase ~4000+ Tokens ueberschreiten, fasse jeden Output auf die Kernpunkte zusammen die der naechste `consumes`-Agent laut dessen Output-Contract tatsaechlich braucht. Ziel: ~40% des Originals, ohne referenzierte Information zu verlieren. Konkret: behalte alle strukturierten Artefakte (Code-Bloecke, Tabellen, Listen mit IDs) und kuerze Prosa-Erklaerungen auf 1-2 Saetze pro Punkt.
8. **Sequentielle Rollen spawnen** (die `consumes` haben):
   - `{Rollenname.output}` Platzhalter mit tatsaechlichen (ggf. zusammengefassten) Outputs ersetzen
   - Output-Validation fuer jede sequentielle Rolle wiederholen

## Phase 4: Integration + Quality Gate

9. **Outputs integrieren**: Alle finalen Outputs einsammeln, auf Konflikte/Widersprueche/Luecken pruefen
10. **Quality Gate spawnen**: `Read references/quality-gate-protocol.md` und folge dem Protokoll
   - `model: "opus"`, team-spezifische Erfolgskriterien + DA Focus
   - Quality Mode wie in Team-Definition (standard oder rigorous)
11. **Ergebnis verarbeiten**:
    - **PASS** → Ergebnis synthetisiert praesentieren
    - **PASS_WITH_NOTES** → Ergebnis praesentieren + Notes hervorheben
    - **FAIL** → Rework-Protokoll ausfuehren

## Rework-Protokoll (max 2 Zyklen)

12. REWORK_NEEDED Items aus Quality Gate parsen
13. Targeted Fix-Agents spawnen:
    - Kleiner Fix → `SendMessage` an bestehenden Worker (bewahrt Kontext)
    - Groesserer Fix → Neuer Sonnet-Agent mit fokussiertem Prompt
14. Quality Gate erneut ausfuehren (gleicher Modus)
15. Nach 2 Zyklen ohne PASS: bestes Ergebnis mit verbleibenden QG-Notes an User praesentieren

## Phase 5: Post-Swarm Learning (optional)

Feuert NUR wenn ein spezialisiertes Team (nicht Generic Fallback) eingesetzt wurde UND:
- QG hat **FAIL** zurueckgegeben (Rework wurde durchgefuehrt), ODER
- QG hat **PASS_WITH_NOTES** mit mindestens einem **MAJOR**-level Item zurueckgegeben

Bei clean PASS oder nur MINOR Notes: Phase 5 ueberspringen.

16. `Read references/trainer-protocol.md` Teil 2 und folge dem Post-Swarm Learning Protokoll
17. Zeige dem User die vorgeschlagene Prompt-Verbesserung und hole Approval ein
18. Bei Approval: Edit die Team-Definition + Changelog-Eintrag ergaenzen

Progress-Update: `[Phase 5/5] Post-Swarm Learning — [Rolle] Prompt verbessert` oder `[Phase 5/5] Uebersprungen (clean PASS)`

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

### Worktree-Merge Protokoll

Wenn parallele Worker in Worktrees gearbeitet haben, gibt jeder Agent seinen Worktree-Branch zurueck. Der Lead merged wie folgt:

1. **Branches sammeln** — Notiere alle zurueckgegebenen Worktree-Branches
2. **Sequentiell mergen** — Merge jeden Branch einzeln in den Hauptbranch (nicht alle gleichzeitig):
   ```
   git merge <worker-branch-1> --no-edit
   git merge <worker-branch-2> --no-edit
   ```
3. **Konflikte aufloesen** — Bei Merge-Konflikten:
   - Lies beide Seiten des Konflikts
   - Entscheide basierend auf den Output-Contracts welche Version korrekt ist
   - Bei echten semantischen Konflikten (beide Seiten aendern die gleiche Logik unterschiedlich): spawne einen Sonnet-Agent mit beiden Versionen und dem Original-Task als Kontext, der die korrekte Synthese schreibt
4. **Build-Check** — Nach allen Merges: pruefe ob der Code kompiliert/lauffaehig ist (z.B. `npm run build` oder `tsc --noEmit`). Falls nicht: Fix-Agent spawnen bevor Quality Gate
5. **Worktree-Cleanup** — Worktrees werden automatisch aufgeraeumt. Falls nicht, `git worktree remove <path>`

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
