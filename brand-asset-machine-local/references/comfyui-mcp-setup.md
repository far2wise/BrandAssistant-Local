# ComfyUI + local MCP setup

The skill drives generation by talking to a **local ComfyUI** through a **ComfyUI
MCP server**. Everything runs on the user's own hardware — no cloud, no per-image
billing. This file gets that connection working.

> Not to be confused with the *official* Comfy MCP (`cloud.comfy.org/mcp`), which
> routes to **Comfy Cloud** — a paid, remote service. For local generation use the
> local-first server below.

## 1. Install and run ComfyUI

Install ComfyUI (desktop app or the portable/CLI build) and update it to the
latest version so it has native FLUX.2 and Wan 2.2 support. Also install
**ComfyUI-Manager** for one-click custom-node and model installs.

Start ComfyUI and confirm it serves its web UI and can generate a test image
before wiring up the MCP. Default endpoints:

- **CLI / portable build:** `http://127.0.0.1:8188`
- **Desktop app:** `http://127.0.0.1:8000`

## 2. Install the local MCP server

Using **`artokun/comfyui-mcp`** — fully local, talks to ComfyUI over its REST +
WebSocket API, auto-detects the port, and can author/run workflows and generate
images, video, and audio. It runs via `npx`, so no repo cloning is needed:

```bash
npx -y comfyui-mcp
```

## 3. Connect it to your agent

### Claude Code

```bash
claude mcp add comfyui -- npx -y comfyui-mcp
```

…or add to `~/.claude/settings.json` directly:

```json
{
  "mcpServers": {
    "comfyui": {
      "command": "npx",
      "args": ["-y", "comfyui-mcp"],
      "env": {
        "COMFYUI_HOST": "127.0.0.1",
        "COMFYUI_PORT": "8188",
        "COMFYUI_PATH": "~/Documents/ComfyUI",
        "CIVITAI_API_TOKEN": ""
      }
    }
  }
}
```

### Claude Desktop

Add the same `mcpServers` block to `claude_desktop_config.json`:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

Then fully restart Claude.

## 4. Environment variables

| Variable | Purpose | Typical value |
| --- | --- | --- |
| `COMFYUI_HOST` | ComfyUI server address | `127.0.0.1` |
| `COMFYUI_PORT` | ComfyUI port | `8188` (CLI) / `8000` (desktop) |
| `COMFYUI_PATH` | Local ComfyUI data dir (for model discovery/downloads) | `~/Documents/ComfyUI` |
| `CIVITAI_API_TOKEN` | Only for pulling gated models from CivitAI | optional |

Generation always runs on the local GPU. The optional HuggingFace / CivitAI APIs
are used only for discovering and downloading models, never for generation.

## 5. Verify before running the skill

1. ComfyUI is running and reachable at its host:port.
2. In Claude Code run `/mcp` (or open the tools panel in Claude Desktop) and
   confirm `comfyui` shows as connected with its tools listed.
3. Query the MCP's model list and confirm the three models for the user's tier
   (`hardware-tiers.md`) are installed — fetch any missing ones.

If the MCP isn't connected, do **not** attempt to generate — fix the connection
first. Common causes: ComfyUI not started, wrong port in `COMFYUI_PORT`, or Claude
not restarted after editing the config.

## Alternatives

- **Other local ComfyUI MCP servers** exist (search "ComfyUI MCP"); any that
  exposes generate/submit-workflow tools against a local ComfyUI works — just keep
  the server name `comfyui` or update the skill's references accordingly.
- **Remote GPU:** to use a rented GPU, run ComfyUI on that box, tunnel/point
  `COMFYUI_HOST`/`COMFYUI_PORT` at it, and the same setup applies.
