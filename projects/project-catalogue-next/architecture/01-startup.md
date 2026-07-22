# Application Startup

## Einstieg

Die Anwendung startet in `main.tsx`.

```text
main.tsx
    ↓
<App />
```

---

## Initialisierung

Die Anwendung wird in folgender Reihenfolge aufgebaut:

```text
AudienceProvider
    ↓
ILIASAuthGuard
    ↓
ExportDataProvider
    ↓
AppContent
```

### AudienceProvider

Ermittelt die aktive Zielgruppe.

### ILIASAuthGuard

Prüft die Authentifizierung und blockiert die Anwendung bis zum Ergebnis.

### ExportDataProvider

Lädt die Exportdaten und stellt sie global zur Verfügung.

### AppContent

Initialisiert die eigentliche Anwendung.

---

## Datenfluss

```text
Exportdaten
      ↓
Training Loader
      ↓
UI
```

---

## Erste Ansicht

Standardmäßig wird das Dashboard angezeigt.

URL-Parameter können den Start überschreiben (Deep Links).