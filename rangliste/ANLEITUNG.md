# EBP-Rangliste mit SharePoint einrichten

Die Rangliste speichert pro Kürzel die beste Wirkung und zeigt die Top 10 im Titelbild und im Schlussbericht. Die Daten liegen in einer SharePoint-Liste in EBPs Microsoft 365. Ein Flow nimmt die Einträge des Spiels entgegen. Er ist eine gerade Linie mit einer einzigen Bedingung am Schluss: Liste lesen, dem Spiel antworten, dann einen besseren Eintrag speichern. Rang und Top 10 rechnet das Spiel selbst aus.

Aufwand: etwa 15 Minuten. Jeder Ausdruck unten steht in einem eigenen Kasten; auf GitHub kopiert das Symbol oben rechts im Kasten den Inhalt.

Solange im Spiel keine Adresse eingetragen ist, läuft es wie bisher, nur ohne Rangliste.

## Wo der Flow entsteht
Zwei Wege führen zum selben Flow:
- **Copilot Studio** (ohne Power-Automate-Premium): **Flows → Neuer Agentenflow**. Abgerechnet wird über die Copilot-Credits von EBP: 13 Credits pro 100 ausgeführte Aktionen. Dieser Flow führt pro Aufruf 9 bis 10 Aktionen aus, also gut 1 Credit oder rund 1 Rappen. Das Spiel lädt die Liste höchstens alle drei Minuten neu und nimmt sie beim Eintragen gleich mit; eine Partie kostet meist einen Aufruf. Das Gratis-Kontingent von Microsoft-365-Copilot-Lizenzen gilt für diesen Auslöser nicht. Vor dem Start mit der IT klären, ob das über die Copilot-Kapazität laufen darf.
- **Power Automate Premium**: [make.powerautomate.com](https://make.powerautomate.com) → **+ Erstellen → Sofortiger Cloud-Flow**. Keine Kosten pro Lauf.

Die Schritte im Designer sind in beiden Fällen gleich.

## 1. SharePoint-Liste anlegen
1. Eine SharePoint-Website öffnen, auf der Sie Listen anlegen dürfen.
2. **+ Neu → Liste → Leere Liste**, Name `Ring frei Rangliste`, **Erstellen**.
3. Spalte «Titel» umbenennen: Spaltenüberschrift **Titel → Spalteneinstellungen → Bearbeiten**, Name `Kürzel`, **Speichern**. Intern heisst sie weiterhin `Title`. «Eindeutige Werte erzwingen» bleibt **aus**: Pro Kürzel kann es mehrere Zeilen geben (siehe «Betrieb»).
4. Mit **+ Spalte hinzufügen** vier Spalten anlegen. Namen genau so eintippen, ohne Umlaute; SharePoint legt dabei den internen Namen fest, mit dem der Flow arbeitet:

   | Name | Typ |
   |---|---|
   | `Wirkung` | Zahl, Dezimalstellen 0 |
   | `Runde` | Zahl, Dezimalstellen 0 |
   | `Ausbaustand` | Einzelne Textzeile |
   | `Datum` | Einzelne Textzeile |

   Falls Sie schon `Stufe` und `Projekt` angelegt haben: stehen lassen, sie bleiben einfach leer.
5. Kontrolle: Zahnrad → **Listeneinstellungen** → eine Spalte anklicken. Die Adresszeile endet auf `Field=Wirkung`. Steht dort etwas anderes (z. B. `Wirkung0`), diesen Namen in den Ausdrücken unten verwenden.
6. Optional als Rangliste direkt in SharePoint: Spaltenüberschrift **Wirkung** → absteigend sortieren → **Ansicht speichern**.

Spielerinnen und Spieler brauchen keinen Zugriff auf die Liste. Der Flow schreibt mit Ihrem Konto.

**Websiteadresse** heisst unten immer die Adresse der SharePoint-Website, auf der die Liste liegt: der vordere Teil des Listen-Links bis und mit dem Websitenamen, z. B. `https://firma.sharepoint.com/sites/Freizeit`. Steht sie nicht in der Auswahl, **«Benutzerdefinierten Wert eingeben»** wählen und einfügen.

## 2. Flow anlegen und Auslöser einstellen
1. Neuen Flow anlegen (siehe oben), Name `Ring frei Rangliste`. Als Auslöser **Beim Empfang einer HTTP-Anforderung** (englisch «When an HTTP request is received») wählen.
2. Im Auslöser:
   - **Wer kann den Flow auslösen?**: **Jeder** (Anyone). Die Spielenden sind bei Microsoft nicht angemeldet.
   - **JSON-Schema für Anforderungstext**: leer lassen.
   - **Methode** (unter «Erweiterte Parameter», falls angeboten): `POST`.
3. Im Auslöser das Register **Einstellungen**: **Parallelitätssteuerung** einschalten, **Grad der Parallelität** `1`. So läuft eine Anfrage nach der anderen. Lässt sich danach nicht mehr ausschalten, nur verstellen.

### So arbeiten Sie im Designer
- Jede Aktion über **+** einfügen und **sofort umbenennen** (auf den Titel klicken). Die Ausdrücke greifen über diese Namen aufeinander zu. Namen genau übernehmen, auch Gross- und Kleinschreibung.
- Ausdruck einfügen: ins Feld klicken → **fx** (oder `/` → «Ausdruck einfügen») → Kasteninhalt einfügen → **Hinzufügen**. Im Feld steht dann nur der Ausdruck, sonst nichts.
- SharePoint-Aktionen findet man im Suchfeld (`Get items`, `Create item`) oder über den Connector **SharePoint** → **Mehr anzeigen** (See more).

### Abkürzung mit Copilot (optional)
Copilot im Designer kann das Gerüst anlegen. Die Ausdrücke müssen Sie trotzdem aus den Kästen unten einsetzen und Namen sowie Reihenfolge prüfen. Text für Copilot, Websiteadresse ergänzen:
```
After the HTTP trigger, add these actions one after another: three Compose actions named Eingabe, Kuerzel and Wirkung; SharePoint Get items named Spieler; SharePoint Get items named Alle; a Select named Liste; a Response; and at the very end a Condition named Speichern with SharePoint Create item in its True branch. Use the SharePoint site <SITE-URL> and the list "Ring frei Rangliste".
```

## 3. Die Schritte im Flow
Alles steht untereinander, in einer geraden Linie. Die einzige Bedingung kommt ganz am Schluss; nach ihr folgt nichts mehr.

```text
Beim Empfang einer HTTP-Anforderung
Eingabe ............ Verfassen
Kuerzel ............ Verfassen
Wirkung ............ Verfassen
Spieler ............ Elemente abrufen  (Filterabfrage, Sortieren, Anzahl 1; keine ID)
Alle ............... Elemente abrufen  (Sortieren, Anzahl 5000)
Liste .............. Auswählen
Antwort ............ Antwort an das Spiel
Speichern .......... Bedingung (If / Else)
   If (Wahr):    Element erstellen
   Else (Falsch): leer
```

Die Antwort kommt vor dem Speichern. Das ist Absicht: Der Flow antwortet dem Spiel und läuft danach weiter. Das Spiel ergänzt den eigenen Eintrag selbst.

### 3.1 Verfassen → Eingabe
**Verfassen** (Compose, Gruppe «Datenvorgang»), umbenennen in `Eingabe`. Eingabe:
```
json(if(empty(string(triggerBody())), '{}', string(triggerBody())))
```

### 3.2 Verfassen → Kuerzel
Ergibt das Kürzel in Grossbuchstaben, oder nichts, wenn es nicht aus 2 bis 4 Buchstaben A bis Z besteht. Der Ausdruck ist lang, weil er jeden Buchstaben einzeln entfernt und dann prüft, ob etwas übrig bleibt:
```
if(and(greaterOrEquals(length(string(outputs('Eingabe')?['k'])), 2), lessOrEquals(length(string(outputs('Eingabe')?['k'])), 4), empty(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(toUpper(string(outputs('Eingabe')?['k'])), 'A', ''), 'B', ''), 'C', ''), 'D', ''), 'E', ''), 'F', ''), 'G', ''), 'H', ''), 'I', ''), 'J', ''), 'K', ''), 'L', ''), 'M', ''), 'N', ''), 'O', ''), 'P', ''), 'Q', ''), 'R', ''), 'S', ''), 'T', ''), 'U', ''), 'V', ''), 'W', ''), 'X', ''), 'Y', ''), 'Z', ''))), toUpper(string(outputs('Eingabe')?['k'])), '')
```

### 3.3 Verfassen → Wirkung
Die Wirkung als ganze Zahl (0, wenn keine geschickt wurde):
```
int(coalesce(outputs('Eingabe')?['s'], 0))
```

### 3.4 Elemente abrufen → Spieler
**Elemente abrufen** (Get items, SharePoint, Mehrzahl), umbenennen in `Spieler`. Diese Aktion hat kein Feld für eine ID; fragt sie nach einer ID, ist es «Element abrufen» (Einzahl): löschen und die Mehrzahl einfügen.
Websiteadresse: Ihre Website; Listenname: `Ring frei Rangliste`. Unter «Erweiterte Parameter»:
- **Filterabfrage**, als Ausdruck:
  ```
  concat('Title eq ''', outputs('Kuerzel'), '''')
  ```
- **Sortieren nach** (Order By): `Wirkung desc`
- **Anzahl** (Top Count): `1`

So liefert «Spieler» den bisher besten Eintrag dieses Kürzels.

### 3.5 Elemente abrufen → Alle
**Elemente abrufen**, umbenennen in `Alle`. Website und Liste wie oben. Unter «Erweiterte Parameter»:
- **Sortieren nach**: `Wirkung desc`
- **Anzahl**: `5000`

### 3.6 Auswählen → Liste
**Auswählen** (Select, Gruppe «Datenvorgang»), umbenennen in `Liste`. **Von**:
```
body('Alle')?['value']
```
**Zuordnen**: auf **Textmodus** umschalten (Symbol rechts neben dem Feld) und diesen Ausdruck einfügen:
```
addProperty(addProperty(addProperty(addProperty(addProperty(json('{}'), 'k', item()?['Title']), 's', coalesce(item()?['Wirkung'], 0)), 'r', coalesce(item()?['Runde'], 0)), 'era', coalesce(item()?['Ausbaustand'], '')), 'd', coalesce(item()?['Datum'], ''))
```
Gibt es keinen Textmodus: fünf Zeilen eintragen, links der Schlüssel als Text, rechts der Wert als Ausdruck: `k` → `item()?['Title']`, `s` → `coalesce(item()?['Wirkung'], 0)`, `r` → `coalesce(item()?['Runde'], 0)`, `era` → `coalesce(item()?['Ausbaustand'], '')`, `d` → `coalesce(item()?['Datum'], '')`.

### 3.7 Antwort
**Antwort** (Response, Gruppe «Anforderung»). Statuscode `200`; Header: Schlüssel `Content-Type`, Wert `application/json`; Text (Body):
```
addProperty(json('{"ok":true}'), 'rows', body('Liste'))
```
Keinen Header `Access-Control-Allow-Origin` ergänzen; Power Automate setzt ihn selbst.

### 3.8 Bedingung → Speichern, ganz am Schluss
**Bedingung** (Condition, Gruppe «Steuerelement»), umbenennen in `Speichern`. Links dieser Ausdruck, Vergleich **ist gleich**, rechts `true` (als Ausdruck über **fx**, nicht als Text):
```
and(equals(outputs('Eingabe')?['op'], 'submit'), not(empty(outputs('Kuerzel'))), greater(outputs('Wirkung'), coalesce(first(body('Spieler')?['value'])?['Wirkung'], 0)), lessOrEquals(outputs('Wirkung'), 500000000), lessOrEquals(outputs('Wirkung'), add(mul(float(coalesce(outputs('Eingabe')?['t'], 0)), 250000), 500000)))
```
Wahr wird es, wenn ein gültiger Eintrag kommt, der besser ist als der beste bisherige dieses Kürzels (oder der erste). Grenzen: höchstens CHF 500 Mio. und höchstens CHF 250 000 pro Spielsekunde plus CHF 500 000. Ungültige Einträge werden still ignoriert.

Im Zweig **If / Wahr**: **Element erstellen** (Create item, SharePoint). Website und Liste wie oben, dann die Felder (fehlende unter «Erweiterte Parameter» → «Alle anzeigen»):

Kürzel:
```
outputs('Kuerzel')
```
Wirkung:
```
outputs('Wirkung')
```
Runde:
```
outputs('Eingabe')?['r']
```
Ausbaustand:
```
take(string(outputs('Eingabe')?['era']), 40)
```
Datum:
```
convertFromUtc(utcNow(), 'W. Europe Standard Time', 'yyyy-MM-dd')
```

Der Zweig **Else / Falsch** bleibt leer. Danach kommt nichts mehr.

## 4. Speichern und Adresse kopieren
1. **Speichern**. In Copilot Studio danach zusätzlich **Veröffentlichen**; erst die veröffentlichte Version beantwortet Aufrufe.
2. Den Auslöser öffnen und die **HTTP-URL** mit dem Kopier-Symbol vollständig kopieren. Sie sieht ungefähr so aus:
   `https://….environment.api.powerplatform.com:443/powerautomate/automations/direct/workflows/…/triggers/manual/paths/invoke?api-version=1&sp=…&sv=1.0&sig=…`
   Fehlt am Ende `sig=…`, steht «Wer kann den Flow auslösen?» nicht auf «Jeder».

## 5. Adresse ins Spiel eintragen
1. Im Repository `index.html` öffnen → Stift-Symbol (Bearbeiten).
2. Mit Ctrl+F (Mac: Cmd+F) nach `boardUrl: ''` suchen.
3. Die URL zwischen die Hochkommas setzen: `boardUrl: 'https://…&sig=…',`
4. **Commit changes**. Nach ein bis zwei Minuten ist die Rangliste online.

Die Adresse enthält mit `sig=` eine geheime Signatur: Wer sie kennt, kann Einträge schicken. Weil das Spiel öffentlich auf GitHub Pages liegt, lässt sie sich im Quelltext finden. Dagegen helfen die Prüfung im Flow, die Moderation in der Liste und notfalls eine neue Adresse (siehe «Betrieb»).

## 6. Testen
1. Die Spielseite öffnen (`https://kasandfis.github.io/ebp-game/`) und die Entwicklerwerkzeuge: Edge und Chrome mit F12 (Mac: Cmd+Option+J), Register **Konsole**. Beim ersten Einfügen verlangt der Browser unter Umständen eine Bestätigung; der Hinweis in der Konsole nennt den Text dafür.
2. Adresse festlegen (eigene URL einsetzen), Enter:
   ```js
   const U = 'https://…&sig=…';
   ```
3. Liste abfragen:
   ```js
   fetch(U, {method: 'POST', headers: {'Content-Type': 'text/plain;charset=utf-8'}, body: JSON.stringify({op: 'top'})}).then(r => r.json()).then(console.log)
   ```
   Bei leerer Liste erscheint nach einigen Sekunden `{ok: true, rows: Array(0)}`.
4. Testeintrag:
   ```js
   fetch(U, {method: 'POST', headers: {'Content-Type': 'text/plain;charset=utf-8'}, body: JSON.stringify({op: 'submit', k: 'TST', s: 1000, r: 1, t: 60, era: 'Test'})}).then(r => r.json()).then(console.log)
   ```
   Die Antwort zeigt die Liste noch ohne «TST», weil der Flow vor dem Speichern antwortet. In der SharePoint-Liste steht danach eine Zeile «TST». Schritt 3 noch einmal ausführen: jetzt ist «TST» dabei.
5. Die Zeile «TST» in der Liste wieder löschen.

## Wenn es nicht klappt
Die Antwort in der Konsole verrät meist den Grund. Zusätzlich zeigt der **Ausführungsverlauf** des Flows jeden Aufruf; ein fehlgeschlagener Schritt ist rot markiert und nennt den Fehler.

- **401, «Unauthorized»:** «Wer kann den Flow auslösen?» steht nicht auf «Jeder», oder die Adresse ist ohne `sig=…` kopiert. Korrigieren, speichern (in Copilot Studio veröffentlichen), Adresse neu kopieren.
- **404, `WorkflowNotFound`:** Flow ausgeschaltet, nicht veröffentlicht oder alte Adresse.
- **502 oder `NoResponse`:** Der Flow ist vor der Antwort abgebrochen. Im Ausführungsverlauf den roten Schritt ansehen. Häufigste Ursachen: eine Aktion nicht exakt umbenannt (z. B. «Verfassen 2» statt «Wirkung»), ein interner Spaltenname weicht ab, ein Ausdruck unvollständig eingefügt.
- **429:** Zu viele Anfragen gleichzeitig. Kurz warten.
- **CORS-Fehler** (`Access-Control-Allow-Origin`): Im Register **Netzwerk** den Statuscode ansehen; meist steckt einer der Fehler oben dahinter. Bleibt ein echter CORS-Fehler, testweise in der **Antwort** den Header `Access-Control-Allow-Origin` = `*` ergänzen; meldet der Browser danach «multiple values», wieder entfernen.
- **Testeintrag erscheint auch beim zweiten Abfragen nicht:** Die Bedingung Speichern war falsch. Im Ausführungsverlauf öffnen; meist ist rechts `true` als Text statt als Ausdruck eingetragen.
- **Auslöser nicht verfügbar oder Flow gesperrt:** Eine Datenrichtlinie der IT verbietet ihn. Dann mit der IT klären.
- **Eine Funktion ist «unbekannt»:** Den Namen notieren und melden; es gibt fast immer einen Ersatzausdruck.

## Betrieb
- **Mehrere Zeilen pro Kürzel:** Jede Verbesserung legt eine neue Zeile an; die alte bleibt stehen. Das Spiel zeigt immer nur die beste Zeile pro Kürzel. Alte Zeilen dürfen Sie jederzeit löschen, in SharePoint z. B. nach Kürzel gruppiert.
- **Einträge korrigieren oder löschen:** direkt in der Liste. Das Spiel zeigt die Änderung spätestens nach drei Minuten.
- **Neue Saison:** alle Zeilen der Liste löschen.
- **Flow ändern:** speichern (in Copilot Studio veröffentlichen). Die Adresse bleibt gleich, solange der Auslöser nicht gelöscht wird.
- **Neue Adresse bei Missbrauch:** Flow kopieren («Speichern unter»), Kopie einschalten, neue Adresse ins Spiel eintragen, alten Flow ausschalten. Microsoft beschreibt zusätzlich, wie man nur die Signatur erneuert («Regenerate the SAS key used in HTTP trigger flows»).
- **Stellvertretung:** In den Flow-Details unter **Besitzer** eine Kollegin oder einen Kollegen hinzufügen. Der Flow läuft mit Ihrer SharePoint-Verbindung; ist Ihr Konto nicht mehr aktiv, steht die Rangliste still.

## Was gespeichert wird
In der Liste: Kürzel (2 bis 4 Buchstaben), Wirkung in CHF, Runde, Ausbaustand der Gemeinde und Datum, eine Zeile pro Verbesserung. Keine Namen, keine E-Mail-Adressen, keine Cookies. Als «Erstellt von» erscheint das Konto der Person, der der Flow gehört. Die Antwort an das Spiel enthält nur diese fünf Werte pro Zeile.

Die Daten bleiben in EBPs Microsoft 365. Power Automate bewahrt jeden Aufruf 30 Tage im Ausführungsverlauf auf; darin können technische Angaben wie die IP-Adresse stehen. Einsehen können ihn nur die Besitzer des Flows.

## Grenzen
- Wer die Adresse kennt, kann theoretisch Fantasiewerte schicken. Der Flow prüft Kürzel und Plausibilität; den Rest erledigt die Moderation in der Liste.
- Eine Antwort dauert meist 2 bis 5 Sekunden. Das Spiel zeigt bis dahin die zuletzt geladene Liste und fragt höchstens alle drei Minuten neu nach; neue Einträge anderer erscheinen deshalb mit bis zu drei Minuten Verzögerung.
- Weil die Anfragen nacheinander laufen, warten bei vielen gleichzeitigen Aufrufen einige etwas länger. Ist die Warteschlange voll, zeigt das Spiel «Rangliste nicht erreichbar.» und läuft normal weiter. Die persönliche Bestleistung bleibt immer im Browser gespeichert.
- Die Antwort enthält alle Zeilen der Liste (höchstens 5000). Für einige hundert Einträge ist das problemlos.
