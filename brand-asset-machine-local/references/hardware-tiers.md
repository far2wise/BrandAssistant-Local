# Hardware Tiers — the VRAM cheat sheet

**Read this every run.** The skill is hardware-agnostic on purpose: it refers to
a PHOTO model, a LOGO model, and a MOTION model. This file maps those three jobs
to the packs this skill actually ships, for the user's actual GPU.

Guiding rule: **VRAM is the ceiling.** A model that needs 24 GB will out-of-memory
on a 12 GB card no matter how you prompt it. When in doubt, drop a tier — a
slightly weaker model that finishes beats a better one that crashes.

## The models: two packs, three jobs

This skill no longer picks from a loose roster of model names. It ships two
pinned installer packs under `packs/`, both **Apache-2.0** (commercial-clean):

| Job | Pack | Model |
| --- | --- | --- |
| **PHOTO** | `packs/photo-logo-qwen-image/` | Qwen-Image 20B distilled (GGUF) |
| **LOGO** | `packs/photo-logo-qwen-image/` | *same model, same graph* — only the prompt changes |
| **MOTION** | `packs/motion-wan22-i2v/` | Wan 2.2 I2V A14B, two-expert (GGUF) |
| **EDIT** *(optional)* | `packs/edit-qwen-image/` | Qwen-Image-**Edit** (GGUF) + mmproj projector |

PHOTO and LOGO share one pack deliberately: Qwen-Image renders readable
typography, so a single install covers both product photography and wordmarks.

Install them with `apply_manifest --path <pack>/manifest.yaml`, then enqueue the
pack's `workflow.json`. Each `pack.yaml` records tested settings and gotchas.

## ⚠️ What is actually verified

**Only the 24GB+ tier has been tested.** Everything below is honest about that —
do not present the lower rows to a user as if they were verified.

| Tier | PHOTO/LOGO quant | MOTION quant | EDIT quant | Status |
| --- | --- | --- | --- | --- |
| **24 GB+** (4090, 5090, A6000) | Qwen-Image Distill **Q8_0** (20.3 GB) | Wan 2.2 I2V **Q4_K_S** (8.1 GB ×2) | Qwen-Image-Edit **Q8_0** (20.3 GB) | ✅ **VERIFIED** on an RTX 5090 / 32 GB |
| **16 GB** | Q5_K_S | Q4_K_S | Q5_K_S | ⚠️ Untested — URLs resolve, should work |
| **12 GB** | Q4_K_S (11.3 GB) | Q4_K_S | Q4_K_S (11.3 GB) | ⚠️ Untested — URLs resolve, expect tight fit |
| **8 GB** | Q4_K_S + offload | marginal | Q4_K_S + offload | ⚠️ Untested — likely needs offload tuning; may not fit |
| **Apple Silicon** | GGUF via MLX | very slow | GGUF via MLX | ⚠️ Untested by us |

To change tier, uncomment the matching quant in the pack's `manifest.yaml` **and**
update the filename in `workflow.json`'s loader node. Both must match.

## Verified numbers (24GB+ tier, RTX 5090 / 32 GB)

Measured, not estimated — from ComfyUI's own `Prompt executed in` lines:

| Asset | Resolution | Time |
| --- | --- | --- |
| First image of a session | 1024×1024 | **50.5 s** (includes the cold 20 GB model load) |
| Each image after that | 1024×1024 | **14.4 s** |
| Motion clip | 832×480, 81 frames | **102.6 s** |
| Motion clip | 1280×720, 81 frames | **182.4 s** |
| Targeted edit (optional pack) | ~1 megapixel | **47.9 s** (includes its own cold 20 GB load) |

Peak VRAM: ~20.9 GB (Qwen UNet) and ~8.5 GB per Wan expert. The two Wan experts
load **sequentially**, so motion peaks at roughly one expert, not both.

**Full-pack estimate on this tier:** ~22 stills ≈ 6 min of GPU time, plus ~3 min
for one 720p clip. Comfortably under 15 minutes of compute for the whole pack.
Budget the cold load once per session, not per asset.

## Resolutions

| Tier | Hero 16:9 | Hero/logo 1:1 | Motion clip |
| --- | --- | --- | --- |
| 24 GB+ (verified) | 1536×864 | **1024×1024** (verified) | **1280×720, 81 frames** (verified) |
| 16 GB | 1536×864 | 1024×1024 | 832×480, 81 frames |
| 12 GB | 1344×768 | 1024×1024 | 832×480, 81 frames |
| 8 GB | 1152×648 | 896×896 | 640×384 (expect retries) |

Wan frame counts must be **4n+1**. 81 frames @ 24 fps = 3.375 s; 121 frames
would give ~5.0 s (untested).

## Decision procedure

1. Run the MCP health check (`get_system_stats` action `health`) — it reports GPU
   and free VRAM in one call.
2. Pick the tier row. If between tiers or unsure, choose the lower one.
3. Tell the user plainly which tier they're on and — if it isn't 24GB+ — that the
   quant swap is untested and may need adjusting.
4. Install both packs, then quote the timings above (24GB+) or warn that timings
   on their tier are unmeasured.
5. If a generation OOMs: drop one resolution step, drop to a smaller GGUF quant
   (remember to update `workflow.json` too), then continue.

## If VRAM is unknown or very low

- **No dedicated GPU / <6 GB:** local generation isn't realistic. Say so honestly;
  suggest renting a GPU and pointing the MCP at that ComfyUI.
- **Unsure:** default to the **12 GB** tier — widest compatibility, degrades
  gracefully — and tell the user it is unverified.
