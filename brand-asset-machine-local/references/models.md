# Models — downloads, folders, licensing

Which open models fill each job, where their files go in ComfyUI, and the
licensing you need to know before putting output on a paying client's site.
Pick the actual model for the user's GPU from `hardware-tiers.md`; this file is
the "how to obtain and where to put it" companion.

The fastest way to install any of these is ComfyUI's **Template browser**
(Workflow → Browse Templates) or the MCP's model-search / installer tools, which
download to the correct folders automatically. The manual paths below are for
when you place files by hand.

## PHOTO models (product photography, textures)

| Model | ComfyUI folders | Notes |
| --- | --- | --- |
| **FLUX.2 [dev]** | `models/diffusion_models/` + text encoders in `models/text_encoders/` + VAE in `models/vae/` | Best local photoreal; fp8 ~24 GB, GGUF Q4 ~19 GB |
| **FLUX.1 [dev]** | same as above | 12 GB fp8; strong photoreal, mature ecosystem |
| **FLUX.1 [schnell]** | same as above | 4-step, 6–8 GB (Q4); faster, slightly lower fidelity |
| **FLUX.2 [klein]** | same as above | ~8 GB distilled; the low-VRAM FLUX option |
| **Qwen-Image** | `models/diffusion_models/` (+ its text encoder/VAE) | Doubles as PHOTO at 16 GB+; Apache-2.0 (commercial-clean) |
| **SD 3.5 / SDXL** | `models/checkpoints/` | 8 GB fallback tier; use a good photoreal SDXL finetune |

## LOGO model (text and vector marks)

| Model | ComfyUI folders | Notes |
| --- | --- | --- |
| **Qwen-Image** | `models/diffusion_models/` (+ encoder/VAE) | Best open model for readable typography/labels. Add **Qwen-Image Lightning** LoRA (`models/loras/`) for ~10× faster sampling |
| **Qwen-Image GGUF (Q4/Q3)** | `models/diffusion_models/` (GGUF loader node) | For 12 GB / 8 GB tiers |
| **SD 3.5** | `models/checkpoints/` | 8 GB fallback; weaker text than Qwen but workable for simple marks |

**Transparent logos:** these models don't reliably output clean alpha. Generate the
mark on flat white, then background-remove in the same workflow (a RemBG / BiRefNet
/ background-removal node) to produce the transparent PNG. Deliver both the
on-white and transparent versions.

## MOTION models (image → video)

| Model | ComfyUI folders | Notes |
| --- | --- | --- |
| **Wan 2.2 I2V (A14B)** | `models/diffusion_models/` + `umt5` text encoder + Wan VAE | Best local motion physics (pours, steam, smoke); ~24 GB fp8, GGUF for less |
| **LTX-2** | per its model card (GGUF variants) | 16 GB tier; strong quality/speed balance |
| **LTX-Video 0.9.5** | `models/checkpoints/` (LTX nodes) | Built-in image-to-video, runs on 12 GB, fastest option (~90 s/clip on a 4090) |
| **HunyuanVideo 1.5** | Tencent repo, native ComfyUI support | ~14 GB with offload; cinematic look, alternative to Wan |

**Custom nodes** (install via ComfyUI-Manager): a GGUF loader (for any quantized
build), the Wan video nodes if your ComfyUI build lacks native Wan 2.2 I2V, and a
background-removal node for logo alpha.

## Licensing — read before commercial use

The skill files in this repo are MIT. The **models are not** — each keeps its own
license, and this matters when the output goes on a paying client's live site.

- **Qwen-Image / Qwen-Image-Edit** — Apache-2.0. Commercial use clean.
- **Wan 2.2** — Apache-2.0. Commercial use clean.
- **LTX-Video / LTX-2** — check the current Lightricks license (has had
  commercial-use terms/tiers); verify before client delivery.
- **FLUX.1 / FLUX.2 [dev]** — Black Forest Labs community license; historically
  **non-commercial for the model weights**. Check BFL's current terms before using
  FLUX output commercially. If you need a guaranteed-commercial photography path,
  **use Qwen-Image for PHOTO too** (Apache-2.0) — it's the safe substitute.
- **SDXL / SD 3.5** — Stability community/enterprise terms; SDXL base is broadly
  usable, SD 3.5 has a size-gated commercial license — confirm for the user's scale.

When licensing is a concern, the fully-commercial-clean stack is:
**Qwen-Image (PHOTO + LOGO) + Wan 2.2 (MOTION)** — all Apache-2.0.
