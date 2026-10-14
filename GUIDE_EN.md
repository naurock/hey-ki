# Hey KI – English setup guide

[Deutsche Anleitung](ANLEITUNG.md) · [Project overview](README.md)

Ask a question using a Twitch chat command:

```text
!ki Why is the sky blue?
```

Streamer.bot sends the question to OpenAI. A short answer is posted in chat,
spoken by Speaker.bot, or both. Input is exclusively through chat.
Select German or English answers in the configuration. The default is German.

## 1. Requirements and files

You need Windows, Streamer.bot connected to your Twitch account, and an OpenAI
API key with available API billing or credits. A ChatGPT subscription does not
replace API billing. Spoken output also requires Speaker.bot and a configured
speech engine. OBS captures the audio for your stream.

Extract the ZIP to a permanent folder, such as `C:\StreamTools\HeyKI`.

| File | Purpose |
|---|---|
| HeyKI.cs | Code for the Streamer.bot action |
| config.json | Configuration without an API key |
| ANLEITUNG.md | German setup guide |
| GUIDE_EN.md | This guide |
| README.md / LICENSE | Overview and MIT license |

This package is not a Streamer.bot import string. Configure the action manually.

## 2. Store your OpenAI key

1. Create a key at [OpenAI API Keys](https://platform.openai.com/api-keys).
   Check API billing and model access for the corresponding project.
2. Search the Windows Start menu for **Edit environment variables for your account**.
3. Under **User variables**, select **New**:
   name `OPENAI_API_KEY`, value = your API key.
4. Save, fully exit Streamer.bot, and start it again.

Keep an existing working key. Do not share keys in chat, screenshots,
or published project files.

## 3. Create the chat command

In Streamer.bot, open **Commands** and create a command:

| Setting | Value |
|---|---|
| Name | Hey KI |
| Enabled | On |
| Mode | Basic |
| Location | Start |
| Commands | !ki |
| Source | Twitch Chat Message |

Under **Permissions**, initially choose **Allow** and select the **Broadcaster**
group. An empty Allow list allows everyone. Add moderators later if desired.
Additional users are subject to the same API costs and output settings.

Use Twitch web chat for the first test. When testing from Streamer.bot's own
chat window, **Ignore Internal Messages** may prevent the command from running.

## 4. Create the action and sub-actions

1. Under **Actions**, create an action named **Hey KI**.
2. Assign a dedicated **non-blocking queue**, for example **HeyKI**.
3. Add the trigger **Core > Commands > Command Triggered** and select
   your **!ki** command.
4. Add exactly these two sub-actions in this order:

| Order | Sub-action | Configuration |
|---|---|---|
| 1 | Core > Arguments > Set Argument | Name: `aiConfigPath`; value: full path to config.json |
| 2 | Core > C# > Execute C# Code | Paste the entire contents of HeyKI.cs and click Compile |

Example `aiConfigPath` value:

```text
C:\StreamTools\HeyKI\config.json
```

Include the filename, not just the folder. Spaces in the path are supported.
Enter the path without surrounding quotation marks; copied surrounding
double quotes are also removed by the code.

If compilation reports missing references, add `System.Net.Http.dll` and
`Newtonsoft.Json.dll` from your Streamer.bot installation in the C# editor's
references section. Continue only after compilation succeeds.

**Do not create aiQuestion or manually set rawInput.**
The command trigger supplies the question after `!ki` as `rawInput`.

## 5. Test chat output first

The included configuration has `"chat": true` and `"speech": false`.
Using your permitted account, enter this in Twitch web chat:

```text
!ki Why is the sky blue?
```

Expect a short German answer prefixed with **KI:**. An empty `!ki` does not
send an API request. Wait at least eight seconds between attempts.
Additional invocations while a request is running are discarded.

## 6. Optional: Speaker.bot and OBS

You can keep an existing working speech setup.

1. [Download Speaker.bot](https://speaker.bot/get-started/installation),
   extract it, and start it.
2. Under **Settings > Speech Engines**, configure an engine:
   **SAPI5** for a locally installed Windows voice, or **ElevenLabs**
   for your ElevenLabs voice.
3. For ElevenLabs, enter its API key in Speaker.bot. Start with these permissions:
   **Text to Speech: Access**, **Voices: Read**, **Models: Read**.
   If an account lookup requires additional access, add **User: Read** specifically.
   Additional requirements depend on your Speaker.bot version.
4. Under **Settings > Voice Aliases**, create the alias **KI**. Select the
   engine, voice, and output device. For ElevenLabs, **Flash v2.5** is suitable
   for short live responses if available; **Multilingual v2** is an alternative.
5. Use **Test Speak** to confirm local playback.
   TTS must be **Enabled** under **Settings > General**.
6. Enable **Settings > WebSocket Server**.
   In Streamer.bot, open **Integrations > Speaker.bot** and connect using
   `127.0.0.1` and the same port. Enable **Auto Connect** and **Auto Reconnect**.
7. Set `speech` to `true` in the configuration. `voiceAlias` must exactly match
   your Speaker.bot alias. Keep both applications running.
8. In OBS, capture the output device with Desktop Audio, or capture Speaker.bot
   with an application audio source. Make a short recording:
   the voice should be audible and captured only once.

The ElevenLabs key belongs in Speaker.bot. The OpenAI key remains in the
Windows environment variable. Select the ElevenLabs speech model in Speaker.bot.

## 7. Configuration

Edit the existing `config.json` in a text editor. Changes are read on the next
invocation. JSON requires double quotes, commas between entries, and unquoted
`true`/`false` values.

| Output | chat | speech |
|---|---|---|
| Chat only | true | false |
| Speech only | false | true |
| Both | true | true |
| Disabled | false | false |

| Setting | Default | Meaning / allowed range |
|---|---|---|
| language | de | Response language: `de` for German, `en` for English |
| model | gpt-5.4-mini | OpenAI model for text answers |
| reasoningEffort | none | Must be supported by the OpenAI model; an empty string omits the API field |
| useBotAccount | true | Use the bot account with broadcaster fallback |
| voiceAlias | KI | Speaker.bot alias name |
| maxChars | 420 | Answer limit, 80–450; longer answers are shortened locally |
| maxOutputTokens | 300 | API output limit, 64–2000 |
| cooldownSeconds | 8 | Minimum interval from request start, at least 0 |
| timeoutSeconds | 20 | Request timeout, 5–60 seconds |

### Choose the response language

Set `"language": "de"` for German or `"language": "en"` for English
in your existing configuration. Example for English answers:

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

The choice applies on the next request to chat output and the text read aloud,
regardless of the question's language. If `language` is absent, German remains
the default. Values other than `de` and `en` are rejected with an error.
Application log messages remain in German. The configuration does not select
a different Speaker.bot voice; choose a voice and engine suitable for your
language there. Flash v2.5, for example, supports both German and English.

Each question is independent, with no conversation history or live web search.
To customize the style, edit the `BuildInstructions` text in `HeyKI.cs`.
Recompile after editing C# code.

## 8. Update an existing installation

1. Back up and keep your customized configuration, especially chat/speech settings.
   Add `"language": "de"` or `"language": "en"` if desired.
2. Replace the entire previous C# code and compile it.
3. Remove old Voice Control triggers and disable the old voice command.
   If unused, stop Voice Control and turn off Auto Start Listen.
4. Remove old sub-actions setting `aiQuestion` or manually setting `rawInput`.
5. Use only the `!ki` command trigger and the two sub-actions from section 4.
   An old `wakePhrase` configuration entry is ignored and can be removed.

## 9. Troubleshooting

Open the **logs** subfolder next to `Streamer.bot.exe`.
Open the latest log file and search for **HeyKI:**. Log messages remain in German.

| Problem / message | Next step |
|---|---|
| No Action History entry | Check command enabled state, source, permissions, and selected trigger |
| rawInput fehlt | Invoke through the chat command; manual execution does not supply a question |
| aiConfigPath fehlt | Add the first sub-action from section 4 |
| Config-Datei nicht gefunden / path points to a folder | Use the actual full path including config.json |
| Ungueltiges JSON | Check configuration syntax and value types |
| Ungueltige Sprache | Set language to de or en |
| OPENAI_API_KEY fehlt | Check the user environment variable and restart Streamer.bot |
| HTTP 401 | Check the OpenAI key |
| HTTP 429 | Check credits or rate limits |
| HTTP 400/404 | Check model access, model name, and reasoningEffort |
| Zeitlimit erreicht | Check connectivity; optionally increase timeoutSeconds up to 60 |
| Antwort nicht vollstaendig | Increase maxOutputTokens; unfinished responses are not delivered |
| Cooldown / previous request still running | Wait and try again later |
| Chat works, speech does not | Check Test Speak, connection, alias, TTS Enabled, and output device |
| Voice plays locally but not in OBS | Check the audio source, mixer, and recording |

For an isolated manual test, temporarily add a `rawInput` argument containing
a fixed question before the C# step, run the action, and
**remove that test sub-action afterwards**.

A log entry confirming output processing does not confirm actual Twitch
delivery or audible playback.

## 10. Data, costs, and license

Questions are sent to OpenAI. With ElevenLabs speech enabled, answer text is
also sent to ElevenLabs. API usage may incur charges. Generated answers may
contain errors. This project uses the MIT License; see [README.md](README.md)
and [LICENSE](LICENSE) for the disclaimer and full terms.

As of October 14, 2026. C# 7.3 / .NET 5 compilation and 35 local checks with a
test double passed. No end-to-end test with your Twitch account, API key,
Speaker.bot, or OBS. Menu labels may vary between application versions.

### Official documentation

- [Streamer.bot Commands](https://docs.streamer.bot/guide/commands)
- [Command Triggered and rawInput](https://docs.streamer.bot/api/triggers/core/commands/command-triggered)
- [Speaker.bot integration](https://docs.streamer.bot/guide/integrations/speakerbot)
- [Speaker.bot Quick Start](https://speaker.bot/get-started/quick-start)
- [ElevenLabs models](https://elevenlabs.io/docs/overview/models)

