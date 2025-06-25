# Ausarbeitung: Smart Bulb Steuerung mit Event-Driven Architecture

## Gewählte Architektur: Event-Driven Architecture (EDA)

Für die Umsetzung unserer Anwendung haben wir uns für eine *Event-Driven Architecture* entschieden. Diese Architektur wurde nicht nur vorgegeben, sondern eignet sich auch ideal für lose gekoppelte, skalierbare und asynchrone Systeme. Die Kommunikation erfolgt über ein Message-Queue-System (RabbitMQ), bei dem ein *Producer* Ereignisse erzeugt und ein *Consumer* diese verarbeitet. Ebenfalls erleichterte die für das Assignment vorgegebene Vorlage unsere Arbeit.

**Vorteile der EDA in unserem Szenario:**
- **Entkopplung:** Producer (API) und Consumer (Smart Bulb Steuerung) sind unabhängig.
- **Asynchronität:** Befehle müssen nicht in Echtzeit verarbeitet werden.
- **Skalierbarkeit:** Beide Komponenten können unabhängig skaliert oder erweitert werden.
- **Erweiterbarkeit:** Weitere Event-Handler (z. B. Logging, Automationen) können leicht integriert werden.

## Funktionsweise der Anwendung

Die Anwendung besteht aus zwei Hauptkomponenten:

### 1. Producer (API)

Der Producer basiert auf Node.js mit Express und stellt REST-Endpunkte zur Verfügung. Jeder Aufruf eines Endpunkts sendet ein JSON-Objekt als Nachricht an die RabbitMQ-Queue `lamp-command`.

**Beispiele:**
- `POST /power` mit `{ "value": "on" }` → Lampe einschalten
- `POST /color` mit `{ "value": "red" }` → Farbe ändern
- `POST /brightness` mit `{ "value": 80 }` → Helligkeit setzen
- `POST /morse` mit `{ "value": "SOS" }` → Morsecode blinken

### 2. Consumer (Smart Bulb Steuerung)

Der Consumer verbindet sich mit einer TP-Link Smart Bulb über die `tplink-bulbs`-Bibliothek. Gleichzeitig lauscht er auf Nachrichten aus der RabbitMQ-Queue und führt je nach Befehl passende Aktionen aus:

- **`on` / `off`** → Schaltet die Lampe ein/aus
- **`brightness`** → Setzt die Helligkeit (0–100)
- **`color`** → Ändert die Lampenfarbe
- **`morse`** → Wandelt Text in Morsecode um und blinkt die Lampe entsprechend

Der aktuelle Zustand der Lampe wird über einen WebSocket-Server an alle verbundenen Clients in Echtzeit übertragen.

### Morsecode-Funktion

Die Morse-Funktion wandelt Textzeichen in Morsecode um und blinkt die Lampe:
- **`.` (Punkt)** → kurzes Blinken
- **`-` (Strich)** → langes Blinken
- Buchstaben- und Wortpausen werden zeitlich berücksichtigt

### Übersicht der Kommunikation

```
[Benutzer] 
    ↓ (HTTP-POST)
[Producer / REST API] 
    ↓ (JSON)
[RabbitMQ Queue: lamp-command]
    ↓ (Event)
[Consumer / Smart Bulb Steuerung] 
    → [TP-Link Bulb]
    → [WebSocket: Status an Clients]
```

## Fazit

Durch den Einsatz von Event-Driven Architecture konnten wir eine flexible, entkoppelte und einfach erweiterbare Smart-Bulb-Steuerung umsetzen. Die Trennung zwischen API und Geräteansteuerung erlaubt sauberes Design, schnelle Fehlerbehebung und spätere Erweiterbarkeit.