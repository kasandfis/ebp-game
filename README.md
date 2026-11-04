# Ring frei.

Ein Arcade-Spiel für EBPler. Drehen Sie den EBP-Ring um eine kleine Schweizer Gemeinde: die Öffnung lässt Aufträge herein, der Ring hält Gefahren ab, der graue Schild stoppt die schweren Brocken.

Alles steckt in einer einzigen Datei, `index.html` (Grafik, Musik, Sprecher). Keine Cookies, kein Login; die persönliche Bestleistung bleibt im eigenen Browser. Optional gibt es eine gemeinsame EBP-Rangliste (Top 10 nach Kürzel), siehe unten.

Erstellt von KAF mit Unterstützung von Claude.

## Spielen
Spielen unter **https://kasandfis.github.io/ebp-game/**
Steuerung: Maus, Finger oder ← →. Rotstift: Klick, ↑ oder Knopf unten rechts. Gutachten: Leertaste, Rechtsklick oder Stadt antippen. Pause: Esc oder P. Vollbild: F.

## Veröffentlichen (einmalig)
1. Repository anlegen (hier: `kasandfis/ebp-game`).
2. Den Inhalt dieses Ordners hochladen (`index.html`, `.nojekyll`, `README.md`, Ordner `rangliste`).
3. Settings → Pages → Build and deployment: «Deploy from a branch», Branch `main`, Ordner `/ (root)`, Save.
4. Nach ein bis zwei Minuten ist das Spiel unter der angezeigten Adresse erreichbar.

Aktualisieren: neue `index.html` hochladen (ersetzen) und committen.

## Auf einer SharePoint-Seite einbetten
1. SharePoint-Admin: unter Websiteeinstellungen → HTML-Feldsicherheit die Domain `kasandfis.github.io` erlauben.
2. Seite bearbeiten → Webpart «Einbetten» → diesen Code einfügen (Adresse ersetzen):

```html
<iframe src="https://kasandfis.github.io/ebp-game/" title="Ring frei." width="100%" height="720" style="border:0;display:block" allow="fullscreen; autoplay; clipboard-write" allowfullscreen></iframe>
```

Zusätzlich lohnt sich ein normaler Link «Im Vollbild öffnen» auf dieselbe Adresse, vor allem für Handys.

## EBP-Rangliste (optional)
Wer am Ende «Ins Ranking eintragen» drückt, speichert die Wirkung unter seinem Kürzel. Die Top 10 erscheinen im Titelbild und im Schlussbericht; pro Kürzel zählt nur der beste Lauf.

Die Liste liegt in einer SharePoint-Liste im EBP-Microsoft-365; ein kurzer Flow (Agentenflow in Copilot Studio oder Power Automate Premium) nimmt die Einträge entgegen. Einrichtung in etwa fünfzehn Minuten: siehe [`rangliste/ANLEITUNG.md`](rangliste/ANLEITUNG.md). Danach die Flow-URL in `index.html` bei `boardUrl: ''` eintragen. Ohne URL läuft das Spiel einfach ohne Rangliste.

## Hinweise
- GitHub Pages ist bei Gratis-Konten öffentlich: Wer die Adresse kennt, kann spielen. Suchmaschinen werden per `noindex` ausgeschlossen. Mit GitHub Enterprise Cloud lässt sich die Seite auf Organisationsmitglieder beschränken.
- Die Teams- und Mail-Nachrichten im Schlussbericht enden automatisch mit «Selber versuchen: <Adresse dieser Seite>».
