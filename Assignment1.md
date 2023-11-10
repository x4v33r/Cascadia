# Assignment 1

**Achtung:** Geänderte Inhalte sind mit einem (⚠️) markiert.

## Aufgaben

Zu den Hauptaufgaben in diesem Assignment zählen:

- Parsen der Netzwerknachrichten des Spielers (`PlaceTileAndToken`)
- Implementieren des einfachen Spielablaufs
  - Initialisierung der Spielfelder.
  - Implementierung des Auswahlbereichs.
  - Implementierung der Wildnisplättchen und Tier-Token Platzierungen.
  - Implementierung der **Einfache Wertung (Für Einsteiger)**

Folgende Bereiche sind explizit **NICHT** in diesem Assignment zu implementieren:

- Umgang mit Überbevölkerung im Auswahlbereich.
- Tannenzapfen-Token (sowie die damit verbundenen Spielzugmöglichkeiten).
- Wertung der Landschaften oder anderen Wertungskarten (B, C, D).


## Rahmenbedingungen

- Konsolenausgaben auf STDOUT und STDERR (`std::cout`, `std::cerr`) sind erlaubt.
- Alle bereitgestellten Klassen dürfen nicht verändert werden.
Explizit frei veränderbar ist `main.cpp`. Außerdem dürfen Sie in `CMakeLists.txt` Ihre Sourcecode-Dateien eintragen (aber keine anderen Änderungen durchführen).
- Dieses Assignment ist in **Gruppenarbeit** zu erledigen, wobei sich jede\*r Gruppenteilnehmer\*in im selben Ausmaß an der Implementierung beteiligen muss.

⚠️ Achtung ⚠️ Die Reihenfolge der Rulesets in den Kommandozeilenargumenten hat sich seit A0 geändert und ist nun konsistent mit den Netzwerknachrichten. Die neue Reihenfolge lautet: { Bär, Elch, Lachs, Bussard, Fuchs }. Solltet ihr die Reihenfolge nicht anpassen wird es bei A3 zu Fehlern führen.

## Spielinhalte

### Auswahlbereich

Der Auswahlbereich ist jener Bereich, von dem die Spieler\*innen eine der 4 Wildnis-/Tierkombinationen (unten visualisiert als *Option A, B, C, D*) auswählen können.

Folgende`ShopState` Nachricht entspricht dabei der darauf folgenden Visualisierung:

> `ShopState RM-HB E FW-BF S PR-ES B FR-ES F`

<img src="pictures/shop.png"  width="375" height="150">

### Spielfeld

Jede\*r Spieler\*in hat ein eigenes Spielfeld. Diese Spielfelder unterliegem einem 2D Koordinatensystem, welches wie folgt aufgebaut ist:

<img src="pictures/Koordinaten.png"  width="350" height="350">

### Deck bzw. Beutel

Darunter fallen konkret

- Stapel für Start-Landschaften
- Stapel für reguläre Landschaften
- Beutel für Animal-Token

Diese sind in einer beliebigen Datenstruktur umzusetzten, die Iteratoren unterstützten. Dazu gehören beispielsweise `std::vector`, `std::array`, `std::deque`, etc.

Der Initialvorgang wird beim Programm- bzw. Spielstart näher beschrieben, grundsätzlich wird jedoch immer zuerst die jeweilige Datenstruktur, die den Stapel / den Beutel repräsentiert, zuerst befüllt und anschließend mittels [`Random::Shuffle(...)`](Framework.md#zufallszahlen-system) gemischt.

Immer, wenn ein Element von einer dieser Datenstrukturen während des Spiels zur Zuweisung an die Spieler\*innen entnommen wird, wird dafür das **erste** Element der jeweiligen Datenstruktur (das sich ganz am Anfang befindet) hergenommen. 
Dieses wird anschließend aus der Datenstruktur **entfernt**.

Immer, wenn ein Animal-Token zurück in den Beutel gelegt wird, wird der betroffene Animal-Token **an das Ende der jeweiligen Datenstruktur** hinzugefügt, worauf **anschließend** erneut [`Random::Shuffle(...)`](Framework.md#zufallszahlen-system) auf diese Datenstruktur angewendet wird.

## Programmstart

Der Programmstart baut auf Assignment 0 auf, es gibt jedoch folgende Änderungen:

- Die ersten 5 (⚠) Zeileneinträge der `tile_deck_file` Textdatei sind im Format [`NET-START-TILE`](Netzwerkprotokoll.md#net-start-tile) und repräsentieren die *Start-Landschaften*. Diese sind separat von den anderen Wildnisplättchen in eine eigene Datenstruktur zu speichern. Dabei ist darauf zu achten, dass die 3 Wildnisplättchen pro Start-Landschaft **zusammen** in der **eingelesenen Reihenfolge** gespeichert werden (siehe Hinweis).

- (⚠) Sollten die ersten 5 Zeileneinträge nicht dem [`NET-START-TILE`](Netzwerkprotokoll.md#net-start-tile) Format entsprechen ODER andere Zeileneinträge innerhalb der Textdatei das [`NET-START-TILE`](Netzwerkprotokoll.md#net-start-tile) Format haben, liegt ein **ungültiger Eintrag** vor (siehe [ErrorHandling.md](ErrorHandling.md#konfigurationsfehler), Fehlercode `7`)

- Die Anzahl der Einträge (= Zeilen) der `tile_deck_file` Textdatei muss damit **mindestens** `(number_of_players * number_of_rounds + 3 + 5)` betragen, um gültig zu sein.

Nach dem Einlesen der Textdateien wird sequentiell

 1. [`Random::Initialize()`](Framework.md#zufallszahlen-system) insgesamt genau 1x aufgerufen (vgl. Assignment 0)
 2. [`Random::Shuffle(...)`](Framework.md#zufallszahlen-system) für die Datenstruktur der Start-Wildnisplättchen aufgerufen
 3. [`Random::Shuffle(...)`](Framework.md#zufallszahlen-system) für die Datenstruktur der verbleibenen Wildnisplättchen aufgerufen (vgl. Assignment 0)
 4. [`Random::Shuffle(...)`](Framework.md#zufallszahlen-system) für die Datenstruktur der Animal-Token aufgerufen (vgl. Assignment 0)

> **Hinweis**: Die Einhaltung dieser Reihenfolge ist dabei äußerst wichtig, da es ansonsten zu Unstimmigkeiten mit unseren Testcases gibt. Bei der Datenstruktur für die Start-Wildnisplättchen auch darauf zu achten, dass die "3" Wildnisplättchen pro Zeileneintrag zu **einem einzigen** Start-Landschaftkomplex gehören (vgl. Originalspiel). Beim Aufruf von `Random::Shuffle(...)` sollen hierbei also die maximal 5 Landschaftskomplexe untereinander gemischt werden, nicht jedoch die einzelnen Wildnisplättchen untereinander.

## Spielablauf

Für Assignment 1 sieht der Spielablauf wie folgt aus:

### 1. Spielstart

Sobald alle Spieler\*innen mit dem Server verbunden sind, soll dieser alle mit einer [`GameStart`](Netzwerkprotokoll.md#gamestart) Netzwerknachricht über den Spielstart informieren.

Anschließend bekommen alle Spieler\*innen die zusammenhängenden **3 Wildnisplättchen** aus der Datenstruktur für die Start-Wildnisplättchen.
Diese werden ab Erhalt sofort in deren Spielbereich platziert. 
Dabei wird das erste Wildnisplättchen an Position (0, 0) gelegt, das zweite an die Position (0, 1) und das dritte an die Position (1, 1).

Sobald diese Prozedur für alle Spieler\*innen abgeschlossen ist, wird anschließend der **Auswahlbereich** mit 4 Wildnisplättchen sowie mit 4 Tier - Token befüllt. Anschließend wartet der Server auf eine Aktionsnachricht vom 1. Spieler (mit der Spieler-ID 0). 

> **Hinweis**: Siehe Beschreibung für `net::waitForNewInput()` in [Framework.md](Framework.md#netzwerk-system).

Der komplette Spielstart wird über foglende Sequenz von Netzwerknachrichten kommuniziert: 

1. Der Server informiert über den Spielstart, sobald alle Spieler\*innen mit dem Server verbunden sind ([`GameStart`](Netzwerkprotokoll.md#gamestart))
2. Der Server verschickt die Information, an welche Position die 3 Start-Wildnisplättchen des 1. Spielers platziert wurden (3x [`PlacedTile`](Netzwerkprotokoll.md#placedtile)).
3. (Selbes Prinzip für alle anderen Spieler\*innen).
4. Der Server verschickt die Information des derzeitigen Auswahlbereichs ([`ShopState`](Netzwerkprotokoll.md#shopstate)) an alle Spieler\*innen.
5. Der Server verschickt die Information, welche\*r Spieler\*in nun am Zug ist ([`RequestInput`](Netzwerkprotokoll.md#requestinput)).

### 2. Spielzug

Nun kann der 1. Spieler (mit der ID 0) eine der 4 Wildnisplättchen/Tier Kombinationen aus dem Auswahlbereich auswählen und anschließend in seinen Spielbereich platzieren lassen ([`PlaceTileAndToken`](Netzwerkprotokoll.md#placetileandtoken)). Dabei ist zu bedenken:

- Die Platzierung des Tieres ist optional (siehe Netzwerprotokoll).
- Sollte die Aktionsnachricht des Spielers nicht dem Netzwerkprotokoll entsprechen, informiert der Server alle Spieler\*innen mit einer [`SyntaxError`](Netzwerkprotokoll.md#syntaxerror) Nachricht (siehe [Error-Handling](ErrorHandling.md#netzwerkparsing-fehler)).
- Sollte die Aktionsnachricht des Spielers nicht den Spielregeln entsprechen, informiert der Server alle Spieler\*innen mit einer [`SemanticError`](Netzwerkprotokoll.md#semanticerror) Nachricht (siehe [Error-Handling](ErrorHandling.md#spielmechanik-fehler)). Mögliche Szenarien dafür wären beispielsweise
  - Die Ziel-Platzierungsposition des Wildnissplättchen ist nicht an ein bereits platziertes Wildnisplättchen anliegend.
  - An der Ziel-Platzierungsposition des Tieres befindet sich kein Wildnisplättchen, auf dem das Tier platzierbar ist.
  - An der Ziel-Platzierungsposition des Tieres befindet sich ein Wildnisplättchen, auf dem bereits ein Tier platziert ist.
  - Die Aktionsnachricht kommt vom einem Spieler, der nicht an der Reihe ist.

Im Falle eines Semantic- oder Syntaxfehlers ist nach dem Versenden der Fehlernachricht erneut eine [`RequestInput`](Netzwerkprotokoll.md#requestinput) Netzwerknachricht an alle Spieler\*innen zu verschicken.

Sobald der/die Spieler\*in einen validen Spielzug getätigt hat, informiert der Server alle Spieler\*innen über die Platzierungen und der nächste Spieler ist an der Reihe.

_**Wichtig**: Sollte der/die Spieler\*in kein Tier-Token platzieren, muss dieses [wie oben beschrieben](#deck-bzw-beutel) zurück in den Beutel gelegt werden._

Das Ende eines Spielzugs wird vom Server wie folgt kommuniziert:

1. Der Server informiert alle über das vom Spieler platzierte Wildnisplättchen ([`PlacedTile`](Netzwerkprotokoll.md#placedtile)).
2. (optional) Der Server informiert alle über das platzierte Tier ([`PlacedToken`](Netzwerkprotokoll.md#placedtoken)).
3. Auffüllen des Auswahlbereichs mit anschließender Information der dort befindlichen Wildnisplättchen und Tier-Token ([`ShopState`](Netzwerkprotokoll.md#shopstate)).
4. Der Server informiert alle, welche\*r Spieler\*in nun an der Reihe ist. ([`RequestInput`](Netzwerkprotokoll.md#requestinput)).

(⚠️) Sollte es sich um den letzten Spielzug vor dem Spielende handeln, wird Schritt 3 und Schritt 4 **ausgelassen**.

*Hinweis: In diesem Assignment findet noch KEINE Überprüfung auf Überbevölkerung statt, und es gibt nur "reguläre Spielzüge", da es noch keine Tannenzapfen-Tokens gibt.*

### 3. Spielende

Das Spiel geht solange, bis jede\*r Spieler\*in insgesamt `number_of_rounds` Mal an der Reihe war. 
Alle Spieler\*innen sollen dann dann mit einer [`GameEnded`](Netzwerkprotokoll.md#gameended) - Nachricht informiert werden.

Da wir in diesem Assignment lediglich die [**Einfache Wertung** (Für Einsteiger)](details/WertungEinfach.md), **OHNE die Landschaftswertungen** implementieren, sollen sämtliche Wertungspunkte innerhalb der Netzwerknachricht, die sich auf

- die Landschaftsinformationen
- die Bonuspunkte für Landschaftsinformationen
- die Anzahl der verbleibenen Tannenzapfen - Token

beziehen, auf `0` gesetzt werden. Für genauere Informationen bzgl. der Berechnung der Punkte für Einfachen Wertung, siehe [Spielwertung](Spielwertung.md).

#### Beispiel: GameEnded Nachricht

> `GameEnded 0 2 4 5 6 9 0 0 0 0 0 0 0 0 0 0 0 26 1 8 9 6 5 4 0 0 0 0 0 0 0 0 0 0 0 33`

| Wert                                | 1. Spieler\*in       | 2. Spieler\*in |
| ----------------------------------- | -------------------- | -------------- |
| Spieler\*in ID                      | `0`                  | `1`            |
| Summe aller Punkte für Bären        | `2`                  | `8`            |
| Summe aller Punkte für Elche        | `4`                  | `9`            |
| Summe aller Punkte für Lachs        | `5`                  | `6`            |
| Summe aller Punkte für Bussarde     | `6`                  | `5`            |
| Summe aller Punkte für Füchse       | `9`                  | `4`            |
| Insgesamte Punkteanzahl             | `26`                 | `33`           |

## Beispiele

Hier ist ein potentieller Auszug eines Logprotokolls eines Servers. Der Verständlichkeit halber sind vom Server ausgehende Nachrichten mit `>>` markiert, während eingehende Nachrichten mit `<<` markiert sind.

```text
>>GameStart 4 E E E E E Markus Alex
>>PlacedTile 0 0,0 0 WW-H
>>PlacedTile 0 0,1 0 RF-SEH
>>PlacedTile 0 1,1 0 PM-BF
>>PlacedTile 1 0,0 0 RR-S
>>PlacedTile 1 0,1 0 PF-SEB
>>PlacedTile 1 1,1 0 MW-FH
>>ShopState MP-FEB S PW-ES F FF-E S FF-F H
>>RequestInput 0
<<PlaceTileAndToken 3 1,0 0 3 1,1
>>PlacedTile 0 -1,0 0 FF-E
>>PlacedToken 0 0,1 S
>>ShopState MP-FEB S PW-ES F WR-FS S FF-F H
>>RequestInput 1
<<PlaceTileAndToken 5 1,0 0 3 1,1
>>SyntaxError
>>RequestInput 1
<<PlaceTileAndToken 3 1,0 0 3 1,1
>>PlacedTile 1 1,0 0 FF-F
>>PlacedToken 1 1,1 H
>>ShopState MP-FEB S PW-ES F WR-FS S PP-F S
>>RequestInput 0
<<PlaceTileAndToken 2 -1,1 0 2 -1,1
>>PlacedTile 0 -1,1 0 WR-FS
>>PlacedToken 0 -1,1 S
>>ShopState MP-FEB S PW-ES F MW-BS S PP-F S
>>RequestInput 1
<<PlaceTileAndToken 1 1,-1 0 1 1,0
>>PlacedTile 1 1,-1 0 PW-ES
>>PlacedToken 1 1,0 F
>>ShopState MP-FEB S PR-EH S MW-BS S PP-F S
>>RequestInput 0
<<PlaceTileAndToken 1 -1,2 0 1 -1,2
>>SemanticError Could not place object!
>>RequestInput 0
<<PlaceTileAndToken 2 -1,2 0 2 -1,2
>>PlacedTile 0 -1,2 0 MW-BS
>>PlacedToken 0 -1,2 S
>>ShopState MP-FEB S PR-EH S MW-EF B PP-F S
>>RequestInput 1
<<PlaceTileAndToken 3 2,-1 0 3 0,0
>>PlacedTile 1 2,-1 0 PP-F
>>PlacedToken 1 0,0 S
>>ShopState MP-FEB S PR-EH S MW-EF B PW-EF E
>>RequestInput 0
<<PlaceTileAndToken 2 0,2 0 2 1,1
>>PlacedTile 0 0,2 0 MW-EF
>>PlacedToken 0 1,1 B
>>ShopState MP-FEB S PR-EH S FW-ES H PW-EF E
>>RequestInput 1
<<PlaceTileAndToken 1 0,-2 0 1 1,-1
>>PlacedTile 1 0,-2 0 PR-EH
>>PlacedToken 1 1,-1 S
>>GameEnded 0 2 0 9 0 0 0 0 0 0 0 0 0 0 0 0 0 11 1 0 0 5 2 2 0 0 0 0 0 0 0 0 0 0 0 9
```
