# Consistency — locking the hero product's look

The single most important thing that separates a real brand pack from a pile of
AI images: **the hero product looks identical across every shot.** The cloud model
this skill replaces (`nano_banana_pro`) did this for free. Local models do not, so
you must choose a method and apply it to every product shot. Pick one with the
user in the setup phase, before batch generation.

Ranked best → cheapest:

## Method A — Product LoRA (best result)

Train a small LoRA on the user's reference images of the hero product, then load
it on every PHOTO (and, for logo lockups, LOGO) generation.

- **Inputs:** 10–20 clean photos of the product from varied angles/lighting.
- **Cost:** one-time ~15–30 min training on a 4090 (longer on smaller cards).
- **Result:** the strongest lock — the exact product recurs across all shots.
- **Use when:** the product is specific/branded (a labeled bottle, a particular
  sneaker) and the user has or can shoot reference images.
- **How:** train with a FLUX/Qwen LoRA trainer (e.g. an in-ComfyUI trainer node or
  a companion trainer), save to `models/loras/`, and reference it in every product
  workflow at a moderate weight (~0.7–1.0). Tune weight if it over- or under-fits.

## Method B — Reference conditioning (no training)

Feed one locked reference still into every generation so each inherits the
product's form and lighting, using **FLUX Redux** or an **IP-Adapter** node.

- **Inputs:** a single clean product image (real photo, or your best first
  generation promoted to "the reference").
- **Cost:** none beyond normal generation.
- **Result:** strong lock without training; slight drift shot-to-shot.
- **Use when:** the user has one good product image but not a training set, or you
  want to move fast. This is the sensible **default**.
- **How:** generate/choose the reference first, wire it into a Redux/IP-Adapter
  input on each product workflow, then vary only the scene/prompt around it.

## Method C — Fixed seed + verbatim build block (cheapest)

Hold the seed constant and reuse an identical "product build + lighting language"
block at the top of every prompt, changing only the scene.

- **Cost:** none.
- **Result:** weakest lock — good enough for backgrounds/textures where exact
  product match matters less; unreliable for hero product shots.
- **Use when:** no reference image exists and training isn't wanted, or for the
  texture/atmosphere assets that don't feature the product.
- **How:** write one canonical description of the product's material, colour
  (with palette hexes), and lighting; paste it verbatim into every prompt; keep the
  seed fixed across the batch.

## Practical guidance

- **Default to Method B.** It gives most of Method A's benefit with none of the
  training overhead, and works on every tier.
- **Combine methods:** a reference (B) plus the verbatim build block (C) is
  stronger than either alone, at no extra cost.
- **Lock lighting too, not just the product.** "Same build *and* lighting language"
  — a consistent key/rim setup is half of what makes a set look like one shoot.
- **Textures and atmosphere overlays** don't need the hero-product lock; generate
  them on plain black/white so they composite cleanly, and just hold the palette.
- **Editing beats regenerating** for small fixes (recolour a label to the palette,
  swap a background): use **Qwen-Image-Edit** rather than rolling the dice on a
  fresh generation that may drift off-model.
  **⚠️ Not installed by default.** `packs/photo-logo-qwen-image/` installs only the
  *text* half of Qwen2.5-VL (no `mmproj` file), which is all txt2img needs. Loading
  it logs `Qwen-Image-Edit will be broken!` — harmless for generation, but the edit
  path genuinely will not work until you install a matching mmproj file and the
  Qwen-Image-Edit UNet. Untested by this skill. If you need edits today, prefer
  Method B (reference conditioning) and regenerate.
