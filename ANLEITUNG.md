# Hey KI: nur Chat-Command

Im Twitch-Chat schreiben: **!ki Warum ist der Himmel blau?**
Kurze deutsche Antwort im Chat, per Speaker.bot oder in beiden.

## Bestehende Action umstellen

1. Deine funktionierende `config.json` behalten. Die beiliegende Config ist eine
   Vorlage: Chat an, Stimme aus. Eine angepasste Config nicht blind ersetzen.
   Den alten Eintrag `wakePhrase` kannst du entfernen; er wird nicht mehr genutzt.
2. Bei **Hey KI** alle Voice-Control-Trigger entfernen. Den alten Sprachbefehl
   deaktivieren. Falls Voice Control sonst nicht gebraucht wird: Stop Listening
   und Auto Start Listen aus.
3. Alte Sub-Actions fuer `aiQuestion` entfernen, auch feste Testfragen und
   `%rawInput%`. `rawInput` ebenfalls nicht manuell setzen: Es kommt vom Trigger.
4. Den C#-Code komplett durch `HeyKI.cs` ersetzen und **Compile** klicken.
5. Genau diese zwei Sub-Actions in dieser Reihenfolge verwenden:

| Reihenfolge | Sub-Action | Inhalt |
|---|---|---|
| 1 | Core > Arguments > Set Argument | Name `aiConfigPath`, Wert z.B. `C:\StreamTools\HeyKI\config.json` |
| 2 | Core > C# > Execute C# Code | Gesamter Inhalt von HeyKI.cs |

Der Pfad muss bis zur **Datei config.json** reichen, nicht nur bis zum Ordner.
Leerzeichen sind erlaubt. Ohne aeussere Anfuehrungszeichen eingeben; kopierte
aeussere doppelte Anfuehrungszeichen werden nun auch automatisch abgefangen.

## Command und Trigger

1. Unter **Commands** den vorhandenen `!ki` bearbeiten oder neu erstellen:
   Enabled an, **Mode Basic**, **Location Start**, **Commands !ki**.
   Als Quelle **Twitch Chat Message** aktivieren.
2. Fuer den Start unter **Permissions > Allow** die Gruppe Broadcaster waehlen.
   Eine leere erlaubte Liste erlaubt allen den Zugriff. Moderatoren kannst du
   spaeter hinzufuegen. API-Anfragen verursachen Kosten.
3. In der Action **Hey KI** genau einen Trigger:
   **Core > Commands > Command Triggered**, Command **!ki** auswaehlen.
4. Eine eigene **nicht blockierende Queue** verwenden; vorhandene Queue HeyKI
   kann bleiben. Nur eine Kopie der Action betreiben.
5. Twitch-Konto verbinden und im Twitch-Webchat testen:
   **!ki Warum ist der Himmel blau?**

Bei Location Start liefert der Trigger die Frage ohne !ki als `rawInput`.
Der Code liest sie direkt. Kein zusaetzliches aiQuestion-Argument erforderlich.
Bei Tests im Streamer.bot-eigenen Chat kann **Ignore Internal Messages** den
Aufruf verhindern. Zuerst im Twitch-Webchat testen oder diese Option ausschalten.

## OpenAI-Key, falls noch nicht vorhanden

Auf https://platform.openai.com/api-keys einen Key erstellen und API-Abrechnung
aktivieren. In Windows **Umgebungsvariablen fuer dieses Konto bearbeiten**:
Benutzervariable `OPENAI_API_KEY` anlegen, Wert = Key. Streamer.bot neu starten.
Ein vorhandener funktionierender Key bleibt unveraendert.
Bei fehlenden C#-Referenzen `System.Net.Http.dll` und `Newtonsoft.Json.dll` aus
der Streamer.bot-Installation im Referenzen-Bereich ergaenzen.

## Chat / Stimme

| Ausgabe | chat | speech |
|---|---|---|
| Nur Chat | true | false |
| Nur Speaker.bot | false | true |
| Beides | true | true |
| Deaktiviert | false | false |

Die Config wird bei jedem Aufruf eingelesen. Fuer deine ElevenLabs-Stimme in der
vorhandenen `config.json` `speech` auf true und `voiceAlias` auf `KI` setzen.
`chat` nach Wunsch setzen. Der Alias muss exakt wie in Speaker.bot heissen.

## Speaker.bot, ElevenLabs und OBS

Eine bereits funktionierende Einrichtung behalten. Fuer Neueinrichtung:

1. https://speaker.bot/get-started/installation : herunterladen, entpacken, starten.
2. **Settings > Speech Engines**: ElevenLabs mit deinem Key einrichten.
   Berechtigungen: Text to Speech = Access, Voices = Read, Models = Read.
   Falls eine Konto-/Guthabenabfrage an fehlendem Zugriff scheitert, gezielt
   User = Read ergaenzen. Keine Schreibrechte zur Stimmenverwaltung noetig.
3. **Settings > Voice Aliases**: Alias **KI**, ElevenLabs, deine Stimme,
   Modell **Flash v2.5**, sofern angeboten, alternativ Multilingual v2.
   Ausgabegeraet waehlen, **Test Speak** pruefen.
4. **Settings > General**: TTS Enabled einschalten.
5. **Settings > WebSocket Server** aktivieren. In Streamer.bot unter
   **Integrations > Speaker.bot** mit 127.0.0.1 und demselben Port verbinden;
   Auto Connect und Auto Reconnect einschalten.
6. OBS: passende Desktop-Audioquelle oder Anwendungs-Audioaufnahme fuer
   Speaker.bot hinzufuegen. Mit einer Aufnahme pruefen, dass die Stimme genau
   einmal im Stream ankommt. Beide Bots muessen laufen.

## Weitere Config-Werte

| Wert | Standard / Bedeutung |
|---|---|
| model | gpt-5.4-mini fuer die OpenAI-Textantwort |
| reasoningEffort | none; bei Modellwechsel Unterstuetzung pruefen, ggf. leere Zeichenfolge |
| maxChars | 420; erlaubt 80 bis 450, lokale harte Begrenzung |
| maxOutputTokens | 300; erlaubt 64 bis 2000 |
| cooldownSeconds | 8 Sekunden ab Anfragestart, auch fuer Broadcaster |
| timeoutSeconds | 20; erlaubt 5 bis 60 Sekunden |
| useBotAccount | true: Bot-Konto, Broadcaster als Fallback |

Jede Frage ist eigenstaendig, ohne Verlauf oder Live-Websuche. Parallele Fragen
werden verworfen. Leeres !ki sendet keine API-Anfrage. Das ElevenLabs-Modell
stellst du in Speaker.bot ein; `model` in der Config steuert nur OpenAI.

## Test und Fehlerdiagnose

1. Zuerst mit chat = true im Twitch-Webchat testen. Zwischen Versuchen mindestens
   8 Sekunden warten. In **Action History** muss **Hey KI** auftauchen.
2. Im Ordner **logs** neben Streamer.bot.exe die neueste Logdatei oeffnen und
   nach **HeyKI:** suchen. Start, ausgelassene Aufrufe und Fehler werden erklaert.
3. Erst Speaker.bot mit Test Speak pruefen, dann speech = true aktivieren.

| Meldung / Verhalten | Pruefung |
|---|---|
| Kein Action-History-Eintrag | Command enabled, Location Start, Quelle, Permissions, Trigger |
| rawInput fehlt | Command-Trigger verwenden; manuelles Starten liefert keine Frage |
| Config-Datei nicht gefunden | aiConfigPath inklusive config.json pruefen |
| Pfad zeigt auf Ordner | Dateiname config.json anhaengen |
| Ungueltiges JSON | Kommas, doppelte Anfuehrungszeichen, true/false ohne Anfuehrungszeichen |
| HTTP 401 | OpenAI-Key pruefen |
| HTTP 429 | Guthaben / API-Limit pruefen |
| HTTP 400/404 | OpenAI-Modell und reasoningEffort pruefen |
| Antwort nicht vollstaendig | maxOutputTokens erhoehen |
| Chat geht, Stimme fehlt | Verbindung, Alias, Engine, TTS Enabled, Ausgabegeraet |
| Stimme lokal, nicht im Stream | OBS-Audioquelle und Mixer |

Die Ausgabemeldung im Log bestaetigt die Verarbeitung, keine hoerbare Wiedergabe
oder Zustellung im Chat. API isoliert testen: voruebergehend vor dem C#-Schritt
Argument `rawInput` mit einer festen Frage setzen, Action manuell starten, danach
**diese Test-Sub-Action wieder entfernen**. Keine API-Keys im Chat oder Log teilen.

## Stand und Quellen

14.10.2026. Kompilation mit C# 7.3 / .NET 5 und 23 lokale Checks mit CPH-Testdouble bestanden; kein Live-Test mit
deinem API-Key, Twitch, Speaker.bot oder OBS. Kein Streamer.bot-Importcode:
Die bestehende Action anhand der Schritte oben aktualisieren.

- https://docs.streamer.bot/guide/commands
- https://docs.streamer.bot/api/triggers/core/commands/command-triggered
- https://docs.streamer.bot/examples/ollama-chat-command
- https://docs.streamer.bot/api/csharp/methods/speakerbot/tts-speak
- https://docs.streamer.bot/guide/integrations/speakerbot
- https://speaker.bot/get-started/quick-start
- https://speaker.bot/get-started/speech-engines
- https://elevenlabs.io/docs/overview/models
- https://developers.openai.com/api/docs/models/gpt-5.4-mini


