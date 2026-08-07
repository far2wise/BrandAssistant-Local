# Hardware Tiers — the VRAM cheat sheet

**Read this every run.** The skill is hardware-agnostic on purpose: it refers to
a PHOTO model, a LOGO model, and a MOTION model. This file maps those three jobs
to concrete open models for the user's actual GPU. Pick the row that matches
their VRAM, tell them the three models, confirm the models are installed
(`references/models.md` has download details), then generate.

Guiding rule: **VRAM is the ceiling.** A model that needs 24 GB will out-of-memory
on a 12 GB card no matter how you prompt it. When in doubt, drop a tier — a
slightly weaker model that finishes beats a better one that crashes.

## Quick selection

| GPU / VRAM tier | PHOTO (product, textures) | LOGO (text/vector) | MOTION (image→video) |
| --- | --- | --- | --- |
| **24 GB+ NVIDIA** (4090, 5090, A6000) | FLUX.2 [dev] (fp8) | Qwen-Image | Wan 2.2 I2V (14B, fp8) |
| **16 GB NVIDIA** (4080, 4070 Ti Super, 4060 Ti 16G) | FLUX.1 [dev] (fp8) *or* Qwen-Image | Qwen-Image | LTX-2 (GGUF) *or* Wan 2.2 I2V (GGUF Q4) |
| **12 GB NVIDIA** (3060 12G, 4070) | FLUX.1 [dev] GGUF Q4 *or* FLUX.1 [schnell] | Qwen-Image GGUF Q4 | LTX-Video 0.9.5 |
| **8 GB NVIDIA** (3050, 2060, 1080) | FLUX.2 [klein] *or* SD 3.5 *or* SDXL | SD 3.5 (or Qwen-Image GGUF Q3, tight) | LTX-Video 0.9.5 (offloaded) — marginal |
| **Apple Silicon** (M-series, unified mem) | FLUX.1 [dev]/[schnell] via MLX/GGUF (≥24 GB unified) | Qwen-Image GGUF (≥32 GB unified best) | LTX-Video 0.9.5 — slow; keep clips short |

Notes on the "or" choices:
- **16 GB PHOTO:** FLUX.2 [dev] can *technically* run via GGUF Q4 with text-encoder
  CPU offload (~19 GB peak) but it's tight on 16 GB — prefer FLUX.1 [dev] fp8, or
  Qwen-Image if you want one model for both PHOTO and LOGO.
- **Qwen-Image doubles as PHOTO** at 16 GB+ and is Apache-2.0, so on any tier where
  FLUX's non-commercial license is a problem (paying client), use Qwen-Image for
  photography too. See `references/models.md` licensing.

## Resolutions per tier

Generate smaller on leaner cards; upscale after if needed (a tiled upscale pass is
cheap compared to native high-res generation).

| Tier | Hero 16:9 | Hero/logo 1:1 | Motion clip |
| --- | --- | --- | --- |
| 24 GB+ | 1536×864 | 1024×1024 | ~5 s @ 720p, 16–24 fps |
| 16 GB | 1536×864 | 1024×1024 | ~5 s @ 480–720p, 16 fps |
| 12 GB | 1344×768 | 1024×1024 | ~4–5 s @ 480p, 16 fps |
| 8 GB | 1152×648 | 896×896 | ~3–4 s @ 384–480p (expect retries) |
| Apple Silicon | 1344×768 | 1024×1024 | short (~3 s); budget lots of time |

## Rough timing (for the Step 2 batch estimate)

Give the user a realistic wait so "generate the pack" isn't a surprise. These are
order-of-magnitude figures; a full core pack is ~15–20 stills + 1 clip.

| Tier | Per still (PHOTO/LOGO) | Per ~5 s clip (MOTION) | Full core pack (approx) |
| --- | --- | --- | --- |
| 24 GB+ (4090/5090) | ~15–25 s | ~3–5 min | well under 1 hour |
| 16 GB | ~30–50 s | ~6–10 min | ~1–1.5 hours |
| 12 GB | ~60–90 s | ~10–15 min (LTX faster) | ~2 hours |
| 8 GB | ~2–4 min | marginal; may skip | half a day — consider images only |
| Apple Silicon | ~2–5 min | very slow | plan for overnight |

If the user is on 8 GB or Apple Silicon and wants the motion loop, warn them it's
the heaviest step and offer to deliver images first, video as a follow-up batch.

## Decision procedure (what to actually do)

1. Ask VRAM (or detect: `nvidia-smi` on NVIDIA; Apple → unified memory size).
2. Pick the tier row above. If they're between tiers or unsure, choose the lower one.
3. Name the three models to the user and confirm each is present in ComfyUI
   (query the MCP's model list). Offer to fetch any that are missing.
4. Use that tier's resolutions and quote its timing in the Step 2 estimate.
5. If a generation OOMs mid-run: drop one resolution step, enable model/text-encoder
   offload, or fall back to that tier's lighter "or" option — then continue.

## If VRAM is unknown or very low

- **No dedicated GPU / <6 GB:** local generation isn't realistic. Tell the user
  honestly; suggest either the cloud version of this skill (Higgsfield/Comfy Cloud)
  or renting a GPU (e.g. a rented 4090) and pointing the MCP at that ComfyUI.
- **Unsure:** default to the **12 GB** tier — it runs on the widest range of cards
  and degrades gracefully.
