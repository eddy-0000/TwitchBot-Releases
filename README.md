# TwitchBot

Java Twitch chatbot for Twitch IRC, OBS integration, built-in chat games, configurable commands, Groq replies, song requests, points, timers, and optional text-to-speech.

Entry points:

- Bot: `twitch.app.TwitchChatClient`
- Editor: `twitch.app.editor.ConfigEditorLauncher`

## Features

- Twitch IRC client with reconnect loop and worker-based command processing
- Built-in commands such as `!help`, `!commands`, `!reload`, `!status`, `!credits`, `!downloads`, `!sr`, `!songrequest`
- Custom commands loaded at runtime from `commands.json`
- OBS WebSocket v5 integration
  - set text sources
  - set media sources
  - toggle scene items
  - refresh browser sources
  - center sources horizontally
- Built-in games
  - Tic-Tac-Toe
  - Wordle
  - Piano
  - Hangman
- SQLite-backed persistence for points, counters, and custom command categories
- Groq integration using `prompts.json` and `emotes.json`
- Song request support with YouTube downloads
- Optional text-to-speech for Groq replies and `!tts`
- Local JavaFX config editor for `config.json`, `commands.json`, `emotes.json`, `prompts.json`, and `timers.json`
- OBS-based auto-start support
- Shutdown when OBS exits

## Requirements

- Java 25, matching the current Maven compiler target in [`pom.xml`](/C:/Java%20Projects/TwitchBot/pom.xml)
- OBS with `obs-websocket` v5 if you use OBS features
- For song downloads:
  - `ffmpeg.exe`
  - `ffprobe.exe`
  - `yt-dlp.exe`

## Project Structure

- `core`: bot, editor, shared runtime code, no TTS dependencies
- `tts`: optional Chorus/OpenVoice/ONNX-based TTS implementation
- `app`: packaging module for shaded jars and Windows app images

## Running

### Development

Run either launcher from IntelliJ or your IDE of choice:

- Bot: `twitch.app.TwitchChatClient`
- Editor: `twitch.app.editor.ConfigEditorLauncher`

You can also use the Maven JavaFX run targets from the packaging module:

```powershell
mvn -pl app -am javafx:run@run-bot
mvn -pl app -am javafx:run@run-editor
```

### Build a shaded Lite JAR without TTS dependencies

```powershell
mvn -pl app -am -P lite -DskipTests package
java -jar app/target/TwitchBotLite-1.0-SNAPSHOT.jar
```

### Build a shaded TTS JAR with TTS dependencies

```powershell
mvn -pl app -am -P tts -DskipTests package
java -jar app/target/TwitchBotTTS-1.0-SNAPSHOT.jar
```

### Build a packaged Windows Lite app image

```powershell
mvn -pl app -am -P lite -DskipTests package
```

The packaged app image is written to:

```text
app/target/dist/TwitchBotLite
```

It contains a shared runtime/app image with two launchers:

```text
app/target/dist/TwitchBotLite/TwitchBotLite.exe
app/target/dist/TwitchBotLite/TwitchBotLiteEditor.exe
```

Use:

- `TwitchBotLite.exe` to run the bot
- `TwitchBotLiteEditor.exe` to open the config editor

### Build a packaged Windows TTS app image

```powershell
mvn -pl app -am -P tts -DskipTests package
```

The packaged app image is written to:

```text
app/target/dist/TwitchBotTTS
```

Because both launchers live inside the same app image, they share the same default `config` directory next to the app.

## Config Directory

The bot uses a single config directory for runtime files.

- Default: `<appHome>/config`
- In development: usually `<project>/config`
- Override via environment variable:
  - `TWITCHBOT_CONFIG_DIR`
- Override via CLI:
  - `--config-dir path/to/config`

You can also override the main config file path:

- `--config path/to/config.json`

## External Tools for Song Downloads

The downloader no longer bundles `ffmpeg.exe`, `ffprobe.exe`, or `yt-dlp.exe` inside the packaged app.

They must exist in:

```text
config/tools/ffmpeg.exe
config/tools/ffprobe.exe
config/tools/yt-dlp.exe
```

For a packaged app image, that usually means:

```text
target/dist/TwitchBot/config/tools/ffmpeg.exe
target/dist/TwitchBot/config/tools/ffprobe.exe
target/dist/TwitchBot/config/tools/yt-dlp.exe
```

Download sources:

- FFmpeg / FFprobe
  - Official FFmpeg download page: https://ffmpeg.org/download.html
  - Recommended Windows build page: https://www.gyan.dev/ffmpeg/builds/
  - Typical file to download: `ffmpeg-release-essentials.zip`
- yt-dlp
  - Official releases page: https://github.com/yt-dlp/yt-dlp/releases
  - Direct latest Windows executable: https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp.exe

After downloading:

- copy `ffmpeg.exe` and `ffprobe.exe` from the FFmpeg zip into `config/tools`
- copy `yt-dlp.exe` into `config/tools`

Behavior:

- On startup, the bot logs whether each external tool was found.
- If a download is attempted and one of the tools is missing, the bot fails with a clear error.

This change keeps the packaged app image much smaller.

## Configuration

The bot stores user-editable runtime files in the config directory:

- `config.json`
- `commands.json`
- `timers.json`
- `emotes.json`
- `prompts.json`
- `Songs.json`
- `twitchbot.db`

You can edit the main runtime files either manually or with the packaged editor:

- `TwitchBotEditor.exe`
- or `twitch.app.editor.ConfigEditorLauncher` in development

### `config.json`

`config.json` is a JSON object. Example:

```json
{
  "bot-name": "your_bot_account",
  "oauth": "oauth:your_twitch_access_token",
  "channel": "your_channel_name",
  "obs-websocket-url": "ws://localhost:4455",
  "obs-password": "your_obs_password",
  "youtube-api-key": "your_youtube_api_key",
  "songplayer": "true",
  "groq": "false",
  "groq-temperature": "0.8",
  "groq-memory": "40",  
  "groq-api-key": "your_groq_api_key",
  "twitch-client-id": "your_client_id",
  "twitch-secret": "your_client_secret",
  "tts-on": "true",
  "tts-model": "kristin",
  "tts-use-open-voice": "true",
  "disabled-commands": ""
}
```

Notes:

- Values are read as strings.
- Booleans accept values like `true`, `false`, `1`, `0`, `yes`, `no`, `on`, `off`.
- `groq-temperature` controls reply creativity.
- `groq-memory` sets the amount of messages Groq remembers, the higher the more Token Groq uses, default value is 20.
- Text-to-speech is available through `!tts`. Groq replies can also be read aloud when TTS is enabled.
- `tts-use-open-voice` enables OpenVoice-based speech generation. This gives higher-quality synthesis but uses noticeably more CPU and potentially GPU resources.
- If `tts-use-open-voice` is disabled, the bot uses the lighter dictionary-based mode instead. That is usually the better choice on lower-spec systems.
- `disabled-commands` lets you turn off certain commands if needed, usage: `"disabled-commands": "!ttson,!ttsoff,!tts,!ttsmodel"`
- In the editor UI, `disabled-commands` is managed through a multi-select command picker that includes built-in and game commands.
- Do not commit real secrets or tokens.

### CLI Overrides

Any of the important config values can be overridden at launch time:

```powershell
TwitchBot.exe `
  --config-dir config `
  --bot-name mybot `
  --oauth oauth:xxxxx `
  --channel mychannel `
  --obs-websocket-url ws://localhost:4455 `
  --obs-password secret `
  --youtube-api-key abc `
  --songplayer false `
  --groq false `
  --groq-api-key xyz `
  --twitch-client-id clientid `
  --twitch-secret clientsecret `
  --tts-on false `
  --tts-model kristin `
  --tts-use-open-voice false
```

## Setup

### Twitch Setup

You need:

- Twitch client ID
- Twitch client secret
- Twitch chat OAuth token

Steps:

1. Open the [Twitch Developer Console](https://dev.twitch.tv/console).
2. Create an application.
3. Add an OAuth redirect URL such as `http://localhost:3000`.
4. Choose `Application Integration`.
5. Copy the generated client ID and secret into `config.json`.

To get a chat OAuth token, open this URL in your browser and replace `YOUR_CLIENT_ID`:

```text
https://id.twitch.tv/oauth2/authorize?client_id=YOUR_CLIENT_ID&redirect_uri=http://localhost:3000&response_type=token&scope=chat:read+chat:edit
```

After redirect, copy the `access_token` value and store it as:

```text
oauth:YOUR_ACCESS_TOKEN
```

### OBS Setup

In OBS:

1. Open `Tools > WebSocket Server Settings`
2. Enable WebSocket Server
3. Enable Authentication
4. Copy the OBS WebSocket URL and password into `config.json`

For local OBS usage, this is usually:

```json
{
  "obs-websocket-url": "ws://localhost:4455"
}
```

### YouTube API Setup

For song request metadata, create a YouTube Data API key in the [Google Developer Console](https://console.developers.google.com/).

Put that key into:

```json
{
  "youtube-api-key": "..."
}
```

### Groq Setup

Create a Groq API key at [Groq](https://groq.com/) and place it in:

```json
{
  "groq-api-key": "..."
}
```

Also set:

```json
{
  "groq": "true"
}
```

## Runtime Files

### `commands.json`

`commands.json` is a JSON array of command objects.

Common fields:

- `command`
- `help`
- `permission`
- `cooldown`
- `successReply`
- `failureReply`
- `credits`
- `requiresCredits`
- `category`
- `save`
- `counterAmount`
- `obsMediaSource`
- `obsMediaFile`
- `obsBroadcasterFile`
- `centerObsSource`
- `obsTextSource`
- `obsTextTemplate`
- `obsCountdownTextSource`

Supported template placeholders:

- `{user}`
- `{target}`
- `{counter}`
- `{countdown}`

Example:

```json
{
  "command": "!quote",
  "category": "Quotes",
  "save": false,
  "cooldown": 5,
  "permission": "everyone",
  "help": {
    "description": "Return a random quote.",
    "usage": "!quote"
  }
}
```

### `timers.json`

Example:

```json
[
  {
    "time": 900,
    "message": "Say something every 900 seconds.",
    "isGroqMessage": true
  }
]
```

### `prompts.json`

Used for user-specific Groq prompts.

Important behavior:

- Prompt lookup is case-insensitive for usernames.
- If no user-specific entry exists, `generell` is used.
- `system` applies to all users.
- `timer` can be used for timed messages.
- `tictactoe` can be used for the built-in Tic-Tac-Toe bot personality.

Example:

```json
[
  {
    "id": "system",
    "text": "You are a childhood friend of everyone."
  },
  {
    "id": "generell",
    "text": "You are a happy chatter in the stream chat."
  },
  {
    "id": "timer",
    "text": "You are a strict professor and give out random interesting facts."
  },
  {
    "id": "tictactoe",
    "text": "You are currently playing tic-tac-toe and should taunt your opponent."
  }
]
```

### `emotes.json`

Example:

```json
[
  {
    "id": "catKISS",
    "text": "Used to show affection in the form of a kiss."
  },
  {
    "id": "catJAM",
    "text": "Used to show a dancing mood."
  }
]
```

### Song Request Files

- `Songs.json`: fallback song store
- `downloads.disabled`: create this file in the config directory to disable downloads

You can also control downloads with:

- `!downloads on`
- `!downloads off`
- `!downloads status`

## Persistence

Created in the config directory as needed:

- `twitchbot.db`
- `commands.json`
- `timers.json`
- `Songs.json`
- `prompts.json`
- `emotes.json`

## Games

### Tic-Tac-Toe

- Start: `!tictactoestart`
- Play: `!tictactoe <x y | position> [bot]`
- Reset: `!tictactoereset`
- Close: `!tictactoeclose`

Examples:

- `!tictactoe 0 2`
- `!tictactoe top right`
- `!tictactoe middle bot`

### Wordle

- Start: `!wordlestart`
- Guess: `!wordle <word>`
- Reset: `!wordlereset`
- Close: `!wordleclose`

Example:

- `!wordle crane`

### Piano

- Start recording: `!pianostart`
- Add notes: `!piano [bpm|] <notes>`
- Play piece: `!pianoplay`
- Reset: `!pianoreset`

Examples:

- `!piano C4:1 E4:1 G4:2`
- `!piano 140|C4:1 E4:1 G4:2`
- `!piano [C4 E4 G4]:2 R:0.5 A4:1`

### Hangman

- Start: `!hangmanstart`
- Guess: `!hangman <letter or word>`
- Reset: `!hangmanreset`
- Close: `!hangmanclose`

Example:

- `!hangman a`

## Text-to-Speech

Commands:

- `!tts <text>`
- `!ttson`
- `!ttsoff`
- `!ttsmodel <model>`
- `!help !ttsmodel`

Using the Config Editor you can define user-specific TTS options.

## Additional Commands

### Logging

- `!loglevel status`
- `!loglevel status root`
- `!loglevel status twitch.app.obs`
- `!loglevel root debug`
- `!loglevel twitch.app.obs trace`
- `!loglevel reset twitch.app.obs`

### Misc

- `!meme`
- `!meme <template|keyword|random> | <line1> | [line2] ...`
- `!meme list <keyword>`
- `!dice`
- `!insult`
- `!compliment`
- `!badword`
- `!flip`
- `!fortune`
- `!8ball`

`!meme` now also shows the generated meme in OBS using a browser source named `Meme`. The bot updates that source with the generated memegen URL, shows it briefly, and then hides it again.

## OBS Auto-Start

### Start OBS and the Bot Together

Example batch file:

```bat
@echo off
start "" "C:\path_to_obs\obs-studio\bin\64bit\obs64.exe"
timeout /t 3 /nobreak >nul
start "" "C:\path_to_bot\TwitchBot\TwitchBot.exe"
```

### Start the Bot When the Stream Starts

In OBS, open:

```text
Tools > Scripts
```

Add [`start_twitchbot_on_stream.lua`](/C:/Java%20Projects/TwitchBot/scripts/start_twitchbot_on_stream.lua).

The script supports the same config-related overrides such as:

- `--config-dir`
- `--config`
- `--bot-name`
- `--oauth`
- `--channel`
- `--obs-websocket-url`
- `--obs-password`
- `--youtube-api-key`
- `--songplayer`
- `--groq`
- `--groq-api-key`
- `--twitch-client-id`
- `--twitch-secret`
- `--tts-on`
- `--tts-model`
- `--tts-use-open-voice`

## Planned Features

### Built-In

- configurable hotkeys
- local LLM support
- Add all other config option to !reload all command

### Games

- Connect 4
- Checkers
- Word Chain / Word Ladder
- Twitch Chat RPG
- Poll / voting game
- Chain Story
- Hot and Cold
