# EBP-Rangliste mit SharePoint einrichten

Die Rangliste speichert pro Kürzel die beste Wirkung und zeigt die Top 10 im Titelbild und im Schlussbericht. Die Daten liegen in einer SharePoint-Liste in EBPs Microsoft 365. Ein Flow nimmt die Einträge des Spiels entgegen und antwortet mit der ganzen Liste; Rang und Top 10 rechnet das Spiel selbst aus. Deshalb ist der Flow kurz: drei Werte vorbereiten, speichern, Liste zurückgeben.

Aufwand: etwa 15 Minuten. Jeder Ausdruck unten steht in einem eigenen Kasten; auf GitHub kopiert das Symbol oben rechts im Kasten den Inhalt.

Solange im Spiel keine Adresse eingetragen ist, läuft es wie bisher, nur ohne Rangliste.

## Wo der Flow entsteht
Zwei Wege führen zum selben Flow:
- **Copilot Studio** (ohne Power-Automate-Premium): **Flows → Neuer Agentenflow**. Abgerechnet wird über die Copilot-Credits von EBP: 13 Credits pro 100 ausgeführte Aktionen. Dieser Flow führt pro Aufruf 8 bis 11 Aktionen aus, also gut 1 Credit oder rund 1 Rappen. Das Spiel lädt die Liste höchstens alle drei Minuten neu und nimmt sie beim Eintragen gleich mit; eine Partie kostet meist einen Aufruf. Das Gratis-Kontingent von Microsoft-365-Copilot-Lizenzen gilt für diesen Auslöser nicht. Vor dem Start mit der IT klären, ob das über die Copilot-Kapazität laufen darf.
- **Power Automate Premium**: [make.powerautomate.com](https://make.powerautomate.com) → **+ Erstellen → Sofortiger Cloud-Flow**. Keine Kosten pro Lauf.

Die Schritte im Designer sind in beiden Fällen gleich.

## 1. SharePoint-Liste anlegen
1. Eine SharePoint-Website öffnen, auf der Sie Listen anlegen dürfen.
2. **+ Neu → Liste → Leere Liste**, Name `Ring frei Rangliste`, **Erstellen**.
3. Spalte «Titel» umbenennen: Spaltenüberschrift **Titel → Spalteneinstellungen → Bearbeiten**, Name `Kürzel`, **Speichern**. Intern heisst sie weiterhin `Title`.
4. Mit **+ Spalte hinzufügen** vier Spalten anlegen. Namen genau so eintippen, ohne Umlaute; SharePoint legt dabei den internen Namen fest, mit dem der Flow arbeitet:

   | Name | Typ |
   |---|---|
   | `Wirkung` | Zahl, Dezimalstellen 0 |
   | `Runde` | Zahl, Dezimalstellen 0 |
   | `Ausbaustand` | Einzelne Textzeile |
   | `Datum` | Einzelne Textzeile |

   Falls Sie nach einer früheren Fassung dieser Anleitung schon `Stufe` und `Projekt` angelegt haben: stehen lassen, sie bleiben einfach leer.
5. Kontrolle: Zahnrad → **Listeneinstellungen** → eine Spalte anklicken. Die Adresszeile endet auf `Field=Wirkung`. Steht dort etwas anderes (z. B. `Wirkung0`), diesen Namen in den Ausdrücken unten verwenden.
6. Optional als Rangliste direkt in SharePoint: Spaltenüberschrift **Wirkung** → absteigend sortieren → **Ansicht speichern**.

Spielerinnen und Spieler brauchen keinen Zugriff auf die Liste. Der Flow schreibt mit Ihrem Konto.

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
- Bei Bedingungen auch `true` oder `false` als Ausdruck eingeben, nicht als Text.

### Abkürzung mit Copilot (optional)
Copilot im Designer kann das Gerüst anlegen. Die Ausdrücke müssen Sie trotzdem aus den Kästen unten einsetzen und Namen sowie Reihenfolge prüfen. Text für Copilot, Website und Liste ergänzen:
```
After the HTTP trigger, add these actions in this order: three Compose actions named Eingabe, Kuerzel and Wirkung; SharePoint Get items named Spieler; a Condition named Speichern; inside its True branch a Condition named Vorhanden with SharePoint Delete item in its True branch, and after Vorhanden a SharePoint Create item; after Speichern a SharePoint Get items named Alle, a Select named Liste and a Response. Use the SharePoint site <SITE-URL> and the list "Ring frei Rangliste".
```

## 3. Die Schritte im Flow
Der fertige Flow sieht so aus. Eingerückt heisst: innerhalb eines Zweigs der Bedingung darüber. Alles andere steht ganz aussen, untereinander.

```text
Beim Empfang einer HTTP-Anforderung
Eingabe ............ Verfassen
Kuerzel ............ Verfassen
Wirkung ............ Verfassen
Spieler ............ Elemente abrufen (Filterabfrage, Anzahl 1; keine ID)
Speichern .......... Bedingung
   Zweig Wahr:
      Vorhanden .... Bedingung
         Zweig Wahr:   Element löschen (hier, und nur hier, steht eine ID)
         Zweig Falsch: leer
      Element erstellen  (unter Vorhanden, noch im Zweig Wahr von Speichern)
   Zweig Falsch: leer
Alle ............... Elemente abrufen (wieder ganz aussen, unter Speichern)
Liste .............. Auswählen
Antwort
```

Wichtig: **Alle**, **Liste** und **Antwort** gehören unter die Bedingung Speichern, nicht in ihren Zweig. Sonst antwortet der Flow nur, wenn gerade etwas gespeichert wurde. Im Designer dafür das **+** unter dem ganzen Bedingungsblock nehmen, nicht das in einem Zweig.

### 3.1 Verfassen → Eingabe
**Verfassen** (Compose, Gruppe «Datenvorgang»), umbenennen in `Eingabe`. Eingabe:
```
json(if(empty(string(triggerBody())), '{}', string(triggerBody())))
```

### 3.2 Verfassen → Kuerzel
Ergibt das Kürzel in Grossbuchstaben, oder nichts, wenn es nicht aus 2 bis 4 Buchstaben A bis Z besteht. Der Ausdruck ist lang, weil er jeden Buchstaben einzeln entfernt und dann prüft, ob etwas übrig bleibt; am besten mit dem Kopier-Symbol übernehmen:
```
if(and(greaterOrEquals(length(string(outputs('Eingabe')?['k'])), 2), lessOrEquals(length(string(outputs('Eingabe')?['k'])), 4), empty(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(toUpper(string(outputs('Eingabe')?['k'])), 'A', ''), 'B', ''), 'C', ''), 'D', ''), 'E', ''), 'F', ''), 'G', ''), 'H', ''), 'I', ''), 'J', ''), 'K', ''), 'L', ''), 'M', ''), 'N', ''), 'O', ''), 'P', ''), 'Q', ''), 'R', ''), 'S', ''), 'T', ''), 'U', ''), 'V', ''), 'W', ''), 'X', ''), 'Y', ''), 'Z', ''))), toUpper(string(outputs('Eingabe')?['k'])), '')
```

### 3.3 Verfassen → Wirkung
Die Wirkung als ganze Zahl (0, wenn keine geschickt wurde):
```
int(coalesce(outputs('Eingabe')?['s'], 0))
```

### 3.4 Elemente abrufen → Spieler
**Elemente abrufen** (Get items, SharePoint, Mehrzahl), umbenennen in `Spieler`. Diese Aktion hat kein Feld für eine ID. Fragt sie nach einer ID, ist es «Element abrufen» (Get item, Einzahl): löschen und die Mehrzahl einfügen. Websiteadresse: Ihre Website; Listenname: `Ring frei Rangliste`. Unter «Erweiterte Parameter»:
- **Filterabfrage**, als Ausdruck:
  ```
  concat('Title eq ''', outputs('Kuerzel'), '''')
  ```
- **Anzahl** (Top Count): `1`

### 3.5 Bedingung → Speichern
**Bedingung** (Condition, Gruppe «Steuerelement»), umbenennen in `Speichern`. Links dieser Ausdruck, Vergleich **ist gleich**, rechts `true` als Ausdruck:
```
and(equals(outputs('Eingabe')?['op'], 'submit'), not(empty(outputs('Kuerzel'))), greater(outputs('Wirkung'), coalesce(first(body('Spieler')?['value'])?['Wirkung'], 0)), lessOrEquals(outputs('Wirkung'), 500000000), lessOrEquals(outputs('Wirkung'), add(mul(float(coalesce(outputs('Eingabe')?['t'], 0)), 250000), 500000)))
```
Das ist wahr, wenn ein gültiger Eintrag kommt, der besser ist als der gespeicherte (oder der erste für dieses Kürzel). Die Plausibilitätsgrenze: höchstens CHF 500 Mio. und höchstens CHF 250 000 pro Spielsekunde plus CHF 500 000. Ungültige Einträge werden still ignoriert.

### 3.6 Im Zweig «Wahr» von Speichern: Bedingung → Vorhanden
**Bedingung**, umbenennen in `Vorhanden`. Links dieser Ausdruck, **ist gleich**, rechts `false` als Ausdruck:
```
empty(body('Spieler')?['value'])
```
Im Zweig **Wahr** von Vorhanden: **Element löschen** (Delete item, SharePoint). Website und Liste wie oben, **ID**:
```
first(body('Spieler')?['value'])?['ID']
```
Der alte, schlechtere Eintrag wird gelöscht; gleich darunter entsteht der neue.

### 3.7 Im Zweig «Wahr» von Speichern, unter Vorhanden: Element erstellen
**Element erstellen** (Create item, SharePoint). Website und Liste wie oben, dann die Felder (fehlende unter «Erweiterte Parameter» → «Alle anzeigen»):

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

### 3.8 Elemente abrufen → Alle
Wieder unterhalb der Bedingung Speichern, auf oberster Ebene. **Elemente abrufen**, umbenennen in `Alle`. Website und Liste wie oben. Unter «Erweiterte Parameter»:
- **Sortieren nach** (Order By): `Wirkung desc`
- **Anzahl** (Top Count): `5000`

### 3.9 Auswählen → Liste, dann Antwort
**Auswählen** (Select, Gruppe «Datenvorgang»), umbenennen in `Liste`. **Von**:
```
body('Alle')?['value']
```
**Zuordnen**: auf **Textmodus** umschalten (Symbol rechts neben dem Feld) und diesen Ausdruck einfügen:
```
addProperty(addProperty(addProperty(addProperty(addProperty(json('{}'), 'k', item()?['Title']), 's', coalesce(item()?['Wirkung'], 0)), 'r', coalesce(item()?['Runde'], 0)), 'era', coalesce(item()?['Ausbaustand'], '')), 'd', coalesce(item()?['Datum'], ''))
```
Gibt es keinen Textmodus: fünf Zeilen eintragen, links der Schlüssel als Text, rechts der Wert als Ausdruck: `k` → `item()?['Title']`, `s` → `coalesce(item()?['Wirkung'], 0)`, `r` → `coalesce(item()?['Runde'], 0)`, `era` → `coalesce(item()?['Ausbaustand'], '')`, `d` → `coalesce(item()?['Datum'], '')`.

**Antwort** (Response, Gruppe «Anforderung»). Statuscode `200`; Header: Schlüssel `Content-Type`, Wert `application/json`; Text (Body):
```
addProperty(json('{"ok":true}'), 'rows', body('Liste'))
```
Keinen Header `Access-Control-Allow-Origin` ergänzen; Power Automate setzt ihn selbst.

## 4. Speichern und Adresse kopieren
1. **Speichern**. In Copilot Studio danach zusätzlich **Veröffentlichen**; erst die veröffentlichte Version beantwortet Aufrufe.
2. Den Auslöser öffnen und die **HTTP-URL** mit dem Kopier-Symbol vollständig kopieren. Sie sieht ungefähr so aus:
   `https://….environment.api.powerplatform.com:443/powerautomate/automations/direct/workflows/…/triggers/manual/paths/invoke?api-version=1&sp=…&sv=1.0&sig=…`
   Fehlt am Ende `sig=…`, steht «Wer kann den Flow auslösen?» nicht auf «Jeder».

## 5. Adresse ins Spiel eintragen
Das Spiel muss in der aktuellen Fassung im Repository liegen (mit Rotstift); ältere Fassungen kennen diese Rangliste nicht.
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
   Erwartet: `{ok: true, rows: [{k: 'TST', s: 1000, r: 1, era: 'Test', d: '…'}]}`, und in der SharePoint-Liste steht eine Zeile «TST».
5. Die Zeile «TST» in der Liste wieder löschen.

## Wenn es nicht klappt
Die Antwort in der Konsole verrät meist den Grund. Zusätzlich zeigt der **Ausführungsverlauf** des Flows jeden Aufruf; ein fehlgeschlagener Schritt ist rot markiert und nennt den Fehler.

- **401, «Unauthorized»:** «Wer kann den Flow auslösen?» steht nicht auf «Jeder», oder die Adresse ist ohne `sig=…` kopiert. Korrigieren, speichern (in Copilot Studio veröffentlichen), Adresse neu kopieren.
- **404, `WorkflowNotFound`:** Flow ausgeschaltet, nicht veröffentlicht oder alte Adresse.
- **502 oder `NoResponse`:** Der Flow ist unterwegs abgebrochen. Im Ausführungsverlauf den roten Schritt ansehen. Häufigste Ursachen: eine Aktion nicht exakt umbenannt (z. B. «Verfassen 2» statt «Wirkung»), ein interner Spaltenname weicht ab, ein Ausdruck unvollständig eingefügt.
- **429:** Zu viele Anfragen gleichzeitig. Kurz warten.
- **CORS-Fehler** (`Access-Control-Allow-Origin`): Im Register **Netzwerk** den Statuscode ansehen; meist steckt einer der Fehler oben dahinter. Bleibt ein echter CORS-Fehler, testweise in der **Antwort** den Header `Access-Control-Allow-Origin` = `*` ergänzen; meldet der Browser danach «multiple values», wieder entfernen.
- **Testeintrag erscheint nicht in der Liste:** Bedingung Speichern war falsch. Im Ausführungsverlauf die Bedingung öffnen; meist ist rechts `true` als Text statt als Ausdruck eingetragen.
- **Auslöser nicht verfügbar oder Flow gesperrt:** Eine Datenrichtlinie der IT verbietet ihn. Dann mit der IT klären.

## Betrieb
- **Einträge korrigieren oder löschen:** direkt in der Liste. Das Spiel zeigt die Änderung spätestens nach drei Minuten.
- **Neue Saison:** alle Zeilen der Liste löschen.
- **Flow ändern:** speichern (in Copilot Studio veröffentlichen). Die Adresse bleibt gleich, solange der Auslöser nicht gelöscht wird.
- **Neue Adresse bei Missbrauch:** Flow kopieren («Speichern unter»), Kopie einschalten, neue Adresse ins Spiel eintragen, alten Flow ausschalten. Microsoft beschreibt zusätzlich, wie man nur die Signatur erneuert («Regenerate the SAS key used in HTTP trigger flows»).
- **Stellvertretung:** In den Flow-Details unter **Besitzer** eine Kollegin oder einen Kollegen hinzufügen. Der Flow läuft mit Ihrer SharePoint-Verbindung; ist Ihr Konto nicht mehr aktiv, steht die Rangliste still.

## Was gespeichert wird
In der Liste: Kürzel (2 bis 4 Buchstaben), Wirkung in CHF, Runde, Ausbaustand der Gemeinde und Datum. Pro Kürzel eine Zeile mit dem besten Lauf. Keine Namen, keine E-Mail-Adressen, keine Cookies. Als «Erstellt von» erscheint das Konto der Person, der der Flow gehört. Die Antwort an das Spiel enthält nur diese fünf Werte pro Zeile.

Die Daten bleiben in EBPs Microsoft 365. Power Automate bewahrt jeden Aufruf 30 Tage im Ausführungsverlauf auf; darin können technische Angaben wie die IP-Adresse stehen. Einsehen können ihn nur die Besitzer des Flows.

## Grenzen
- Wer die Adresse kennt, kann theoretisch Fantasiewerte schicken. Der Flow prüft Kürzel und Plausibilität; den Rest erledigt die Moderation in der Liste.
- Eine Antwort dauert meist 2 bis 5 Sekunden. Das Spiel zeigt bis dahin die zuletzt geladene Liste und fragt höchstens alle drei Minuten neu nach; neue Einträge anderer erscheinen deshalb mit bis zu drei Minuten Verzögerung.
- Weil die Anfragen nacheinander laufen, warten bei vielen gleichzeitigen Aufrufen einige etwas länger. Ist die Warteschlange voll, zeigt das Spiel «Rangliste nicht erreichbar.» und läuft normal weiter. Die persönliche Bestleistung bleibt immer im Browser gespeichert.
- Die Antwort enthält alle Zeilen der Liste. Für einige hundert Kürzel ist das problemlos.
