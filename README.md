# Ring frei.

Ein Arcade-Spiel für EBPler. Drehen Sie den EBP-Ring um eine kleine Schweizer Gemeinde: die Öffnung lässt Aufträge herein, der Ring hält Gefahren ab, der graue Schild stoppt die schweren Brocken.

Alles steckt in einer einzigen Datei, `index.html` (Grafik, Musik, Sprecher). Kein Server, keine Datenbank, keine Cookies; die persönliche Bestleistung bleibt im eigenen Browser.

## Spielen
Öffnen Sie die GitHub-Pages-Adresse dieses Repositorys, z. B. `https://BENUTZERNAME.github.io/ring-frei/`.
Steuerung: Maus, Finger oder ← →. Gutachten: Leertaste oder Stadt antippen. Pause: Esc oder P. Vollbild: F.

## Veröffentlichen (einmalig)
1. Neues Repository anlegen, z. B. `ring-frei`.
2. Den Inhalt dieses Ordners hochladen (`index.html`, `.nojekyll`, `README.md`).
3. Settings → Pages → Build and deployment: «Deploy from a branch», Branch `main`, Ordner `/ (root)`, Save.
4. Nach ein bis zwei Minuten ist das Spiel unter der angezeigten Adresse erreichbar.

Aktualisieren: neue `index.html` hochladen (ersetzen) und committen.

## Auf einer SharePoint-Seite einbetten
1. SharePoint-Admin: unter Websiteeinstellungen → HTML-Feldsicherheit die Domain `BENUTZERNAME.github.io` erlauben.
2. Seite bearbeiten → Webpart «Einbetten» → diesen Code einfügen (Adresse ersetzen):

```html
<iframe src="https://BENUTZERNAME.github.io/ring-frei/" title="Ring frei." width="100%" height="720" style="border:0;display:block" allow="fullscreen; autoplay; clipboard-write" allowfullscreen></iframe>
```

Zusätzlich lohnt sich ein normaler Link «Im Vollbild öffnen» auf dieselbe Adresse, vor allem für Handys.

## Hinweise
- GitHub Pages ist bei Gratis-Konten öffentlich: Wer die Adresse kennt, kann spielen. Suchmaschinen werden per `noindex` ausgeschlossen. Mit GitHub Enterprise Cloud lässt sich die Seite auf Organisationsmitglieder beschränken.
- Die Teams- und Mail-Nachrichten im Schlussbericht enden automatisch mit «Selber versuchen: <Adresse dieser Seite>».
