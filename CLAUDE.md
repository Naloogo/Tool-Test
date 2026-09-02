# Auslegung-WP — Interne Tools der Lutz Spring GmbH

Sammlung interner Web-Tools für Heizungs-/Wärmepumpenplanung. Läuft als
statische Seite über GitHub Pages, ohne Build-Schritt und ohne Backend.

## Dateien

| Datei | Tool | Zweck |
|---|---|---|
| `index.html` | Landingpage | Einstiegsseite mit Kacheln zu allen Tools |
| `auslegung.html` | Wärmepumpenauslegung | Heizlast- und WP-Auslegungsrechner |
| `heizkoerper.html` | Heizkörperdimensionierung | Heizkörperauslegung, gleicht gegen Lagerbestand ab |
| `kostenvergleich.html` | Kostenvergleich | Wirtschaftlichkeitsvergleich Heizsysteme |
| `bestandsaufnahme.html` | Lagerbestand | Erfassung/Ansicht des Heizkörperlagers |
| `bad-angebot.html` | Bestandsaufnahme Bad | Bauvorhaben, Sanitärobjekte und Ausführungen aufnehmen; exportiert GAEB X83 für Hero |
| `lager.json` | Datenbasis | Lagerbestand, gelesen von `heizkoerper.html` + `bestandsaufnahme.html` |
| `theme.css` | Design-System | **Zentrale Quelle für alle Farben** |
| `_vorlage.html` | Vorlage | Startpunkt für ein neues Tool, kein produktives Tool |
| `logo.png` | Logo | Firmenlogo, von `_vorlage.html` genutzt |
| `robots.txt` | — | Suchmaschinen ausgeschlossen (internes Tool) |

## Architekturregeln

**Ein Tool = eine eigenständige HTML-Datei.** CSS im `<style>`-Block, JS im
`<script>`-Block. Einzige gemeinsame Datei ist `theme.css`. Keine externen
CDNs, keine npm-Pakete, kein Build-Prozess.

**Kein Framework.** Vanilla JS, direkte DOM-Manipulation. Nicht auf React,
Vue o.ä. umstellen, auch nicht teilweise.

**Navigation:** Jedes Tool verlinkt per `href="index.html"` zurück zur
Landingpage. Die Landingpage verlinkt auf alle Tools. Bei einem neuen Tool
beide Richtungen ergänzen.

**Daten:** `lager.json` liegt im selben Ordner und wird per `fetch` geladen.
Feldstruktur ist im `_hinweis`-Feld der Datei selbst dokumentiert. Beim
Ändern des Schemas beide lesenden Tools mit anpassen. Wegen des `fetch` muss
lokal über einen Webserver getestet werden (`python3 -m http.server`),
nicht per Doppelklick auf die Datei.

**iOS/PWA:** Die Tools werden auf iPhones als Web-App vom Homescreen genutzt.
`viewport-fit=cover`, `env(safe-area-inset-*)` und
`-webkit-text-size-adjust` beibehalten. Änderungen immer auch in schmaler
Viewport-Breite prüfen.

## Design-System

**Alle Farben stehen in `theme.css`.** Für ein Redesign ausschließlich diese
Datei ändern — die Änderung schlägt automatisch auf alle Tools durch.

Jede HTML-Datei bindet `theme.css` ein und übersetzt anschließend in einem
kleinen `:root`-Block ihre historisch gewachsenen Variablennamen auf die
zentralen Tokens, zum Beispiel:

```css
:root{
  --navy: var(--brand-navy);
  --line: var(--border-cool);
}
```

Diese Alias-Blöcke sind Absicht: So mussten die über 5.000 bestehenden
CSS-Regeln nicht angefasst werden. Die Namen dürfen bleiben.

**Regeln für neue Arbeit:**

- Niemals einen Hex-Wert direkt in eine HTML-Datei schreiben. Immer
  `var(--token)` aus `theme.css` benutzen.
- Fehlt ein passendes Token, erst `theme.css` erweitern, dann verwenden.
- Der `<link rel="stylesheet" href="theme.css">` muss **vor** dem
  `<style>`-Block stehen, damit die Alias-Blöcke gewinnen.

**Fest verdrahtete Farben: erledigt.** Alle Tools nutzen ausschließlich
Tokens; ein `grep` nach Hex- oder `rgba()`-Werten liefert nichts mehr.
Zwei Ausnahmen sind Absicht und müssen Literale bleiben:
`<meta name="theme-color" content="#1a2b4a">` in `index.html` und
`_vorlage.html` — in einem `<meta>`-Tag ist `var()` ungültig.

Beim Überführen weiterer Farben gilt: **eine Farbe = ein Token mit exakt
demselben Wert.** Nicht beim Aufräumen gleichzeitig Farben zusammenlegen –
das ist eine Design-Entscheidung und gehört in einen eigenen Schritt.
Ob das Ergebnis unverändert aussieht, prüft man zuverlässig so: in beiden
Fassungen jedes `var(--token)` vollständig zu seinem Endwert auflösen; die
Dateien müssen danach zeichengleich sein.

Zwei Fallen, die dabei aufgetreten sind:

- **HTML-Entities sehen aus wie Farben.** `&#128293;` (🔥), `&#9888;` (⚠)
  oder `&#215;` (×) matchen jedes naive `#[0-9a-f]{3,6}`-Muster. In
  `auslegung.html` und `kostenvergleich.html` stecken 28 solcher Entities.
  Immer erst `&#…;` ausklammern, dann nach Farben suchen.
- **`#fff` ist nicht immer dasselbe.** Als `color` ist es `--text-on-dark`,
  als `background` `--card`. Stumpf ersetzt geht die Semantik verloren.

**Farbgleiche Paare, bewusst nicht zusammengelegt** (Zusammenlegen wäre eine
Design-Entscheidung): `--danger-bg-soft-hover` `#e8bcbc` vs.
`--danger-bg-soft-hover-alt` `#eec3c3`, sowie `--blue-hover` `#d5dfec` vs.
`--blue-soft-hover` `#d5dfeb`. Die Paare unterscheiden sich um ein Bit und
sind mit hoher Wahrscheinlichkeit historische Tippfehler.

**Zwei Flächenfamilien:** `index/auslegung/kostenvergleich` nutzen einen
warmen Beigeton (`--bg`), `heizkoerper/bestandsaufnahme` einen kühlen
Blauton (`--bg-cool`). Historisch gewachsen. Um zu vereinheitlichen, in
`theme.css` einfach `--bg-cool: var(--bg);` setzen.

## Schnittstelle zu Hero (Kalkulationssoftware)

Kalkuliert und bepreist wird **nicht** in diesen Tools, sondern in Hero.
Die Tools liefern die Positionen, Hero macht daraus das Angebot.

Was Hero kann (vom Betrieb bestätigt): IDS Connect, DATANORM-Artikelstamm
von Wiedemann (bereits eingespielt), UGL-Export/-Import, GAEB-Import,
OpenTrans-Import. **Bestellen und Preise kann Hero also selbst** — das
gehört nicht in diese Tools.

**Der Weg ist GAEB, Austauschphase 83 (Angebotsaufforderung).** Grund:
Artikelnummern allein sparen keine Zeit, der Aufwand steckt im Abtippen
der Positionstexte. GAEB überträgt Kurztext, **Langtext**, Menge und
Einheit — also genau das.

`bad-angebot.html` erzeugt **GAEB DA XML 3.2, Belegtyp X83**
(Namensraum `http://www.gaeb.de/GAEB_DA_XML/DA83/3.2`). Aufbau:

    GAEB > Award(DP=83) > BoQ > BoQBody > BoQCtgy > Itemlist > Item

Je Gegenstandsart eine `BoQCtgy`, je Position ein `Item` mit `Qty`, `QU`
und `Description/CompleteText` (`DetailTxt` = Langtext aus den
Ausführungsfeldern, `OutlineText` = Kurztext). Preise stehen bewusst nicht
drin — das ist der Sinn von Phase 83.

Beim Ändern des Generators: Die Struktur folgt einer verifizierten
Beispieldatei, nicht dem Bauchgefühl. Keine Elemente erfinden — GAEB wird
gegen ein XSD geprüft, und ein unbekanntes Element lässt den Import
scheitern. Vor dem Download prüft der Code selbst per `DOMParser`, ob das
XML wohlgeformt ist.

`heizkoerper.html` hat denselben Export, aber in der Gliederung des
regulären Angebots (geprüft gegen ANG-255):

    Pos. 1  Heizkörper             -> Untergruppe je Raum
    Pos. 2  Installationsmaterial  -> Konsolen/Blenden über alle Räume summiert
    Pos. 3  Arbeits- und Serviceleistungen

Drei Punkte dazu:

- **Artikelbezeichnung.** Der Kurztext wird im Format des Großhandels-
  Artikelstamms erzeugt, z. B. `KERMI X2 Profil-K Typ22 BH600x100x1200mm
  QN1999, weiß, 10 bar, m. Abdeckung`. `QN` ist die Normwärmeleistung
  75/65/20 und ergibt sich aus `DB().normW()` (= `sl` × Baulänge). Gegen
  zwei Positionen aus ANG-255 verifiziert. Nur für Kermi therm-x2 belegt,
  sonst greift der `klartext` des Tools.
- **Konsolenregeln sind dupliziert.** `gaebKonsolen()` ist eine bewusste
  Kopie von `addKonsole()` aus dem Druckbericht, damit der funktionierende
  Bericht nicht angefasst werden musste. **Ändert sich eine Regel dort,
  muss sie hier mitgezogen werden.**
- **Montagezeit wird nicht geraten.** Das Feld „Montagezeit je HK" ist
  leer voreingestellt; bleibt es leer, entfällt die Position „Arbeitszeit
  Monteur". Es wird keine Zeitnorm erfunden.

GAEB legt in Hero **Leistungen** an, keine Artikel (`Artikel 0 /
Leistungen 10`). Das ist bauartbedingt: GAEB ist ein Leistungsverzeichnis
und hat kein Feld für eine Großhandels-Artikelnummer. Der Artikel wird in
Hero je Position zugeordnet — am schnellsten über einen eigenen
Leistungskatalog. Deshalb ist die exakte Artikelbezeichnung im Kurztext
wichtig: darüber lässt sich der Artikel im Stamm direkt finden.

**Wichtig zum Artikelstamm** (an einem echten Wiedemann-Datensatz geprüft,
Kermi Profil-K Typ 33, BH 500, BL 1800):

| Feld in Hero | Inhalt (Werte hier anonymisiert) |
|---|---|
| Artikelname | `KERMI X2 Profil-K Typ33 BH500x155x1800mm QN3499, weiß, 10 bar, …` |
| Artikelnummer / Lieferanten-Nr. | 7-stellig numerisch |
| Matchcode | `KK<Typ>FHK<Bauhöhe><Baulänge>` |
| EAN | 13-stellig |
| Hersteller-Nr. | **leer** |

Die konkreten Nummern stehen bewusst nicht hier — das Repository ist
öffentlich, und die Werte stammen aus der Großhandels-DATANORM.

Daraus folgt zweierlei:

1. **Der Kermi-Bestellschlüssel (`FK0330514`) taugt nicht zur Suche.**
   Wiedemanns DATANORM füllt „Hersteller-Nr." nicht, die Nummer existiert im
   Stamm also gar nicht. Sie bleibt trotzdem im Langtext — zum Bestellen bei
   Kermi direkt —, ist aber als `Kermi-Bestellschlüssel:` gekennzeichnet,
   damit sie nicht mit der Großhandelsnummer verwechselt wird.
2. **Die vom Tool erzeugte Bezeichnung ist zeichengleich mit dem
   Artikelnamen im Stamm.** Sie steht deshalb als **erste Zeile im Langtext**
   — nicht nur im Kurztext, denn Hero zeigt den GAEB-Kurztext nicht an
   (in der Positionsliste steht „Position"). Sichtbar ist nur der Langtext,
   und darüber wird gesucht. **Diese Reihenfolge nicht ändern.**

### Artikeldaten aus DATANORM

`heizkoerper.html` kann optional eine Artikeltabelle laden („Artikeldaten
laden"). Ist sie geladen, schreibt der GAEB-Export **Großhandels-Artikel-
nummer, EAN und den echten Artikelnamen** in den Langtext.

**Diese Datei gehört NICHT ins Repository.** Das Repo ist öffentlich
erreichbar (GitHub Pages), und der DATANORM-Vorlaufsatz sagt ausdrücklich:
„Daten bleiben unser Eigentum … Weitergabe an Dritte nicht erlaubt".
Die Tabelle wird deshalb pro Gerät über die Dateiauswahl geladen und im
`localStorage` gehalten. Preise und Rabattgruppen sind im Konverter
bewusst ausgeschlossen.

Erzeugt wird sie aus `DATANORM.001` (DATANORM 4.0, **CP850**-kodiert):

    A;kz;ArtNr;TextKz;Kurztext1;Kurztext2;PreisKz;PreisEh;ME;PREIS;RABGRP;WarenGrp
    B;kz;ArtNr;Matchcode;Herstellernummer;…;EAN;…

Der **Artikelname** ist `Kurztext1 + " " + Kurztext2`. Feld 5 des B-Satzes
enthält den **Kermi-Bestellschlüssel** — genau die Nummer, die das Tool
berechnet. Hero übernimmt dieses Feld beim DATANORM-Import nicht in
„Hersteller-Nr.", deshalb findet die Suche danach nichts.

Die Tabelle hat zwei Indizes:

- `dim` — `Baureihe|Typ|Bauhöhe|Baulänge[|li/re]`, aus dem Artikelnamen
  geparst. **Der belastbarere Weg**, weil unabhängig vom Nummernformat.
- `bn` — Bestellschlüssel, als Rückfallebene.

Abdeckung über das gesamte Tool-Sortiment: **Kermi therm-x2 92 %,
Verteo 74 %**. Die Lücken sind echte Sortimentslücken des Großhandels
(z. B. Typ 11 Bauhöhe 700), kein Fehler der Zuordnung.

**Nicht die Bezeichnung selbst zusammenbauen, wenn die Tabelle da ist.**
Die berechnete Form stimmt nicht überall: Typ 33 heißt `KERMI X2 Profil-K …`,
Typ 10 dagegen nur `KERMI Profil-K …`. Deshalb hat der Name aus der Tabelle
Vorrang; `gaebBezeichnung()` ist nur die Notlösung ohne geladene Daten.

Noch offen: Bedarfspositionen lassen sich in Hero je Position ankreuzen;
ob GAEB sie mitliefern kann, ist nicht verifiziert. `auslegung.html` hat
noch keinen GAEB-Export. Arbonia und Zehnder liefern keinen Bestellschlüssel
und sind in der Artikeltabelle nicht abgedeckt.

## Neues Tool anlegen

1. `_vorlage.html` kopieren und benennen (klein, ohne Umlaute/Leerzeichen)
2. Kopfbereich, Eingabefelder und Rechenlogik ausfüllen (`TODO`-Markierungen)
3. Kachel in `index.html` ergänzen
4. Zeile in der Tabelle oben in dieser Datei ergänzen

## Arbeitsweise

Commits auf `main` gehen direkt live über GitHub Pages. Vor dem Push kurz
gegenprüfen, dass die geänderte Datei im Browser fehlerfrei lädt
(Konsole ohne Errors).

**Rechenlogik ist fachlich kritisch.** Formeln in den Auslegungstools nicht
ohne Rückfrage umbauen. Bei neuen Formeln die Quelle als Kommentar dazu
(Norm, Handbuch, Seite).

Bei Änderungen am Aussehen: vorher/nachher im Browser vergleichen. Ein
Screenshot-Vergleich über alle fünf Tools ist der zuverlässigste Weg,
ungewollte Nebenwirkungen zu erkennen.
