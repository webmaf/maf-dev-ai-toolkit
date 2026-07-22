# Domänenmodell

## Ziel

Die Anwendung verwaltet einen Trainingskatalog mit zielgruppenspezifischer Darstellung, zentral gepflegten Metadaten und verschiedenen Trainingsarten.

---

# Kernobjekte

## Training

Zentrales Objekt der Anwendung.

Es existieren drei Trainingsarten:

- Education
- Next Campus
- WBT

Ein Training besitzt u.a.

- Kategorien
- Kompetenzen
- Zielgruppen
- Termine
- Dozenten
- Buchungsinformationen

---

## Job Family

Beschreibt eine Zielgruppe.

Beispiele:

- Führungskräfte
- IT
- Sachbearbeitung

Eine Job Family bestimmt

- Dashboard
- Kategorien
- Filter
- Carousel
- Kompetenzen

---

## Category

Hierarchische Klassifizierung eines Trainings.

Training
↓

Category

---

## Competency

Beschreibt die Kompetenzen, die durch Trainings vermittelt werden.

---

## ExportData

Single Source of Truth.

Enthält

- Kategorien
- Kompetenzen
- JobFamilyConfigs
- TrainingMetadata
- Bilder

Trainings werden beim Laden mit diesen Daten angereichert.

---

## TrainingProgram

Beschreibt mehrstufige Qualifizierungsprogramme.

Programme bestehen aus

Phasen

↓

Modulen

↓

Trainings

---

## Compass

Eigenständige Wissensplattform.

Nicht Teil des Trainingskatalogs.

---

## Community of Practice

Community-Plattform.

Kann mit Trainings verknüpft werden.

---

# Domänenbeziehungen

ExportData

↓

TrainingMetadata

↓

Training

↓

Category

↓

Competency

↓

JobFamily

---

# Architekturprinzip

Trainings enthalten nur Basisinformationen.

Metadaten werden zentral gepflegt und beim Laden angereichert.

Dadurch bleiben Trainings und Katalogstruktur voneinander getrennt.