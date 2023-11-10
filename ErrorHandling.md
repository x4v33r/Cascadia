# Error-Handling 

Wir unterscheiden zwischen

- [Netzwerkparsing-Fehler](#netzwerkparsing-fehler)
- [Spielmechanik Fehler](#spielmechanik-fehler)
- [Fatale Fehler](#fatale-fehler)

## Netzwerkparsing-Fehler

Wenn eine Nachricht nicht dem [Netzwerkprotokoll](Netzwerkprotokoll.md) entspricht, dann handelt es sich um einen *Netzwerkparsing-Fehler*. In diesem Fall sollen alle Spieler\*innen mit einer [`SyntaxError`](Netzwerkprotokoll.md#syntaxerror)-Netzwerknachricht benachrichtigt werden.

## Spielmechanik-Fehler

Diese Fehler betreffen sämtliche Fehler die auftreten, wenn ein\*e Spieler\*in Aktionen anfragt, die laut der aktuellen Assignmentbeschreibung nicht erlaubt sind. Voraussetzung für diese Fehler ist immer eine gültige Nachricht, die dem Netzwerkprotokoll entspricht.

Im Falle eines Spielmechanik-Fehlers soll die ungültige angefragte Aktion der Spieler\*innen nicht ausgeführt werden, sondern lediglich mit einer [`SemanticError`](Netzwerkprotokoll.md#semanticerror)-Netzwerknachricht über den Fehler informiert werden.

Gründe für einen solchen Spielmechanik-Fehler sind beispielsweise: 

- **Falsche Spieler\*innen-Reihenfolge**: Sollte ein\*e Spieler\*in eine Nachricht abschicken, obwohl diese\*r jedoch nicht an der Reihe ist.
- **Unzureichende Ressourcen**: Sollte eine Aktion Tannenzapfen-Token benötigen, welcher der\*die Spieler\*in jedoch nicht hat.

*Hinweis: Werte die in einer Größenordnung oder in einem ungültigen Wertebereich liegen, die nicht vom Netzwerkprotkoll unterstützt werden, fallen unter Netzwerkparsing-Fehler, nicht unter Spielmechanik-Fehler.*

## Fatale Fehler

Fatale Fehler beinhalten Fehler die zur sofortigen Beendigung des Spieles führen sollen. Dazu gehören vor allem:

- [Konfigurationsfehler](#konfigurationsfehler)
- [Netzwerkfehler](#netzwerkfehler)
- [Unzureichender Speicher](#unzureichender-speicher)
- [Sonstige Laufzeitfehler](#sonstige-laufzeitfehler)

In sämtlichen Fällen sollen dennoch alle allokierten Ressourcen ordnungsgemäß freigegeben werden, bevor das Programm beendet wird.

### Konfigurationsfehler

Sollte das Programm mit ungültigen Argumenten gestartet werden, dann ist das Programm mit einem vordefinierten Rückgabewert zu beenden.

| Grund                                   | Rückgabewert   |
| --------------------------------------- | -------------- |
| Ungültige Anzahl der Startargumente     | `1`            |
| Ungültiger Wert für `number_of_players` | `2`
| Ungültiger Wert für `rulesets`          | `3`            |
| Ungültige Datei für `tile_deck_file` (Dateipfad existiert nicht, fehlende Lese-Berechtigung, etc.) | `4`           |
| Ungültige Datei für `animal_deck_file` (Dateipfad existiert nicht, fehlende Lese-Berechtigung, etc.) | `5`           |
| Ungültiger Wert für `number_of_rounds`  | `6`           |
| Nicht alle Dateieinträge von `tile_deck_file` entsprechen dem vorgegebenen Format   | `7`            |
| Ungültige Anzahl an Einträgen der NET-TILES in `tile_deck_file`                     | `8`            |
| Nicht alle Dateieinträge von `animal_deck_file` entsprechen dem vorgegebenen Format | `9`            |
| Invalide Anzahl an Einträgen der NET-ANIMAL in `animal_deck_file`                   | `10`            |

Dabei ist zu beachten, dass diese Fehler (aufsteigend) geordnet nach Priorität sind.

### Netzwerkfehler

Im Falle eines Netzwerkfehlers (z.B. bei einem *Timeout*) soll das Programm mit dem Rückgabewert `-1` beendet werden.

> **Hinweis**: Siehe [Framework.md](Framework.md#exceptions)

### Unzureichender Speicher

Bei dem klassischen *Out-out-Memory*-Problem soll das Programm mit dem Rückgabewert `-2` beendet werden.

> **Hinweis**: Siehe [std::bad_alloc](https://en.cppreference.com/w/cpp/memory/new/bad_alloc)-Exception.

### Sonstige Laufzeitfehler

Bei allen anderen Laufzeitfehlern, die zu keinen der oben genannten Fehlern gehören, soll das Programm mit dem Rückgabewert `-3` beendet werden.

> **Hinweis**: Siehe [std::runtime_error](https://en.cppreference.com/w/cpp/error/runtime_error)-Exception.
