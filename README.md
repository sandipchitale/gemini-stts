# gemini-stts

**IMPORTANT** This will work when this PR is merged: https://github.com/google-gemini/gemini-cli/pull/22681

A Gemini CLI extension that adds speech-to-text (STT) and text-to-speech (TTS) capabilities to the Gemini CLI. Speak your prompts instead of typing them, and hear Gemini CLI's responses read aloud.

## Features

- **Speech-to-Text (`/t`)** - Dictate your prompt using your microphone via a Chrome-based UI powered by the Web SpeechRecognition API
- **Text-to-Speech (`/h`)** - Have Gemini CLI's response (or any text) spoken aloud using the browser's SpeechSynthesis API
- **Combined STT + TTS (`/th`)** - Speak your prompt and automatically hear the response -- a full voice conversation flow
- **Looping STT (`/tl`)** - Like `/t`, but keeps re-opening the dictation dialog after each prompt until you cancel it
- **Looping STT + TTS (`/thl`)** - Like `/th`, but stays in a continuous speak-and-listen loop until you cancel the dialog

### Voice Commands (STT)

While dictating, you can use voice commands for hands-free editing:

| Command | Action |
|---|---|
| `insert comma / period / question mark / exclamation mark / tab` | Inserts the corresponding punctuation or character |
| `new paragraph` | Ends the current sentence and starts a new paragraph |
| `go to start / go to end` | Moves cursor to the beginning or end of the text |
| `select all` | Selects all text |
| `unselect selection` | Collapses the selection without deleting |
| `delete selection` | Deletes the selected text |
| `undo it / redo it` | Undo or redo the last action |

A built-in cheat sheet for these voice commands is also available via the 🗣️ button in the STT dialog.

### Keyboard Shortcuts (STT)

| Shortcut | Action |
|---|---|
| `Ctrl+R` | Toggle speech recognition on/off |
| `Enter` | Send the prompt |
| `Shift+Enter` | Insert a new line |
| `Escape` | Stop recording / cancel |

## How It Works

The plugin launches a Chrome instance (via [chrome-launcher](https://www.npmjs.com/package/chrome-launcher)) in app mode and connects to it using [puppeteer-core](https://www.npmjs.com/package/puppeteer-core). This approach leverages the browser's native `SpeechRecognition` and `SpeechSynthesis` APIs, which provide high-quality speech processing without requiring any external API keys or services.

- **STT flow**: Opens a Chrome window with a textarea where you can type or dictate. Microphone permission is automatically granted via Puppeteer. When you click "Send" or press Enter, the text is printed to stdout, which Gemini CLI captures and processes as your prompt.

~[Speak](screenshots/stt.png)

- **TTS flow**: Opens a Chrome window that receives text via Puppeteer's `evaluateOnNewDocument`, then speaks it using `SpeechSynthesisUtterance`. Supports a `--oneshot` flag to automatically close after speaking.

~[Hear](screenshots/tts.png)


## Architecture

```
gemini-stts/
├── commands/
│   ├── t.toml                 # /t command - speech-to-text
│   ├── tl.toml                # /tl command - looping speech-to-text
│   ├── h.toml                 # /h command - text-to-speech
│   ├── th.toml                # /th command - combined STT + TTS
│   └── thl.toml               # /thl command - looping STT + TTS
├── src/
│   ├── chrome-sidekick.ts   # Chrome launcher and Puppeteer connection utilities
│   ├── stt.ts               # STT entry point - launches Chrome with speech recognition UI
│   ├── stt_ui.html          # STT frontend - textarea with SpeechRecognition integration
│   ├── tts.ts               # TTS entry point - launches Chrome with speech synthesis UI
│   └── tts_ui.html          # TTS frontend - textarea with SpeechSynthesis integration
├── dist/                    # Pre-built bundles (committed so users don't need `npm install`)
│   ├── stt.mjs              # Bundled STT entry point
│   ├── tts.mjs              # Bundled TTS entry point
│   ├── stt_ui.html          # Copied at build time
│   └── tts_ui.html          # Copied at build time
├── build.mjs                # esbuild build script
├── gemini-extension.json    # Gemini CLI extension manifest
├── package.json
└── tsconfig.json
```

## Prerequisites

- **Node.js** (v18+)
- **Google Chrome** or Chromium installed on your system
- A working **microphone** for speech-to-text

## Installation

### As a Gemini CLI Extension

Install directly from the repository:

```bash
gemini extension link .
```

The repository ships with pre-built bundles in `dist/`, so **no `npm install` is required** for end users. The slash commands invoke `node ${extensionPath}/dist/stt.mjs` and `dist/tts.mjs` directly.

### For Local Development

```bash
git clone https://github.com/sandipchitale/gemini-stts.git
cd gemini-stts
npm install
npm run build   # bundles src/*.ts into dist/ via esbuild
```

Then add it as a local marketplace and plugin in Gemini CLI:

```bash
git clone https://github.com/sandipchitale/gemini-stts.git
cd gemini-stts
gemini extension link .
```

The build script (`build.mjs`) uses [esbuild](https://esbuild.github.io/) to bundle `src/stt.ts` and `src/tts.ts` into standalone ESM files under `dist/`, and copies the HTML UI assets alongside them. Re-run `npm run build` after any change in `src/`.

## Usage

Once installed, use the slash commands in Gemini CLI:

```
/t [initial text]   # Speak your prompt
/tl                 # Speak your prompt in a loop until cancelled
/h                  # Hear Gemini CLI's last response
/h prompt text      # Hear specific text spoken aloud
/th [initial text]  # Speak your prompt and hear the response
/thl [initial text] # Speak-and-hear in a loop until cancelled
```

All slash commands accept optional text arguments that pre-populate the dictation textarea, e.g.:

```
/t Write a haiku about
/h The quick brown fox jumps over the lazy dog
```

## Dependencies

- [chrome-launcher](https://www.npmjs.com/package/chrome-launcher) - Launches Chrome with custom flags
- [puppeteer-core](https://www.npmjs.com/package/puppeteer-core) - Connects to and controls the Chrome instance
- [commander](https://www.npmjs.com/package/commander) - CLI argument parsing
- [esbuild](https://esbuild.github.io/) (dev) - Bundles `src/*.ts` into `dist/*.mjs` so the plugin runs without `npm install`

## License

MIT

## Author

Sandip Chitale
