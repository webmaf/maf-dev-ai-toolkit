# Phase 1: Das Projekt verstehen lassen (einmalig)

Lass Copilot zunächst nicht programmieren, sondern das Projekt analysieren.

Prompts wie:

Analysiere die Architektur dieses Projekts. Erstelle eine Übersicht über Module, Verantwortlichkeiten und Datenfluss. Nimm noch keine Änderungen vor.

Danach:

Welche Coding-Konventionen erkennst du?

Dann:

Welche Design Patterns werden verwendet?

Dann:

Welche Risiken oder technischen Schulden siehst du?

So baut Copilot ein gutes mentales Modell auf.

# Phase 2: Dokumentation erzeugen

Bei großen Projekten lohnt es sich, eine KI-lesbare Dokumentation zu erstellen.

Zum Beispiel:

docs/
    architecture.md
    frontend.md
    backend.md
    database.md
    api.md
    coding-guidelines.md

Diese Dateien dienen weniger den Menschen als der KI. Je besser sie sind, desto besser werden die Antworten.

# Phase 3: Copilot Instructions

Hier gehört alles hinein, was Claude "implizit wusste".

Zum Beispiel:

# Projekt

Legacy jQuery Projekt.

## Architektur

- MVC
- kein React
- kein Angular
- kein TypeScript

## Coding Style

- ES6
- keine globalen Variablen
- Funktionen klein halten
- Kommentare nur bei komplexer Logik

## Bei Änderungen

- vorhandenen Stil beibehalten
- keine unnötigen Refactorings
- Accessibility beachten
- Performance beachten
# Phase 4: In kleinen Schritten arbeiten

Nicht:

Baue das komplette Modul neu.

Sondern:

Erkläre zuerst dieses Modul.

Dann:

Welche Klassen sind beteiligt?

Dann:

Wie würdest du das verbessern, ohne das Verhalten zu ändern?

Dann:

Implementiere nur Schritt 1.

So bleibt die Kontrolle bei dir.