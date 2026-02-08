# System Design: Proaktive Wetter-Benachrichtigung für Mitarbeiter

## Übersicht
Dieses System extrahiert Mitarbeiter-Standorte aus SAP-Systemen (S/4HANA oder SuccessFactors) und sendet proaktive Wetterwarnungen.

## Architektur (Mermaid)
```mermaid
sequenceDiagram
    participant S4 as S/4HANA / SuccessFactors
    participant IS as BTP Integration Suite (iFlow)
    participant WAPI as Weather API (external)
    participant NT as Notification Service (Email/Teams/Telegram)

    Note over IS: Job-basiertes Triggering (z.B. täglich 07:00)
    
    IS->>S4: Fetch Employee Data (Location, Contact)
    S4-->>IS: List of Employees & Locations
    
    loop Pro Mitarbeiter
        IS->>WAPI: Get Forecast (Location)
        WAPI-->>IS: Weather Data (Temp, Rain, etc.)
        
        alt Wetter-Warnung nötig? (z.B. Sturm/Regen)
            IS->>IS: Format Personal Message
            IS->>NT: Send Proactive Notification
            NT-->>IS: Delivery Status
        else Wetter OK
            Note over IS: Kein Handlungsbedarf
        end
    end
    
    Note over IS: Log Summary & Error Handling
```

## Best Practices & Ausnahmen
### Best Practices
- **Batch Processing:** Abruf der Mitarbeiterdaten gesammelt, um API-Calls zu minimieren.
- **Payload-Filterung:** Nur notwendige Felder (Email, City) von SAP abfragen.
- **Asynchrone Benachrichtigung:** Die Integration Suite wartet nicht auf die Zustellung der Nachricht, um den Prozess nicht zu blockieren.

### Ausnahmen (Error Handling)
- **API Timeout:** Implementierung eines Exponential Backoff bei der Wetter-API.
- **Fehlende Standortdaten:** Mitarbeiter ohne validen Standort werden geloggt, aber der Prozess läuft für andere weiter.
- **Rate Limiting:** Einhaltung der Quotas der externen Wetter-API.
