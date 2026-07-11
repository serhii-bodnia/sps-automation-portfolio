# **Übungsprojekt: SPS-Programmierung mit TIA Portal, Factory I/O.** 

# **Automatisierung einer modularen Abfüll- und Sortieranlage mit Siemens S7-1200**

## Screenshots

**Gesamtanlage (Factory I/O Simulation)**
![Factory I/O Übersicht](screenshots/factory-io-overview.png)

**Gesamtlayout der Anlage**
![Factory I/O Layout](screenshots/factory-io-layout-full.png)

**HMI – Abfüllbereich**
Rezeptauswahl, Soll-/Istwerte für Füllstand, Füllmenge und Gewicht je Linie, NOT-AUS-Taster.
![HMI Abfüllbereich](screenshots/hmi-abfuellbereich.png)

**HMI – Übersicht**
Statusanzeige beider Linien, Produktionsdaten, Palettenzähler, globaler STOP-ALL-Taster.
![HMI Übersicht](screenshots/hmi-uebersicht.png)

## Code-Beispiele (SCL)

Auszug aus den zentralen Funktionsbausteinen des Projekts:

- [`FB_Dose_Control.scl`](code/FB_Dose_Control.scl) — Zweistufige Dosierregelung
  (Schnellfüllung/Feindosierung) mit Volumenstromintegration über OB30-Zykluszeit
- [`FB_Gewichtkontrolle_SCL.scl`](code/FB_Gewichtkontrolle_SCL.scl) — Schrittkette
  zur Gewichtsprüfung mit Toleranzband und automatischer Ausschussaussortierung
- [`FB_Pick_and_Place.scl`](code/FB_Pick_and_Place.scl) — Schrittkette für
  XYZ-Handling mit palettenweiser Ablage (4 Positionen) und Palettenwechsel-Logik



---
# **1. Projektbeschreibung**   
**1.1 Gesamtkonzept des Projekts** 

Das System simuliert eine Produktionslinie zum Befüllen und Sortieren von Flüssigkeitskanistern. Das Projekt umfasst: 

- Lagerung und Dosierung von Flüssigkeiten aus zwei Tanks. 
- Präzise volumetrische Befüllung mittels Durchflussmessern (Flowmeter). 
- Gewichtskontrolle und Ausschuss-Erkennung (Ausschleusung). 
- Transportbänder und Fördersysteme. 
- Sortierung nach Produkttyp (unter Verwendung eines Vision Sensors). 
- Palettierung der Fertigprodukte mittels eines Pick-and-Place-Roboters. 
- Visualisierung und Bedienung über ein HMI-Bedienpanel. 
Jede Linie arbeitet mit einem eigenen Reservoir sowie einem separaten Befüll- und Dosiersystem. Nach der Befüllung durchlaufen die Produkte eine Gewichtskontrolle, werden anschließend auf einer gemeinsamen Transportlinie weitergeleitet und je nach Produkttyp sortiert. 

**1.2 Hardware-Komponenten** 

Im Projekt werden folgende Komponenten verwendet:

- SPS-Steuerung: Siemens S7-1200 CPU 1211C DC/DC/DC 
- Analoge Füllstandssensoren (Levelmeter) 
- Analoge Wägezellen (Gewichtssensoren) 
- Diffuse Reedsensoren / Lichttaster (Präsenzerkennung) 
- Aktoren / Spiegelmechanismen: 
  - Band- und Rollenförderer (Transportbänder)
  - Emitters und Removers (Objekterzeugung/-löschung)
  - Befüllventile (Fill Valves)
  - Sortier- und Schwenkmechanismen ○ Pusher
  - Signalleuchten (Meldeleuchten) 
Die SPS-Steuerung übernimmt: 

- Die Verarbeitung der Eingangssignale 
- Die Ansteuerung der Aktoren 
- Den Datenaustausch mit dem HMI über PROFINET 

**1.3. Software-Architektur**

Das Projekt ist modular aufgebaut und nutzt parametrisierte Funktionsbausteine (FB). 
Die Aufgabenverteilung ist klar strukturiert: OB1 übernimmt die allgemeine Steuerungslogik, während OB30 für die hochpräzise Dosierung zuständig ist. 
Wichtigste Software-Module: 

- "FC_Mode_Manager" – Zentrale Steuerung der Betriebsarten 
- "FB_Tank" – Steuerung und Füllstandsregelung des Reservoirs 
- "FB_Dose_Control" – Dosierung von Flüssigkeit in Kanistern nach vorgegebenem Sollwert 
- "FB_Transport_Band" – Standard-Förderband (gerade) 
- "FB_Transportband mit Verzögerung" – Förderband mit Einschalt-/Ausschaltverzögerung 
- "FB_Transport_T-Band" – T-Kreuzung / T-Stück-Förderband 
- "FB_Band mit Positionierleisten" – Förderband mit Positionierungsanschlägen 
- "FB_Palette_Linie" – Palettenzuführung 
- "FB_AbfüllmaschineBand" – Steuerung des Bands der Abfüllanlage 
- "FB_Gewichtkontrolle" – Gewichtskontrolle und Toleranzprüfung der Produkte 
- "FB_Sortierabschnitt" – Steuerung des Sortierbereichs 
- "FB_Laufzeit" – Betriebsstundenzähler des Systems 
Für die Verarbeitung analoger Signale wird ein separater Skalierungsbaustein verwendet: 

- FC_Scalierung 

**1.4. Sicherheitssystem** 

Im Projekt ist ein mehrstufiges Not-Halt- und Fehlersystem integriert: 

- Hardware-Not-Halt-Taster (Emergency Stop) 
- NOT-AUS-Betriebsmodus 
- Stauüberwachung (Erkennung von Blockaden auf den Förderbändern) 
- Störungsquittierung durch den Bediener über das HMI 
- Optische Zustandsanzeige über Signalleuchten 
Beim Auslösen eines Not-Halts oder einer kritischen Störung: 

- Werden alle Förderbänder sofort abgeschaltet 
- Wird der Dosiervorgang abrupt abgebrochen 
- Werden alle Aktoren und Ventile verriegelt (sicherer Zustand) 

**1.5. HMI-Schnittstelle** 

Auf dem HMI-Bedienpanel (KTP900 Basic) sind folgende Funktionen umgesetzt: 

- Anzeige des aktuellen Tankfüllstands 
- Anzeige der dosierten Gutmenge 
- Anzeige des aktuellen Gewichts 
- Rezepturverwaltung und Rezeptauswahl 
- Laden von Rezepten in die Steuerung (Rezept hochladen) 
- Anzeige von Produktionsdaten (Stückzahlzähler) 
- Grafische Visualisierung von Fehlern direkt im Anlagenlayout 
- Anzeige von Alarm- und Systemmeldungen (Alarm Logging) 
- Starten und Stoppen der jeweiligen Produktionslinie 
Die Benutzeroberfläche beinhaltet: 

- Grafische Darstellung der Tanks mit Füllstandsanimation 
- Statusanzeigen (Meldungen/Leuchten) 
- Start- / Stop-Taster 
- Virtueller NOT-AUS-Taster 
- Eingabe-und Ausgabefelder (E/A-Felder) für Prozessparameter 

# **2. Funktionsprinzip der Anlage**  
**2.1 Bereich Lagerung und Rohstoffaufbereitung**
   
Das System übernimmt die Soll-Füllvolumen der Reservoirs aus den HMI-Rezepturen und überwacht den tatsächlichen Füllstand kontinuierlich mithilfe der analogen Füllstandssensoren. 
Bei Absinken des Füllstands: 
  
- Öffnet sich das Befüllventil automatisch 
- Wird das Reservoir bis zum definierten Sollwert befüllt 
- Schließt das Ventil exakt nach Erreichen des Sollwerts 
Realisierte Logik: 

- Proportional-Integral-Regellogik zur Ansteuerung des Befüllventils (FB_Tank) mit einer zweistufigen Regelung plus  dynamischer Komponente zur exakten Dosierung. 
- Grafische Füllstandsvisualisierung sowie Plausibilitätsprüfungen gegen fehlerhafte Sollwerteingaben. 

**2.2 Abfüllstation (Abfüllmaschine)**

Nach erfolgreicher Befüllung der Tanks startet die Zuführung der leeren Kanister. 
Sobald ein Kanister durch die Sensoren der Abfüllstation erfasst wird, stoppt das Förderband und der Dosiervorgang beginnt. Die Dosierwerte werden über die HMI- Rezeptur vorgegeben. 
Realisierte Logik: 
  
- Synchronisation mit dem Fördersystem: Automatischer Bandstopp während der Befüllung. 
- Die präzise volumetrische Dosierung der Flüssigkeit erfolgt im Baustein FB_Dose_Control (SCL), der im Weckalarm-Organisationsbaustein OB30 (10 ms) aufgerufen wird, um maximale Genauigkeit zu garantieren. 
- Nutzung eines integrierten Durchflusssignals (Flowmeter) und zweistufige Ventilansteuerung. 
- Echtzeit-Visualisierung des Kanister-Füllstands und Animation des geöffneten Dosierventils. 

**2.3 Gewichtskontrolle** 

Nach dem Befüllen durchlaufen die Produkte die Kontrollstation. 
Das System übernimmt: 

- Die Überprüfung des Produktgewichts auf Konformität mit dem geladenen Rezept 
- Den Gesamtzähler der produzierten Einheiten (Gutmenge) 
- Den Ausschusszähler (Schlechtmenge) 
Bei Gewichtsabweichungen (Unter- oder Übergewicht): 
- Wird das fehlerhafte Produkt durch einen Pusher (Auswerfer) und einen Remover sofort aus der Linie entfernt. 
    
**2.4 Transportbänder und Fördersysteme** 

Die Steuerung der Transportbänder erfolgt über die Signale der Lichttaster. 
Im Baustein FB_Transportband mit Verzögerung ist eine Verriegelungslogik ("Belegt-Meldung") integriert, um Kollisionen zwischen den Kanistern effektiv zu verhindern. 

**2.5 Produktsortierung** 

Ein Vision Sensor erkennt den jeweiligen Produkttyp auf dem Band. Basierend auf dem Produkttyp und den Rezepturvorgaben erfolgt die Zuweisung auf die entsprechende Spur mittels eines Schwenkverteilers (Pivot Arm Sorter). 

**2.6 Palettierbereich** 

Dieser Bereich besteht aus: 

- Einem Pick & Place Roboter zur vordefinierten Stapelung der Kanister auf Paletten (2x2 Muster) 
- Einem Rollenfördersystem zur automatischen Zuführung von Leerpaletten und zum Abtransport der vollbeladenen        Paletten. 
Realisierte Logik: 
- Erfassung und Zählung fertiggestellter Paletten. 

# **3. Kurzbedienungsanleitung**   
**3.1 Systemstart** 

1. Benutzer-Anmeldung durchführen:
    - Login: Wolf, Passwort: 555 
    - Login: Meyer, Passwort: 111 
2. Im HMI-Bild **"Abfüllbereich"** das gewünschte Rezept aus der Dropdown-Liste für die entsprechende Linie auswählen. 
3. Den Taster **"Rezept hochladen"** betätigen. Die Sollwert-Anzeigen auf dem HMI müssen sich entsprechend den Rezeptvorgaben aktualisieren.

   ***Hinweis: Nach dem Start der Linie ist eine Rezeptänderung erst nach einem System- Reset wieder möglich.*** 
4. Den Taster **START** drücken, um die jeweilige Linie in Betrieb zu nehmen.

   ***Achtung! Solange kein Rezept erfolgreich geladen wurde, bleibt der START-Taster gesperrt (ausgegraut).*** 
5. Das System wechselt automatisch in den Automatikbetrieb. 
6. Das Betätigen des Tasters **STOP** im Bild **Abfüllbereich** führt zu einem kontrollierten Halt: 
    - Wenn nur eine Abfülllinie aktiv ist, stoppt die gesamte Anlage. 
    - Wenn beide Abfülllinien aktiv sind, stoppt nur die     Linie, bei der der entsprechende STOP-Taster gedrückt        wurde. 
    - Ein Wiederanlauf erfolgt durch erneutes Drücken von **START**. 
7. Das Drücken des **NOT-AUS-Taster** führt zu einem sofortigen, unkontrollierten Stillstand der gesamten Anlage. 
8. Der Taster **STOP ALL** im HMI-Bild **"Übersicht"** führt zu einem kontrollierten, koordinierten Herunterfahren        aller Anlagenkomponenten. 

**3.2 Anlagenbedienung** 

  **Hauptfunktionen des Bedieners** 
  
  Über das HMI kann der Bediener: 
  
- Die Linien starten und stoppen 
- Vor dem Start die entsprechende Rezeptur auswählen 
- Den Flüssigkeitsstand der Tanks überwachen 
- Die Dosiermenge kontrollieren 
- Produktionskennzahlen einsehen (Stückzähler) 
- Anstehende Fehlermeldungen einsehen und quittieren 
    
**Verhalten im Störungsfall (Alarmhandling)** 

Sollte einer der Lichttaster bei laufendem Förderband länger als 10 Sekunden ununterbrochen belegt sein (Stauerkennung), löst die Steuerung eine automatische Schutzabschaltung aus. Es wird eine Fehlermeldung mit genauer Orts- und Typangabe der Störung generiert. 
Im HMI-Bild **"Übersicht"** wird die genaue Störungsquelle im grafischen Anlagenlayout blinkend hervorgehoben. 
Um den Betrieb nach einer Störung wiederaufzunehmen:
  
1. Die physikalische Ursache der Blockade/Störung beseitigen. 
2. Den Fehler über die Quittiertaste (Ack) am HMI **quittieren**. 
3. Die Linie durch Drücken der Taste **START** erneut anfahren.
   
**Produktionsdaten (Betriebsdaten)**

Das HMI-Bild **"Abfüllbereich"** zeigt: 

- Den aktuellen Füllstand der Reservoirs 
- Die aktuell abgefüllte Dosiermenge 
- Den Schaltzustand des Dosierventils 
- Das aktuelle Gewicht des Produkts auf der Waage 
- Die Gesamtzahl der produzierten Gutteile 
- Die Anzahl der fehlerhaften Ausschussteile
Das HMI-Bild **"Übersicht"** zeigt: 

- Die optische Betriebszustandsanzeige der Gesamtanlage (Automatik / Hand / Störung) 
- Die Anzahl der fertig beladenen und abtransportierten Paletten 
- Die Anzahl der Produkte, die an nachgelagerte Produktionsbereiche übergeben wurden

  ---
**Kontakt:** [LinkedIn](https://www.linkedin.com/in/serhii-bodnia) · s.a.bodnya@gmail.com
