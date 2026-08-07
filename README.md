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
2. **Add the MCP server**, from that same shell (see
   [`references/comfyui-mcp-setup.md`](brand-asset-machine-local/references/comfyui-mcp-setup.md)):
   ```bash
   claude mcp add comfyui -- npx -y comfyui-mcp
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
without touching WSL2.

1. **Install [Node.js for Windows](https://nodejs.org/)** (needed for `npx`).
2. **Get ComfyUI running natively on Windows** — desktop build defaults to
   `http://127.0.0.1:8000`, the portable/CLI build to `http://127.0.0.1:8188`.
3. **Add the MCP server.** Desktop's Connectors panel only lists pre-published
   services — it won't find `comfyui-mcp` by search. Instead, install the CLI
   separately (Desktop bundles the engine but not the `claude` command) and
   use it to add the server:
   ```powershell
   irm https://claude.ai/install.ps1 | iex
   claude mcp add comfyui -- npx -y comfyui-mcp
   ```
   Set `COMFYUI_PORT` to match step 2. See
   [`references/comfyui-mcp-setup.md`](brand-asset-machine-local/references/comfyui-mcp-setup.md)
   for the manual `%USERPROFILE%\.claude.json` alternative if you'd rather
   skip the CLI install.
4. **Install the skill** — copy `brand-asset-machine-local/` into
   `%USERPROFILE%\.claude\skills\brand-asset-machine-local`. Symlinks work too
   (`mklink /D` in an elevated prompt), but a plain copy is less fuss — just
   re-copy after `git pull`.
5. **Open the project in Claude Code Desktop without the WSL environment
   picker**, so it stays on the native Windows config. Restart and the skill
   loads automatically.

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
