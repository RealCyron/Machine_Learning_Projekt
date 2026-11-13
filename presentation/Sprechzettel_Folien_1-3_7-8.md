# Sprechzettel: Folien 1, 2, 3, 7 und 8

Präsentation: `Feinstaub schätzen, wo keine Messstation steht.pptx`
Zahlenquelle: `lukas_feature_engineering_alle_features.ipynb` (Branch `lukas_ms3`), also der Durchlauf **mit** Satelliten-Geometrie (177 Merkmale).

> Achtung Foliennummern: Folie 7 trägt unten rechts die Nummer **9**, Folie 8 die **10** (die Fußzeilen-Nummern passen nicht zur Reihenfolge im Foliensatz). „Folie 7“ = „Die wichtigsten Schadstoffwerte“, „Folie 8“ = „Die Schwachstelle“.

---

## Vorab: Was sind „die Gruppen“?

Im Vortrag tauchen **zwei verschiedene Arten von Gruppen** auf. Nicht verwechseln:

**1. Merkmalsgruppen (die „neun Gruppen“ auf Folie 3 und 4)**
- Merkmal = eine Eingangsspalte fürs Modell (z. B. „mittlerer CO-Wert dieses Ortes“).
- Lukas hat aus den 74 Rohspalten 177 Merkmale gebaut und sie **nach Idee sortiert** in neun Gruppen:

| Gruppe | Anzahl | Was steckt drin | Alltagssprache |
|---|---|---|---|
| Ausgangsmodell | 8 | Wind, HCHO, CO, NO₂, Aerosol, Bewölkung, Tag im Jahr, Wochentag | der Vergleichsmaßstab |
| Übrige Rohdaten | 39 | alle anderen Mess- und Wetterspalten | „einfach alles nehmen“ |
| Satelliten-Geometrie | 35 | Blickwinkel, Sonnenstand, Bahnhöhe | Aufnahmebedingungen |
| Datum | 3 | Tag im Jahr, Wochentag, Wochenende | Jahreszeit, Verkehr |
| Abgeleitetes Wetter | 5 | Windstärke/-richtung, Temperatur × Feuchte, Taupunktabstand | Ausbreitung der Partikel |
| Satelliten-Kennzahlen | 5 | Gasverhältnisse, mittlere Bewölkung, Zahl fehlender Werte | Messqualität |
| **Orts-Profile** | 42 | Mittel, Streuung, Abweichung **je Ort über alle Tage** | „Wie ist dieser Ort normalerweise?“ |
| Tages-Lage | 28 | Mittel je Tag **über alle Orte** | großräumige Wetterlage |
| Glättung | 20 | gleitende Mittel über 3 und 7 Tage je Ort | gleicht verrauschte Satellitenwerte aus |

- „Gruppen prüfen“ (Schritt 2) heißt: Gruppe für Gruppe hinzufügen und messen, ob der Fehler sinkt (zusätzlich jede Gruppe einzeln zum Ausgangsmodell).
- Gewinner: **Orts-Profile** (rund zwei Drittel der Wichtigkeit). 13 der 20 finalen Merkmale sind Orts-Profile.
- Wichtig: Orts-Profile werden **nur aus Satelliten- und Wetterdaten** berechnet, nie aus dem gemessenen Feinstaub. Deshalb lassen sie sich auch für einen Ort ohne Messstation bilden.

**2. Orts-Gruppen für den Test (Kreuzvalidierung)**
- Die 340 Orte werden in **fünf Gruppen à ~68 Orte** geteilt.
- Das Modell lernt auf vier Gruppen und schätzt die fünfte. Das wird fünfmal wiederholt, bis jeder Ort einmal „unbekannt“ war.
- So bekommt jeder der 30.557 Tageswerte genau eine Schätzung von einem Modell, das den Ort nie gesehen hat. Auf diesen Schätzungen beruhen alle Fehlerzahlen (auch Folie 8).

Merksatz: *Merkmalsgruppen = was das Modell bekommt. Orts-Gruppen = wie wir ehrlich testen.*

---

## Folie 1: Titel (≈ 0:30)

- Begrüßung, wer wir sind.
- Kernfrage in einem Satz: **Können wir Feinstaub (PM2.5) dort schätzen, wo keine Messstation steht, und zwar nur mit frei verfügbaren Satelliten- und Wetterdaten?**
- Zweite Frage: **Welche dieser Daten braucht man dafür wirklich?**
- Ausblick: Am Ende steht, was davon für eine Umweltverwaltung nutzbar ist und wo die Grenzen liegen.

## Folie 2: Die Datengrundlage (≈ 1:00)

- **340 Orte** mit gemessenem Feinstaub (µg/m³), Daten aus dem Zindi-Wettbewerb „Urban Air Pollution“.
- **30.557 Tageswerte**: ein Wert = ein Ort an einem Tag, Januar bis April 2020 (ca. 90 Tage je Ort).
- **74 Eingangsspalten** pro Ort und Tag: Satellitenmessungen zu Spurengasen (NO₂, CO, O₃, SO₂, Formaldehyd, Methan), Aerosolen und Wolken, dazu Wetter. Davon sind 35 reine Aufnahme-Geometrie (Winkel, Bahnhöhe), keine Schadstoffe.
- Darum der Satz unten: **Das Modell muss auf Orte übertragbar sein, die es nie gesehen hat.** Genau das ist auch der Anwendungsfall in der Verwaltung.
- Falls Nachfrage zur Region: Orte sind nur als anonyme IDs angegeben, keine Koordinaten, Region unbekannt.

## Folie 3: Das Vorgehen in drei Schritten (≈ 1:15)

- Grundidee: Das Vorhersagemodell (Gradient Boosting) ist hier **das Messinstrument**. Es bleibt in allen Versuchen gleich eingestellt, damit Unterschiede nur von den Merkmalen kommen.
- **1 Merkmale bauen:** Aus 74 Rohspalten werden 177 Merkmale, sortiert in neun Merkmalsgruppen (z. B. Orts-Profile, Tages-Lage, Glättung). Details auf der nächsten Folie.
- **2 Gruppen prüfen:** Jede Merkmalsgruppe wird hinzugefügt und gemessen, ob der Fehler sinkt. Getestet wird immer an Orten, die das Modell nicht kennt (fünf Orts-Gruppen, siehe oben).
- **3 Auswählen:** Das Modell bewertet selbst, welche Merkmale es braucht (Durchmischen eines Merkmals → wie stark wird die Schätzung schlechter?). Übrig bleibt das **kleinste Set, das praktisch genauso gut ist: 20 Merkmale**.
- **Satz unten, kurz erklären:** „Gegen geschönte Ergebnisse“
  - keine Ortskennung als Merkmal (sonst lernt das Modell Orte auswendig),
  - keine Information aus den Feinstaub-Messwerten in den Merkmalen,
  - Spalten, die es nur im Training gibt (Min/Max/Varianz des Messwerts), bleiben draußen.
- Übergabe an Folie 4 (Lukas).

## Folie 7: Die wichtigsten Schadstoffwerte (≈ 1:30)

- Einstieg: Von den 20 ausgewählten Merkmalen beziehen sich viele auf **fünf Spurengase**. Die Tabelle zeigt, wie weit vorne sie stehen und woher sie stammen.
- **Rang** = bester Platz des Stoffs unter den 20 Merkmalen (1 = wichtigstes).
- **Formaldehyd (Rang 1)** und **Kohlenmonoxid (Rang 2)** führen, jeweils als **langfristiges Ortsmittel**, also „Wie hoch ist der Wert an diesem Ort normalerweise?“. CO steckt viermal in den Top 20, Formaldehyd dreimal.
- Warum sagen Gase etwas über Feinstaub? Sie kommen großteils **aus denselben Quellen** wie die Partikel: Verbrennung in Verkehr, Heizungen, Industrie, Bränden. NO₂ und organische Verbindungen sind außerdem Vorläufer, aus denen sich neue Partikel bilden.
- Kurz zu den anderen:
  - Methan (Rang 8): Landwirtschaft, Deponien, Förderung fossiler Brennstoffe.
  - Ozon (Rang 11): **Vorsicht**, der Satellit misst die ganze Luftsäule, also vor allem die Ozonschicht, nicht das bodennahe Ozon.
  - NO₂ (Rang 14): heiße Verbrennung, v. a. Verkehr und Kraftwerke.
  - SO₂ wurde geprüft, ist aber nicht unter den 20.
- Satz unter der Tabelle: **20 statt 177 Merkmale bei fast gleichem Fehler (30,4 statt 30,2)**. Ebenfalls weit vorn: Windstärke (Rang 3, ebenfalls als Ortsmittel) und Jahreszeit (Rang 6).
- **Pflichtsatz (Fußzeile):** Rang = Bedeutung für die Vorhersage, **kein Nachweis einer Ursache**. Die Spalte „So entsteht er“ ist allgemeines Fachwissen, nicht aus unseren Daten.

## Folie 8: Die Schwachstelle, Tage mit hoher Belastung (≈ 1:30)

- Kernaussage zuerst: **Das Modell wird überall besser, aber gerade die gesundheitlich wichtigen Spitzentage trifft es am schlechtesten.**
- Tabelle lesen:
  - Die Tageswerte sind nach dem **gemessenen** Feinstaub in sechs Bereiche sortiert.
  - „Fehler“ = wie weit die Schätzung im Mittel danebenliegt, in µg/m³ (ohne Vorzeichen).
  - „vorher“ = Ausgangsmodell (8 Merkmale), „jetzt“ = 20 ausgewählte Merkmale.
- Beispiel vorlesen: Über 200 µg/m³ gibt es 256 Tageswerte. Das Ausgangsmodell lag im Schnitt 131 daneben, das neue Modell 101. Besser, aber immer noch sehr weit weg.
- In jedem Bereich sinkt der Fehler, am stärksten bei hohen Werten (z. B. 150–200: von 64,7 auf 46,6).
- **Die 42 %:**
  - Nur **8 %** der Tageswerte liegen über 150 µg/m³.
  - Sie verursachen aber **42 %** des Gesamtfehlers (22 % + 20 %, gemessen am quadratischen Fehler, also dem Maß hinter unserer Hauptkennzahl RMSE).
  - Grund: Beim Quadrieren zählt ein Fehler von 100 als 10.000, ein Fehler von 15 nur als 225. Wenige große Ausreißer dominieren.
- Richtung: Das Modell **unterschätzt** Spitzen (sichtbar im Streudiagramm im Notebook, nicht in der Tabelle selbst).
- Konsequenz fürs Ministerium: Für Warnungen und Grenzwert-Alarme braucht es weiter Messstationen. Das Modell eignet sich zum Priorisieren, nicht zum Alarmieren. Überleitung zu den Empfehlungen.

---

## Mögliche Rückfragen

- **„Was heißt RMSE 30?“** Typischer Fehler von rund 30 µg/m³ an unbekannten Orten. Ein Modell, das immer den Mittelwert schätzt, läge bei knapp 47.
- **„Warum nicht alle 177 Merkmale?“** Gleicher Fehler, aber 20 sind einfacher zu erklären, zu pflegen und weniger anfällig für Zufallsmuster.
- **„Wie ehrlich ist die 30,4?“** Leicht optimistisch, weil die Auswahl der 20 auf allen Daten lief. Die streng verschachtelte Schätzung liegt bei 30,7. Die Streuung zwischen den fünf Testdurchgängen beträgt etwa 4, kleine Unterschiede sind daher nicht belastbar.
- **„Funktioniert das für einen Ort mit wenigen Tagen Daten?“** Eher nicht, Orts-Profile brauchen eine längere Datenreihe (hier ca. 90 Tage je Ort).
- **„Der unterste Bereich (bis 25) hat ja auch 17,9 Fehler?“** Stimmt, relativ zum Messwert ist das ebenfalls ungenau. Bei sauberer Luft liegt die Schätzung in der Größenordnung des Werts selbst.
