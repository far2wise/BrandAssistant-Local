# ComfyUI + local MCP setup

The skill drives generation by talking to a **local ComfyUI** through a **ComfyUI
MCP server**. Everything runs on the user's own hardware — no cloud, no per-image
billing. This file gets that connection working.

> Not to be confused with the *official* Comfy MCP (`cloud.comfy.org/mcp`), which
> routes to **Comfy Cloud** — a paid, remote service. For local generation use the
> local-first server below.

> **Windows users:** ComfyUI, the MCP server, and Claude Code all need to run
> in the *same* environment (all-WSL2 or all-native-Windows) — `127.0.0.1`
> means a different machine depending on which side of that split you're on.
> See the README's [Install](../../README.md#install) section for the two
> supported lanes, and its Windows walkthrough for the full set of
> Windows-specific gotchas (CLI install, MCP scope, the Home-vs-Code-tab
> trap) — this file covers the mechanics, that one covers the order to do
> them in.

## 1. Install and run ComfyUI

Install ComfyUI (desktop app or the portable/CLI build) and update it to the
latest version so it has native Wan 2.2 and Qwen-Image support. Also install
**ComfyUI-Manager** for one-click custom-node and model installs.

Once the MCP is connected, the skill installs everything else it needs from its
own packs — see `models.md`. You do not need to pre-install any model by hand.

Start ComfyUI and confirm it serves its web UI and can generate a test image
before wiring up the MCP. **Don't trust a remembered default — read the port
from the ComfyUI log every time**, it varies by build and version:

- **Portable/CLI build:** typically `http://127.0.0.1:8188`.
- **Comfy-Desktop build:** typically `http://127.0.0.1:8000`, but current
  versions print the real answer on startup — look for a line like
  `To see the GUI go to: http://0.0.0.0:8000` and use that port.

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
claude mcp add --scope user comfyui -- npx -y comfyui-mcp
```

**Always pass `--scope user`.** The default (`local`) scope only registers
the server for the project folder you ran the command from — run it without
the flag from, say, your home directory, and the skill won't find the server
when it runs from a different project later. Check what's registered and
where with `claude mcp list`; inside a session, `/mcp` shows live connection
status too.

The Desktop app **includes** Claude Code's engine but does **not** put a
`claude` command on your PATH — `claude mcp add` needs the CLI installed
separately:

- **macOS / Linux / WSL2:** `curl -fsSL https://claude.ai/install.sh | bash`
- **Native Windows (PowerShell):** `irm https://claude.ai/install.ps1 | iex`

Also note the Desktop app's **Connectors** settings panel only lists
pre-published services (GitHub, Slack, etc.) — searching it for `comfyui-mcp`
won't find anything, since it's not a listed connector. A custom local stdio
server like this one always goes through the CLI command above, or by editing
the config file by hand:

- **Project scope:** `.mcp.json` in the project root
- **User scope:** `~/.claude.json` (macOS/Linux/WSL2) or
  `%USERPROFILE%\.claude.json` (native Windows)

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

Restart Claude Code (CLI or Desktop) after editing the file by hand.

### Claude Desktop

Add the same `mcpServers` block to `claude_desktop_config.json`:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

Then fully restart Claude.

## 4. Environment variables

| Variable | Purpose | Typical value |
| --- | --- | --- |
| `COMFYUI_HOST` | ComfyUI server address | `127.0.0.1` |
| `COMFYUI_PORT` | ComfyUI port | `8188` (CLI) / `8000` (desktop) — but always confirm from the actual startup log, not this table |
| `COMFYUI_PATH` | Local ComfyUI data dir (for model discovery/downloads) | `~/Documents/ComfyUI` |
| `CIVITAI_API_TOKEN` | Only for pulling gated models from CivitAI | optional |

Generation always runs on the local GPU. The optional HuggingFace / CivitAI APIs
are used only for discovering and downloading models, never for generation.

## 5. Verify before running the skill

1. ComfyUI is running and reachable at its host:port.
2. In Claude Code run `/mcp` (or open the tools panel in Claude Desktop) and
   confirm `comfyui` shows as connected with its tools listed.
3. Install the skill's two packs with `apply_manifest` (see `models.md`), then
   confirm their models are listed by the MCP. Pick the GGUF quant for the
   user's VRAM first — see `hardware-tiers.md`.

If the MCP isn't connected, do **not** attempt to generate — fix the
connection first using the table below.

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| `claude` not recognized in a terminal right after installing | Desktop bundles the engine but not the CLI; the installer's PATH change hasn't reached your current shell | Open a **new** terminal window, or call the full path: `& "$env:USERPROFILE\.local\bin\claude.exe" ...` (Windows) |
| Searching Desktop's Connectors panel for `comfyui-mcp` finds nothing | Connectors only lists pre-published services (GitHub, Slack, etc.), not arbitrary local stdio servers | Add it via `claude mcp add`, not the Connectors search box |
| `comfyui` works from one project/terminal but not another | Registered with the default `local` scope, which ties it to the one folder you ran the command from | Re-add with `--scope user`: `claude mcp add --scope user comfyui -- npx -y comfyui-mcp`; confirm with `claude mcp list` |
| Browser can't reach `127.0.0.1:8188` even though ComfyUI is running | Wrong port assumed — Comfy-Desktop often actually serves on `8000`, not the documented default | Read the actual port from the ComfyUI startup log, not from memory |
| Skill folder copied into `~/.claude/skills/` but Claude still says it doesn't exist | A skills directory is only watched from when the session *started* — a new chat in an already-running session doesn't re-scan it | Fully quit and reopen Claude Code (Desktop or CLI), not just a new chat/task |
| Repo commands (`git clone`, symlink) fail to find files that "should" be there | WSL2 and native Windows are separate filesystems — a repo checked out only in WSL2 is invisible to native Windows and vice versa | Clone/copy the repo fresh on whichever side you're actually running Claude Code from |
| `comfyui` shows connected via `claude mcp list`, but a Desktop **Home tab** task still can't see it (while another server like Blender works fine) | Home-tab/Dispatch-style tasks route through a cloud "device bridge" that only proxies a specific allowlist of local servers — a freshly-added server may not be bridged at all | Use Desktop's **Code** tab with the **Local** environment instead, pointed at the project folder — that's a genuinely local session with no bridge involved |
| ComfyUI-Manager logs `security_level must be normal or below, and network_mode must be personal_cloud` | Manager's `network_mode` is set to `public`, which blocks some registry actions the skill relies on to auto-fetch missing models | Not urgent to fix immediately, but if model auto-fetch fails later, check Manager's settings for `network_mode` |
| ComfyUI-Manager cache is stale / registry unreachable | `Cannot connect to comfyregistry` — outbound network issue, unrelated to the MCP connection | Fine to ignore for local generation; only affects discovering new nodes/models |

Other general causes: ComfyUI not started, Claude not restarted after
hand-editing the config file, or — on Windows — ComfyUI and the MCP server
ended up split across WSL2 and native Windows (see the callout above).

## Alternatives

- **Other local ComfyUI MCP servers** exist (search "ComfyUI MCP"); any that
  exposes generate/submit-workflow tools against a local ComfyUI works — just keep
  the server name `comfyui` or update the skill's references accordingly.
- **Remote GPU:** to use a rented GPU, run ComfyUI on that box, tunnel/point
  `COMFYUI_HOST`/`COMFYUI_PORT` at it, and the same setup applies.
