# Netzwerkprotokoll

Das Netzwerkprotokoll definiert sämtliche Nachrichten, die über das Netzwerk verschickt und empfangen werden können.

Eine Netzwerknachricht ist stets wie folgt aufgebaut: 

> `[NETWORK_MESSAGE_IDENTIFIER] [NETWORK_MESSAGE_PARAM_1] ... [NETWORK_MESSAGE_PARAM_N]`

Dabei können zwischen dem Netzwerknachricht-Identifier und den abgeschlossenen Netzwerknachricht-Parametern beliebig viele Leerzeichen (>= 1) stehen.

Unterstützte Netzwerknachricht Identifier sind

- [GameStart](#gamestart)
- [PlaceTileAndToken](#placetileandtoken)
- **(🛑 Assignment 2)** [ExchangeTokens](#exchangetokens-assignment-2) 
- [ShopState](#shopstate)
- [RequestInput](#requestinput)
- [PlacedTile](#placedtile)
- [PlacedToken](#placedtoken)
- [GameEnded](#gameended)
- [SyntaxError](#syntaxerror)
- [SemanticError](#semanticerror)
- **(🛑 Assignment 2)** [UpdatedNatureToken](#updatednaturetoken-assignment-2) 

*Hinweis: Vor dem Netzwerknachricht-Identifier als auch nach dem letzten Netzwerknachricht-Parameter dürfen KEINE Leerzeichen stehen. Zudem hat eine Netzwerknachricht KEINE Zeilenumbrüche (\n) am Ende!*

## Datentypen

### NET-INT

Eine Kombination von Ganzzahlen (Dezimalzahlensystem), die in einen (signed) 32-bit Integer passen.

Beispiele für keinen NET-INT Typen:

- 0-3
- x152
- 3\.
- 2147483648

### NET-POSITION

Ist ein Tuple zur Repräsentation der (x, y) - Koordinaten am Spielfeld.

Format: `[NET-INT],[NET-INT]`

<img src="pictures/Koordinaten.png" alt="Koordinaten" width="350"/>

### NET-ENUM

Repräsentiert einen genau definierten Auswahlbereich.

#### NET-ROTATION

Ist ein NET-ENUM mit dem Auswahlbereich { `0`, `1`, `2`, `3`, `4`, `5` }.

Der Wert entspricht der Anzahl der Rotationen um eine Kante im Uhrzeigersinn.

<img src="pictures/rotation_long.png" alt="Rotation" width="450"/>

#### NET-HABITAT

Ist ein NET-ENUM mit dem Auswahlbereich { `M`, `W`, `F`, `P`, `R` }.

| Wert         | Bedeutung                                            |
| ------------ | ---------------------------------------------------- |
| `M`          | Gebirge (Mountain)                                   |
| `W`          | Feuchtgebiet (Wetland)                               |
| `F`          | Wald (Forest)                                        |
| `P`          | Prärie (Prairie)                                     |
| `R`          | Fluss (River)                                        |

#### NET-ANIMAL

Ist ein NET-ENUM mit dem Auswahlbereich { `B`, `S`, `H`, `E`, `F` }.

| Wert         | Bedeutung                                            |
| ------------ | ---------------------------------------------------- |
| `B`          | Bär (Bear)                                           |
| `S`          | Lachs (Salmon)                                       |
| `H`          | Bussard (Hawk)                                       |
| `E`          | Elch (Elk)                                           |
| `F`          | Fuchs (Fox)                                          |

#### NET-SCORE

Ist ein NET-ENUM mit dem Auswahlbereich { `A`, `B`, `C`, `D`, `E` }.

| Wert         | Bedeutung                                            |
| ------------ | ---------------------------------------------------- |
| `A`          | Verwendung der Wertung 'A'.                          |
| `B`          | Verwendung der Wertung 'B'.                          |
| `C`          | Verwendung der Wertung 'C'.                          |
| `D`          | Verwendung der Wertung 'D'.                          |
| `E`          | Verwendung der 'Einfachen Wertung (Für Einsteiger)'. |

### NET-TILE

Ein NET-TILE repräsentiert ein Wildnisplättchen und ist daher eine Zusammensetzung aus 2 verschiedenen Informationen.

Format: `[NET-HABITAT][NET-HABITAT]-[NET-ANIMAL]([NET-ANIMAL]([NET-ANIMAL]))`

| Argument     | Bedeutung                                                    |
| ----------   | ------------------------------------------------------------ |
| 1            | Die beiden Landschaftinformation des Wildnisplättchen.       |
| 2            | Tiere, die auf dem Wildnisplättchen platziert werden können. |

Wichtig: Sollte das Wildnisplättchen nur eine einzelne Landschaft aufweisen, dann sind die Landschaftsinformationen ident (siehe Beispiel).

(⚠) Alle `[NET-ANIMAL]`-Einträge müssen verschieden sein, d.h. `FF-F` wäre korrekt, `FF-FFF` jedoch nicht. (⚠)

*Hinweis: Man beachte die Leerzeichensetzung-Zwischen den beiden NET-HABITAT Einträgen, aber auch den NET-ANIMAL Einträgen ist KEIN Leerzeichen, es zählt daher als jeweils als ein Argument.*

Beispiele

- `FF-F`: Wildnisplättchen mit Wald-Landschaft, bewohnbar von Fuchs.
- `MP-BF`: Wildnisplättchen mit Gebirge / Prarie Landschaft, bewohnbar von Bär und Fuchs.

### NET-START-TILE

Ein `NET-START-TILE` repräsentiert eine *Start-Landschaft*, d.h. einen Komplex aus 3 regulären Wildnisplättchen. 

Format: `[NET-TILE] [NET-ROTATION] [NET-TILE] [NET-ROTATION] [NET-TILE] [NET-ROTATION]`

| Argument     | Bedeutung                                                      |
| ----------   | -------------------------------------------------------------- |
| 1            | Das 1. Tile der Start-Landschaft (wird auf (0, 0) platziert).  |
| 2            | Die Rotation des 1. Tiles der Start-Landschaft.                |
| 3            | Das 2. Tile der Start-Landschaft (wird auf (0, 1) platziert).  |
| 4            | Die Rotation des 2. Tiles der Start-Landschaft.                |
| 5            | Das 3. Tile der Start-Landschaft (wird auf (1, 1) platziert).  |
| 6            | Die Rotation des 3. Tiles der Start-Landschaft.                |

### NET-STRING

Ein `NET-STRING` repräsentiert eine beliebige ASCII-Zeichenkette der Länge >= 1, die jedoch

- keine Leerzeichen
- keine Kontrollzeichen (wie z.B. `\n`, `\r`, `\t`, ...)

enthält.

### NET-PLAYER

Ein NET-PLAYER ist eine Zusammensetzung aus 18 verschiedenen Informationen.

Format: `[NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT] [NET-INT]`

| Argument     | Bedeutung                                                                         |
| ----------   | --------------------------------------------------------------------------------- |
| 1            | ID des Spielers.                                                                  |
| 2            | Summe aller Punkte für Bären.                                                     |
| 3            | Summe aller Punkte für Elche.                                                     |
| 4            | Summe aller Punkte für Lachs.                                                     |
| 5            | Summe aller Punkte für Bussard.                                                   |
| 6            | Summe aller Punkte für Füchse.                                                    |
| 7            | Anzahl der größten, zusammenhäng. Wildnisplättchen mit Landschaft 'Berg'.         |
| 8            | Anzahl der erhaltenen Bonuspunkte für den Landschaftstypen 'Berg'.                |
| 9            | Anzahl der größten, zusammenhäng. Wildnisplättchen mit Landschaft 'Wald'.         |
| 10           | Anzahl der erhaltenen Bonuspunkte für den Landschaftstypen 'Wald'.                |
| 11           | Anzahl der größten, zusammenhäng. Wildnisplättchen mit Landschaft 'Prarie'.       |
| 12           | Anzahl der erhaltenen Bonuspunkte für den Landschaftstypen 'Prarie'.              |
| 13           | Anzahl der größten, zusammenhäng. Wildnisplättchen mit Landschaft 'Feuchtgebiet'. |
| 14           | Anzahl der erhaltenen Bonuspunkte für den Landschaftstypen 'Feuchtgebiet'.        |
| 15           | Anzahl der größten, zusammenhäng. Wildnisplättchen mit Landschaft 'Fluss'.        |
| 16           | Anzahl der erhaltenen Bonuspunkte für den Landschaftstypen 'Fluss'.               |
| 17           | Anzahl der verbleibenen Zapfen-Token.                                           |
| 18           | Punkte des Spielers insgesamt.                                                    |

Hinweis: Für die genauen Punkte-Berechnungen und weitere Beispiele, siehe [Spielwertung](Spielwertung.md).

## Nachrichten

### GameStart

Information über den Spielstart und die aktuelle Spielkonfiguration.

Typ: Servernachricht (Server -> Client)

> `GameStart [NET-INT] [NET-SCORE] [NET-SCORE] [NET-SCORE] [NET-SCORE] [NET-SCORE] [NET-STRING] [NET-STRING]( [NET-STRING]( [NET-STRING]))`

| Argument     | Bedeutung                                            |
| ------------ | ---------------------------------------------------- |
| 1 (⚠️)        | Anzahl der Runden des Spieles.                       |
| 2            | Wertungstyp für Bären.                               |
| 3            | Wertungstyp für Hirsche.                             |
| 4            | Wertungstyp für Lachse.                              |
| 5            | Wertungstyp für Bussarde.                            |
| 6            | Wertungstyp für Füchse.                              |
| 7            | Name des 1. Spielers (ID 0).                         |
| 8            | Name des 2. Spielers (ID 1).                         |
| 9 (optional) | Name des 3. Spielers (ID 2), falls vorhanden.        |
| 10 (optional) | Name des 4. Spielers (ID 3), falls vorhanden.       |

*Hinweis: Die (semantisch) korrekte Anzahl der Parameter hängt von der Anzahl der Spieler\*innen ab.*

### ShopState

Information über die Wildnisplättchen und Tier-Tokens des derzeitigen Auswahlbereichs.

Typ: Servernachricht (Server -> Client)

> `ShopState [NET-TILE] [NET-ANIMAL] [NET-TILE] [NET-ANIMAL] [NET-TILE] [NET-ANIMAL] [NET-TILE] [NET-ANIMAL]`

| Parameter    | Bedeutung                                            |
| ------------ | ---------------------------------------------------- |
| 1            | Wildnisplättchen des Auswahlbereichs (Index 0).      |
| 2            | Tier-Token des Auswahlbereichs (Index 0).          |
| 3            | Wildnisplättchen des Auswahlbereichs (Index 1).      |
| 4            | Tier-Token des Auswahlbereichs (Index 1).          |
| 5            | Wildnisplättchen des Auswahlbereichs (Index 2).      |
| 6            | Tier-Token des Auswahlbereichs (Index 2).          |
| 7            | Wildnisplättchen des Auswahlbereichs (Index 3).      |
| 8            | Tier-Token des Auswahlbereichs (Index 3).          |

*Hinweis: Diese Nachricht muss vom Server immer verschickt werden, sobald sich der Auswahlbereich ändert, d.h. im Falle vom Austausch (durch Überbevölkerung oder Zapfen-Token) sowie nach jedem Spielzug.*

### PlaceTileAndToken

Aktionsnachricht, welches Wildnisplättchen und welcher Tier-Token aus dem Auswahlbereich genommen werden soll und wohin diese anschließend platziert werden sollen.

Typ: Clientnachricht (Client -> Server)

> `PlaceTileAndToken [NET-INT] [NET-POSITION] [NET-ROTATION] [NET-INT]( [NET-POSITION])`

| Parameter    | Bedeutung                                            |
| ------------ | ---------------------------------------------------- |
| 1            | Index des Tiles vom Auswahlbereich.                  |
| 2            | Position, an den das Tile gelegt werden soll.        |
| 3            | Rotation des platzierten Tiles.                      |
| 4            | Index des Tier-Tokens vom Auswahlbereich.          |
| 5 (optional) | Position, an den das Tier-Token gelegt werden soll.|

*Hinweis: Wenn der 5. Parameter nicht gegeben ist, wird der Tier-Token verworfen. Für die Implementierung der Spielmechanik ist zu beachten, dass der Spieler sequentiell zuerst das Wildnisplättchen, und erst anschließend den Tier-Token legt, d.h. er könnte diesen ggf. auch auf den gerade positionierten Plättchen legen*

### ExchangeTokens (🛑 Assignment 2)

Aktionsnachricht, welche Tier-Tokens im Auswahlbereich ausgetauscht werden sollen.

Typ: Clientnachricht (Client -> Server)

> `ExchangeTokens [NET-INT]( [NET-INT]( [NET-INT]( [NET-INT])))`

| Parameter    | Bedeutung                                                   |
| ------------ | ----------------------------------------------------------- |
| 1            | Index des 1. Tier-Tokens, der ausgetauscht werden soll.   |
| 2 (optional) | Index des 2. Tier-Tokens, der ausgetauscht werden soll.   |
| 3 (optional) | Index des 3. Tier-Tokens, der ausgetauscht werden soll.   |
| 4 (optional) | Index des 4. Tier-Tokens, der ausgetauscht werden soll.   |

Wichtig: Diese Aktionsnachricht kann vom Spieler verschickt werden, wenn

- dieser gegen Tannenzapfen-Token beliebige Tokens aus dem Auswahlbereich austauscht (Assignment 2)
- der Spieler einen Tausch wegen 'Einfachere Überbevölkerung' anfordert (Assignment 2)

### RequestInput

Information, dass der Server auf die Eingabe eines Spielers wartet.

Typ: Servernachricht (Server -> Client)

> `RequestInput [NET-INT]`

| Parameter    | Bedeutung                                            |
| ----------   | ---------------------------------------------------- |
| 1            | ID des Spielers, der nun am Zug ist.                 |

### PlacedTile

Information, dass ein\*e Spieler\*in im Spielzug soeben ein Wildnisplättchen platziert hat.

Typ: Servernachricht (Server -> Client)

> `PlacedTile [NET-INT] [NET-POSITION] [NET-ROTATION] [NET-TILE]`

| Parameter    | Bedeutung                                            |
| ----------   | ---------------------------------------------------- |
| 1            | ID des Spielers, der das Wildnisplättchen gelegt hat.|
| 2            | Position, an die das Wildnisplättchen gelegt wurde.  |
| 3            | Rotation des gelegten Wildnisplättchen
| 4            | Wildnisplättchen, welches gelegt wurde.              |

*Hinweis: Diese Nachricht wird nur verschickt, wenn der jeweilige Spieler auch ein Wildnisplättchen erfolgreich platziert hat.*

### PlacedToken

Information, dass ein\*e Spieler\*in im Spielzug soeben ein Token platziert hat.

Typ: Servernachricht (Server -> Client)

> `PlacedToken [NET-INT] [NET-POSITION] [NET-ANIMAL]`

| Parameter    | Bedeutung                                            |
| ----------   | ---------------------------------------------------- |
| 1            | ID des Spielers, der den Tier-Token gelegt hat.    |
| 2            | Position, an die der Tier-Token gelegt wurde.      |
| 3            | Tier-Token, welcher gelegt wurde.                  |

*Hinweis: Diese Nachricht wird nur verschickt, wenn der jeweilige Spieler auch ein Tier-Token erfolgreich platziert hat.*

### UpdatedNatureToken (🛑 Assignment 2)

Information, dass sich die Anzahl der Tannenzapfen-Tokens eines Spielers verändert hat.

Typ: Servernachricht (Server -> Client)

> `UpdatedNatureToken [NET-INT] [NET-INT]`

| Parameter    | Bedeutung                                                                     |
| ----------   | ----------------------------------------------------------------------------- |
| 1            | ID des Spielers, der den Tannenzapfen-Token verwendet hat.                  |
| 2            | Aktuelle Anzahl an Tannenzapfen-Token des jeweiligen Spielers.              |

### GameEnded

Information über das Spielende, inkl. der finalen Punkteauswertung für jede\*n Spieler\*in.

Typ: Servernachricht (Server -> Client)

> `GameEnded [NET-PLAYER] [NET-PLAYER]( [NET-PLAYER]( [NET-PLAYER]))`

| Parameter    | Bedeutung                                                              |
| ----------   | ---------------------------------------------------------------------- |
| 1            | Punkte-Aufschlüsslung für Spieler mit ID 1.                          |
| 2            | Punkte-Aufschlüsslung für Spieler mit ID 2.                          |
| 3 (optional) | Punkte-Aufschlüsslung für Spieler mit ID 3 (falls vorhanden).        |
| 4 (optional) | Punkte-Aufschlüsslung für Spieler mit ID 4 (falls vorhanden).        |

*Hinweis: Die (semantisch) korrekte Anzahl der Parameter hängt von der Anzahl der Spieler\*innen ab.*

### SyntaxError

Information über den Erhalt einer Eingabe, die nicht dem Netzwerkprotokoll entspricht.

Typ: Servernachricht (Server -> Client)

`SyntaxError( [NET-STRING_1]( ...( [NET-STRING_N])))`

| Parameter       | Bedeutung                                                              |
| --------------- | ---------------------------------------------------------------------- |
| 1 - n (optional)| Beliebige Informationen                                                |

*Hinweis: Ihr könnt hier beliebige Informationen angeben, die euch bei der Fehlersuche helfen können. Wichtig ist jedoch immer noch, dass das Grundformat einer Netzwerknachricht eingehalten wird, d.h. nicht mit einem Leerzeichen oder Sonderzeichen wie '\n' beginnt oder endet. Die Informationen werden in der GUI für euch angezeigt.*

*Tipp: Es empfiehlt sich sehr, zumindest die betroffene User-ID mitzugeben, damit für euch ersichtlich ist, welche\*r Spieler\*in diesen verursacht hat.* 

Beispiele 

- `SyntaxError`
- `SyntaxError 1337`
- `SyntaxError player 1 tried to place token, but tile does not support it!`
- `SyntaxError <playerid = 4> cannot <TilePlaceAndTokens 1 2,2 3 4,4> (tile not placable).`

### SemanticError

Information über den Erhalt einer Eingabe, die nicht dem Spielregeln entspricht.

Typ: Servernachricht (Server -> Client)

> `SemanticError( [NET-STRING_1]( ...( [NET-STRING_N])))`

| Parameter       | Bedeutung                                                              |
| --------------- | ---------------------------------------------------------------------- |
| 1 - n (optional)| Beliebige Informationen                                                |

*Hinweis: Ihr könnt hier beliebige Informationen angeben, die euch bei der Fehlersuche helfen können. Wichtig ist jedoch immer noch, dass das Grundformat einer Netzwerknachricht eingehalten wird, d.h. nicht mit einem Leerzeichen oder Sonderzeichen wie '\n' beginnt oder endet. Die Informationen werden in der GUI für euch angezeigt.*

*Tipp: Es empfiehlt sich sehr, zumindest die betroffene User-ID mitzugeben, damit für euch ersichtlich ist, welche\*r Spieler\*in diesen verursacht hat.* 

Beispiele

 - `SyntaxError`
 - `SyntaxError 404`
 - `SyntaxError player <1> send <  ExchangeTokens 0 1 2 3>`
 - `SyntaxError 1 "  ExchangeTokens 0 1 2 3 " (ill formed whitespace)`