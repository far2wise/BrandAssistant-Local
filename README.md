# BrandAssistant-Local

A **Claude skill** that builds a complete, brand-unique visual asset pack —
logo, product photography, a cinematic hero motion loop, textures, and atmosphere
overlays — **entirely on your own hardware** using [ComfyUI](https://github.com/comfyanonymous/ComfyUI),
then embeds it into a website and hands you a ready-to-paste Claude Design brief.

It's a local, self-hosted rebuild of a cloud pipeline: every paid image/video API
is swapped for an open model running in your ComfyUI, driven by Claude through a
local MCP server. You spend GPU time instead of credits.

## What it does

Given a short interview about your brand, the skill:

1. **Interviews** you (product, vibe, hero items, things to avoid).
2. **Researches** your category and builds a named **colour palette** (hex codes).
3. Proposes a **Custom Asset Plan** — the brand-specific extras (smoke, grain,
   glow, textures) that make a site stop looking like a template.
4. **Generates** the pack locally: logo, banner, hero shots, action/detail/
   lifestyle photography, a ~5s looping hero video, and the approved extras.
5. **Embeds** the assets into your site (or writes copy-paste HTML/CSS snippets).
6. **Hands off** a `claude-design-brief.md` formatted for Claude Design.

The design goal is a *bespoke* pack: one product build photographed many ways, a
palette held across every asset, and motion/texture templates never have.

## Requirements

- A working [ComfyUI](https://github.com/comfyanonymous/ComfyUI) install.
- A ComfyUI MCP server connected to your Claude session
  (this skill assumes [`artokun/comfyui-mcp`](https://github.com/artokun/comfyui-mcp), fully local).
- An **NVIDIA GPU is strongly recommended**; Apple Silicon works with reduced
  speed. Model choices adapt to your VRAM — see the tier table below.
- Claude Code (CLI or Desktop app) to run the skill and the MCP. See
  [Install](#install) below for the Linux/WSL2 vs. Windows-native path.

## Model tiers at a glance

The skill picks models to fit **your** GPU. Full detail (resolutions, timings,
fallbacks) is in [`brand-asset-machine-local/references/hardware-tiers.md`](brand-asset-machine-local/references/hardware-tiers.md).

| VRAM | Photography | Logo / text | Motion (image→video) |
| --- | --- | --- | --- |
| 24 GB+ | FLUX.2 [dev] | Qwen-Image | Wan 2.2 I2V |
| 16 GB | FLUX.1 [dev] / Qwen-Image | Qwen-Image | LTX-2 / Wan 2.2 (GGUF) |
| 12 GB | FLUX.1 [dev] GGUF | Qwen-Image GGUF | LTX-Video 0.9.5 |
| 8 GB | FLUX.2 [klein] / SD 3.5 | SD 3.5 | LTX-Video 0.9.5 (marginal) |
| Apple Silicon | FLUX via MLX/GGUF | Qwen-Image GGUF | LTX-Video 0.9.5 (slow) |

> **Licensing:** the skill files here are MIT, but the **models are not** — each
> keeps its own license. FLUX `[dev]` weights are historically non-commercial;
> for guaranteed-commercial output use the all-Apache-2.0 stack (Qwen-Image +
> Wan 2.2). Details in [`references/models.md`](brand-asset-machine-local/references/models.md).

## Install

Pick **one lane** and stay in it: Claude Code, the local MCP server, and
ComfyUI all need to run in the *same* environment. Split them — e.g. Claude
Code Desktop on native Windows talking to ComfyUI running inside WSL2 — and
`127.0.0.1` stops meaning the same thing on both sides, so nothing connects.
If you're not sure which lane you're in, match whichever environment already
runs your GPU tools.

### Linux / WSL2

Use this if ComfyUI runs in a Linux shell — including inside WSL2 on Windows.

1. **Get ComfyUI running** at `http://127.0.0.1:8188` and install
   [ComfyUI-Manager](https://github.com/ltdrdata/ComfyUI-Manager).
2. **Add the MCP server**, from that same shell, with **user scope** so it's
   visible from any project (the default, local scope, only applies to the
   folder you ran the command from — see
   [`references/comfyui-mcp-setup.md`](brand-asset-machine-local/references/comfyui-mcp-setup.md)):
   ```bash
   claude mcp add --scope user comfyui -- npx -y comfyui-mcp
   ```
3. **Install the skill** — copy or symlink `brand-asset-machine-local/` into a
   skills folder:
   - **Every project:** `~/.claude/skills/brand-asset-machine-local/`
   - **Just one project:** `<project>/.claude/skills/brand-asset-machine-local/`
   ```bash
   ln -s "$(pwd)/brand-asset-machine-local" ~/.claude/skills/brand-asset-machine-local
   ```
4. **Run Claude Code** — the CLI in this same shell, or Desktop with its
   environment picker pointed at this WSL2/Linux folder (not opened as a plain
   Windows project). Restart and it picks the skill up automatically.

### Windows (native, no WSL2)

Use this if ComfyUI runs directly on Windows and you want Claude Code Desktop
without touching WSL2. Every step below was hit-tested against real footguns —
follow it in order and you'll skip all of them.

1. **Install [Node.js for Windows](https://nodejs.org/)** (needed for `npx`).

2. **Get ComfyUI running natively on Windows**, and note which port it prints
   on startup — it varies by build and the docs' "default" is not reliable:
   - **Comfy-Desktop build:** actually serves on `http://127.0.0.1:8000` in
     current versions (check your ComfyUI log for a line like
     `To see the GUI go to: http://0.0.0.0:8000`).
   - **Portable/CLI build:** `http://127.0.0.1:8188`.
   Load that URL in a browser and generate a test image before going further.

3. **Install the Claude Code CLI separately.** Desktop bundles the engine but
   *not* a `claude` command on PATH — you need the CLI for the next step:
   ```powershell
   irm https://claude.ai/install.ps1 | iex
   ```
   If a command right after this says `claude` is not recognized, the
   installer put it in `%USERPROFILE%\.local\bin`, which isn't on your PATH
   until you open a **new** PowerShell window. Either open a new window, or
   prefix the next commands with the full path:
   `& "$env:USERPROFILE\.local\bin\claude.exe" ...`.

4. **Get this repo onto the Windows filesystem, then install the skill.** If
   you've only ever used this repo from inside WSL2, it isn't visible to
   native Windows — `C:\...` and `/home/...` are separate filesystems. Clone
   it fresh on Windows (don't try to copy across the WSL2 boundary). These
   commands are copy-pasteable as-is — `$env:USERPROFILE` fills in your
   username automatically, nothing to edit:
   ```powershell
   git clone https://github.com/far2wise/BrandAssistant-Local.git "$env:USERPROFILE\BrandAssistant-Local"
   Copy-Item -Recurse "$env:USERPROFILE\BrandAssistant-Local\brand-asset-machine-local" "$env:USERPROFILE\.claude\skills\brand-asset-machine-local"
   ```
   A symlink works too, if you'd rather not re-copy after `git pull` (run in
   an elevated PowerShell):
   ```powershell
   New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\brand-asset-machine-local" -Target "$env:USERPROFILE\BrandAssistant-Local\brand-asset-machine-local"
   ```
   Verify it landed — this should print `True`:
   ```powershell
   Test-Path "$env:USERPROFILE\.claude\skills\brand-asset-machine-local\SKILL.md"
   ```

5. **Add the MCP server with `--scope user` and the port from step 2.** Do
   *not* use Desktop's Connectors settings panel for this — it only lists
   pre-published services (GitHub, Slack, etc.) and will never find
   `comfyui-mcp` no matter how you search:
   ```powershell
   claude mcp add --scope user comfyui --env COMFYUI_HOST=127.0.0.1 --env COMFYUI_PORT=8000 -- npx -y comfyui-mcp
   ```
   `--scope user` is not optional — the default (`local`) scope ties the
   server to the one folder you ran the command from, and it silently
   disappears when the skill runs from anywhere else. Verify it's live:
   ```powershell
   claude mcp list
   ```
   should show `comfyui` as connected.

6. **In Claude Code Desktop, use the Code tab with the Local environment —
   not the Home tab.** This is the step most likely to trip you up even after
   everything above is correct. Desktop's **Home** tab runs tasks through a
   cloud-routed "device bridge" that only proxies a specific allowlist of
   local MCP servers back to your machine — a server you just added with
   `claude mcp add` may simply never appear there, no matter how correctly
   it's configured. The **Code** tab with the **Local** environment is a
   genuinely local session: it reads `~/.claude.json` and `~/.claude/skills/`
   directly, no proxy involved, and is what every step above assumes. Open
   Code → Local → the `BrandAssistant-Local` folder from step 4.

7. **Fully quit and reopen Claude Code Desktop** before your first attempt —
   a new chat/task inside an already-running Desktop isn't enough, since it
   only starts watching a skills directory that existed when the session
   began. Then ask it directly: "are you using the brand-asset-machine-local
   skill for this?" to confirm before you go further.

**If something still doesn't connect**, see the troubleshooting table in
[`references/comfyui-mcp-setup.md`](brand-asset-machine-local/references/comfyui-mcp-setup.md#troubleshooting).

## Usage

With ComfyUI running and the MCP connected, just ask Claude:

> "Build me a local brand asset pack for my cold-brew coffee company."

The skill checks your MCP connection, matches models to your GPU, sets up a
consistency method, and walks you through interview → palette → generate → embed →
handoff. It always shows you a generation count and time estimate and waits for
your go before spending GPU time.

## Repo layout

```
BrandAssistant-Local/
├── README.md
├── LICENSE                         (MIT — covers the skill files, not the models)
└── brand-asset-machine-local/      (the installable skill)
    ├── SKILL.md
    └── references/
        ├── hardware-tiers.md       (VRAM cheat sheet — model picks per GPU)
        ├── models.md               (downloads, folders, licensing)
        ├── comfyui-mcp-setup.md    (ComfyUI + local MCP install/config)
        └── consistency.md          (locking the hero product's look)
```

## License

MIT — see [LICENSE](LICENSE). This covers the skill's own files. The AI models the
skill uses each carry their own licenses; check them before commercial use.

## Credits

Built on [ComfyUI](https://github.com/comfyanonymous/ComfyUI), the
[`artokun/comfyui-mcp`](https://github.com/artokun/comfyui-mcp) local control
plane, and the open model community (Black Forest Labs FLUX, Qwen-Image,
Wan, LTX-Video, Stability).
