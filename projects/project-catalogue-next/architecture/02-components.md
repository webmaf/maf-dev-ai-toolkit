# Komponentenübersicht

## Ziel

Die Anwendung ist in vier Ebenen gegliedert:

```text
Infrastruktur
    ↓
Navigation
    ↓
Business Views
    ↓
UI-Komponenten
```

---

# Komponentenbaum

```text
App
│
├── Provider Layer
│   ├── AudienceProvider
│   ├── ILIASAuthGuard
│   └── ExportDataProvider
│
└── AppContent
    │
    ├── SharedHeader
    ├── Dashboard
    ├── Catalogue
    ├── DetailView
    ├── Compass
    ├── Workshops
    ├── Coaching
    ├── Community
    │
    └── Overlays
         ├── Onboarding
         ├── GuidedTour
         └── Support
```

---

# Komponenten

## App

Root-Komponente der Anwendung. Initialisiert die Provider und startet die Anwendung.

---

## Provider Layer

Stellt globale Funktionalität bereit.

| Provider | Aufgabe |
|----------|----------|
| AudienceProvider | Verwaltung der aktiven Zielgruppe |
| ILIASAuthGuard | Authentifizierung |
| ExportDataProvider | Bereitstellung der Exportdaten |

---

## AppContent

Orchestriert die komplette Anwendung.

Verantwortlich für:

- Laden der Trainingsdaten
- Wechsel zwischen den Hauptansichten
- Initialisierung der Anwendung
- Verwaltung des globalen UI-Zustands

---

## SharedHeader

Zentrale Navigation der Anwendung.

Verantwortlich für:

- Navigation
- Zielgruppenwechsel
- Branding

---

## Business Views

| View | Aufgabe |
|------|----------|
| Dashboard | Einstieg und Übersicht |
| Catalogue | Trainings suchen und filtern |
| DetailView | Trainingsdetails anzeigen |
| Compass | Wissensportal |
| Workshops | Workshopprogramme |
| Coaching | Coaching-Angebote |
| Community | Community-Inhalte |

---

## Overlays

Zusätzliche UI-Komponenten, die unabhängig von den Hauptansichten angezeigt werden.

- Onboarding
- Guided Tour
- Support