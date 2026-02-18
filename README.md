# STT (Toggle Mode Fork)

> Fork of [bokan/stt](https://github.com/bokan/stt) with **hands-free toggle mode** — tap once to start recording, tap again to stop. No need to hold a key down.

**Like SuperWhisper, but free. Like Wispr Flow, but local. Now truly hands-free.**

Tap a key, speak as long as you want, tap again — your words appear wherever your cursor is. Built for vibe coding and long-form dictation to AI agents like Claude Code.

![Demo](demo.gif)

- **Free & open source** — no subscription, no cloud dependency
- **Runs locally** on Apple Silicon via MLX Whisper or Parakeet
- **Toggle mode** — tap to start, tap to stop (no holding keys down)
- **Long recordings** — dictate for minutes, not just seconds
- **One command install** — `uv tool install git+https://github.com/cyberkenn/stt.git`

## What This Fork Adds

### Toggle Mode (`TOGGLE_MODE=true`)

Upstream stt requires **holding** the hotkey while speaking. This fork adds a **tap-toggle** alternative:

| Mode | Workflow | Best for |
|---|---|---|
| Hold (upstream default) | Hold key → speak → release | Short commands |
| **Toggle (this fork)** | Tap key → speak hands-free → tap key | Long dictation, accessibility |

Enable it by adding to `~/.config/stt/.env`:
```bash
TOGGLE_MODE=true
```

**How it works:**
1. **Tap** the hotkey (e.g., Right Option) — recording starts, you hear a sound
2. **Speak** as long as you want — hands completely free
3. **Tap** the hotkey again — recording stops, audio is transcribed, text appears at cursor
4. **Esc** — cancel recording at any time
5. **Hold Shift** during the second tap — auto-presses Enter after typing (submit to Claude Code)

Set `TOGGLE_MODE=false` (or remove it) to use the original hold-to-record behavior.

## All Features

- **Global hotkey** — works in any application, configurable trigger key
- **Toggle or hold-to-record** — choose your preferred recording style
- **Auto-type** — transcribed text is typed directly into the active field
- **Shift+record** — automatically sends Enter after typing (great for chat interfaces)
- **Audio feedback** — subtle system sounds confirm recording state (can be disabled)
- **Silence detection** — automatically skips transcription when no speech detected
- **Slash commands** — say "slash help" to type `/help`
- **Context prompts** — improve accuracy with domain-specific vocabulary
- **Auto-updates** — notifies when a new version is available

## Requirements

- macOS with Apple Silicon (M1/M2/M3/M4)
- [UV](https://docs.astral.sh/uv/) package manager
- **For cloud mode (optional):** [Groq API key](https://console.groq.com)

## Installation

```bash
# This fork (with toggle mode):
uv tool install git+https://github.com/cyberkenn/stt.git

# Or upstream (hold-to-record only):
uv tool install git+https://github.com/bokan/stt.git
```

On first run, a setup wizard will guide you through configuration.

To update:

```bash
uv tool install --reinstall git+https://github.com/cyberkenn/stt.git
```

## Permissions

STT needs macOS permissions to capture the global hotkey and type text into other apps.

Grant these to **your terminal app** (iTerm2, Terminal, Warp, etc.) — not "stt":

- **Accessibility** — System Settings → Privacy & Security → Accessibility
- **Input Monitoring** — System Settings → Privacy & Security → Input Monitoring

## Usage

```bash
stt
```

| Action | Keys |
|--------|------|
| Record (hold mode) | Hold **Right Command** (default) |
| Record (toggle mode) | Tap **Right Command** to start, tap again to stop |
| Record + Enter | Hold **Shift** while recording (hold) or during second tap (toggle) |
| Cancel recording / stuck transcription | **ESC** |
| Quit | **Ctrl+C** |

## Configuration

Settings are stored in `~/.config/stt/.env`. Run `stt --config` to reconfigure, or edit directly:

```bash
# Transcription provider: "mlx" (default), "whisper-cpp-http", "parakeet", or "groq"
PROVIDER=mlx

# Local HTTP server URL (default: http://localhost:8080)
WHISPER_CPP_HTTP_URL=http://localhost:8080

# Required for cloud mode only
GROQ_API_KEY=gsk_...

# Audio device (saved automatically after first selection; device name, not index)
AUDIO_DEVICE=MacBook Pro Microphone

# Language code for transcription
LANGUAGE=en

# Trigger key: cmd_r, cmd_l, alt_r, alt_l, ctrl_r, ctrl_l, shift_r
HOTKEY=cmd_r

# Context prompt to improve accuracy for specific terms
PROMPT=Claude, Anthropic, TypeScript, React, Python

# Disable audio feedback sounds
SOUND_ENABLED=true

# Toggle mode: tap to start/stop instead of hold (this fork only)
TOGGLE_MODE=true

# Transcription timeout in seconds (increase for long recordings)
WHISPER_TIMEOUT_S=600
```

## Prompt Overlay (Optional)

STT includes a prompt overlay (triggered by Right Option by default) for quickly pasting common prompts.

Prompts live in:

`~/.config/stt/prompts/*.md`

### Local Mode (MLX Whisper) — Default

Local transcription uses Apple Silicon GPU acceleration via MLX. On first run, the Whisper large-v3 model (~3GB) will be downloaded and cached. Subsequent runs load from cache.

Runs completely offline — no API key required. Supports 99 languages and context prompts.

### Local Mode (Parakeet)

Nvidia's Parakeet model via MLX. Faster than Whisper (~3000x realtime factor) with comparable accuracy.

```bash
PROVIDER=parakeet
```

On first run, the model (~2.5GB) will be downloaded and cached.

**Limitations:**
- English only

**Phonetic correction:** While Parakeet doesn't support Whisper-style prompts, it uses the `PROMPT` setting for phonetic post-processing. Terms like `Claude Code, WezTerm` will correct sound-alike ASR errors (e.g., "cloud code" → "Claude Code", "Vez term" → "WezTerm").

### Cloud Mode (Groq)

To use cloud transcription instead:

```bash
PROVIDER=groq
GROQ_API_KEY=gsk_...
```

Requires a [Groq API key](https://console.groq.com) (free tier available).

### HTTP Mode (Local Server)

Run a local HTTP server with Whisper transcription. Useful for performance or custom integration.

```bash
PROVIDER=whisper-cpp-http
WHISPER_CPP_HTTP_URL=http://localhost:8080
```

**Start the server:**

```bash
# Terminal 1: Start the whisper.cpp server
./whisper-server -m models/ggml-large-v3.bin -f

# Or run in background with a custom port
./whisper-server -m models/ggml-large-v3.bin -f -t 4 -ngl 32 -p 8080
```

The server provides a whisper.cpp-compatible endpoint:

```bash
curl -X POST http://localhost:8080/inference \
  -H "Content-Type: multipart/form-data" \
  -F "file=@audio.wav" \
  -F "language=en"
```

**Benefits:**
- Fast HTTP API for integrating with other services
- Reuse whisper.cpp model across multiple applications
- Hardware accelerated on CPU/NVIDIA
- Configurable temperature, model, and decoding options

### Prompt Examples

The `PROMPT` setting helps Whisper recognize domain-specific terms:

```bash
# Programming
PROMPT=TypeScript, React, useState, async await, API endpoint

# AI tools
PROMPT=Claude, Anthropic, OpenAI, Groq, LLM, GPT
```

## Syncing with Upstream

This fork tracks [bokan/stt](https://github.com/bokan/stt). To pull in upstream updates:

```bash
git fetch upstream
git merge upstream/master
git push
```

## Development

```bash
git clone https://github.com/cyberkenn/stt.git
cd stt
uv sync
uv run stt
```

## License

MIT — same as upstream. See [LICENSE](LICENSE).
