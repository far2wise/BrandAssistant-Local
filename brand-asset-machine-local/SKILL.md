---
name: brand-asset-machine-local
description: >-
  Build a complete, brand-unique visual asset pack — logo, product photography,
  cinematic motion loop, textures, and atmosphere overlays — entirely on local
  hardware using ComfyUI, then embed it into a website and produce a Claude
  Design handoff brief. Use this skill whenever the user wants to generate brand
  assets, product photos, a logo, brand textures, or a hero video loop LOCALLY
  (self-hosted, offline, on their own GPU) instead of a paid cloud image API —
  including phrasings like "local brand asset generator", "ComfyUI brand kit",
  "generate my product photography locally", "make brand assets on my own GPU",
  or "self-hosted alternative to Higgsfield / Midjourney / nano-banana for
  branding". Adapts model choices to the user's VRAM before generating.
license: MIT
compatibility: >-
  Requires a running local ComfyUI and a ComfyUI MCP server connected to this
  session (e.g. artokun/comfyui-mcp). An NVIDIA GPU is strongly recommended;
  Apple Silicon works with reduced speed. See references/comfyui-mcp-setup.md.
---

# The Brand Asset Machine (Local Edition)

Build everything a brand's website *wears* — the logo, the photography, one
cinematic motion loop, and a texture/atmosphere pack — entirely on the user's
own hardware through a local ComfyUI, then embed it and hand off a Claude Design
brief. This skill does **not** design the website itself; it produces the assets
the site is dressed in.

The whole point is a pack that looks *bespoke*: one product build photographed
many ways, a palette held across every asset, and motion/texture that templates
never have. Consistency and palette discipline matter more than raw model power.

## Before you generate anything: three setup checks

Do these first, in order. Skipping them is the most common way this fails.

1. **Confirm the MCP is live and check the hardware.** A ComfyUI MCP server must
   be connected with its ComfyUI running. Run the MCP's health check
   (`get_system_stats` action `health`) — one call confirms reachability AND
   reports GPU / free VRAM / queue depth. If it isn't connected, stop and walk
   the user through `references/comfyui-mcp-setup.md` — do not try to generate
   without it.

2. **Install the two packs.** This skill ships its own installer packs under
   `packs/` — pinned, licence-vetted, and tested. Do **not** hand-build graphs or
   go hunting through ComfyUI-Manager's registry. Apply each manifest:

   ```
   apply_manifest --path brand-asset-machine-local/packs/photo-logo-qwen-image/manifest.yaml
   apply_manifest --path brand-asset-machine-local/packs/motion-wan22-i2v/manifest.yaml
   ```

   - **PHOTO + LOGO** → `packs/photo-logo-qwen-image/` (Qwen-Image, Apache-2.0).
     One model does both jobs — photography *and* readable wordmarks.
   - **MOTION** → `packs/motion-wan22-i2v/` (Wan 2.2 I2V, Apache-2.0).

   Each pack's `workflow.json` is a ready API-format graph: enqueue it directly
   with `enqueue_workflow`, overriding the prompt/seed/size per asset. Read the
   pack's `pack.yaml` before you run it — it records the tested settings, what is
   **not** yet verified, and the install gotchas (custom-node placement, skipped
   pip deps, and a misleading "model NOT VISIBLE" warning). Only the 24GB+ tier
   is verified today; see `references/hardware-tiers.md` before promising more.

3. **Pick a consistency method.** The brand's hero product must look identical
   across every shot, and no local model gives that for free. Read
   `references/consistency.md` and choose one method (product LoRA, reference
   conditioning, or fixed-seed) with the user before batch generation.

Throughout, refer to the three chosen models by their **job**, not a brand name,
so the workflow stays hardware-portable:
- **PHOTO model** — photography, banner, heroes, action, detail, lifestyle, textures
- **LOGO model** — logo and any text/vector marks (the text-rendering specialist)
- **MOTION model** — the image-to-video hero loop

## Workflow

### Step 0 — Interview (generate nothing yet)

Ask one round, then reflect your understanding back before continuing:

1. What's the product or business?
2. Do they have a brand name, or should you pitch 3 with personality?
3. The vibe — 2-3 adjectives, or a film / place / era it should feel like.
4. How many hero products, and do they have names?
5. Anything to avoid — colours they hate, category clichés, competitors not to resemble.
6. Is there an existing website project folder to embed into, or are we building the pack first?
7. Do they have reference photos of the hero product (for the consistency method)? Where?

### Step 1 — Research, palette, and the Custom Asset Plan

WebSearch real brands in the category and bold logos for *reference energy only,
never copies*. Then write an art-direction note containing:

- **The colour palette** — 5-6 named colours with exact hex codes. Print it and
  save it to `palette.md`. Every asset stays inside this palette. Local models
  obey palette best when you paste the **hex codes directly into the prompt**
  (e.g. "deep oxblood #4A0E0E background, warm brass #B08D57 rim light"), so do that.
- **Logo direction** — 2-3 concepts, one line each.
- **The Custom Asset Plan** — think about what *this* brand's site actually needs.
  The core pack (Step 3) is the floor, not the ceiling. Propose 5-10
  brand-specific extras, each with a one-line purpose: atmosphere overlays
  (smoke, steam, neon haze, flour dust — on plain black or white so they
  composite cleanly), a texture pack (3-4 section backgrounds in the palette),
  patterns, badges, macro details, environment shots. The user approves or edits
  this list before you generate.

### Step 2 — Parameters and a batch estimate

Because this is local, there are no credits — the budget is GPU time. Give the
user, and wait for their go before queuing anything:

- **Resolutions:** heroes 16:9 = 1536×864 and 1:1 = 1024×1024; banner and most
  stills 16:9 = 1536×864; logo 1:1 = 1024×1024. (On 8-12 GB tiers, drop to
  1344×768 / 896×896 — see `hardware-tiers.md`.)
- **Motion:** MOTION model, ~5 s loop, 480-720p per tier, image-to-video from the
  best action still.
- One generation per asset first pass; regenerate only failures.
- Report: **total number of generations** and a **rough GPU-time estimate** using
  the per-image / per-clip figures in `hardware-tiers.md` for their tier.

### Step 3 — Generate (after their yes)

Queue through the MCP, saving into subfolders. Apply the consistency method to
every product shot.

- `logo/` (LOGO model): flat vector-style mark, 3 candidate directions, each on
  plain white and on transparent, no photo texture. (For clean alpha, generate on
  white then background-remove — see `models.md`.)
- `banner/` (PHOTO): 1 flagship hero still, product on one side, clean headline space on the other.
- `heroes/` (PHOTO): each hero product, same build + lighting language, 16:9 + 1:1.
- `action/` (PHOTO): 3 macro stills of the product being made/served — the messiest, most physical moments.
- `detail/` (PHOTO): 2 quality-story shots — raw materials, styled flat-lay or prep scene.
- `lifestyle/` (PHOTO): 2 shots with human hands in the brand's world, no readable text in frame.
- `motion/` (MOTION, from the best action still): ONE ~5 s slow-motion loop of the
  single most satisfying physical moment, loopable, plus its poster still. This is
  the website hero background — spend the retries here.
- `extras/`: everything approved in the Custom Asset Plan, in subfolders by type.

### Step 4 — Embed in the website

If given a project folder in Step 0, wire the assets in:

- The motion loop as the hero: autoplaying, muted, looped background video with
  the poster still as fallback.
- Hero shots on product cards; the banner wherever a static hero is needed.
- Textures behind sections at low opacity; atmosphere overlays composited above them.
- The logo in the nav and as the favicon.

If there's no project yet, write `embed-guide.md` with exact copy-paste HTML/CSS
snippets for each placement.

### Step 5 — The Claude Design handoff

Save everything to `~/Desktop/[brand-slug]-assets/` (or a path the user gives) plus:

- `palette.md` — the hex codes.
- `manifest.md` — every file, the model + workflow used, the prompt, seed, and generation time.
- `claude-design-brief.md` — exactly what Claude Design asks for: company name;
  company blurb (2-3 sentences in the brand's voice); design-system name
  (e.g. "Fast-Food Noir"); other notes (the palette hexes, one font-pairing
  suggestion, tone words, 2-3 do/don't rules).

Finish by printing `claude-design-brief.md` in full so the user can paste it
straight into Claude Design.

## Rules that make or break the pack

- **One product build, everywhere.** A brand is the same thing photographed many
  ways, not many things. Use the locked reference/LoRA on every product shot.
- **Stay inside the palette.** Put the hex codes directly in the prompts.
- **No text baked into photographs.** Only `logo/` assets carry the mark — that's
  what the LOGO model is for.
- **Overlays and textures stay subtle** enough to sit under foreground text.
- **Off-brand or garbled result:** regenerate once (nudge seed/prompt) before showing the user.

## Reference files

- `packs/` — the two installer packs this skill ships (`photo-logo-qwen-image/`,
  `motion-wan22-i2v/`). Each `pack.yaml` records what's actually tested, the
  install gotchas, and any fix-ups made to the graph — read it before running
  that pack for the first time.
- `references/hardware-tiers.md` — **read this every run.** Which GGUF quant to
  use per GPU tier, resolutions, and per-image/per-clip timings.
- `references/models.md` — what each pack installs, ComfyUI folders, alpha/
  background removal, and licensing (matters for commercial client work).
- `references/comfyui-mcp-setup.md` — install ComfyUI + connect the local MCP.
- `references/consistency.md` — the three ways to lock the hero product's look.
