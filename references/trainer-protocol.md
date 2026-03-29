# Trainer-Protokoll

Der Trainer ist der Coach aller SWAT-Teams. Er erstellt neue Teams wenn noetig, verbessert bestehende nach jedem Einsatz, und sorgt dafuer dass der Skill ueber Sessions hinweg an Performance gewinnt.

**Persistence-Mechanismus:** Alle Aenderungen werden direkt auf Disk geschrieben (Edit/Write auf Team-Definitionen). Naechste Session liest Claude die verbesserte Version. Disk IST der State.

---

## Teil 1: Auto-Creation (Neues Team erstellen)

### Wann ein neues Team erstellen?

Wird vom Dispatcher in SKILL.md getriggert wenn KEIN existierendes Team passt. Vor dem Generic Fallback wird geprueft:

**Erstelle ein neues Team wenn ALLE 3 Bedingungen erfuellt sind:**

1. **Wiederkehrender Task-Typ** — Der Task gehoert zu einer erkennbaren Kategorie die wahrscheinlich erneut vorkommt (z.B. "Performance-Optimierung", "API-Design-Review", "Datenbank-Tuning"). NICHT erstellen fuer einmalige, hochspezifische Tasks (z.B. "fix diesen einen CSS-Bug", "schreibe diese E-Mail").

2. **Distinct von existierenden Teams** — Der Task-Typ wird nicht bereits von einem bestehenden Team abgedeckt. Prüfe explizit: Koennte Security Audit, Frontend Feature, Backend API, Migration/Refactor, oder Full-Stack Feature diesen Task sinnvoll bearbeiten? Wenn ja → kein neues Team noetig, nutze das existierende.

3. **Spezialisierung bringt Mehrwert** — Der Task-Typ profitiert von domaenenspezifischen Prompts, Output-Contracts und Workflow-Strukturen. Ein generischer Swarm wuerde signifikant schlechtere Ergebnisse liefern als ein spezialisiertes Team.

**Im Zweifelsfall: Generic Fallback.** Lieber einmal zu wenig ein Team erstellen als den Skill mit Nischen-Teams aufblaehem.

### Auto-Creation Ablauf

1. **User informieren:**
   ```
   Kein passendes SWAT-Team fuer diesen Task-Typ vorhanden.
   Task-Typ "[erkannter Typ]" scheint wiederkehrend — soll ich ein neues SWAT-Team dafuer erstellen?
   Das Team wird automatisch fuer zukuenftige Sessions verfuegbar sein.
   [Ja / Nein, nutze Generic Fallback]
   ```

2. **Bei Ja: Team Builder deployen** — `Read references/teams/team-builder.md` und starte den Team Builder Swarm (Research-Agent → Prompt-Engineer → Validator)

3. **Nach erfolgreicher Validation:**
   - Neues Team-File nach `references/teams/<name-slug>.md` schreiben
   - Team-Index in SKILL.md updaten (neue Zeile in der Team-Tabelle)
   - User informieren: "Neues SWAT-Team '[Name]' erstellt und einsatzbereit."

4. **Sofort deployen** — Das gerade erstellte Team wird DIREKT fuer den aktuellen Task deployed (kein zweiter Aufruf noetig). Der User wartet nicht bis zur naechsten Session.

---

## Teil 2: Post-Swarm Learning (Bestehende Teams verbessern)

### Wann feuert Post-Swarm Learning?

**NUR in diesen Faellen:**
- Quality Gate hat **FAIL** zurueckgegeben (Rework wurde durchgefuehrt)
- Quality Gate hat **PASS_WITH_NOTES** mit mindestens einem **MAJOR**-level Item zurueckgegeben

**NICHT feuern bei:**
- Clean PASS — das Team hat gut funktioniert, nichts zu verbessern
- PASS_WITH_NOTES mit nur MINOR Items — zu geringfuegig fuer Prompt-Aenderungen
- Generic Fallback wurde verwendet (kein Team-File zum Verbessern)

### Analyse-Schritte

Nach Abschluss des Swarms (inkl. aller Rework-Zyklen):

**Schritt 1: Failure-Pattern identifizieren**

Analysiere die QG-Issues und ordne sie einer dieser Kategorien zu:

| Pattern | Ursache | Prompt-Fix |
|---------|---------|------------|
| **Unvollstaendiger Output** | Worker hat Teile des Output-Contracts uebersprungen | Fehlende Sektionen im Prompt expliziter machen, "MUST include" Hinweis |
| **Falsches Format** | Worker hat Output in anderem Format als Contract geliefert | Output-Format-Sektion mit konkreterem Beispiel versehen |
| **Fehlende Edge Cases** | Worker hat Standard-Fall bearbeitet aber Edge Cases ignoriert | Edge-Case-Checkliste zum Prompt hinzufuegen |
| **Inkonsistenz zwischen Workern** | Parallele Worker haben widersprüchliche Annahmen getroffen | Shared Constraints-Sektion im Prompt ergaenzen |
| **Scope Creep** | Worker hat mehr gemacht als seine Rolle verlangt | Boundaries-Sektion schaerfen ("Du bist NICHT verantwortlich fuer...") |
| **Mangelnde Tiefe** | Worker hat oberflaechlich gearbeitet statt gruendlich | Spezifischere Analyse-Schritte hinzufuegen |
| **Integration Failure** | Outputs passen nicht zusammen | Consumes-Rollen-Prompts mit expliziterem Format-Matching versehen |

**Schritt 2: Betroffene Prompt-Sektion lokalisieren**

Identifiziere EXAKT welche Zeilen in welcher Rolle betroffen sind:
- Welcher Worker hat den fehlerhaften Output produziert?
- Welche Prompt-Sektion hat die Schwaeche verursacht? (Objective, Schritte, Rules, Output Format, Constraints)
- Ist es ein fehlender Hinweis oder ein zu vager bestehender Hinweis?

**Schritt 3: Targeted Fix formulieren**

Formuliere die minimale Prompt-Aenderung die das Problem behebt:
- **Hinzufuegen**: Fehlende Instruktion oder Constraint ergaenzen
- **Praezisieren**: Vage Instruktion durch spezifischere ersetzen
- **Beispiel hinzufuegen**: Konkretes Beispiel zum Output-Format hinzufuegen
- **NICHT**: Ganze Prompts umschreiben, Rollen-Struktur aendern, oder neue Rollen hinzufuegen

### Aenderung anwenden

**Schritt 4: User-Approval einholen**

Zeige dem User die geplante Aenderung:
```
Post-Swarm Learning: QG hat [Issue] gefunden.
Ursache: [Pattern] in Rolle [Name], Prompt-Sektion [Section]
Vorgeschlagener Fix:

  Vorher: "[bestehende Zeile]"
  Nachher: "[verbesserte Zeile]"

Soll ich die Team-Definition aktualisieren? [Ja / Nein]
```

**Schritt 5: Edit ausfuehren (nur bei Ja)**

- `Edit` auf `references/teams/<team-name>.md` — nur die betroffene Stelle
- Aenderungskommentar am Ende der Datei ergaenzen (falls noch nicht vorhanden, neue Sektion):

```markdown
## Changelog

- [YYYY-MM-DD] [Rolle]: [1-Satz Beschreibung der Aenderung] (Trigger: QG [FAIL/PASS_WITH_NOTES])
```

### Guardrails

1. **Max 1 Team-Edit pro Swarm-Einsatz** — Nicht mehrere Aenderungen auf einmal. Eine Aenderung, ein Lerneffekt.
2. **Nur targeted Edits** — Nie mehr als 5 Zeilen pro Edit aendern. Wenn das Problem groesser ist: dem User empfehlen den Team Builder fuer eine Ueberarbeitung zu nutzen.
3. **Nie strukturelle Aenderungen** — Rollen hinzufuegen/entfernen, Workflow-Graph aendern, oder consumes umstrukturieren ist NICHT Aufgabe des Post-Swarm Learning. Das erfordert einen bewussten Redesign-Schritt.
4. **User-Approval ist Pflicht** — Keine stille Aenderung an Team-Files. Der User muss jede Aenderung sehen und bestaetigen.
5. **Changelog fuehren** — Jede Aenderung wird im Team-File dokumentiert. So kann der User die Evolution nachvollziehen.

---

## Teil 3: Team Fitness Audit (Explizit getriggert)

Wird NICHT automatisch ausgefuehrt. Nur wenn der User explizit fragt:
- "Pruefe ob die SWAT-Teams noch optimal sind"
- "Audit die Team-Definitionen"
- `/agent-swat-swarm team-audit`

### Audit-Protokoll

Fuer jedes Team in `references/teams/`:

**1. Structural Check**
- Hat jede Rolle model, subagent_type, consumes?
- Ist der Workflow-Graph konsistent mit den Rollen?
- Hat jede Rolle einen Output-Contract?
- Gibt es DA Focus (5+ Fragen) und Success Criteria (5+ Kriterien)?

**2. Prompt Quality Check**
- Ist jede Instruktion domaenenspezifisch (nicht generisch)?
- Enthaelt jeder Prompt `{task_context}`?
- Sind alle `{Rollenname.output}` Platzhalter korrekt?
- Gibt es vage Formulierungen die praezisiert werden koennten?

**3. Changelog Review**
- Hat das Team einen Changelog? Wie viele Aenderungen gab es?
- Gibt es wiederkehrende Patterns in den Aenderungen (gleiche Rolle, gleiches Problem)?
- Wenn ja: die Rolle braucht moeglicherweise ein grundlegenderes Redesign

**4. Coverage Gap Analysis**
- Welche Task-Typen kommen haeufig vor aber haben kein spezialisiertes Team?
- Gibt es Teams die nie genutzt werden (Kandidaten zum Entfernen)?

### Audit-Output

```
## Team Fitness Report

| Team | Structural | Prompts | Changelog | Verdict |
|------|-----------|---------|-----------|---------|
| Security Audit | PASS | PASS | 0 changes | Healthy |
| Frontend Feature | PASS | 1 vage Stelle | 2 changes | Minor improvement suggested |
| ... | ... | ... | ... | ... |

### Empfehlungen
1. [Team]: [Konkrete Empfehlung]
2. [Coverage Gap]: [Vorschlag fuer neues Team]
```
