# Hey KI for Streamer.bot

A lightweight Twitch chat command that sends questions to the OpenAI API and
returns short answers. Output can be sent to Twitch chat, spoken through
Speaker.bot, or both. Voice recognition is not used.

```text
!ki Why is the sky blue?
```

Answers are limited to 420 characters by default. Set `"language": "de"` for
German (default) or `"language": "en"` for English in `config.json`.
This applies to both chat output and the text read aloud by Speaker.bot.

## Contents

- `HeyKI.cs` — C# action for Streamer.bot.
- `config.json` — model, output, length, timeout, and cooldown settings.
- [ANLEITUNG.md](ANLEITUNG.md) - setup and troubleshooting guide in German.
- [GUIDE_EN.md](GUIDE_EN.md) - setup and troubleshooting guide in English.
- `LICENSE` — MIT license terms.

## Setup

Requires Streamer.bot, a connected Twitch account, and an OpenAI API key with
API billing enabled. Speaker.bot is optional; OBS is needed to include spoken
answers in your stream. ElevenLabs is an optional speech provider.

1. Follow [GUIDE_EN.md](GUIDE_EN.md) or [ANLEITUNG.md](ANLEITUNG.md) to configure the `!ki` command and action.
2. Set `aiConfigPath` to the full path of `config.json`, including the filename.
3. Store your OpenAI key in the Windows user environment variable
   `OPENAI_API_KEY`, then restart Streamer.bot.
4. Enable `chat`, `speech`, or both in the configuration and test the command.

When updating, keep your existing configuration. Do not publish API keys.
Questions are sent to OpenAI; when using ElevenLabs, answer text is also sent
to ElevenLabs for speech generation. External services may charge usage fees.

## License and disclaimer

This project is open-source software released under the [MIT License](LICENSE).
You may use, modify, and redistribute it under that license.

Use this software at your own risk. It is provided without warranty. To the
maximum extent permitted by applicable law, the authors and maintainers accept
no liability for its use, generated content, service charges, interruptions,
or resulting loss or damage. Nothing in this notice excludes liability that
cannot legally be excluded. See `LICENSE` for the full terms.

This project is independent and is not affiliated with or endorsed by Twitch,
OpenAI, Streamer.bot, Speaker.bot, OBS, or ElevenLabs. Third-party software and
services remain subject to their own licenses and terms.
