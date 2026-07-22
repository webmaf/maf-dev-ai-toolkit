# State Management

## Architektur

Die Anwendung verwendet kein zentrales State-Management wie Redux oder Zustand.

Stattdessen kombiniert sie drei Mechanismen:

```text
React Context
        ↓
AppContent
        ↓
URL State
        ↓
Persistierung (SCORM/localStorage)
```

---

# Globale Zustände

## AudienceContext

Verantwortlich für:

- aktuelle Zielgruppe
- Zielgruppenwechsel
- Zielgruppen-Konfiguration

---

## ExportDataContext

Verantwortlich für:

- Exportdaten
- Kategorien
- Kompetenzen
- JobFamily-Konfiguration
- Trainings-Metadaten

---

# Persistierung

Persistierbare Daten werden über den `userStateService` gespeichert.

Speicherorte:

- SCORM
- localStorage (Fallback)

Persistiert werden u.a.:

- Audience
- Navigation
- Filter
- Suchbegriff
- Onboardingstatus

---

# URL als State

Die URL ist Teil des State-Managements.

Sie speichert beispielsweise:

- aktive View
- aktiven Tab
- Suchbegriff
- Filter
- Deep Links

Dadurch sind Browser-History und Lesezeichen möglich.

---

# Verantwortlichkeiten

| Bereich | Aufgabe |
|----------|----------|
| AudienceContext | Zielgruppe |
| ExportDataContext | Katalogdaten |
| AppContent | UI-State |
| userStateService | Persistierung |
| URL | Navigation & Deep Links |

---

# Architekturentscheidung

Globaler Zustand wird bewusst auf mehrere Verantwortlichkeiten verteilt.

- Fachliche Daten → Context
- Navigation → URL
- Persistenz → Service
- UI-Zustand → AppContent

Dadurch bleibt kein einzelner "God Store" bestehen.