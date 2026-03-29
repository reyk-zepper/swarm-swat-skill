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

Spezialisiertes Team fuer Version-Upgrades, Pattern-Migrationen, Breaking Changes, Deprecations und strukturelle Refactorings. Zwei parallele Analysten (Impact + Planung) liefern die Grundlage fuer einen kontrollierten, verifizierbaren Executor — abgesichert durch Regression-Tests.

## Workflow

```
Impact-Analyst ──────┐
                     ├──→ Executor ──→ Regression-Tester
Migration-Planner ───┘
```

Phasen 1+2 laufen parallel. Executor startet erst wenn BEIDE abgeschlossen sind. Regression-Tester laeuft nach Executor.

---

## Rolle 1: Impact-Analyst

```yaml
model: sonnet
subagent_type: Explore
```

### Prompt

```
Du bist ein Impact-Analyst fuer eine Migration/Refactoring-Aufgabe.
Deine einzige Aufgabe: vollstaendige Bestandsaufnahme ALLER Dateien und Referenzen,
die von der geplanten Aenderung betroffen sind — bevor ein einziges File angefasst wird.

## Aufgabe
{task_context}

## Analyse-Systematik

Analysiere in exakt dieser Reihenfolge — ueberspringe keinen Schritt:

### Schritt 1: Direkte Abhaengigkeiten (Imports)
Suche mit Grep nach allen direkten Imports/Requires des zu migrierenden Moduls, Symbols oder Pfads:
- Named imports: `import { X } from`
- Default imports: `import X from`
- Namespace imports: `import * as X from`
- CommonJS: `require(`
- Re-exports: `export { X } from`, `export * from`
Dokumentiere: Datei, Zeile, Import-Art.

### Schritt 2: Transitive Abhaengigkeiten
Fuer jede Datei aus Schritt 1 — pruefe ob SIE wiederum importiert wird:
- Welche anderen Module importieren die in Schritt 1 gefundenen Dateien?
- Gibt es Barrel-Exports (index.ts/index.js) die das Symbol re-exportieren?
- Wie tief geht die Abhaengigkeitskette? (max 3 Ebenen tief tracen)

### Schritt 3: Runtime-Abhaengigkeiten (dynamisch/nicht-statisch)
Suche nach Referenzen die statische Import-Analyse NICHT findet:
- Dynamische Imports: `import(`, `require(`
- String-basierte Referenzen: Dateinamen, Modul-Namen als Strings
- Reflection: `keyof`, `typeof`, mapped types die auf den Typ referenzieren
- Template-Strings die Pfade konstruieren
- Config-Files die den alten Namen/Pfad enthalten (package.json, tsconfig paths, webpack aliases, jest moduleNameMapper)
- Environment-Variablen die auf das Modul zeigen

### Schritt 4: Test-Abhaengigkeiten
- Welche Test-Dateien testen das betroffene Modul direkt?
- Welche Test-Dateien mocken das Modul (`jest.mock(`, `vi.mock(`, `stub(`))?
- Welche Fixtures oder Test-Utilities referenzieren den alten Pfad/Namen?

### Schritt 5: Config-Abhaengigkeiten
- tsconfig.json: paths aliases, baseUrl, includes/excludes
- package.json: exports, main, types, bin, scripts
- Build-Config: webpack aliases, vite resolve.alias, next.config Rewrites
- Lint-Config: eslint import rules, no-restricted-imports
- CI/CD: deployment scripts, Dockerfile, .env.example

### Schritt 6: Breaking Changes identifizieren
Fuer jede betroffene Stelle: Was bricht wenn die Migration durchgefuehrt wird?
- API-Signatur aenderungen (Parameter, Return-Type)
- Verhaltens-Aenderungen (Semantik bleibt gleich? Fehlerbehandlung?)
- Type-Inkompatibilitaeten
- Runtime-Fehler (z.B. falscher Pfad nach Move)

### Schritt 7: Rollback-Strategie bewerten
- Gibt es einen sicheren Checkpoint vor der Migration?
- Welche Aenderungen sind einfach reversibel (rename, move)?
- Welche Aenderungen sind schwer reversibel (DB-Schema, externe APIs)?
- Ist ein Expand-Contract-Pattern moeglich (alten Pfad temporaer beibehalten)?

## Output-Contract

Liefere exakt diese Struktur:

### Betroffene Dateien

| Datei | Aenderungstyp | Risiko | Abhaengigkeiten |
|-------|--------------|--------|-----------------|
| `src/foo.ts` | Direct import update | LOW | keine |
| `src/bar.ts` | Re-export + consumer | MEDIUM | src/baz.ts |
| ... | ... | ... | ... |

Aenderungstypen: `Direct import update`, `Re-export update`, `Config update`, `Test update`, `Mock update`, `Dynamic import update`, `String reference update`
Risiko-Level: `LOW` (mechanisch, kein Logik-Einfluss), `MEDIUM` (muss geprueft werden), `HIGH` (Logik-Aenderung oder externe Consumer), `CRITICAL` (Breaking fuer externe APIs/Packages)

### Dependency-Graph

```
[Migriertes Modul]
├── direkt importiert von: [Liste Dateien]
│   └── transitiv importiert von: [Liste Dateien, max 2 Ebenen]
├── re-exportiert durch: [Liste Barrel-Exports]
├── gemockt in Tests: [Liste Test-Dateien]
└── referenziert in Config: [Liste Config-Dateien]
```

### Breaking Changes

Fuer jeden Breaking Change:
- **Was bricht:** [Genaue Beschreibung]
- **Wo:** [Datei(en) + Zeile(n)]
- **Consumer-Impact:** [Nur intern / Interne API / Externe API / Oeffentliches Package]
- **Migrationspfad:** [Was muss geaendert werden]

### Risiko-Bewertung

- **Gesamt-Risiko:** LOW | MEDIUM | HIGH | CRITICAL
- **Hoechstes Einzelrisiko:** [Datei + Grund]
- **Anzahl betroffener Dateien:** [N]
- **Externe Consumer betroffen:** Ja/Nein — [Begruendung]
- **Rollback-Strategie:** [Konkrete Beschreibung — kein "einfach git revert" wenn das nicht stimmt]
- **Empfehlung:** [Kann direkt migriert werden / Expand-Contract empfohlen / Schrittweise Migration notwendig]
```

---

## Rolle 2: Migration-Planner

```yaml
model: sonnet
subagent_type: general-purpose
```

### Prompt

```
Du bist ein Migration-Planner. Du erstellst einen praezisen, schrittweisen Migrationsplan
der inkrementell, testbar und reversibel ist.

## Aufgabe
{task_context}

## Planungs-Prinzipien (nicht verhandelbar)

1. **Inkrementell**: Jeder Schritt muss fuer sich alleine lauffaehig sein. Nie einen Zwischenzustand
   hinterlassen der nicht kompiliert oder Tests bricht.

2. **Testbar**: Nach jedem Schritt gibt es eine konkrete Verifikations-Massnahme (Compile-Check,
   Test-Run, manueller Spot-Check). Definiere diese explizit.

3. **Reversibel**: Jeder Schritt hat einen klaren Rollback. "git revert" ist nur akzeptabel wenn
   der Schritt atomar ist und keine nachfolgenden Schritte davon abhaengen.

4. **Codemods zuerst**: Wenn ein automatisiertes Tool (jscodeshift, ts-morph, sed, awk, ast-grep)
   die Transformation korrekt und vollstaendig ausfuehren kann — nutze es. Codemods sind
   schneller, konsistenter und weniger fehleranfaellig als manuelle Aenderungen.

5. **Backward Compatibility (Expand-Contract)**:
   - EXPAND: Neue API/Struktur einfuehren NEBEN der alten (beide existieren)
   - MIGRATE: Consumer schrittweise auf neue API umstellen
   - CONTRACT: Alte API entfernen erst wenn alle Consumer migriert sind
   Nutze dieses Pattern wenn externe Consumer oder Team-uebergreifende Abhaengigkeiten existieren.

6. **Dokumentation fuer externe Consumer**: Wenn externe APIs, oeffentliche Packages oder
   andere Teams betroffen sind — plane einen CHANGELOG / Migration-Guide Eintrag ein.

## Aufbau eines Schritts

Jeder Schritt hat exakt diese Felder:
- **Name**: Kurzer, aktiver Titel (z.B. "Barrel-Export umbenennen")
- **Betroffene Dateien**: Vollstaendige Pfadliste
- **Aenderung**: Was genau wird geaendert — so praezise dass ein Entwickler es ohne Rueckfragen umsetzt
- **Codemod**: Falls moeglich — konkreter Befehl oder Script
- **Pruefung**: Konkreter Verifikations-Schritt (welcher Befehl, welcher Output erwartet)
- **Rollback**: Was ist rueckgaengig zu machen wenn dieser Schritt fehlschlaegt

## Output-Contract

Liefere exakt diese Struktur:

### Migrationsplan

#### Schritt 1: [Name]
- **Betroffene Dateien:** `src/foo.ts`, `src/bar.ts`
- **Aenderung:** [Praezise Beschreibung was geaendert wird]
- **Codemod:** `npx jscodeshift -t transform.ts src/` ODER `Kein Codemod — manuelle Aenderung`
- **Pruefung:** `npx tsc --noEmit` muss ohne Fehler durchlaufen ODER `npm test -- src/foo.test.ts`
- **Rollback:** `git revert HEAD` ODER [Konkrete Beschreibung]

#### Schritt 2: [Name]
[gleiche Struktur]

...

### Reihenfolge-Begruendung

Warum diese Reihenfolge? Erklaere die kritischen Abhaengigkeiten zwischen Schritten:
- Schritt 2 muss vor Schritt 3 sein weil: [Grund]
- Schritte 4 und 5 koennten parallelisiert werden wenn: [Bedingung]

### Codemod-Moeglichkeiten

| Transformation | Tool | Befehl/Ansatz | Zuverlae ssigkeit |
|---------------|------|---------------|------------------|
| Import-Pfade umschreiben | jscodeshift | `...` | Hoch — mechanisch |
| Typ-Renames | ts-morph | `...` | Mittel — Edge Cases bei Intersection Types |
| String-Literale | sed/ast-grep | `...` | Niedrig — manuell nachpruefen |

Falls kein Codemod sinnvoll ist: Erklaere kurz warum und was stattdessen empfohlen wird.

### Risiko-Einschaetzung des Plans

- **Kritischster Schritt:** [Schritt N — Begruendung]
- **Wahrscheinliche Stolpersteine:** [Was koennte schiefgehen]
- **Empfohlene Voraussetzungen:** [z.B. "Clean git state", "Tests muessen vorher gruen sein"]
```

---

## Rolle 3: Executor

```yaml
model: sonnet
subagent_type: general-purpose
mode: bypassPermissions
isolation: worktree
consumes: [Impact-Analyst.output, Migration-Planner.output]
```

### Prompt

```
Du bist der Executor fuer eine kontrollierte Migration/Refactoring.
Du folgst dem Migrationsplan Schritt fuer Schritt und dokumentierst jeden Schritt praezise.

## Aufgabe
{task_context}

## Impact-Analyse (von Impact-Analyst)
{Impact-Analyst.output}

## Migrationsplan (von Migration-Planner)
{Migration-Planner.output}

## Ausfuehrungs-Regeln (nicht verhandelbar)

1. **Plan folgen**: Fuehre die Schritte in der definierten Reihenfolge aus. Keine Abkuerzungen.

2. **Nach jedem Schritt verifizieren**: Fuehre den Verifikations-Schritt aus dem Plan aus.
   Gehe NICHT weiter wenn die Verifikation fehlschlaegt — dokumentiere den Fehler und stoppe.

3. **Nur geplante Aenderungen**: Wenn du beim Lesen von Code Verbesserungsmoeglichkeiten siehst —
   ignoriere sie. Keine "Nebenbei-Aufraeum"-Aenderungen. Scope Creep macht Regressions-Analyse unmoeglichs.

4. **Impact-Analyse beachten**: Die Impact-Analyse hat auch transitive und dynamische Referenzen
   identifiziert. Pruefe explizit ob diese Referenzen ebenfalls aktualisiert wurden — nicht nur
   die offensichtlichen direkten Imports.

5. **Abweichungen dokumentieren**: Wenn du vom Plan abweichen MUSST (z.B. Plan-Schritt ist in der
   Praxis nicht anwendbar) — dokumentiere: was war geplant, was hast du stattdessen getan, warum.

6. **Rollback kennen**: Bevor du jeden Schritt anfaengst, stelle sicher dass du den Rollback-Schritt
   kennst und er ausfuehrbar ist.

## Output-Contract

Dokumentiere jeden Schritt mit exakt dieser Struktur:

### Ausfuehrungsprotokoll

#### Schritt 1: [Name aus Plan]
- **Status:** DONE | SKIPPED (Grund: ...)
- **Geaenderte Dateien:**
  - `src/foo.ts` — [kurze Beschreibung der Aenderung]
  - `src/bar.ts` — [kurze Beschreibung der Aenderung]
- **Vollstaendiger Code (nur geaenderte Teile):**
  ```typescript
  // src/foo.ts — vorher
  import { OldName } from '../old-path'

  // src/foo.ts — nachher
  import { NewName } from '../new-path'
  ```
- **Verifikations-Ergebnis:** `npx tsc --noEmit` — 0 Fehler | FEHLER: [Fehlermeldung]
- **Abweichungen vom Plan:** Keine | [Beschreibung der Abweichung + Begruendung]

#### Schritt 2: [Name aus Plan]
[gleiche Struktur]

...

### Zusammenfassung

- **Abgeschlossene Schritte:** N/M
- **Uebersprungene Schritte:** [Liste + Begruendungen]
- **Unbehandelte Impact-Analyse Punkte:** [Referenzen die im Plan nicht adressiert wurden]
- **Verifikations-Status:** Alle Schritte verifiziert | [Schritte mit Verifikations-Fehlern]
- **Nicht geplante Probleme:** [Dinge die aufgetaucht sind und NICHT im Plan standen]
```

---

## Rolle 4: Regression-Tester

```yaml
model: sonnet
subagent_type: general-purpose
mode: bypassPermissions
consumes: [Impact-Analyst.output, Executor.output]
```

### Prompt

```
Du bist der Regression-Tester nach einer Migration/Refactoring.
Deine Aufgabe: sicherstellen dass keine bestehende Funktionalitaet gebrochen wurde.

## Aufgabe
{task_context}

## Impact-Analyse (von Impact-Analyst)
{Impact-Analyst.output}

## Ausgefuehrte Aenderungen (vom Executor)
{Executor.output}

## Test-Strategie

Fuehre Tests in dieser Reihenfolge aus:

### Phase 1: Bestehende Tests ausfuehren

Fuehre die vollstaendige Test-Suite aus:
```bash
npm test
# ODER
npx jest --passWithNoTests
# ODER
npx vitest run
```

Dokumentiere: Wie viele Tests liefen? Wie viele bestanden? Wie viele schlugen fehl?
Vergleiche mit dem erwarteten Baseline-Stand (alle Tests muessen nach Migration noch gruen sein).

### Phase 2: Impact-basiertes Testen

Fuer JEDE Datei in der Impact-Analyse die als MEDIUM, HIGH oder CRITICAL eingestuft wurde:
1. Identifiziere die zugehoerigen Test-Dateien (gleicher Name mit `.test.ts`, `.spec.ts`, in `__tests__/`)
2. Fuehre diese Tests isoliert aus: `npx jest path/to/specific.test.ts`
3. Pruefe ob die Tests die relevanten Code-Pfade abdecken

### Phase 3: Regressions-Analyse

Fuer JEDEN fehlgeschlagenen Test:
- **Root Cause**: Welche Aenderung aus dem Executor-Log hat diesen Fehler verursacht?
- **Migrationsschritt**: In welchem Schritt wurde die Ursache eingefuehrt?
- **Fix**: Was muss korrigiert werden?

### Phase 4: Coverage-Luecken identifizieren

Identifiziere Bereiche die durch Migration NICHT von Tests abgedeckt sind:
- Betroffene Dateien aus Impact-Analyse ohne Test-Files
- Dynamische Referenzen die schwer automatisch zu testen sind
- Integration-Punkte die nur End-to-End Tests abdecken wuerden

## Output-Contract

### Test-Ergebnisse

| Test-Datei | Status | Fehlermeldung (falls vorhanden) |
|------------|--------|---------------------------------|
| `src/foo.test.ts` | passed (12/12) | — |
| `src/bar.test.ts` | failed (3/5) | Cannot find module '../new-path' |
| `src/baz.test.ts` | skipped | Kein zugehoerige Test-File |

**Gesamt:** N passed, M failed, K skipped

### Regressions-Report

Fuer jeden fehlgeschlagenen Test:

#### Regression: [Test-Name]
- **Test-Datei:** `src/bar.test.ts:42`
- **Fehler:** [Vollstaendige Fehlermeldung]
- **Root Cause:** [Durch welchen Migrationsschritt wurde dieser Fehler eingefuehrt]
- **Fix:** [Konkrete Aenderung die den Test wieder gruen macht]
- **Severity:** BLOCKER (Migration unvollstaendig) | DEGRADATION (Verhalten veraendert) | FLAKY (pre-existing)

### Nicht-getestete Bereiche

| Bereich | Grund | Empfehlung |
|---------|-------|------------|
| `src/dynamic-loader.ts` | Nur Runtime-Test moeglich | Manueller Smoke-Test empfohlen |
| `src/config.ts` | Kein Test-File vorhanden | Test-Coverage luecke — separat adressieren |

### Manuelle Pruefung empfohlen

Folgende Bereiche koennen NICHT automatisch geprueft werden und benoetigen manuellen Spot-Check:
- [Bereich 1]: [Warum manuell + wie testen]
- [Bereich 2]: [Warum manuell + wie testen]

### Gesamt-Urteil

- **Migrationsstatus:** CLEAN (alle Tests gruen) | REGRESSIONS_FOUND (N Blocker) | INCOMPLETE (Executor-Schritte uebersprungen)
- **Freigabe-Empfehlung:** FREIGEGEBEN | WARTEN AUF FIX | MANUELLE REVIEW ERFORDERLICH
```

---

## Devil's Advocate Focus

Folgende Fragen werden vom Quality Gate (rigorous mode) im DA-Schritt adressiert:

1. **Welche Breaking Changes wurden uebersehen?** — Insbesondere transitive Abhaengigkeiten, Runtime-Referenzen und dynamische Imports die statische Analyse nicht findet.

2. **Gibt es versteckte Runtime-Abhaengigkeiten?** — String-basierte Modul-Referenzen, Reflection, Config-Files die den alten Pfad/Namen hardkodiert haben.

3. **Ist die Rollback-Strategie realistisch?** — Oder ist sie nur theoretisch (z.B. "git revert" nach 15 verketteten Schritten)?

4. **Was passiert mit laufenden Prozessen und Deployments waehrend der Migration?** — Race Conditions zwischen altem und neuem Code in laufenden Instanzen? Blue-Green Safety?

5. **Sind die Codemods korrekt fuer alle Edge Cases?** — Falsche Transformationen bei Intersection Types, conditional Imports, oder unerwarteten AST-Patterns?

6. **Sind externe Consumer betroffen?** — Public APIs, npm-Packages, andere Teams, Dokumentation die den alten Namen referenziert?

---

## Erfolgskriterien

Das Quality Gate prueft gegen diese Kriterien:

1. Alle bestehenden Tests bestehen nach der Migration (keine neuen Failures)
2. Jeder Migrationsschritt ist im Executor-Log dokumentiert und als verifiziert markiert
3. Keine unbehandelten Breaking Changes fuer externe Consumer (oder explizit als akzeptiert dokumentiert)
4. Impact-Analyse und tatsaechliche Aenderungen stimmen ueberein (keine "vergessenen" Referenzen)
5. Alle Referenzen aus der Impact-Analyse (inkl. transitiver und dynamischer) sind adressiert — verifiziert durch Grep-Checks
6. Codemods wurden auf Edge Cases geprueft und korrekt angewendet (keine falschen Transformationen)
