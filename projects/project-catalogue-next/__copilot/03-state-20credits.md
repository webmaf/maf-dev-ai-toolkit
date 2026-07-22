Ich analysiere die globale State-Management-Architektur. Dafür lese ich die Context-Provider und ihren Zusammenhang.  

Searched for text `createContext|Context.Provider|useContext` (`**/src/contexts/**`), 9 results

Searched for files matching `**/src/contexts/*.tsx`, 2 matches

Searched for regex `useAudience|useExportData|useRefresh|userStateService` (`**/src/**`), 87 results

Read [](file:///Users/matthias/projects/catalogue-next/src/services/userStateService.ts#1-1), lines 1 to 60

Read [](file:///Users/matthias/projects/catalogue-next/src/services/userStateService.ts#50-50), lines 50 to 150

Read [](file:///Users/matthias/projects/catalogue-next/src/hooks/useExportRefresh.ts#1-1), lines 1 to 100

---

## Analyse: Global State Management Strategie

Die Anwendung nutzt ein **hybrids State-Management-Modell** aus **React Context + Service-Layer + URL-basiertem State** statt eines zentralisierten Redux/Zustand-Stores.

---

### **Überblick: State Management Architektur**

```
┌─────────────────────────────────────────────────────────────┐
│                    Persistierungs-Layer                      │
│  (SCORM API ← → localStorage Fallback via userStateService) │
└─────────────────────┬───────────────────────────────────────┘
                      ↑
        ┌─────────────┼─────────────┐
        │             │             │
   [AudienceContext]  │   [ExportDataContext]
   • currentAudience  │   • metadata
   • setAudience()    │   • taxonomy
        │             │   • jobFamilyConfigs
        └─────────────┼─────────────┘
                      ↓
        ┌─────────────────────────────┐
        │  AppContent (Orchestrator)  │
        │  • currentView              │
        │  • selectedTags             │
        │  • searchQuery              │
        │  • filters                  │
        └─────────────────────────────┘
                      ↓
        ┌─────────────────────────────┐
        │  URL Query Parameters       │
        │  ?audience=, ?view=,        │
        │  ?tags=, ?search=, etc.     │
        └─────────────────────────────┘
```

---

### **Provider 1: AudienceContext**

**Datei**: AudienceContext.tsx

#### **Verantwortung**
Verwaltet die **aktuell gewählte Jobfamily** (Zielgruppe) und deren zugehörige Konfiguration.

#### **Globale State-Variablen**
| Variable | Typ | Beschreibung |
|----------|-----|-------------|
| `currentAudienceId` | `string` | ID der aktiven Zielgruppe (z. B. `'fuehrungskraefte'`, `'it-experten'`) |
| `currentAudience` | `Audience` | Komplette Audienc-Konfiguration (aus `config/audiences.json`) |
| `availableAudiences` | `Audience[]` | Liste aller aktivierten Zielgruppen |
| `audienceTags` | `string[]` | Gefilterte Tags für aktuelle Audience |
| `audienceTagCategories` | `Record<string, any>` | Tag-Kategorien der Audience |

#### **Public API (Hooks & Functions)**
```typescript
useAudience() → {
    currentAudience,      // aktuelle Audience-Config
    currentAudienceId,    // Audience-ID
    availableAudiences,   // alle Audiences
    setAudience(id, resetFilters?) → void,  // Audience wechseln
    audienceTags,
    audienceTagCategories
}
```

#### **Initialisierungslogik**
1. URL-Parameter prüfen (`?audience=...`)
2. Sonst localStorage (falls `persistAudienceSelection: true`)
3. Sonst Default aus `config/audiences.json`

#### **Interaktion mit anderen Systemen**
- **→ userStateService**: Persistiert `audienceId` bei jedem `setAudience()` Call
- **→ URL**: Updated Browser-History beim Audience-Wechsel
- **← AppContent**: AppContent ruft `setAudience()` auf bei Audience-Auswahl
- **→ ExportDataContext**: Audience-ID wird zur Filterung von `jobFamilyConfigs` verwendet

---

### **Provider 2: ExportDataContext**

**Datei**: ExportDataContext.tsx

#### **Verantwortung**
Lädt und verteilt **katalog-metadaten aus dem Catalogue Manager Export** (zentrale Datenquelle für Kategorien, Kompetenzen, Training-Zuordnungen, Job-Family-Configs, Bilder).

#### **Globale State-Variablen**
| Variable | Typ | Quelle |
|----------|-----|--------|
| `exportData` | `ExportData \| null` | Remote: ILIAS-Datensammlung (HTML-Parse) oder Lokal: catalogue-export.json |
| `isLoading` | `boolean` | Loading-Zustand des Remote-Fetch |
| `error` | `string \| null` | Fehler beim Laden |
| `metadata` | `TrainingMetadata[]` | Training → Kategorien/Kompetenzen/Zielgruppen |
| `taxonomy` | `Category[]` | Hierarchische Kategorien (Parent-Child) |
| `competencies` | `Competency[]` | Kompetenz-Definitionen |
| `jobFamilyConfigs` | `JobFamilyConfig[]` | Dashboard-Config, Carousel, Buttons pro Zielgruppe |
| `trainingImages` | `TrainingImage[]` | Base64-kodierte Training-Bilder |
| `jobFamilies` | `JobFamily[]` | Zielgruppen-Definitionen |
| `compassContent` | `CompassContent` | FührungsKompass-Artikel und Struktur |

#### **Public API (Hooks & Accessor Functions)**
```typescript
useExportData() → {
    exportData, isLoading, error, isAvailable,
    metadata, taxonomy, competencies, jobFamilyConfigs, compassContent,
    // O(1)-Lookups via Maps
    getMetadataForTraining(trainingTypeNumber),
    getImageForTraining(trainingTypeNumber),
    getJobFamilyConfig(jobFamilyId),
    getCategory(categoryId),
    getMainCategories(),
    getSubcategories(parentId),
    getCompetency(competencyId),
    getCategoryColor(categoryId),
    // Actions
    refresh(),     // Manueller Refresh (Ctrl+Shift+E)
    clearCache()   // Cache löschen
}

// Convenience Hooks
useTrainingMetadata(trainingTypeNumber)
useTrainingImage(trainingTypeNumber)
useCurrentJobFamilyConfig(jobFamilyId)
useMainCategories()
useCompassContent()
```

#### **Lade-Strategie**
1. **Remote**: `loadExportData()` versucht URL aus dataSourceConfig.ts (Catalogue Manager Export JSON)
2. **Fallback (DEV)**: catalogue-export.json
3. **Fallback (PROD)**: Nur lokal wenn Remote fehlschlägt
4. **Caching**: localStorage mit 1h Expiration (optional via `CACHE_REMOTE_DATA`)

#### **Performance-Optimierungen**
- Pre-computed Maps für O(1)-Lookups statt O(n)-Suchen
- `useMemo()` für alle Collections
- `useCallback()` für Accessor-Functions (stabile Referenzen)
- Cancellation-Support in `useEffect` (StrictMode-safe)

#### **Interaktion mit anderen Systemen**
- **← App**: Initial Load durch `ExportDataProvider` Mount
- **→ useExportRefresh**: `refresh()` wird via `Ctrl+Shift+E` Keyboard-Shortcut getriggert
- **→ TrainingLoaders**: `metadata` wird zur Training-Enrichment (`enrichTrainingsWithMetadata()`) verwendet
- **→ Komponenten**: Direct Hook-Zugriff für Training-Details, Kategorien, Kompetenzen

---

### **Service Layer: userStateService**

**Datei**: userStateService.ts

#### **Verantwortung**
Persistiert & restauriert **persistierungsfähigen Anwendungs-State** via **SCORM API** (mit localStorage-Fallback).

#### **Verwalteter globaler State**
```typescript
UserState {
    // Navigation
    currentView: 'dashboard' | 'catalogue'
    currentTab: 'catalogue' | 'kompass' | 'workshops' | 'coaching' | 'community'
    listingView?: 'calendar' | 'catalogue'
    catalogueLayout?: '1col' | '2col'
    
    // Zielgruppe & Filter
    audienceId: string
    selectedTags: string[]
    searchQuery: string
    trainingType: 'all' | 'scheduled' | 'selfpaced'
    
    // Onboarding
    onboardingCompleted: boolean
    catalogueTourCompleted: boolean
    calendarTourCompleted: boolean
    
    // Job Family
    jobFamilyId?: string
    isLeader?: boolean
    
    // Meta
    favorites: string[]          // (vorbereitet für Zukunft)
    lastVisit: ISO8601-timestamp
    version: string             // für Schema-Migration
}
```

#### **Public API**
```typescript
// Singleton
class UserStateService {
    initialize(): void         // SCORM init + load
    terminate(): void          // SCORM cleanup + final save
    loadState(): UserState     // Restore von SCORM/localStorage
    saveState(partial): void   // Merge & persist
    getState(): UserState      // Current in-memory state
    updateState(partial): void // Merge + save
    migrateState(state): UserState // Version-Migration
}

// Singleton export
const userStateService = new UserStateService()
```

#### **Persistierungs-Mechanismus**
1. **SCORM 2004**: `cmi.suspend_data` als JSON-String
2. **Fallback**: localStorage mit Key `scorm_cmi.suspend_data` (wenn SCORM nicht verfügbar)
3. **Timing**: 
   - Load: Bei App.tsx Mount (sync)
   - Save: Bei jedem `updateState()` Call (sync) + `window.beforeunload` Event

#### **Interaktion mit anderen Systemen**
- **← AudienceContext**: `setAudience()` triggert `userStateService.updateState({ audienceId })`
- **← AppContent**: Manuelle Calls zu `updateState()` für Navigation, Filter, Tour-Status
- **→ App.tsx**: Initialisierung + Restore beim Start
- **→ SCORM API**: `scormService.setSuspendData()` für Persistierung

---

### **Helper: useExportRefresh Hook**

**Datei**: useExportRefresh.ts

#### **Verantwortung**
Trigger für **Keyboard-Shortcut** (`Ctrl+Shift+E`) zur manuellen Aktualisierung der Export-Daten.

#### **State-Management** (lokal, aber wichtig für globales System)
| Variable | Typ | Bedeutung |
|----------|-----|----------|
| `status` | `'idle' \| 'loading' \| 'success' \| 'error'` | Refresh-Status |
| `message` | `string` | Toast-Meldung |
| `isToastVisible` | `boolean` | Toast-Sichtbarkeit |

#### **API**
```typescript
useExportRefresh() → {
    status,
    message,
    isToastVisible,
    triggerRefresh()  // ruft exportDataContext.refresh()
}
```

#### **Interaktion**
- **→ ExportDataContext**: Ruft `refresh()` auf → triggert Reload via `refreshTrigger`-Dependency
- **← App.tsx**: Hook-State wird für `RefreshToast`-Komponente verwendet

---

### **URL als State-Quelle**

Die Applikation nutzt **URL-Parameter als primäre Quelle für Filter & Navigation**. Das ist kein klasslicher Context, aber **kritisch für State-Management**:

#### **URL-Parameter** (in `AppContent` geparsed)
```
?audience=fuehrungskraefte          // Audience-ID
&view=catalogue                     // View (dashboard|catalogue|training-detail)
&tab=catalogue                      // Active Tab
&listingView=calendar               // Listing-View (calendar|catalogue)
&categories=...                     // Kategorie-Filter (IDs)
&competencies=...                   // Kompetenzen-Filter
&tags=...                           // Tag-Filter
&search=...                         // Suchtext
&type=scheduled                     // Training-Typ-Filter
&cost=free                          // Kosten-Filter
&training=5010.1020-2025-09-15      // Deep-Link zu Training-Detail (mit Datum)
```

#### **Synchronisations-Muster**
1. **URL → State**: `parseURLParams()` beim Mount + `popstate` Event (Back-Button)
2. **State → URL**: `updateURL()` bei Filteränderungen, Navigation, Modal-Open
3. **URL → localStorage**: Über `userStateService` (nur `audienceId`, `view`, `tab`)

---

### **Interaktionsmuster zwischen Context Providern**

#### **1. Audience-Wechsel Flow**
```
User klickt Audience-Selector
   ↓
SharedHeader.setAudience(newId)
   ↓
AudienceContext.setAudience(newId)
   ├→ localStorage.setItem(AUDIENCE_STORAGE_KEY)
   ├→ userStateService.updateState({ audienceId })
   ├→ URL wird aktualisiert
   └→ Komponenten re-rendern via useAudience()
   ↓
AppContent liest neue Audience
   ├→ loadTrainingsForAudience() wird neu getriggert
   ├→ Filter werden zurückgesetzt
   └→ currentView → 'catalogue' oder Dashboard
```

#### **2. Filter-Änderung Flow**
```
User wählt Tag/Kategorie in FilterBar
   ↓
TrainingCatalogue.setSelectedTags()
   ↓
URL wird aktualisiert via updateURL()
   ├→ Browser-History Push
   └→ URL-Parameter gespiegelt in Local-State
   ↓
useExportData() liefert Metadaten
   └→ Komponenten filtern mit getMetadataForTraining()
   ↓
Gefilterte Trainings werden gerendert
```

#### **3. Persistierungs-Flow**
```
App startet
   ↓
AppContent: userStateService.initialize()
   ├→ SCORM/localStorage initialisieren
   └→ userStateService.loadState() auslesen
   ↓
savedState anwenden
   ├→ AudienceContext: setAudience(savedState.audienceId, false)
   ├→ Filters/Search/View wiederherstellen
   └→ currentTab/catalogueLayout setzen
   ↓
Komponenten nutzen restored State
   ↓
User navigiert/filtert
   ↓
userStateService.updateState() aufrufen
   └→ SCORM cmi.suspend_data wird aktualisiert
```

#### **4. Export-Refresh Flow**
```
User drückt Ctrl+Shift+E
   ↓
useExportRefresh.triggerRefresh()
   ├→ exportDataContext.refresh()
   │  └→ clearExportDataCache()
   │  └→ setRefreshTrigger() → triggert useEffect
   ├→ exportDataContext re-loads Remote-Daten
   └→ State aktualisiert
   ↓
Toast zeigt Status (loading → success/error)
   ↓
Komponenten mit useExportData() re-rendern automatisch
```

---

### **Daten-Flow-Zusammenfassung**

```
┌────────────────────────────────────────────────────────────┐
│                     Remote Sources                         │
│  • ILIAS Catalogue Manager Export (JSON)                   │
│  • nextCampus Excel (via parser)                           │
│  • Education Catalogue JSON (~7MB)                         │
│  • WBT Links Excel                                         │
└────────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────────┐
│                   Data Loaders                             │
│  • exportDataLoader → ExportDataContext                    │
│  • trainingLoader → loadTrainingsForAudience()            │
│  • enrichTrainingsWithMetadata()                           │
└────────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────────┐
│          Global Context State (React Context)              │
│  • AudienceContext: currentAudienceId, currentAudience     │
│  • ExportDataContext: metadata, taxonomy, configs          │
└────────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────────┐
│         AppContent Local State + URL Parameters            │
│  • currentView, currentTab, filters, search                │
│  • Synced zu Browser-URL & userStateService                │
└────────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────────┐
│              Component Level (Local useState)              │
│  • Form inputs, UI interactions, animation states          │
│  • NOT global, not persisted                               │
└────────────────────────────────────────────────────────────┘
```

---

### **State-Management Design Decisions**

| Entscheidung | Begründung |
|---|---|
| **Zwei Contexts statt Redux** | Katalog-Daten ≠ Navigation. Export statisch (Lifecycle), Navigation dynamisch (URL-getrieben). |
| **URL als Quelle** | Deep-linking, Browser-History, Lesezeichen. Kein State-Sync-Problem. |
| **userStateService als Service** | SCORM ist synchron, kein Hook. Service-Pattern passt besser. |
| **O(1)-Maps in ExportData** | Export-Daten können Tausende Einträge haben. Maps statt Array-Suchen. |
| **useDeferredValue in TrainingCatalogue** | Filter-State ist teuer zu berechnen. Defer separiert fast User-Input von langsamen Renders. |
| **Lazy-Load schwerer Contexts/Hooks** | CompassTab, GuidedTour, ChangelogDialog nur bei Bedarf. Reduces initial JS. |
| **Local-Fallback für SCORM** | SCORM nicht immer verfügbar (DEV, lokale Tests). localStorage Fallback essentiell. |

---

### **Kritische State-Quellen-Priorität**

1. **URL-Parameter** (höchste Priorität)
   - Browser-History, Deep-Links
   - `parseURLParams()` → State

2. **SCORM/localStorage** (bei App-Start)
   - Audience, Onboarding-Flag, Filter
   - `userStateService.loadState()` → State

3. **Context Defaults** (Fallback)
   - AudienceContext: `config/audiences.json`
   - ExportDataContext: lokale Fallback-JSON

4. **Component Local State** (UI-only)
   - Form-Inputs, Hover-States
   - Nicht persistiert