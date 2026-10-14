# Hey KI – Einrichtung auf Deutsch

[English setup guide](GUIDE_EN.md) · [Projektübersicht](README.md)

Mit diesem Paket stellst du Fragen über einen Twitch-Chatbefehl:

```text
!ki Warum ist der Himmel blau?
```

Streamer.bot sendet die Frage an OpenAI. Die kurze Antwort erscheint im Chat,
wird von Speaker.bot vorgelesen oder beides. Die Eingabe erfolgt ausschließlich
per Chat. Die Antwortsprache wird in der Config gewählt: Deutsch oder Englisch.

## 1. Voraussetzungen und Dateien

Du brauchst Windows, Streamer.bot mit verbundenem Twitch-Konto und einen
OpenAI-API-Key mit verfügbarer API-Abrechnung bzw. Guthaben. Ein ChatGPT-Abo
ersetzt die API-Abrechnung nicht. Für Sprachausgabe brauchst du zusätzlich
Speaker.bot und eine eingerichtete Sprachengine. OBS nimmt die Ausgabe für den Stream auf.

Entpacke das ZIP dauerhaft, zum Beispiel nach `C:\StreamTools\HeyKI`.

| Datei | Zweck |
|---|---|
| HeyKI.cs | Code für die Streamer.bot-Action |
| config.json | Konfiguration ohne API-Key |
| ANLEITUNG.md | Diese Anleitung |
| GUIDE_EN.md | Englische Anleitung |
| README.md / LICENSE | Übersicht und MIT-Lizenz |

Das Paket enthält keinen Streamer.bot-Importcode. Die Action wird manuell eingerichtet.

## 2. OpenAI-Key hinterlegen

1. Auf [OpenAI API Keys](https://platform.openai.com/api-keys) einen Key erstellen.
   Im zugehörigen API-Projekt Abrechnung und Modellzugriff prüfen.
2. Im Windows-Startmenü **Umgebungsvariablen für dieses Konto bearbeiten** suchen.
3. Unter **Benutzervariablen** auf **Neu** klicken:
   Name `OPENAI_API_KEY`, Wert = dein API-Key.
4. Speichern und Streamer.bot vollständig beenden und neu starten.

Ein vorhandener funktionierender Key kann bleiben. Keys nicht im Chat,
in Screenshots oder im veröffentlichten Paket teilen.

## 3. Chatbefehl erstellen

In Streamer.bot unter **Commands** einen Befehl erstellen:

| Einstellung | Wert |
|---|---|
| Name | Hey KI |
| Enabled | An |
| Mode | Basic |
| Location | Start |
| Commands | !ki |
| Quelle | Twitch Chat Message |

Unter **Permissions** zunächst **Allow** und die Gruppe **Broadcaster** auswählen.
Eine leere Allow-Liste erlaubt allen den Zugriff. Moderatoren kannst du später
hinzufügen. Für weitere Nutzer gelten dieselben API-Kosten und Ausgaberegeln.

Zum Ersttest den Twitch-Webchat verwenden. Bei Eingaben im Streamer.bot-eigenen
Chat kann **Ignore Internal Messages** den Befehl blockieren.

## 4. Action und Sub-Actions einrichten

1. Unter **Actions** eine Action **Hey KI** anlegen.
2. Eine eigene **nicht blockierende Queue** zuweisen, zum Beispiel **HeyKI**.
3. Als Trigger **Core > Commands > Command Triggered** hinzufügen und genau
   deinen Command **!ki** auswählen.
4. Die folgenden zwei Sub-Actions in dieser Reihenfolge hinzufügen:

| Reihenfolge | Sub-Action | Einstellung |
|---|---|---|
| 1 | Core > Arguments > Set Argument | Name: `aiConfigPath`; Wert: vollständiger Pfad zur config.json |
| 2 | Core > C# > Execute C# Code | Gesamten Inhalt von HeyKI.cs einfügen und Compile klicken |

Beispiel für `aiConfigPath`:

```text
C:\StreamTools\HeyKI\config.json
```

Der Dateiname muss enthalten sein. Leerzeichen im Pfad sind erlaubt.
Ohne äußere Anführungszeichen eingeben; kopierte äußere doppelte
Anführungszeichen werden vom Code ebenfalls entfernt.

Falls beim Kompilieren Referenzen fehlen: Im Referenzen-Bereich des C#-Editors
`System.Net.Http.dll` und `Newtonsoft.Json.dll` aus der Streamer.bot-Installation
ergänzen. Erst fortfahren, wenn die Kompilation erfolgreich ist.

**Kein Argument aiQuestion anlegen. rawInput nicht selbst setzen.**
Der Command-Trigger liefert die Frage nach `!ki` automatisch als `rawInput`.

## 5. Zuerst nur den Chat testen

Die mitgelieferte Config enthält `"chat": true` und `"speech": false`.
Schreibe mit deinem zugelassenen Konto im Twitch-Webchat:

```text
!ki Warum ist der Himmel blau?
```

Erwartet: Eine kurze Antwort mit dem Präfix **KI:**. Ein leeres `!ki` erzeugt
keine API-Anfrage. Zwischen Versuchen mindestens acht Sekunden warten.
Weitere Aufrufe während einer laufenden Anfrage werden verworfen.

## 6. Optional: Speaker.bot und OBS

Eine bereits funktionierende Einrichtung kannst du weiterverwenden.

1. [Speaker.bot herunterladen](https://speaker.bot/get-started/installation),
   entpacken und starten.
2. Unter **Settings > Speech Engines** eine Engine einrichten:
   **SAPI5** für eine lokal installierte Windows-Stimme oder **ElevenLabs**
   für deine ElevenLabs-Stimme.
3. Bei ElevenLabs den Key in Speaker.bot hinterlegen. Als Startkonfiguration:
   **Text to Speech: Access**, **Voices: Read**, **Models: Read**.
   Verlangt eine Kontoabfrage zusätzlich Zugriff, gezielt **User: Read** ergänzen.
   Zusätzliche Rechte hängen von der verwendeten Speaker.bot-Version ab.
4. Unter **Settings > Voice Aliases** den Alias **KI** anlegen. Engine,
   Stimme und Ausgabegerät wählen. Bei ElevenLabs eignet sich **Flash v2.5**
   für kurze Live-Antworten, sofern angeboten; alternativ **Multilingual v2**.
5. Mit **Test Speak** prüfen, ob die Stimme lokal hörbar ist.
   Unter **Settings > General** muss TTS **Enabled** sein.
6. Unter **Settings > WebSocket Server** den Server aktivieren.
   In Streamer.bot unter **Integrations > Speaker.bot** mit `127.0.0.1`
   und demselben Port verbinden. **Auto Connect** und **Auto Reconnect** aktivieren.
7. In der Config `speech` auf `true` setzen. `voiceAlias` muss exakt
   dem Alias in Speaker.bot entsprechen. Beide Programme laufen lassen.
8. In OBS das Ausgabegerät über Desktop-Audio oder Speaker.bot über eine
   Anwendungs-Audioaufnahme erfassen. Eine kurze Aufnahme prüfen:
   Die Stimme soll hörbar sein und nur einmal aufgenommen werden.

Der ElevenLabs-Key gehört in Speaker.bot. Der OpenAI-Key bleibt in der
Windows-Umgebungsvariable. Das ElevenLabs-Sprachmodell wird in Speaker.bot gewählt.

## 7. Konfiguration

Ändere die vorhandene `config.json` mit einem Texteditor. Änderungen werden bei
der nächsten Anfrage gelesen. JSON benötigt doppelte Anführungszeichen,
Kommas zwischen Einträgen und `true`/`false` ohne Anführungszeichen.

| Ausgabe | chat | speech |
|---|---|---|
| Nur Chat | true | false |
| Nur Stimme | false | true |
| Beides | true | true |
| Deaktiviert | false | false |

| Einstellung | Standard | Bedeutung / erlaubter Bereich |
|---|---|---|
| language | de | Antwortsprache: `de` für Deutsch, `en` für Englisch |
| model | gpt-5.4-mini | OpenAI-Modell für Textantworten |
| reasoningEffort | none | Muss zum OpenAI-Modell passen; leere Zeichenfolge lässt das API-Feld weg |
| useBotAccount | true | Bot-Konto nutzen, Broadcaster als Fallback |
| voiceAlias | KI | Name des Speaker.bot-Alias |
| maxChars | 420 | Antwortlimit, 80–450; längere Texte werden lokal gekürzt |
| maxOutputTokens | 300 | API-Ausgabelimit, 64–2000 |
| cooldownSeconds | 8 | Mindestabstand ab Anfragestart, mindestens 0 |
| timeoutSeconds | 20 | Anfragezeitlimit, 5–60 Sekunden |

### Antwortsprache wählen

Setze in der vorhandenen Config `"language": "de"` für Deutsch oder
`"language": "en"` für Englisch. Beispiel für englische Antworten:

```json
{
  "language": "en",
  "model": "gpt-5.4-mini",
  "reasoningEffort": "none",
  "chat": true,
  "speech": false,
  "useBotAccount": true,
  "voiceAlias": "KI",
  "maxChars": 420,
  "maxOutputTokens": 300,
  "cooldownSeconds": 8,
  "timeoutSeconds": 20
}
```

Die Auswahl gilt ab der nächsten Anfrage für Chat und vorgelesenen Antworttext,
unabhängig von der Sprache der Frage. Ohne `language` bleibt Deutsch der Standard.
Andere Werte als `de` und `en` werden mit einer Fehlermeldung abgelehnt.
Programm- und Logmeldungen bleiben auf Deutsch. Die Config wählt keine andere
Speaker.bot-Stimme; verwende dort eine für die jeweilige Sprache geeignete Stimme
und Engine. Für Deutsch und Englisch eignet sich beispielsweise Flash v2.5.

Jede Frage ist eigenständig, ohne Gesprächsverlauf oder Live-Websuche.
Den Stil kannst du im `BuildInstructions`-Text in `HeyKI.cs` anpassen.
Nach Änderungen am C#-Code erneut kompilieren.

## 8. Bestehende Installation aktualisieren

1. Deine angepasste Config sichern und behalten, besonders Chat-/Sprachoptionen.
   Bei Bedarf `"language": "de"` oder `"language": "en"` ergänzen.
2. Den bisherigen C#-Code vollständig ersetzen und kompilieren.
3. Alte Voice-Control-Trigger entfernen und den früheren Sprachbefehl deaktivieren.
   Wenn ungenutzt: Voice Control stoppen und Auto Start Listen ausschalten.
4. Alte Sub-Actions für `aiQuestion` und manuell gesetztes `rawInput` entfernen.
5. Nur den `!ki`-Command-Trigger und die zwei Sub-Actions aus Abschnitt 4 verwenden.
   Ein alter Config-Eintrag `wakePhrase` wird ignoriert und kann entfernt werden.

## 9. Fehler finden

Im Ordner von `Streamer.bot.exe` den Unterordner **logs** öffnen.
In der neuesten Logdatei nach **HeyKI:** suchen.

| Problem / Meldung | Nächster Schritt |
|---|---|
| Kein Eintrag in Action History | Command aktiv? Quelle, Berechtigungen und ausgewählten Trigger prüfen |
| rawInput fehlt | Über den Chat-Command aufrufen; manuelles Starten liefert keine Frage |
| aiConfigPath fehlt | Erste Sub-Action aus Abschnitt 4 ergänzen |
| Config-Datei nicht gefunden / Pfad zeigt auf Ordner | Tatsächlichen vollständigen Dateipfad inklusive config.json verwenden |
| Ungueltiges JSON | Config auf Syntax und Datentypen prüfen |
| Ungueltige Sprache | language auf de oder en setzen |
| OPENAI_API_KEY fehlt | Benutzervariable prüfen und Streamer.bot neu starten |
| HTTP 401 | OpenAI-Key prüfen |
| HTTP 429 | Guthaben bzw. Rate-Limit prüfen |
| HTTP 400/404 | Modellzugriff, Modellname und reasoningEffort prüfen |
| Zeitlimit erreicht | Verbindung prüfen; bei Bedarf timeoutSeconds bis maximal 60 erhöhen |
| Antwort nicht vollstaendig | maxOutputTokens erhöhen; nicht abgeschlossene Antworten werden nicht ausgegeben |
| Cooldown / vorherige Anfrage läuft | Warten und später erneut senden |
| Chat geht, Stimme fehlt | Test Speak, Verbindung, Alias, TTS Enabled und Ausgabegerät prüfen |
| Stimme lokal, nicht in OBS | Audioquelle, Mixer und Aufnahme prüfen |

Für einen isolierten manuellen Test vorübergehend vor dem C#-Schritt ein Argument
`rawInput` mit einer festen Frage setzen, die Action starten und
**diese Test-Sub-Action danach wieder entfernen**.

Ein Logeintrag zur verarbeiteten Ausgabe bestätigt keine tatsächliche
Chat-Zustellung oder hörbare Wiedergabe.

## 10. Daten, Kosten und Lizenz

Die Frage wird an OpenAI übertragen. Bei ElevenLabs-Sprachausgabe wird der
Antworttext zusätzlich an ElevenLabs übertragen. API-Nutzung kann Kosten auslösen.
Antworten können Fehler enthalten. Das Projekt steht unter der MIT-Lizenz;
Haftungsausschluss und Bedingungen stehen in [README.md](README.md) und [LICENSE](LICENSE).

Stand: 14.10.2026. C# 7.3 / .NET 5 und 35 lokale Checks mit Testdouble bestanden.
Kein End-to-End-Test mit deinem Twitch-Konto, API-Key, Speaker.bot oder OBS.
Menübezeichnungen können je nach Programmversion abweichen.

### Offizielle Dokumentation

- [Streamer.bot Commands](https://docs.streamer.bot/guide/commands)
- [Command Triggered und rawInput](https://docs.streamer.bot/api/triggers/core/commands/command-triggered)
- [Speaker.bot-Anbindung](https://docs.streamer.bot/guide/integrations/speakerbot)
- [Speaker.bot Quick Start](https://speaker.bot/get-started/quick-start)
- [ElevenLabs-Modelle](https://elevenlabs.io/docs/overview/models)

