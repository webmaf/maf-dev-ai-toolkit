Ich lese die Typdefin­itionen und Domänenmodelle, um die fachliche Kernstruktur zu verstehen.  

Searched for files matching `**/src/types/*.ts`, 9 matches

Read [](file:///Users/matthias/projects/catalogue-next/src/types/index.ts)

Read [](file:///Users/matthias/projects/catalogue-next/src/types/training.ts)

Read [](file:///Users/matthias/projects/catalogue-next/src/types/catalogueExport.ts)

Read [](file:///Users/matthias/projects/catalogue-next/src/types/audience.ts)

Read [](file:///Users/matthias/projects/catalogue-next/src/types/location.ts)

Read [](file:///Users/matthias/projects/catalogue-next/src/types/person.ts)

Read [](file:///Users/matthias/projects/catalogue-next/src/types/trainingProgram.ts#1-1), lines 1 to 150

---

## Analyse: Fachliche Kernobjekte und Domänenmodelle

Die Anwendung organisiert sich um **fünf zentrale Domänenkonzepte**, die hierarchisch und durch Beziehungen miteinander verbunden sind.

---

### **1. Training (Zentral-Objekt)**

**Fachliche Bedeutung**: Ein Training ist das **Kernprodukt** — ein Schulungs-/Fortbildungsangebot mit konkreten Terminen, Ort, Dozent:innen und Buchungsinformationen. Trainings sind **polymorp**: Es gibt drei Spezialisierungen.

#### **Spezialisierungen**

| Typ | Beschreibung | Besonderheiten |
|---|---|---|
| **EducationTraining** | Bildungs-Schulungen (z.B. IT-Fachkurse, Fachtrainings) | Kurscode, Credits, Voraussetzungen, Zertifikat, Registrierungs-Deadline, Meldeschluss, Kapazitätsmodell (min/opt/max/frei/Warteliste) |
| **NextCampusTraining** | Führungskräfte-Trainings am next campus Lernort | Raumbezeichnung, Daily Schedule, Buchungsmethode (buchungstool/mail/reference/direct), Venue-Fotos, Kalkulation |
| **WBTTraining** | Web-Based Trainings (Selbstlernmodule, ∼40 Stück) | Keine Termine, Dauer in Min., Schwierigkeitsstufe, immer verfügbar, Direktzugang |

#### **Gemeinsame Basis-Eigenschaften** (in `BaseTraining`)
- **Identität**: `id`, `title`, `description`
- **Zeitlich**: `startDate`, `endDate` (optional für WBTs)
- **Ort**: `location` (String)
- **Personen**: `instructors[]` (E-Mail), `contacts[]` (Ansprechpartner)
- **Klassifikation**: `tags[]`, `tagsByCategory` (Legacy vs. strukturiert)
- **Zielgruppen**: `audiences[]` (z.B. `['berufsanfaenger', 'it-experten']`)

---

### **2. Category (Taxonomie)**

**Fachliche Bedeutung**: Eine **hierarchische Klassifizierung** für Trainings. Ein Training kann mehreren Kategorien zugeordnet werden, um es über verschiedene Perspektiven filterbar zu machen.

#### **Struktur**
- **Hierarchie**: Parent-Child-Beziehung (`parentId: null` = Hauptkategorie, sonst untergeordnet)
- **Eigenschaften**: `id`, `label`, `description`, `color` (für Hauptkategorien), `order`
- **Beispiele**: 
  - Hauptkategorien: "Leadership", "IT-Sicherheit", "Projektmanagement"
  - Unterkategorien: "Leadership" → "Change Management", "Delegation", "Feedback"

#### **Beziehung zu Training**
Ein Training hat `categories: string[]` (Category-IDs) und optional `primaryCategory: string` (primäre Kategorie, muss in `categories` enthalten sein). Eine Kategorie kennt ihre Trainings nicht (unidirektional).

---

### **3. Competency (Kompetenzen)**

**Fachliche Bedeutung**: Ein **trainiertes Kompetenzfeld** — fachliche oder persönliche Fähigkeit, die durch Trainings entwickelt wird.

#### **Struktur**
- `id`, `label`, `color`, `description`, `characteristics[]`
- **Global definiert**: Im Export als zentrale Liste mit ~10–20 Kompetenzen

#### **Beziehung zu Training**
- Training hat `competencies: string[]` (Competency-IDs)
- Eine Competency wird in mehreren Trainings adressiert
- **Job-Family-spezifische Schwerpunkt-Kompetenzen**: `JobFamilyConfig.additionalCompetencyIds` (2–3 wichtigste Kompetenzen pro Zielgruppe)

---

### **4. JobFamily (Audience / Zielgruppe)**

**Fachliche Bedeutung**: Eine **Nutzergruppe mit eigener Perspektive** auf den Katalog — unterschiedliche Dashboard-Konfiguration, Kategorien, Filter und Angebote.

#### **Struktur** (in catalogueExport.ts)
```
JobFamily {
  id: 'fuehrungskraefte' | 'it-experten' | 'sachbearbeitung'
  label, color, icon (compass|monitor|filetext|graduationcap)
}
```

#### **Dynamische Config** (in `JobFamilyConfig`)
Jede Job-Family hat eine spezifische Konfiguration mit:
- **Carousel**: Highlight-Trainings auf der Startseite
- **Dashboard-Buttons**: Schnellzugriff-Kacheln mit Filtern/Links
- **Counters**: Statistik-Widgets (z.B. "15 Trainings verfügbar")
- **Custom Categories**: Hierarchische Kategorien speziell für diese Zielgruppe
- **Additional Competencies**: Spezifische Schwerpunkt-Kompetenzen
- **Display Mode**: `'all'` (alle Kategorien), `'single'` (eine), `'custom'` (gepflegte Auswahl)

#### **Beziehung zu Training**
- Training hat `jobFamilies: string[]` (JobFamily-IDs)
- Training kann für bestimmte Jobfamilies andere Kategorien haben: `jobFamilyCategories: Record<jobFamilyId, categoryIds[]>`
- Eine JobFamily sieht nur Trainings, die ihr in `jobFamilies[]` oder `audiences[]` zugeordnet sind

---

### **5. ExportData (Zentraler Metadaten-Container)**

**Fachliche Bedeutung**: Ein **Export-Paket aus dem Catalogue Manager** mit allen verwalteten Metadaten, Definitionen und Dashboard-Konfigurationen. Das ist die **Single Source of Truth** für Kategorisierung, Kompetenzen, Job-Family-Configs und Training-Metadaten.

#### **Struktur** (verschachtelt)
```
ExportData {
  version: number
  exportedAt: ISO-Timestamp
  data: {
    // Verwaltete Daten (Training-Zuordnung)
    metadata: TrainingMetadata[]     // Pro TrainingTypeNumber
    trainingImages: TrainingImage[]  // Base64-Bilder
    jobFamilyConfigs: JobFamilyConfig[]  // Pro JobFamily-ID
    globalSettings: GlobalSettings   // Farb-Overrides, Base-Kompetenzen
    
    // Statische Definitionen (Selbstbeschreibung)
    taxonomy: Category[]
    competencies: Competency[]
    jobFamilies: JobFamily[]
    
    // Redaktioneller Content (v11+)
    compassContent?: CompassContent  // FührungsKompass-Artikel
  }
}
```

#### **TrainingMetadata** (pro Training)
Speichert die Zuordnung eines Trainings zu Kategorien, Kompetenzen, Job-Families:

```
TrainingMetadata {
  trainingTypeNumber: string      // z.B. "5010.1002"
  categories: string[]            // Category-IDs
  primaryCategory?: string        // Hauptkategorie
  jobFamilies?: string[]          // Welche Jobfamilies sollen das sehen?
  jobFamilyCategories?: {         // Jobfamily-spezifische Kategorien-Zuordnung
    'fuehrungskraefte': ['cat-1', 'cat-2'],
    'it-experten': ['cat-3']
  }
  competencies?: string[]         // Competency-IDs
  relatedItems?: RelatedItem[]    // Verknüpfung zu anderen Trainings / CoPs
  bookingHighlights?: [...]       // Pflichttext-Felder vor Buchung
}
```

---

### **6. Audience (Konfiguration für Nutzer)**

**Fachliche Bedeutung**: Eine **Nutzer-Segment-Definition** mit Basiskonfiguration aus `config/audiences.json`. Wird bei Runtime durch `JobFamilyConfig` aus dem Export ergänzt.

#### **Struktur** (in audience.ts)
```
Audience {
  id: string
  label, description, icon
  isDefault: boolean
  enabled: boolean
  dataSources: string[]           // Welche Datenquellen nutzen?
  tagCategories: Record<string, AudienceTagCategory>  // Filter-Struktur
  // Legacy Dashboard-Config (wird durch JobFamilyConfig ersetzt)
  dashboardButtons, carousel, counters
}
```

#### **Beziehung zu JobFamily**
- `Audience.id` ≈ `JobFamily.id` (1:1 Mapping)
- `Audience` ist die **Laufzeit-Instanz** mit User-Prefs (localStorage)
- `JobFamily` ist die **exportierte Definition** mit zentraler Konfiguration

---

### **7. Person (Dozent:innen & Ansprechpartner)**

**Fachliche Bedeutung**: Ein **Lehrperson** oder **Kontaktperson** für ein Training.

#### **Struktur**
```
Person {
  email: string              // Eindeutiger Identifier
  firstName, lastName
  title?, phone?, department?, bio?
  active: boolean
}

ContactPerson (Training-Kontext) {
  fullName, firstName?, lastName?
  email?, phoneNumber?
  image? (Avatar)
  iliasContactReference: string  // ILIAS-ID
}
```

#### **Beziehung zu Training**
- Training hat `instructors: string[]` (Personen-E-Mails)
- Training hat `contacts: Record<'content'|'event'|'speaker', ContactPerson[]>` (verschiedene Kontakttypen)
- **NextCampusTraining** zusätzlich: `instructorBios: Record<name, biography>` für Sprecher-Beschreibung

---

### **8. Location (Veranstaltungsort & Raum)**

**Fachliche Bedeutung**: Ein **physischer Ort** mit Raumdefinitionen und zeitlichem Stundenplan.

#### **Struktur**
```
Location {
  id, title
  streetLine1, streetNumber, zipCode, city
}

Room {
  id, type, short, label
  locationId  // Zuordnung zu Location
  link?       // z.B. 360°-Rundgang
}

ScheduleEntry {
  date: string              // Konkretes Datum
  begin, end: string        // Uhrzeit (HHmmss)
  rooms: Room[]             // Welche Räume?
}
```

#### **Beziehung zu Training**
- EducationTraining kann `fullSchedule: ScheduleEntry[]` haben (Termine + Räume)
- NextCampusTraining hat `location`, `address`, `room` als String-Eigenschaften
- WBT hat keine Location

---

### **9. TrainingProgram & ProgramCohort (Qualifizierungsprogramme)**

**Fachliche Bedeutung**: Ein **Curriculum aus mehreren zusammenhängenden Modulen** mit fester Reihenfolge. Ein `TrainingProgram` ist eine **Struktur**, ein `ProgramCohort` eine **konkrete Durchführung** mit Termine.

#### **Struktur**
```
TrainingProgram {
  id, title, description, shortDescription
  targetAudience?, prerequisites?
  phases: ProgramPhase[]  // Grundlagen, Vertiefung, Abschluss
}

ProgramPhase {
  id, name, description, order, color
  modules: ProgramModule[]
}

ProgramModule {
  id, isOptional, order
  trainingTypeNumber?: string    // Link zu Training-Katalog
  trainingTypeId?: string        // Inline-Definition
  resolvedTraining?: Training    // Zur Runtime aufgelöst
}

ProgramCohort {
  id, name, startDate, endDate
  enrolledCount, maxCapacity
  registrationDeadline, status
  moduleTrainings: Record<moduleId, trainingId>  // Modul → konkretes Training-Termin
}
```

#### **Beziehung zu Training**
- Ein Modul referenziiert entweder einen TrainingTypeNumber (aus Katalog) oder eine Inline-Definition
- Im Cohort wird jedes Modul auf ein konkretes Training mit Datum + Ort gemappt
- **Auflösungs-Pattern**: programModuleResolver.ts kombiniert Program + Trainings zur Runtime

---

### **10. Tag & Category vs. JobFamilyCategories (Klassifikations-Patterns)**

**Fachliche Besonderheit**: Die Anwendung hat **zwei orthogonale Klassifikationssysteme**:

#### **Tag-basiert** (Legacy, aus `config/tags.ts`)
- Flache Liste: `praesenz`, `online`, `hybrid`, `grundlagen`, `recht`, `fachkompetenz`, etc.
- **Probleme**: Keine Hierarchie, keine Neustrukturierbarkeit pro Jobfamily

#### **Kategorie-basiert** (Modern, aus Export)
- Hierarchisch: Main-Category → Sub-Categories
- **Job-Family-spezifisch**: Verschiedene Jobfamilies sehen andere Kategorien-Hierarchien
- **Erweiterbar**: Im Export definierbar, ohne Code zu ändern

#### **Hybrid-Ansatz im Code**
- Trainings haben weiterhin `tags[]` für Kompatibilität
- Neue Filter nutzen `categories[]` + `jobFamilyCategories`
- Filter-Matching prüft beide (mit "smart matching" für `online` ↔ `hybrid`)

---

### **11. CompassContent (Führungs-Kompass Inhalte)**

**Fachliche Bedeutung**: **Redaktionelle Knowledge Base** für Führungskräfte — Artikel organisiert in Themenfeldern mit Stichwort-Index.

#### **Struktur**
```
CompassContent {
  subthemes: Subtheme[]     // Themenfelder (z.B. "Organisationsentwicklung")
  articles: Article[]       // Einzelne Artikel
  frequentlySearchedChips: string[]  // Schnellzugriff-Stichwörter
}

Subtheme {
  id, name, description
}

Article {
  id, subthemeId, title, content (HTML)
}
```

#### **Beziehung zu anderen Modellen**
- Unabhängig von Trainings (separate Wissensbasis)
- Optional im Export enthalten (`compassContent?`)
- Wird von `CompassTab` visualisiert

---

### **12. RelatedItem (v9+: Verknüpfungen)**

**Fachliche Bedeutung**: Eine **unidirektionale Verknüpfung** zwischen Trainings oder zwischen Training und Community of Practice.

#### **Struktur**
```
RelatedItem {
  type: 'training' | 'cop'
  id: string              // TrainingTypeNumber oder CoP-Slug
  relationKind?: string   // (reserviert: 'prerequisite', 'deepening')
}
```

#### **Beziehung**
- Gespeichert in `TrainingMetadata.relatedItems`
- Im Frontend aufgelöst: Suche nach Training mit gleicher `trainingTypeNumber` oder CoP mit gleicher `id`
- **Defensives Lesen**: Verwaiste Referenzen ignorieren (Ziel wurde gelöscht)

---

### **13. CommunityOfPractice (v7+: Communities)**

**Fachliche Bedeutung**: Ein **Community-Netzwerk** für Wissensaustausch — ähnlich wie ein Trainings-Forum, aber global gepflegt.

#### **Struktur**
```
CommunityOfPractice {
  id: string              // Stabiler Slug
  name, description
  responsibleEmails: string[]
  link: string
  targetGroup?, sourceKey?
  inactive?: boolean
}
```

#### **Beziehung zu Training**
- Training kann in `relatedItems` auf eine CoP verweisen
- CoP kennt ihre Trainings nicht
- CoP wird im `globalSettings.communitiesOfPractice[]` central verwaltet

---

### **Beziehungs-Übersicht (Entity Relationship)**

```
┌─────────────┐
│  JobFamily  │
│ ('Führungs-│
│  kräfte')   │
└──────┬──────┘
       │ (1:1 mapped)
       ↓
┌─────────────────────────────────┐
│   JobFamilyConfig               │
│   - carousel (items)            │
│   - dashboardButtons            │
│   - customCategories            │
│   - additionalCompetencyIds     │
└──────────────────────────────────┘

┌──────────┐                ┌───────────────┐
│ Category │◄───(parent)───┤ Category      │
│ (Main)   │                │ (Sub)         │
└──────────┘                └───────────────┘
       ▲                            ▲
       │ (has)                      │
       │                            │
┌──────────────────────────────────┐
│       Training                   │
│  - id, title, description        │
│  - categories[]                  │
│  - primaryCategory               │
│  - jobFamilies[]                 │
│  - jobFamilyCategories{}         │
│  - competencies[]                │
│  - instructors[], contacts[]     │
│  - location                      │
└──────────────────────────────────┘
       │ (has)
       ├─→ Competency (skills gained)
       ├─→ Person (instructors/contacts)
       ├─→ Location (venue)
       └─→ RelatedItem (training/cop)
              │
              ├─→ Training (other course)
              └─→ CommunityOfPractice

┌────────────────────────┐
│   TrainingProgram      │
│  - phases[]            │
│    - modules[]         │
│      - trainingTypeNo  │─→ Training (resolved)
└────────────────────────┘
       │
       └─→ ProgramCohort (Durchführung)
           - moduleTrainings{} (modul → training)

┌──────────────────┐
│   ExportData     │
│  - metadata[]    │──→ Training-Zuordnungen
│  - taxonomy[]    │──→ Category-Definitionen
│  - competencies[]│──→ Kompetenz-Definitionen
│  - jobFamilyConfigs[]
│  - trainingImages[]
│  - globalSettings
└──────────────────┘
```

---

### **Daten-Kohäsion & Verantwortlichkeiten**

| Objekt | Verwaltet | Referenziert | Besitzt |
|--------|-----------|--------------|---------|
| **Training** | Trainings-Inhalt, Termine, Dozent:innen | Category-IDs, JobFamily-IDs, Competency-IDs | Konkrete Daten-Instanz |
| **Category** | Hierarchie (parent), Label, Farbe | Trainings kennen sie nicht | Taxonomie-Definition |
| **Competency** | Label, Farbe, Beschreibung | Trainings referenzieren | Skill-Definition |
| **JobFamily** | Grund-Metadaten (id, label, icon) | JobFamilyConfig kennt Details | Segment-Definition |
| **JobFamilyConfig** | Dashboard-Layout, Carousel, Buttons, Custom-Categories | TrainingMetadata prüft | Job-Family-spezifische Konfiguration |
| **ExportData** | Alle Metadaten + Definitionen | Trainings werden davon angereichert | Zentrale Wahrheitsquelle |
| **TrainingMetadata** | Training-Zuordnung zu Kategorien/Kompetenzen/JobFamilies | Training muss existieren | Verwaltete Zuordnung |
| **CompassContent** | Artikel, Themen, Stichwörter | Unabhängig von Trainings | Redaktionelle Basis |

---

### **Konsistenzbedingungen & Invarianten**

1. **Jedes Training hat genau eine Job-Family-Zuordnung** (`jobFamilies[]` bei Mindestwertiger Erfassung)  
2. **Kategorien sind hierarchisch oder Flach** (zyklische Dependencies ausgeschlossen durch Parent-Validierung)  
3. **primaryCategory muss in categories[] enthalten sein** (Validierung beim Export)  
4. **TrainingMetadata.trainingTypeNumber muss auf ein Training im Katalog zeigen** (defensiv: wenn nicht, wird ignoriert)  
5. **JobFamilyCategories-Einträge müssen Category-IDs sein** (Validierung bei Export)  
6. **Competencies sind global eindeutig** (eine ID = eine Competency Definition)  
7. **RelatedItems sind defensiv lesbar** (nicht gefundenes Ziel → überspringen)

---

### **Design-Pattern: Anreicherung (Enrichment)**

Trainings werden beim Laden mit Export-Daten **angereichert**:

```
Raw Training (von ILIAS/Excel)
       ↓
TrainingMetadata (aus Export)
       ↓
enrichTrainingsWithMetadata()
       ↓
Training + categories[] + competencies[] + jobFamilies[]
       ↓
Filter, Suche, Anzeige
```

Das ermöglicht, dass Trainings **unabhängig gepflegt** werden (in ILIAS), während **Kategorisierung zentral** im Export erfolgt.