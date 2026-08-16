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

**Offener Restbestand:** In den Tools stecken noch rund 100 Einzelfarben für
spezielle Badges und Hinweisboxen, überwiegend in `heizkoerper.html`. Sie
sind optisch untergeordnet, aber beim nächsten größeren Redesign gehören sie
nach `theme.css` überführt. Vorsicht bei automatischen Ersetzungen: Im Code
stehen HTML-Entities wie `&#128293;` (Emoji), die wie Hex-Farben aussehen,
aber keine sind.

**Zwei Flächenfamilien:** `index/auslegung/kostenvergleich` nutzen einen
warmen Beigeton (`--bg`), `heizkoerper/bestandsaufnahme` einen kühlen
Blauton (`--bg-cool`). Historisch gewachsen. Um zu vereinheitlichen, in
`theme.css` einfach `--bg-cool: var(--bg);` setzen.

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
