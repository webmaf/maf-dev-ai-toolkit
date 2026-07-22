Ich lese die Struktur der Hauptkomponenten aus App.tsx und ihren Sub-Komponenten, um einen aussagekräftigen Architektur-Baum mit fachlichen Verantwortungen zu erstellen.   

Read [](file:///Users/matthias/projects/catalogue-next/src/components/TrainingDashboard.tsx)

Read [](file:///Users/matthias/projects/catalogue-next/src/components/TrainingCatalogue.tsx#1-1), lines 1 to 150

Read [](file:///Users/matthias/projects/catalogue-next/src/components/SharedHeader.tsx#1-1), lines 1 to 150

Read [](file:///Users/matthias/projects/catalogue-next/src/components/FilterBar.tsx#1-1), lines 1 to 100

Read [](file:///Users/matthias/projects/catalogue-next/src/components/DashboardShowcase.tsx#1-1), lines 1 to 100

Read [](file:///Users/matthias/projects/catalogue-next/src/components/DetailView/DetailView.tsx#1-1), lines 1 to 100

---

## Analyse: Komponentenarchitektur des Training Catalogue

Ich habe die Komponentenstruktur analysiert und erstelle jetzt einen Komponentenbaum bis Ebene 3 mit fachlicher Verantwortung für jede Hauptkomponente.

---

### **Komponentenbaum**

```
App (Root)
├── Provider-Wrapper (Kontext & Auth)
│   ├── AudienceProvider (Zielgruppen-Management)
│   ├── ILIASAuthGuard (Authentifizierung)
│   └── ExportDataProvider (Metadaten-Versorgung)
│
└── AppContent (Main Container)
    ├── SharedHeader (Navigation & Tabs)
    │   ├── NextStepLogo (Branding)
    │   ├── TabNavigation (Catalogue/Kompass/Workshops/Coaching/Community)
    │   ├── AudienceSelector (Jobfamily-Wahl)
    │   └── JobFamilySelectorDialog (Audience-Dialog)
    │
    ├── DecorativeBand (Visual Background)
    │
    ├── LoadingScreen (Initial Loading Overlay)
    │   └── AnimatedStats (Logo Animation)
    │
    ├── MainContent-Shell (View Container)
    │   ├── TrainingDashboard (View: Dashboard)
    │   │   └── DashboardShowcase (Bento Grid + Carousel)
    │   │       ├── SearchBentoCard (Bento-Kachel)
    │   │       ├── DashboardActionButtons (Filter-Buttons)
    │   │       ├── CarouselSection (Highlight-Trainings)
    │   │       └── AnimatedStats (Training Count)
    │   │
    │   ├── TrainingCatalogue (View: Katalog/Kalender)
    │   │   ├── FilterBar (Filter UI)
    │   │   │   ├── FilterDropdownButton (Einzelne Filter)
    │   │   │   ├── FilterOverflowMenu ("Weitere" Menü)
    │   │   │   └── SearchField (Suche)
    │   │   │
    │   │   ├── TrainingTypeToggle (Kalender/Katalog Umschalt)
    │   │   │
    │   │   ├── CatalogueView (Themensektion-Ansicht)
    │   │   │   └── TrainingTypeCard (Trainings-Karte mit Inline-Termine)
    │   │   │
    │   │   ├── Virtuoso (Virtualisiertes Kalender-Scroll)
    │   │   │   ├── WeekGroupHeader (Woche-Header)
    │   │   │   ├── TrainingCard (Einzelner Termin)
    │   │   │   ├── MonthTimeline (Monat-Navigation)
    │   │   │   ├── WBTTeaserBanner (WBT-Teaser oder gefilterte Liste)
    │   │   │   └── WBTFilteredSection (WBT-Trainings)
    │   │   │
    │   │   └── TrainingDetailPanel (Modal: Alt-Panel-Slide-in)
    │   │
    │   ├── WorkshopsTab (View: Workshops)
    │   │   └── ProgramCard / ProgramTimeline
    │   │
    │   ├── CoachingTab (View: Coaching)
    │   │
    │   ├── CommunityOfPracticeTab (View: Community)
    │   │
    │   └── CompassTab (Lazy: Führungs-Kompass)
    │       ├── CompassOverview (Themenseite)
    │       │   └── BentoCard (Themenkachel)
    │       ├── CompassThemenfeldView (Unterseite)
    │       │   ├── CompassArticleBody (Artikel-Inhalt)
    │       │   └── LinkedText (Hyperlinks)
    │       ├── CompassSearchView (Suche)
    │       │   └── SearchBentoCard (Treffer-Karte)
    │       └── CompassSearchBar (Suchfeld)
    │
    ├── DetailView (Full-Page Detail: NEU)
    │   ├── DetailHeader (Hero-Bereich)
    │   ├── EducationDetailView (Bildungs-Trainings)
    │   │   ├── DescriptionMarkdown (Fließtext)
    │   │   ├── AccordionSection (Ziele/Methoden/Voraussetzungen)
    │   │   ├── SidebarSpeakers (Dozent:innen)
    │   │   ├── SidebarTimeline (Termine + Buchung)
    │   │   ├── SidebarVenue (Ort)
    │   │   ├── LearnEnvironment (Lernumgebung)
    │   │   └── RelatedOffersSection (Empfehlung)
    │   │
    │   ├── NextCampusDetailView (next campus)
    │   │   ├── NextCampusDescription (Fließtext)
    │   │   ├── NextCampusTimeline (Termine)
    │   │   ├── NextCampusSpeakers (Dozent:innen)
    │   │   ├── CurrentDateCard (Datums-Vergleich)
    │   │   ├── NextCampusBooking (Buchungs-Logik)
    │   │   └── AvailableDatesModal (Datumsliste)
    │   │
    │   └── WBTDetailView (Web-Based Trainings)
    │       └── WBTAccessCard (Direktlink)
    │
    ├── OnboardingWizard (Overlay: Erstes Mal)
    │   └── JobFamilyWizardConfig (Audience-Auswahl)
    │
    ├── GuidedTour (Lazy: Orientierungs-Tour)
    │   └── TourTooltip (Tour-Sprechblasen)
    │
    ├── NameReveal (Lazy: Gamification-Feature)
    │
    └── SupportFab (Hilfe & Feedback)
        ├── PageHelpDialog (Seiten-spezifische Hilfe)
        └── FeedbackDialog (Rückmeldungs-Formular)
```

---

### **Fachliche Verantwortung – Pro Komponente**

#### **Level 0: Root & Provider**

| Komponente | Verantwortung |
|---|---|
| **App** | Globale Provider-Hülle. Wrappt AppContent in drei Context-Provider (Audience, AuthGuard, ExportData). Keine direkte UI-Rendering. |
| **ILIASAuthGuard** | **Auth-Türsteher**: Prüft, ob Nutzer im lokalen ILIAS (Vite-Proxy) oder Produktion authentifiziert ist. Bei `localhost`: kontaktiert `/ilias.php?baseClass=ilDashboardGUI` via Proxy. Falls fehlgeschlagen oder Fallback-Modus (`?perfMode=1`): lässt Nutzer durch. Zeigt Login-iframe nur bei echtem Auth-Fehler. |
| **AudienceProvider** | **Zielgruppen-Verwaltung**: Verwaltet aktive Job-Family (Führungskräfte, IT-Experten, etc.). Liest aus URL (`?audience=...`), localStorage oder Default (audiences.json). Stellt `setAudience()`, `currentAudienceId`, `availableAudiences` bereit. |
| **ExportDataProvider** | **Metadaten-Verteilung**: Lädt Catalogue-Export (JSON) aus ILIAS oder localem Fallback. Parsed Training-Kategorien, Job-Family-Configs, Kompetenzen, Bilder. Stellt O(1)-Lookups via Maps bereit (`getMetadataForTraining()`, `getCategory()`, etc.). |

---

#### **Level 1: Main Views/Layouts**

| Komponente | Verantwortung |
|---|---|
| **AppContent** | **Orchestrator**: Koordiniert alle Views (Dashboard/Catalogue/Detail). Managed top-level State (currentView, currentTab, Filters, selectedTraining). Lädt Trainings-Daten, startet Preloading, zeigt LoadingScreen. Verwaltet URL-Parameter & Browser-Back-Button. Setzt ScrollContainer (OverlayScrollbarsComponent) für alle Views. |
| **SharedHeader** | **Navigation & Branding**: Fixierter Header mit Logo, Audience-Selector, Tab-Pills (Catalogue/Kompass/Workshops/Coaching/Community), Easter Eggs (Logo-Tap → Changelog, Konami-Code → Legacy-Branding). Zeigt Pill-Animation bei Tab-Wechsel. |
| **DecorativeBand** | **Visual Context**: Dekoratives Hintergrund-Band (fixed), sichtbar hinter allen Views. Rein visuell, ohne Funktionalität. |
| **LoadingScreen** | **Bootstrap-Overlay**: Zeigt DRV-Branding + AnimatedStats (Logo-Wellenanimation) beim Start. Wartet auf Mindestzeit (∼2500ms) + Daten-Laden. Fade-out nach beide Bedingungen erfüllt. |

---

#### **Level 1–2: Content Views**

| Komponente | Verantwortung | Subkomponenten (Ebene 2) |
|---|---|---|
| **TrainingDashboard** | **Startseite**: Zeigt Willkommensgruß + dynamisches Showcase-Layout. Lädt Job-Family-Config aus Export. Warnt, wenn Export-Daten fehlen. | `DashboardShowcase` |
| **TrainingCatalogue** | **Katalog-Container**: Große List-View mit Filterung, Suche, Sortierung. Managt Listing-View Toggle (Kalender/Katalog). Nutzt Virtuoso für virtualisiertes Rendering großer Listens. Koordiniert `TrainingCard`, `TrainingTypeCard`, Filterung. | `FilterBar`, `TrainingTypeToggle`, `CatalogueView`, `Virtuoso` (virtualisiert), `TrainingDetailPanel` (alt) |
| **DetailView** | **Trainings-Detailseite (NEU)**: Fullpage-Detailansicht mit Scroll-Overlay statt Modal. Dispatch auf spezialisierte Sub-Views (`EducationDetailView`, `NextCampusDetailView`, `WBTDetailView`). Hero + Akkordeons + Sidebar-Panels. | `EducationDetailView`, `NextCampusDetailView`, `WBTDetailView` |
| **WorkshopsTab** | **Workshops-Seite**: Zeigt Workshop-Programme (Trainings-Serien/Cohorts). Ähnlich zu Katalog, aber thematische Struktur. | `ProgramCard`, `ProgramTimeline` |
| **CoachingTab** | **Coaching-Beratung**: Spezielle Inhalte zu Coaching-Angeboten. | (spezifische Subkomponenten) |
| **CommunityOfPracticeTab** | **Community-Netzwerk**: Sektion für Community-of-Practice. | (spezifische Subkomponenten) |
| **CompassTab (Lazy)** | **Führungs-Kompass**: Wissensportal für Führungskräfte. Drei Sub-Ansichten: Themenübersicht, Themenseite (Artikel), Suche. Lazy-loaded (react-markdown). | `CompassOverview`, `CompassThemenfeldView`, `CompassSearchView` |

---

#### **Level 2: Shared/Filter Components**

| Komponente | Verantwortung |
|---|---|
| **FilterBar** | **Filter-UI-Shell**: Koordiniert Dropdown-Filter (Kategorien, Kompetenzen, Format, Ort, Kosten) + Suchfeld. Managt Responsive-Overflow (wenn Filter nicht passen → "Weitere" Menü). Zeigt aktive Filter-Badges. |
| **FilterDropdownButton** | **Einzelner Dropdown**: Ein Filter-Dropdown (z. B. "Thema", "Format"). Öffnet Liste der Optionen, zeigt Auswahlstatus. |
| **FilterOverflowMenu** | **"Weitere"-Menü**: Popup-Menü für Filter, die nicht im Viewport passen. |
| **SearchField** | **Suchfeld**: Text-Input für Freiwort-Suche. Nutzt Kölner Phonetik für deutsche Texte. Debounced (200ms). |
| **TrainingTypeToggle** | **Kalender ↔ Katalog**: Toggle-Button für zwei Listenansichten. Kalender: chronologisch nach Woche/Monat. Katalog: nach Thema/Kategorie. |

---

#### **Level 2–3: Training Cards & Detail Sections**

| Komponente | Verantwortung | Subkomponenten |
|---|---|---|
| **TrainingCard** | **Kalender-Karte**: Einzelner Termin in der Kalenderansicht. Zeigt Datum, Titel, Location, Capacity-Status (grün/orange/rot), Anmelde-Status. | (–) |
| **TrainingTypeCard** | **Katalog-Karte**: Training-Typ mit Inline-Termin-Aufklappung. Zeigt Trainingstyp-Info, Klick öffnet Termine. | (–) |
| **CatalogueView** | **Themenansicht**: Gruppiert Trainings nach Kategorie (statt Datum). Jede Kategorie hat Kopfzeile + Trainings-Grid. | `WeekGroupHeader`, `TrainingTypeCard` |
| **WBTTeaserBanner** | **WBT-Teaser oder Liste**: Je nach Config "Teaser" (1–2 Highlights) oder "Liste" (gefilterte WBT). | (–) |
| **DetailView (Education)** | **Detailseite für Bildungs-Trainings**: Accordion-Struktur (Ziele, Methoden, Voraussetzungen, Kompetenzen). Sidebar mit Dozent:innen, Termine, Ort, Lernumgebung. | `DetailHeader`, `AccordionSection`, `SidebarSpeakers`, `SidebarTimeline`, `SidebarVenue`, `LearnEnvironment` |
| **DetailView (next campus)** | **Detailseite für next campus**: Spezialseite mit next campus Räume (Fotos), Buchungslogik (buchungstool/mail/reference/direct), Alternative Termine, Map-Dropdown. | `DetailHeader`, `NextCampusTimeline`, `NextCampusSpeakers`, `CurrentDateCard`, `AvailableDatesModal` |
| **DetailView (WBT)** | **Detailseite für Web-Based Trainings**: Minimale View. Hauptsächlich Beschreibung + direkter Link zu WBT-Plattform (WBT-Zugang). | `DetailHeader`, `WBTAccessCard` |
| **AccordionSection** | **Aufklappbare Sektion**: Ziele, Methoden, Voraussetzungen, Kompetenzen expandierbar. Mit Animation. | (–) |
| **SidebarTimeline** | **Termine-Panel**: Listet alle zukünftigen Termine für Bildungs-Training. Buchungs-Button pro Termin. | (–) |
| **SidebarBooking** | **Buchungs-Logik (Education)**: Zeigt Buchungs-Status (buchbar/ausgebucht/fixiert), Capacity-Farbe (grün/orange/rot), Meldeschluss. | (–) |
| **NextCampusBooking** | **Buchungs-Logik (next campus)**: Zeigt Buchungs-Methode (buchungstool/mail/reference/direct), Formular oder Link. | `BookingConfirmDialog` |

---

#### **Level 1–2: Overlays & Dialog Components**

| Komponente | Verantwortung | Subkomponenten |
|---|---|---|
| **OnboardingWizard** | **First-time Setup**: Interaktiver Wizard zur Wahl der Jobfamily. Nur beim ersten Start (Flag in SCORM). Nach Completion: Dashboard-Renderung hinter Blur, dann Tour startet. | (Job-Family-Selection) |
| **GuidedTour (Lazy)** | **Orientierungstour**: React-Joyride basiert, mit Tour-Steps (tourSteps.ts). Zeigt Spotlight + Sprechblasen für UI-Elemente. Lazy-loaded, da joyride schwer ist. | `TourTooltip` |
| **SupportFab** | **Hilfe & Feedback**: Runder Button unten rechts. Bei Klick: zwei kleine Buttons (Hilfe, Feedback). Hilfe zeigt Seiten-spezifischen Dialog, Feedback öffnet Fragebogen. | `PageHelpDialog`, `FeedbackDialog` |
| **NameReveal (Lazy)** | **Gamification**: Easter Egg. Mit `?reveal=true` starten → animiertes Zahlenratespiel mit confetti. Lazy-loaded (canvas-confetti). | (–) |
| **TrainingDetailPanel (ALT)** | **Detail-Modal (Slide-in)**: Alte Version. Slide-in Panel von rechts. Nutzt Framer Motion. Feature-gated; kann via `useEducationDetailView`-Flag deaktiviert werden. | (–) |

---

### **Datenflussmuster**

1. **Context-Kommunikation** (nicht Prop-Drilling):  
   - `useAudience()` → Jobfamily-Change  
   - `useExportData()` → Metadaten-Zugriff (kategorialisiert, O(1)-Lookups)  
   - `useTrainingMetadata()` → Training-spezifische Anreicherung

2. **URL als State-Quelle**:  
   - `?audience=`, `?view=`, `?tab=`, `?training=`, `?tags=`, `?categories=`, `?search=`  
   - `AppContent` parst URL, setzt lokalen State, synced zu Browser-History

3. **SCORM Persistence**:  
   - `userStateService` speichert Audience, Filters, View, Tab auf `unload`  
   - Restore beim Start (vor Renderung)

---

### **Architektur-Highlights**

| Aspekt | Lösung |
|---|---|
| **Große Listen** | `react-virtuoso` mit customScrollParent (App-Level OverlayScrollbars) |
| **Filter-Performance** | `useDeferredValue()` für Debouncing + Virtuoso-Windowing |
| **Lazy-Code-Split** | CompassTab, GuidedTour, NameReveal, ChangelogDialog (jeweils Suspense) |
| **Responsive Overflow** | FilterBar misst Buttons → visibleCount → "Weitere"-Menü |
| **Scroll-Overlay** | OverlayScrollbars mit maskImage (nur über Pillen-Navigation sichtbar bei Katalog/Dashboard) |
| **Tab-Animation** | Two-speed State: `currentTab` (UI), `mountedTab` (Content) → spring-delay auf Heavy-Mount |
| **Auth** | LocalAuth (Proxy) vs. Prod-Auth (JSON); Fallback bei Fehler (`?perfMode=1` auch ohne Auth) |