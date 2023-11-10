# Framework

Dafür stellen wir folgende Systeme zur Verfügung:

- [das Netzwerk-System](Framework.md#Netzwerk-System)
- [das Zufallzahlen-System](Framework.md#Zufallszahlen-System)

Im Folgenden ist die Verwendung der beiden Komponenten beschrieben.

> **_Wichtig:_** : Alle **bereitgestellten Klassen** dürfen **nicht verändert** werden.
> Explizit frei veränderbar ist *main.cpp*. Außerdem dürfen Sie in *CMakeLists.txt*
> Ihre Sourcecode-Dateien eintragen, aber nichts anderes verändern. Sollten Sie sich
> nicht an diese Vorgabe halten, kann dies zu rigorosem Punkteabzug (bis inkl. 100%) führen.

## Netzwerk-System

Sämtliche netzwerkspezifischen Implementierungen (Empfangen einer Nachricht, Versenden einer Nachricht, etc.) sind bereits vorhanden.
Sie müssen also lediglich die dafür vorgesehenen Funktionen an der richtigen Stelle (mit den richtigen Parametern) aufrufen.

Um das Netzwerk-System verwenden zu können, muss das Header-File mittels `#include "network/Server.hpp"` eingebunden werden.

### Initialisierung

Das Initialisieren der Netzwerk-Klasse wird von der bereitgestellten *main.cpp* bereits übernommen.
Dort wird eine Instanz des Objektes `net::Server` erstellt:

`net::Server server(unt number_of_player);`

- **Beschreibung**: Initialisieren des Netzwerksystems. Dabei wird die weitere Ausführung des Codes solange pausiert, bis sich alle Clients verbunden haben.
- **Parameter `number_of_player`**: Anzahl an Spieler, auf die für die Verbindung gewartet wird.

### Funktionalitäten

Das `net::Server` Objekt verfügt über folgende Methoden:

`Message waitForNewInput();`

- **Beschreibung**: Der Server wartet so lange, bis eine Nachricht von einem Client empfangen wird.
- **Rückgabewert**: Nachricht, die von einem Client empfangen wurde.

`void send(const std::string& content)`

- **Beschreibung**: Der Server verschickt eine Nachricht an alle verbundenen Clients.
- **Parameter `content`**: Die zu verschickende Nachricht.

`std::size_t getNumberOfPlayers() const;`

- **Beschreibung**: Der Server liest die Anzahl der verbunden Clients aus seinem Speicher aus.
- **Rückgabewert**: Anzahl der Spieler.

`const std::string& getPlayerName(std::size_t player_index) const;`

- **Beschreibung**: Der Server liest den Spielernamen eines verbundenen Clients aus seinem Speicher aus.
- **Parameter `player_index`**: SpielerID jenes Clients, dessen Name ausgegeben werden soll.
- **Rückgabewert**: Der Spielername des angegebenen Clients.

### Exceptions

Beim Aufruf des Konstruktors sowie der obigen Methoden kann es zu folgenden Fehlern kommen:

- `net::TimeoutException`: Tritt im Falle eines Timeouts auf.
- `net::NetworkException`: Tritt auf, wenn über den angegebene Port keine Verbindung erzeugt werden konnte, oder es zu einem internen Netzwerkfehler kommt.

Der korrekte Umgang in diesen Fehlerfällen ist in [ErrorHandling.md](ErrorHandling.md) beschrieben.

> **Hinweis**: Um diese Exceptions in der *main.cpp* verwenden zu können, müssen Sie das entsprechende Header-File mittels `#include network/Exceptions.hpp` inkludieren.

## Zufallszahlen-System

Das Mischen der Start-Wildnissplättchen, der *regulären* Wildnisplättchen des Ziehstapels als auch das Mischen der Animal-Tokens vom Beutel wird von unserem bereitgestellten Zufallszahlen-System übernommen. Dieses kann ganz einfach über `#include random/Random.hpp` inkludiert werden.

Hier bietet die bereitgestellte Klasse `Random` folgende Methoden:

`static void initialize();`

- **Beschreibung**: Initialisieren der Zufalls-Datenstruktur. 
- **Anmerkung**: Diese Funktion darf insgesamt nur **EINMAL** am Beginn des Programms (noch VOR Aufruf der Shuffle Funktion!) aufgerufen werden.

`static void Shuffle(Container_t& container);`

- **Beschreibung**: Mischt sämtliche Elemente einer Datenstruktur *zufällig* durch.
- **Anmerkung**: Es ist unbedingt notwendig, dass die richtige Reihenfolge beachtet wird, *welche* Datenstrukturen *wann* gemischt werden sollen, da es ansonsten zu Unstimmigkeiten mit unseren Testcases kommt!
- **Parameter `container`**: Eine beliebige Datenstruktur, welche *Iteratoren* unterstützt. Dazu gehören beispielsweise [`std::vector`](https://en.cppreference.com/w/cpp/container/vector), [`std::array`](https://en.cppreference.com/w/cpp/container/array), [`std::deque`](https://en.cppreference.com/w/cpp/container/deque) oder [`std::map`](https://en.cppreference.com/w/cpp/container/map).

> **Hinweis**: Es handelt sich hier um sowohl bei `initialize`, als auch bei `Shuffle` um [**statische Methoden**](https://www.learncpp.com/cpp-tutorial/static-member-functions/). Das heißt, der Zugriff erfolgt nicht über eine Objektinstanz, sondern direkt über den Klassentypen.
Beispielaufruf: `Random::Shuffle(my_vector);`, wobei `my_vector` in diesem Beispiel den Typen `std::vector<int>` hat.
