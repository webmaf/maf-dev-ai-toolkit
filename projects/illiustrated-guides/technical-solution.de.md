# Manuskript: Technische Lösung der „Bebilderten Anleitungen“

**Vorgesehene Dauer:** etwa 8 bis 10 Minuten  
**Zielgruppe:** fachlich interessierte Personen ohne tiefe Programmierkenntnisse

## 1. Einstieg

[Folie: Titel und Ziel des Projekts]

Ich stelle heute die technische Lösung für das Projekt „Bebilderte Anleitungen“ vor. Das Ziel der Anwendung ist, technische oder fachliche Anleitungen mit Texten und Screenshots einfach und in einem einheitlichen Format zu erstellen.

Bei der technischen Konzeption waren drei Anforderungen besonders wichtig: Erstens sollen auch sensible Screenshots verarbeitet werden können. Zweitens soll die Anwendung ohne aufwendige Server-Infrastruktur funktionieren. Drittens soll sie später um Bildmarkierungen, den Austausch von Projektdateien und einen kontrollierten PDF-Export erweitert werden können.

Ich unterscheide im Folgenden bewusst zwischen dem heute funktionsfähigen Stand und der geplanten Ausbaustufe.

## 2. Grundentscheidung: eine lokale Browser-Anwendung

[Folie: Browser → lokale Verarbeitung; keine Serverübertragung]

Die Anwendung ist als Single-Page-Anwendung aufgebaut. Das bedeutet: Nach dem Aufruf wird die Benutzeroberfläche im Browser ausgeführt. Für die eigentliche Bearbeitung ist keine Server-API nötig. Texte und Screenshots werden derzeit ausschließlich lokal im Browser verarbeitet und nicht an einen Anwendungsserver übertragen.

Diese Entscheidung wurde vor allem aus Datenschutz- und Betriebsgründen getroffen. Screenshots können personenbezogene oder interne Informationen enthalten. Wenn sie den Browser nicht verlassen, wird das Risiko einer unbeabsichtigten Übertragung deutlich reduziert. Gleichzeitig ist die Bereitstellung einfach: Die Anwendung besteht aus statischen Dateien und kann deshalb über eine vorhandene Webumgebung ausgeliefert werden.

Die Kehrseite dieser Entscheidung ist, dass Funktionen wie zentrale Zusammenarbeit, geräteübergreifende Synchronisation oder serverseitige Backups nicht automatisch vorhanden sind. Für den vorgesehenen lokalen Einzelarbeitsplatz ist dieser Kompromiss im ersten Schritt jedoch passend.

## 3. Eingesetzte technische Basis

[Folie: React, TypeScript und Vite]

Die Oberfläche wurde mit React entwickelt. React eignet sich für diese Anwendung, weil sich der Editor in klar getrennte, wiederverwendbare Bereiche zerlegen lässt. Dazu gehören die Startseite, die Schrittübersicht, der Deckblatt-Editor, der Editor eines einzelnen Arbeitsschritts und der Bereich für Screenshots.

TypeScript ergänzt JavaScript um feste Typen. Das Projektmodell legt zum Beispiel eindeutig fest, welche Daten ein Projekt, ein Schritt und ein Screenshot enthalten. Dadurch werden Fehler beim Weiterentwickeln früher sichtbar und neue Funktionen können kontrollierter ergänzt werden.

Vite übernimmt die Entwicklungs- und Build-Umgebung. Es sorgt für kurze Ladezeiten während der Entwicklung und erzeugt am Ende die statischen Dateien für die Bereitstellung. Eine umfangreichere Lösung wie Next.js wäre hier derzeit unnötig, weil keine serverseitig gerenderten Seiten und keine eigene Serverlogik gebraucht werden.

## 4. Aufbau der Anwendung

[Folie: Komponentenübersicht]

Die Anwendung ist in mehrere fachlich abgegrenzte React-Komponenten aufgeteilt. Die zentrale App-Komponente koordiniert den aktuellen Zustand und die Benutzeraktionen. Der Project Workspace bildet den sichtbaren Arbeitsbereich. Darin verwaltet der Step Navigator die Reihenfolge und Auswahl der Schritte. Der Cover Editor bearbeitet die Projektdaten des Deckblatts. Der Step Editor bearbeitet Titel, Erklärung und Screenshot eines Schritts. Der Screenshot Canvas stellt das Bild und die Zoomsteuerung dar.

Diese Trennung wurde gewählt, damit nicht die gesamte Anwendung in einer einzigen großen Datei steckt. Jede Komponente hat eine überschaubare Verantwortung. Das verbessert Lesbarkeit, Wartbarkeit und die spätere Erweiterbarkeit. Wenn beispielsweise die Bildannotation ergänzt wird, kann sie weitgehend im Screenshot-Bereich umgesetzt werden, ohne die Schritt-Navigation neu bauen zu müssen.

Wichtig ist dabei: Der sogenannte Screenshot Canvas ist im aktuellen Stand noch kein vollwertiger Zeichen-Canvas. Er zeigt das importierte Bild, unterstützt Zoomen, Ersetzen und Entfernen und enthält Platzhalter für spätere Werkzeuge. Rahmen, Pfeile, Texte, Nummerierungen, Schwärzungen und Bildzuschnitt sind noch nicht funktionsfähig.

## 5. Datenmodell und Zustandsverwaltung

[Folie: Projekt → Metadaten → geordnete Schritte → optionaler Screenshot]

Das Datenmodell ist bewusst einfach gehalten. Ein Projekt enthält eine ID, Titel, Datum, Organisationseinheit, Autor und eine geordnete Liste von Schritten. Jeder Schritt besitzt ebenfalls eine eindeutige ID sowie Titel, Erklärung und optional einen Screenshot. Zum Screenshot werden Dateiname, Dateityp, Bilddaten und Originalabmessungen gespeichert.

Die zentrale Zustandsverwaltung verwendet den in React eingebauten Reducer-Ansatz. Benutzeraktionen wie „Schritt hinzufügen“, „Schritt bearbeiten“, „löschen“ oder „verschieben“ werden als klar benannte Aktionen beschrieben. Eine zentrale Reducer-Funktion erzeugt daraus jeweils den neuen Projektzustand.

Der Vorteil ist, dass Änderungen nachvollziehbar und vorhersehbar bleiben. Die Komponenten müssen nicht selbst wissen, wie eine Schrittliste intern verändert wird. Außerdem arbeitet der Reducer mit unveränderlichen Datenstrukturen: Statt vorhandene Daten unkontrolliert zu verändern, wird ein neuer Zustand erzeugt. Das passt zu React und reduziert schwer nachvollziehbare Seiteneffekte.

Eine fachliche Schutzregel verhindert außerdem vollständig leere Schritte: Mindestens Titel, Erklärung oder Screenshot müssen erhalten bleiben. Beim Löschen eines ganzen Schritts gibt es zusätzlich eine Bestätigung.

## 6. Verarbeitung und Speicherung der Screenshots

[Folie: Bilddatei → FileReader → Data URL → Projektzustand]

Ein Screenshot kann über eine Dateiauswahl oder per Drag-and-drop eingefügt werden. Der Browser liest die Bilddatei mit seiner eingebauten FileReader-Schnittstelle ein. Das Bild wird als Data URL gespeichert, also als Textdarstellung der Bilddaten. Zusätzlich werden die natürlichen Abmessungen des Bildes ermittelt.

Diese Lösung ist für den MVP praktisch, weil Textdaten und Bild in einem gemeinsamen Projektobjekt gespeichert werden können. Es entstehen keine separaten lokalen Dateiverweise, die nach einem Neustart ungültig sein könnten.

Der Nachteil ist ein höherer Speicherbedarf: Eine Base64-Darstellung ist größer als die ursprüngliche Binärdatei. Bei vielen sehr großen oder hochauflösenden Screenshots müssen deshalb noch Belastungstests durchgeführt und gegebenenfalls Grenzen oder eine effizientere Blob-Speicherung ergänzt werden.

## 7. Lokale Persistenz mit IndexedDB

[Folie: Projektzustand → automatische Speicherung → IndexedDB]

Damit die Arbeit nach einem Schließen oder Neuladen des Browsers nicht verloren geht, wird der jeweils letzte Projektstand automatisch in IndexedDB gespeichert. IndexedDB ist eine im Browser integrierte lokale Datenbank. Beim Start des Editors wird geprüft, ob ein gespeichertes Projekt vorhanden ist, und dieses wird wiederhergestellt.

Die gespeicherten Daten besitzen eine Versionsnummer und werden beim Laden strukturell geprüft. Dadurch wird nicht ungeprüft irgendein Inhalt als gültiges Projekt übernommen. Fehler der lokalen Speicherung unterbrechen außerdem nicht die aktuelle Bearbeitung im Arbeitsspeicher.

Im derzeitigen Code wird IndexedDB direkt über die Browser-Schnittstelle angesprochen. Die ursprünglich erwogene Bibliothek Dexie ist zwar als Abhängigkeit vorgesehen, wird aktuell aber nicht eingesetzt. Für das kleine Schema vermeidet die direkte Lösung eine zusätzliche Abstraktionsschicht. Wenn später mehrere Projekte, Migrationen oder komplexere Abfragen hinzukommen, kann Dexie wieder sinnvoll werden.

Die lokale Speicherung ist allerdings kein echtes Backup. Sie gehört zum jeweiligen Browserprofil und kann durch das Löschen von Browserdaten verloren gehen. Deshalb ist der portable Import und Export einer Projektdatei der empfohlene nächste Entwicklungsschritt.

## 8. Bedienung und Barrierearmut

[Folie: Navigation, Tastatur und Statusmeldungen]

Der aktuelle MVP unterstützt das Anlegen, Auswählen, Bearbeiten, Löschen und Umsortieren von Schritten. Die Reihenfolge kann per Drag-and-drop geändert werden. Zusätzlich können ausgewählte Schritte mit den Pfeiltasten verschoben werden. Eine für Screenreader vorgesehene Statusmeldung bestätigt die neue Position.

Diese zweite Bedienmöglichkeit ist wichtig, weil Drag-and-drop allein nicht für alle Nutzerinnen und Nutzer zugänglich ist. Auch semantische HTML-Elemente, Beschriftungen und Bestätigungsdialoge wurden eingesetzt. Eine vollständige Barrierefreiheitsprüfung steht allerdings noch aus.

## 9. Geplante Ausbaustufe

[Folie: Heute / Als Nächstes / Danach]

Der aktuelle Stand deckt den grundlegenden Bearbeitungsweg ab: Projektdaten pflegen, Schritte anlegen und sortieren, Texte schreiben, Screenshots einfügen und den letzten Stand lokal wiederherstellen.

Als Nächstes soll der Import und Export portabler Projektdateien umgesetzt werden. Damit können Projekte gesichert und auf einen anderen Arbeitsplatz übertragen werden, ohne dass dafür ein zentraler Server nötig ist.

Danach sind interaktive Bildwerkzeuge vorgesehen. Für Rahmen, Pfeile, Nummern, Text und Schwärzungen ist Konva beziehungsweise react-konva eingeplant. Diese Bibliothek baut auf dem HTML-Canvas auf und erleichtert das Auswählen, Verschieben und Skalieren grafischer Objekte. Die Positionen sollen relativ zur nativen Bildgröße gespeichert werden, damit Markierungen unabhängig von Bildschirmgröße und Auflösung an der richtigen Stelle bleiben.

Für den PDF-Export ist jsPDF vorgesehen. Es ermöglicht eine vollständig clientseitige Ausgabe mit festem A4-Layout, Deckblatt sowie Kopf- und Fußzeilen. Vor allem Schwärzungen sollen vor dem Einbetten fest in eine neue Bilddarstellung eingerechnet werden. Damit darf das ungeschwärzte Original nicht versehentlich aus dem PDF extrahierbar sein. Diese Sicherheitsanforderung muss vor einer Freigabe gezielt getestet werden.

Konva, react-konva und jsPDF sind bereits als Projektabhängigkeiten eingetragen, werden im aktuellen Funktionsstand aber noch nicht produktiv verwendet.

## 10. Fazit

[Folie: Datenschutzfreundliche und erweiterbare Grundlage]

Zusammengefasst wurde eine bewusst schlanke, lokale und modular aufgebaute Browser-Anwendung gewählt. React strukturiert die Oberfläche, TypeScript sichert das Datenmodell ab, Vite erzeugt eine einfach auslieferbare Anwendung und IndexedDB erhält den letzten Arbeitsstand lokal.

Die wichtigste Architekturentscheidung ist die Verarbeitung im Browser ohne Serverübertragung. Sie passt zum Schutz potenziell sensibler Screenshots und hält den technischen Betrieb einfach. Die modulare Struktur und das zentrale Projektmodell schaffen zugleich die Grundlage für die noch offenen Funktionen: Projektdateien, Bildannotation und einen kontrollierten PDF-Export.

Der aktuelle Stand ist damit ein belastbarer MVP, aber noch nicht das vollständige Zielprodukt. Die nächsten Schritte sollten insbesondere Datensicherung, große Bildmengen, Browserkompatibilität, Barrierefreiheit und die sichere Schwärzung im PDF absichern.

## Mögliche Rückfragen und kurze Antworten

**Warum gibt es keinen Server?**  
Weil die Kernbearbeitung lokal möglich ist und sensible Screenshots den Browser möglichst nicht verlassen sollen. Außerdem sinkt dadurch der Betriebsaufwand.

**Sind die Daten damit automatisch sicher?**  
Die lokale Verarbeitung reduziert Übertragungsrisiken, ersetzt aber kein Sicherheitskonzept. Browserprofil, Endgerät, Backups und die spätere sichere Schwärzung müssen ebenfalls berücksichtigt werden.

**Warum React statt einer einfachen HTML-Seite?**  
Der Editor besitzt viele zusammenhängende Zustände und Interaktionen. React erleichtert deren Aufteilung in wartbare Komponenten.

**Warum werden Bilder als Data URL gespeichert?**  
Damit Bild und Projektdaten im MVP gemeinsam lokal gespeichert und zuverlässig wiederhergestellt werden können. Für große Datenmengen muss die Speicherstrategie noch geprüft werden.

**Was ist bereits fertig?**  
Deckblattdaten, Schrittverwaltung, Texte, Screenshot-Import, Zoom, Sortierung, Löschung sowie automatische lokale Speicherung und Wiederherstellung.

**Was fehlt noch?**  
Portable Projektdateien, echte Bildannotation und Zuschnitt, PDF-Export sowie umfassende Tests für große Projekte, Zielbrowser, Barrierefreiheit und sichere Schwärzung.
