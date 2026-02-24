# ollama-openclaw-discord-bot

# 🤖 adie-clawbot

A local AI-powered Discord bot using **OpenClaw** + **Ollama** — runs 100% on your own Windows machine. No cloud API costs, no data leaving your PC.

---

## 📋 Table of Contents

- [What is this?](#what-is-this)
- [How it works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Bot](#running-the-bot)
- [Config Reference](#config-reference)
- [Troubleshooting](#troubleshooting)
- [What You Can Build](#what-you-can-build)
- [Useful Commands](#useful-commands)
- [Known Issues](#known-issues)

---

## What is this?

This repo documents the setup of a fully local Discord AI bot using:

| Tool | Role |
|---|---|
| [Ollama](https://ollama.com) | Runs LLM models locally on your Windows machine |
| [OpenClaw](https://openclaw.ai) | Gateway that bridges Discord ↔ your local LLM |
| Discord Bot | The interface users talk to |
| `qwen2.5-coder:7b` | The local LLM model (runs on your GPU/CPU) |

Everything runs on **your own PC** — no OpenAI API, no monthly AI costs.

---

## How it works

```
Discord User
    ↓  (message)
Discord Bot (adie-clawbot)
    ↓
OpenClaw Gateway (port 18789)
    ↓
Ollama (http://127.0.0.1:11434)
    ↓
qwen2.5-coder:7b (local model)
    ↓  (response)
Back to Discord
```

---

## Prerequisites

Before starting, make sure you have:

- Windows 10/11
- [Node.js 22+](https://nodejs.org)
- [Ollama for Windows](https://ollama.com/download)
- A [Discord account](https://discord.com)
- A Discord server where you have admin rights

---

## Installation

### Step 1 — Install Ollama and pull a model

Download Ollama from [ollama.com/download](https://ollama.com/download) and install it.

Then pull your model:

```powershell
ollama pull qwen2.5-coder:7b
```

Verify it's available:

```powershell
ollama list
```

### Step 2 — Install OpenClaw

```powershell
npm install -g openclaw@latest
```

### Step 3 — Run the setup wizard

```powershell
ollama launch openclaw
```

This launches the onboarding wizard. Select:
- Provider → **Local Ollama**
- Ollama URL → `http://127.0.0.1:11434` (default, just press Enter)
- Model → pick your pulled model
- Messaging platform → **Discord**

### Step 4 — Create a Discord Bot

1. Go to [discord.com/developers/applications](https://discord.com/developers/applications)
2. Click **New Application** → name it
3. Go to **Bot** tab → click **Reset Token** → copy the token
4. Enable **Message Content Intent** under Privileged Gateway Intents
5. Go to **OAuth2 → URL Generator**:
   - Scope: `bot`
   - Permissions: `Send Messages`, `Read Message History`
6. Copy the OAuth URL → open in browser → invite bot to your server

### Step 5 — Get your IDs

Enable Developer Mode in Discord:
> User Settings (⚙️) → Advanced → Developer Mode → ON

Then:
- **Server ID**: Right-click your server icon → Copy Server ID
- **Channel ID**: Right-click your channel → Copy Channel ID
- **Your User ID**: Right-click your own username → Copy User ID

---

## Configuration

Config file location:
```
C:\Users\<YourName>\.openclaw\openclaw.json
```

### Full working config template

```json
{
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://127.0.0.1:11434/v1",
        "apiKey": "ollama",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen2.5-coder:7b",
            "name": "qwen2.5-coder:7b",
            "reasoning": false,
            "input": ["text"],
            "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
            "contextWindow": 32768,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/qwen2.5-coder:7b"
      },
      "workspace": "C:\\Users\\YourName\\.openclaw\\workspace"
    }
  },
  "commands": {
    "native": "auto",
    "nativeSkills": "auto",
    "restart": true,
    "ownerDisplay": "raw"
  },
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "boot-md": { "enabled": true },
        "tts": { "enabled": false }
      }
    }
  },
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN_HERE",
      "groupPolicy": "allowlist",
      "streamMode": "off",
      "guilds": {
        "YOUR_SERVER_ID": {
          "users": ["YOUR_DISCORD_USER_ID"],
          "channels": {
            "YOUR_CHANNEL_ID": {
              "allow": true
            }
          }
        }
      }
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "your-gateway-token"
    }
  },
  "plugins": {
    "entries": {
      "discord": { "enabled": true }
    }
  }
}
```

### Key config values explained

| Field | Value | Notes |
|---|---|---|
| `apiKey` | `"ollama"` | Must be exactly `"ollama"` — not anything else |
| `contextWindow` | `32768` | Max for qwen2.5-coder:7b — don't set higher |
| `ownerDisplay` | `"raw"` or `"hash"` | Only these two values are valid |
| `streamMode` | `"off"` | Keep off to avoid raw JSON output |
| `users` | `["your_user_id"]` | Array format — not an object |
| `tts` enabled | `false` | Disable to stop JSON-wrapped responses |

---

## Running the Bot

### Every time you start your PC:

**Step 1 — Start Ollama** (if not already running as a background service):
```powershell
ollama serve
```

**Step 2 — Launch OpenClaw:**
```powershell
ollama launch openclaw
```

**Step 3 — Verify:**
```powershell
openclaw status
```

You should see:
```
Discord  : ON  |  OK
Model    : qwen2.5-coder:7b
Sessions : 1
```

### Auto-start on Windows boot (optional)

```powershell
openclaw daemon install
```

This installs OpenClaw as a Windows service so it starts automatically.

Also set Ollama to keep your model loaded (prevents cold start delays):
```powershell
setx OLLAMA_KEEP_ALIVE "24h"
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Bot offline in Discord | Gateway not running | Run `ollama launch openclaw` |
| Raw JSON in Discord | `ownerDisplay: "raw"` + tts skill | Set `tts: { enabled: false }` in hooks |
| `memory_search` JSON output | Model using native skills | Normal behavior, disable tts via hooks |
| Config invalid error | JSON syntax error or wrong value | Run `openclaw doctor --fix` |
| `users: expected array` | Wrong format for users field | Use `"users": ["id"]` not `"users": {"id": {}}` |
| `allowFrom` unrecognized | Wrong key name | Use `guilds > server_id > users` array instead |
| Gateway service missing | Gateway not installed | Run `openclaw gateway install` then `openclaw gateway` |
| Model not found | Model not pulled | Run `ollama pull qwen2.5-coder:7b` |
| Slow responses | Model unloads between messages | Set `OLLAMA_KEEP_ALIVE=24h` |
| Bot online but no response | User ID not in allowlist | Add your user ID to `guilds > users` array |

### Diagnostic commands

```powershell
openclaw doctor          # check config for errors
openclaw doctor --fix    # auto-fix config errors
openclaw logs --follow   # live log stream
openclaw status          # full status overview
openclaw status --deep   # deeper channel diagnostics
```

---

## Config Reference

### Valid values quick reference

```json
"ownerDisplay": "raw"          // only "raw" or "hash" accepted
"nativeSkills": "auto"         // only "auto" or "on" accepted
"streamMode": "off"            // "off" | "paragraph" | "full"
"groupPolicy": "allowlist"     // "allowlist" | "allowall"
"api": "openai-completions"    // for Ollama provider
```

### Recommended models for OpenClaw

| Model | Size | Best for | Context |
|---|---|---|---|
| `qwen2.5-coder:7b` | ~5GB | Code tasks | 32k |
| `qwen2.5:7b` | ~5GB | General chat | 32k |
| `llama3.2:3b` | ~2GB | Fast responses | 128k |
| `qwen3-coder` | varies | Best coding | 128k+ |
| `deepseek-r1:14b` | ~9GB | Reasoning | 128k |

> OpenClaw requires **at least 64k context window** for full agent features.

---

## What You Can Build

Now that you have a local AI Discord bot, here are things you can extend it to do:

### 🛠️ Coding Assistant
Your bot already uses `qwen2.5-coder` — ask it to write functions, debug code, explain errors directly in Discord.

### 📁 File & Workspace Agent
OpenClaw has a workspace directory (`~/.openclaw/workspace`). You can give it files to read, edit, and create — all triggered from Discord.

### 🔌 Connect more messaging platforms
OpenClaw supports WhatsApp, Telegram, Slack, iMessage, and more — same config, just add more channel entries.

### 🧠 Custom system prompts
Add a custom agent persona via the `boot-md` hook — create a file at:
```
~/.openclaw/agents/main/boot.md
```
Write any system instructions there (personality, rules, context).

### 🔄 Multiple models
Add multiple models in the config and switch between them per-channel:
```json
"agents": {
  "defaults": {
    "model": {
      "primary": "ollama/qwen2.5-coder:7b",
      "fallback": "ollama/llama3.2:3b"
    }
  }
}
```

### 🌐 Web search skill
Enable web search so your bot can look things up in real time (requires OpenClaw web skill configuration).

### 🖥️ Run on a VPS for 24/7 uptime
Deploy this entire setup on a Linux VPS (Hetzner, DigitalOcean, Contabo) so the bot stays online even when your PC is off. Costs ~$10-15/month.

---

## Useful Commands

```powershell
# Gateway management
ollama launch openclaw          # start everything
openclaw gateway                # start gateway only
openclaw gateway install        # install as Windows service
openclaw gateway restart        # restart gateway
openclaw gateway stop           # stop gateway

# Monitoring
openclaw status                 # quick status
openclaw status --all           # full status
openclaw status --deep          # deep channel check
openclaw logs --follow          # live logs
openclaw doctor                 # diagnose config issues
openclaw doctor --fix           # auto-fix issues

# Ollama
ollama list                     # see pulled models
ollama pull <model>             # download a model
ollama ps                       # see running models
ollama serve                    # start ollama server
```

---

## Known Issues

- `ownerDisplay: "rendered"` is **not** a valid value — only `"raw"` or `"hash"` work
- `nativeSkills: "off"` is **not** a valid value — only `"auto"` or `"on"` work  
- `users` field must be an **array** `["id"]` not an object `{"id": {}}`
- `tts` skill wraps responses in JSON when `ownerDisplay` is `"raw"` — disable tts via hooks
- `memory_search` tool calls appear as JSON — this is normal native skill behavior
- Context window in config should match actual model capability (qwen2.5-coder:7b = 32768, not 131072)

---

## License

MIT — use freely for personal or commercial projects.

---

> **Security note:** Never commit your `openclaw.json` to GitHub — it contains your Discord bot token. Add it to `.gitignore`:
> ```
> .openclaw/
> *.json
> ```
