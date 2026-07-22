Schritte:
1. Dateizählung aus dem Workspace.
2. Grobe SLOC‑Schätzung basierend auf Anzahl und Typen der Dateien.
3. Erkennung des Tech‑Stacks aus package.json und vite.config.ts.
4. Herkunfts- und Zweck‑Zusammenfassung aus CLAUDE.md und misc-Dokumenten.

Antworten

1) Projektgröße (grobe Werte)
- Dateien: ca. 411 Dateien insgesamt im Repo (davon ~273 unter src).  
- Zeilen Code (grob): geschätzt ca. 40k–80k Zeilen (stark abhängig von großen JSON/CSV-Dateien in sources — z. B. education_catalogue_complete_v5.json enthält viele Zeilen). Methode: Anzahl Quelldateien × mittlere Dateigröße; wenn du eine exakte Zählung willst, kann ich sie berechnen.  
- Art: Frontend (React‑Single‑Page‑App). Es gibt keine eigenständige Backend‑API im Repo — Integration zu ILIAS geschieht über Proxies / externe ILIAS‑Instanz (siehe vite.config.ts und dataSourceConfig.ts).

2) Verwendete Technologien (Kernliste)
- Framework / Sprache: React 19 + TypeScript (~5.8). (siehe package.json und `src/*.tsx`)  
- Build / Dev: Vite 7 (vite.config.ts)  
- Styling: Tailwind CSS  
- Animation / UI: Framer Motion, radix/ui (Radix Komponenten), shadcn/ui utilities  
- Data / Misc libs: `react-virtuoso` (virtuelles Scrolling), `xlsx` (Excel), `react-markdown`, `cologne-phonetic` (phonetische Suche), `pipwerks-scorm-api-wrapper` (SCORM)  
- Tools / Scripts: Node/npm, verschiedene scripts unter scripts für SCORM/ZIP etc.  
- Datenquellen: lokale JSON/Excel + Catalogue Manager Export (ILIAS) — Fallback‑Dateien unter sources.

3) Wie ist das Projekt entstanden?
- Basis: offensichtlich ein Vite + React Projekt, anschließend stark weiterentwickelt (eigenes vite.config.ts, viele Komponenten, Loader).  
- Daten/Content: Trainingsdaten wurden aus Word‑Dokumenten extrahiert (siehe `misc/*.docx` und die generierten berufanfaenger-trainings-data.json) — es gibt begleitende Arbeitsdokus in misc.  
- Hinweise auf AI‑Unterstützung / agenten: Projekt enthält agent/skill‑Dateien und CLAUDE.md, außerdem Hinweise, dass Teile (Daten/Docs) automatisiert erzeugt wurden.  
- Integration: Konzipiert zur Laufzeit mit einem externen ILIAS Backend (Production) und Vite‑Proxy für lokale Entwicklung.

4) Wofür ist das Projekt gedacht?
- Zweck: Einheitlicher Trainingskatalog für zwei Bereiche (`next campus` und `Weiterbildung`) — Anzeigen von Veranstaltungen, Filtern nach Kategorien/Zielgruppe, Buchungs‑/Terminlogik, Dashboard, WBT/SCORM‑Support. (Siehe CLAUDE.md und docs.)  
- Features: Katalogansicht, Kalenderansicht, Dashboard, phonetische Suche, Kategorie-/Tag‑Filter, SCORM‑Packaging für LMS, Export/Import von Catalogue Manager Daten, Offline‑Fallback für Entwicklung (`?perfMode=1`).

Kurz‑Zusammenfassung und Angebot
- Repo ≈ 400 Dateien, Frontend React/TypeScript, Kern-Stacks: Vite + Tailwind + Framer Motion + react-virtuoso.  
- Entstehung: Vite template → domain‑spezifische Erweiterung + automatisierte Datengenerierung / AI‑Hilfen.  
- Zweck: Trainingskatalog mit ILIAS‑Integration und lokalen Fallbacks.

---

Ich öffne main.tsx, dann folge den wichtigsten Imports (App.tsx, Context-Provider, Router) und erkläre den Render‑Flow bis zur ersten sichtbaren Seite. Ich lese die beteiligten Dateien dafür kurz ein.

Read [](file:///Users/matthias/projects/catalogue-next/src/main.tsx)

Read [](file:///Users/matthias/projects/catalogue-next/src/App.tsx#1-1), lines 1 to 400

Read [](file:///Users/matthias/projects/catalogue-next/src/App.tsx#400-400), lines 400 to 800

Read [](file:///Users/matthias/projects/catalogue-next/src/App.tsx#800-800), lines 800 to 1200

Read [](file:///Users/matthias/projects/catalogue-next/src/App.tsx#1200-1200), lines 1200 to 1230

Read [](file:///Users/matthias/projects/catalogue-next/src/contexts/AudienceContext.tsx)

Read [](file:///Users/matthias/projects/catalogue-next/src/contexts/ExportDataContext.tsx)

Ich erkläre jetzt schrittweise den Start‑/Renderfluss ab main.tsx bis zur ersten sichtbaren Seite; ich habe die beteiligten Dateien gelesen.

Schritte (kurz)
1. Start in main.tsx → React-Root rendern.  
2. `App`-Komposition: Provider → AuthGuard → ExportData → `AppContent`.  
3. Contexts initialisieren (`Audience`, `ExportData`).  
4. Auth‑Check (`ILIASAuthGuard`) entscheidet, ob Kinder gerendert werden.  
5. `ExportDataProvider` lädt Export-Daten.  
6. `AppContent` lädt Trainings, zeigt LoadingScreen, dann Dashboard/Katalog.

Detaillierte Ablaufbeschreibung (Datei‑Referenzen)

1) Bootstrapping
- main.tsx lädt Styles, registriert Offline‑Icons und ruft:
  main.tsx
  createRoot(...).render(<App />).

2) App‑Hülle und Provider
- `App` setzt die globale Struktur:
  App.tsx
  Es wrappt `AppContent` in:
  - `AudienceProvider` — bestimmt aktuelle Audience (URL / localStorage / default) (AudienceContext.tsx).
  - `ILIASAuthGuard` — führt ILIAS‑Auth‑Prüfung (lokal via Vite‑Proxy oder Prod‑Check) durch (ILIASAuthGuard.tsx).
  - `ExportDataProvider` — lädt Catalogue‑Exportdaten (JSON) und stellt sie als Context bereit (ExportDataContext.tsx).

3) Auth-Guard Wirkung (sichtbar zuerst)
- Beim ersten Mount ist `ILIASAuthGuard` im „checking“-Zustand → zeigt einen vollflächigen Loader (Preloader/iframe‑Fallback). Wenn der lokale ILIAS‑Proxy nicht erreichbar, setzt die Guard‑Logik in vielen Fällen `isAuthenticated=true` (zum Entwickeln) — sonst kann sie das ILIAS‑Login in einem iframe anzeigen. Sie bestimmt also, ob App‑Inhalt überhaupt gerendert wird (siehe `checkLocalAuth` / Fehlerbehandlung) (ILIASAuthGuard.tsx).

4) Export‑Daten laden
- Sobald `ILIASAuthGuard` erlaubt, mountet `ExportDataProvider`, der `loadExportData()` asynchron aufruft. Währenddessen ist `isLoading` true; die Datenquelle versucht Remote → falls fehlschlägt oder in DEV, lädt es lokale Fallbacks (z. B. catalogue-export.json) (ExportDataContext.tsx und src/data/loaders/exportDataLoader.ts ).

5) `AppContent` Initialisierung und Preload
- `AppContent` liest `useAudience()` und `useExportData()` und startet:
  - SCORM/UserState (restore von saved state),
  - Preloading (carousel colors, phonetic pregeneration, `startUserInfoFetch()`),
  - sowie das Laden und Enrichen der Trainings via `loadTrainingsForAudience(...)` sobald Export‑Metadata verfügbar ist. (`AppContent` initialisiert `currentView='dashboard'` standardmäßig) (App.tsx, `loadTrainingsForAudience` in trainingLoader.ts).

6) LoadingScreen & Timing
- Die UI zeigt ein Branding/LoadingScreen solange zwei Bedingungen erfüllt sein müssen: Mindest‑Zeit (∼2500ms) und Daten‑Ladevorgang (`dataLoaded`) fertig. Erst danach wird der Loader ausgeblendet und das Hauptlayout sichtbar (App.tsx).

7) Erste sichtbare Seite
- Standardpfad: `currentView === 'dashboard'` → `TrainingDashboard` wird als erste gerenderte Seite angezeigt (unter der SharedHeader/Hauptstruktur). Falls URL‑Parameter (`?view=catalogue` / `?audience=...` / `?training=...`) vorhanden sind, weichen Startansicht und Fokus ab (z. B. direkte Katalogansicht oder Training‑Detail). Relevant: `AppContent` parst URL‑Parameter nach dem Laden und navigiert entsprechend (App.tsx, App.tsx).

Kurz‑Zusammenfassung der Abhängigkeiten und Entscheidungsstellen
- Sichtbar zuerst: entweder AuthGuard‑Loader/iframe (falls ILIAS‑Login nötig) oder das App‑LoadingScreen (Branding). Erst wenn Auth + ExportData + Trainings fertig (plus Mindestzeit) → SharedHeader + `TrainingDashboard` (default) oder `TrainingCatalogue` / `TrainingDetail` bei Deeplinks.
- Wichtige Kontrollpunkte: `ILIASAuthGuard` (block/allow), `ExportDataProvider.isLoading`, `AppContent.dataLoaded` + min timer.

Was ich noch prüfen kann (optional)
- Exakte Stelle, die `loadExportData()` verwendet (größere Fehlerquelle beim DEV‑Proxy).  
- Ablauf bei `?perfMode=1` (schaltet Auth + Remote aus → schneller Start mit lokalen Fallbacks).  
Sag kurz, welche der drei Optionen du als Nächstes willst: exakte SLOCzählung, Top‑Files nach Größe, oder detaillierte Erklärung eines der Loader (z. B. `exportDataLoader` oder `trainingLoader`).
