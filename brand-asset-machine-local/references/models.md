# Models — what this skill installs, and the licensing

This skill installs models through its own two packs under `packs/`. Both are
pinned to **Apache-2.0** models so output is safe on a paying client's site.
Pick the quant for the user's GPU from `hardware-tiers.md`; this file is the
"what it is, where it goes, and may I sell the output" companion.

Install with the MCP's `apply_manifest`:

```
apply_manifest --path brand-asset-machine-local/packs/photo-logo-qwen-image/manifest.yaml
apply_manifest --path brand-asset-machine-local/packs/motion-wan22-i2v/manifest.yaml
apply_manifest --path brand-asset-machine-local/packs/edit-qwen-image/manifest.yaml   # optional
```

The first two are the core pack. The third is optional — install it only if you
want the touch-up path (`consistency.md`), since it adds ~21.5 GB.

Do **not** hand-build graphs or hunt through the ComfyUI-Manager registry — the
packs exist precisely so that stops being necessary.

## PHOTO + LOGO — `packs/photo-logo-qwen-image/`

One model, both jobs. Qwen-Image renders legible typography, which is why it
covers the wordmark as well as the photography.

| File | ComfyUI folder | Size (Q8_0) |
| --- | --- | --- |
| `Qwen_Image_Distill-Q8_0.gguf` | `models/unet/` | 20.3 GB |
| `Qwen2.5-VL-7B-Instruct-UD-Q4_K_S.gguf` | `models/text_encoders/` | 4.5 GB |
| `qwen_image_vae.safetensors` | `models/vae/` | 0.2 GB |
| `Qwen-Image-Lightning-8steps-V1.0.safetensors` | `models/loras/` | 1.6 GB |

Custom node required: **ComfyUI-GGUF** only.

**Transparent logos:** generate the mark on flat white, then background-remove to
produce the alpha PNG. Deliver both versions.

**Text-to-image only — no mmproj.** This manifest installs the *text* half of
Qwen2.5-VL. On load you will see `Can't find mmproj file ... Qwen-Image-Edit will
be broken!` and a long `clip missing: visual.*` list. This is **expected and
harmless for txt2img** (verified: clean photo and logo output). Editing lives in
its own pack — see below.

## EDIT — `packs/edit-qwen-image/`

Targeted touch-ups to an existing asset (recolour a label to the palette, swap a
background) instead of re-rolling a generation that drifts off-model.

| File | ComfyUI folder | Size (Q8_0) |
| --- | --- | --- |
| `Qwen_Image_Edit-Q8_0.gguf` | `models/unet/` | 20.3 GB |
| `Qwen2.5-VL-7B-Instruct-mmproj-BF16.gguf` | `models/text_encoders/` | 1.3 GB |

Custom node required: **ComfyUI-GGUF** only. Reuses the photo pack's encoder,
VAE and Lightning LoRA.

**Two things are required, and missing either one breaks editing:**

1. **The mmproj vision projector** — completes Qwen2.5-VL's vision half.
   ComfyUI-GGUF pairs it **by filename, in the same directory**: it strips the
   encoder's quant suffix and scans `models/text_encoders/` for a `.gguf`
   containing both `mmproj` and that stem. Keep the upstream filename and put it
   beside the encoder, or it silently stays broken. One BF16 mmproj serves every
   quant of the encoder.
2. **The Qwen-Image-Edit UNet** — a *different model* from `Qwen_Image_Distill`.
   Installing the mmproj alone only silences the warning; it does not enable edits.

Verified 2026-08-07: the log flipped from `Can't find mmproj file ... will be
broken!` to `Using mmproj 'Qwen2.5-VL-7B-Instruct-mmproj-BF16.gguf' ...`, and a
cap-recolour edit changed 0.98% of pixels, concentrated in the cap band.

## MOTION — `packs/motion-wan22-i2v/`

Wan 2.2 image-to-video as a two-expert mixture: a high-noise expert denoises
steps 0-2, a low-noise expert finishes 2-4. Each has its own Lightning LoRA.

| File | ComfyUI folder | Size (Q4_K_S) |
| --- | --- | --- |
| `Wan2.2-I2V-A14B-HighNoise-Q4_K_S.gguf` | `models/unet/` | 8.1 GB |
| `Wan2.2-I2V-A14B-LowNoise-Q4_K_S.gguf` | `models/unet/` | 8.1 GB |
| `umt5-xxl-encoder-Q5_K_S.gguf` | `models/text_encoders/` | 3.8 GB |
| `wan_2.1_vae.safetensors` | `models/vae/` | 0.3 GB |
| `wan2.2_i2v_lightx2v_4steps_lora_v1_{high,low}_noise.safetensors` | `models/loras/` | ~0.6 GB each |

Custom nodes required: **ComfyUI-GGUF** and **ComfyUI-VideoHelperSuite**.

**Why Wan and not LTX-2.3:** both were inspected. The bundled `ltx-2.3-img2vid`
workflow hides its whole pipeline inside a subgraph definition (4 top-level
nodes), which is awkward to enqueue headlessly; the Wan i2v workflow is an
explicit 107-node graph. Wan is also Apache-2.0 where LTX-2.3 is Lightricks-
licensed. Wan won on both counts, so **LTX was not adopted and its licence did
not need resolving.** If you ever switch to LTX, resolve that licence first.

## Licensing — read before commercial use

The skill files in this repo are MIT. The **models are not** — each keeps its own
license, and this matters when output goes on a paying client's live site.

- **Qwen-Image** — Apache-2.0. Commercial use clean. ✅ *(what we ship for PHOTO+LOGO)*
- **Wan 2.2** — Apache-2.0. Commercial use clean. ✅ *(what we ship for MOTION)*
- **LTX-Video / LTX-2.x** — Lightricks license; has had commercial-use terms/tiers.
  **Not used by this skill.** Resolve the current terms before adopting it.
- **FLUX.1 / FLUX.2 [dev]** — Black Forest Labs community license, historically
  **non-commercial for the weights**. **Not used by this skill**, deliberately.
- **SDXL / SD 3.5** — Stability community/enterprise terms. Not used by this skill.

The shipped stack is commercial-clean end to end: **Qwen-Image (PHOTO + LOGO) +
Wan 2.2 (MOTION)**, both Apache-2.0.

## Install gotchas (hit on a real machine, 2026-08-06)

1. **Custom nodes can land in the wrong tree.** `apply_manifest` clones into
   `<ComfyUI install>/custom_nodes`, but a Comfy-Desktop install launched with
   `--base-directory` loads them from `<base-directory>/custom_nodes`. If nodes
   don't appear after install, move them there and restart ComfyUI.
2. **pip deps are skipped.** `apply_manifest` reports "Python dependencies were
   NOT installed". Install each node's `requirements.txt` into ComfyUI's own venv
   (e.g. `<base-directory>/.venv/Scripts/python.exe -m pip install -r ...`).
3. **A "model NOT VISIBLE" warning can be false.** ComfyUI's REST `/models`
   endpoint does not list `.gguf` files under `text_encoders`, so the MCP may
   warn about a file that works fine. Confirm against the loader node's own enum
   (`create_workflow action:"node_info" node_type:CLIPLoaderGGUF verbose:true`).
4. **Video output may not appear in `/history`.** VHS_VideoCombine writes the mp4
   without registering it, so a finished render can look empty. Confirm with
   `get_image action:"list_outputs"`, which scans the filesystem.
5. **`apply_manifest` may report custom nodes as "failed" while still cloning
   them.** Read the message — it often says it fell back to a direct git clone.
   Verify on disk before retrying.
