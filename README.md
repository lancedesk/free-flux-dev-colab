# Free FLUX.1 dev on Google Colab (ComfyUI) — v2

Run [Black Forest Labs' FLUX.1 dev](https://huggingface.co/black-forest-labs/FLUX.1-dev) — a 12B parameter Diffusion Transformer — for free on Google Colab T4 GPU via [ComfyUI](https://github.com/comfyanonymous/ComfyUI). No local GPU required.

**v2 Features:** Text-to-Image + **Image-to-Image** (img2img) workflows!

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/lancedesk/free-flux-dev-colab/blob/main/flux_dev_colab.ipynb)

---

## Why FLUX.1 dev beats Stable Diffusion

| Feature | SD 1.5 / SDXL | FLUX.1 dev |
|---------|--------------|------------|
| Parameters | 860M / 3.5B | **12B** |
| Architecture | UNet | **Diffusion Transformer (DiT)** |
| Prompt adherence | Moderate | **Excellent** |
| Text in images | Poor | **Good** |
| Hands / anatomy | Needs ADetailer | **Much better natively** |
| Realism | Good (fine-tuned) | **Excellent out of the box** |
| License | Open | Non-commercial (research/personal) |

## Features

- **FLUX.1 dev GGUF Q5** — 12B DiT model, Q5 quantized (~8.6 GB) for reliable T4 compatibility
- **ComfyUI** — node-based interface with native FLUX support
- **ComfyUI-Manager** — install extra nodes (face detailers, LoRAs, upscalers) directly in the browser
- **ComfyUI-GGUF** — custom node for loading GGUF quantized models (pre-installed)
- **Dual text encoders** — T5-XXL fp8 (CPU-offloaded) + CLIP-L for rich prompt understanding
- **Cloudflare Tunnel** — reliable public URL, replaces Gradio's unstable `--share`
- **Two pre-built workflows:**
  - `flux_dev_default.json` — **Text-to-Image** (generate from prompt)
  - `flux_dev_img2img.json` — **Image-to-Image** (transform existing images)

## Quick Start

1. Click the **Open in Colab** badge above, or go to [Google Colab](https://colab.research.google.com/) → **File → Upload notebook** → select `flux_dev_colab.ipynb`
2. **Runtime → Change runtime type** → Hardware accelerator: **T4 GPU** → **Save**
3. **Runtime → Run all** — ~6 minutes total (model downloads dominate)
4. **Click the `trycloudflare.com` URL** that appears in Cell 3 output → ComfyUI opens!
5. **Load a workflow:** Click **Load** (right sidebar) → select:
   - `flux_dev_default.json` for **text-to-image**
   - `flux_dev_img2img.json` for **image-to-image**
6. Click **Queue Prompt** → wait ~60-90 sec

## Pre-configured Defaults

| Setting | Value | Notes |
|---------|-------|-------|
| **Model** | **FLUX.1 dev GGUF Q5** | 12B DiT, Q5 quantized (~8.6 GB) |
| **Sampler** | **euler** | Recommended for FLUX |
| **Scheduler** | **simple** | Optimal for FLUX |
| **Steps** | **20** | Quality sweet spot (try 25-30 for max detail) |
| **Guidance** | **3.5** | FLUX's CFG equivalent (try 2.5–4.5) |
| **Resolution** | **1024 × 1024** | Native FLUX resolution |
| **Denoise (img2img)** | **0.75** | Higher = more changes, lower = preserve original |
| T5-XXL | fp8, CPU-offloaded | Keeps GPU free for the UNet |
| VRAM mode | `--lowvram` | Streams model layers to GPU as needed |

## VRAM Breakdown (T4 — 15 GB)

| Component | Size | Location |
|-----------|------|----------|
| FLUX.1 dev GGUF Q5 | ~8.6 GB | GPU (streamed via `--lowvram`) |
| T5-XXL fp8 | ~4.9 GB | CPU (offloaded) |
| CLIP-L | ~246 MB | GPU |
| VAE (ae) | ~335 MB | GPU |

## Resolution Guide

Use `EmptySD3LatentImage` in the workflow to change resolution:

| Format | Width | Height |
|--------|-------|--------|
| Square | 1024 | 1024 |
| Landscape | 1360 | 768 |
| Portrait | 768 | 1360 |
| Wide | 1536 | 640 |

## Prompting Tips

### For text-to-image
- FLUX.1 dev has **excellent prompt adherence** — be specific and descriptive
- No negative prompt needed (anatomy handled natively)
- Include camera/lens details: `85mm lens, f/1.8, Sony A7R IV, film grain`
- Text in images works: `a neon sign that says "Hello World"`
- Style keywords: `cinematic, editorial, golden hour, studio lighting`

### For img2img
- Describe the **transformation**, not just the desired result
- Good: `oil painting version with dramatic lighting and rich colors`
- Good: `convert to anime style, Studio Ghibli aesthetic`
- Good: `make it look like a vintage photograph from the 1970s`
- The **denoise value** is key:
  - **0.3–0.5** = Subtle changes (color grading, minor style shifts)
  - **0.6–0.8** = Moderate changes (style transfer, artistic interpretation) — **recommended**
  - **0.9–1.0** = Major changes (nearly full regeneration, only composition preserved)

## API Usage

Once ComfyUI is running, modify and submit the workflow via HTTP:

```python
import requests, json, time
from PIL import Image
from io import BytesIO

BASE = "https://your-url.trycloudflare.com"

with open('flux_dev_colab.ipynb') as f:  # or load the saved workflow JSON
    pass

# Minimal programmatic workflow submission
wf = json.load(open('/content/ComfyUI/user/default/workflows/flux_dev_default.json'))
wf["4"]["inputs"]["text"] = "your prompt here"
wf["8"]["inputs"]["noise_seed"] = 12345

prompt_id = requests.post(f"{BASE}/prompt", json={"prompt": wf}).json()["prompt_id"]

while True:
    history = requests.get(f"{BASE}/history/{prompt_id}").json()
    if prompt_id in history:
        break
    time.sleep(2)

for node_id, output in history[prompt_id]["outputs"].items():
    for img in output.get("images", []):
        data = requests.get(f"{BASE}/view", params={
            "filename": img["filename"], "subfolder": img["subfolder"], "type": img["type"]
        }).content
        Image.open(BytesIO(data)).save("output.png")
```

Full API docs: `https://your-url.trycloudflare.com/docs`

## How It Works

| Cell | What it does | Time |
|------|-------------|------|
| Cell 1 | Clone ComfyUI + ComfyUI-Manager + ComfyUI-GGUF, install deps | ~3-4 min |
| Cell 2 | Download FLUX.1 dev GGUF Q5, T5-XXL fp8, CLIP-L, VAE (~14 GB) | ~3-4 min |
| Cell 3 | Write workflows (txt2img + img2img), start Cloudflare tunnel, launch ComfyUI | ~1 min |

**Model sources (no HF token required):**
- FLUX.1 dev GGUF Q5 → [city96/FLUX.1-dev-gguf](https://huggingface.co/city96/FLUX.1-dev-gguf)
- T5-XXL fp8 + CLIP-L → [comfyanonymous/flux_text_encoders](https://huggingface.co/comfyanonymous/flux_text_encoders)
- VAE → [cocktailpeanut/xulf-dev](https://huggingface.co/cocktailpeanut/xulf-dev) (public mirror)

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Out of memory | `--lowvram` is already set. Reduce resolution to 768×768 or lower batch size. |
| Slow generation | Normal for T4 — ~60–90 sec/image. First image also compiles CUDA kernels (~60 sec extra). |
| Workflow not loading | Click **Load** (right sidebar) → select `flux_dev_default.json` or `flux_dev_img2img.json` |
| Black / corrupted images | Re-run Cell 2 to re-download `ae.safetensors` |
| `UNETLoader` node missing | ComfyUI may be outdated — re-run Cell 1 with a fresh `git clone` |
| Tunnel URL missing | Look in Cell 3 output for `trycloudflare.com` |
| T5 memory warnings | Normal — T5 runs on CPU in fp8 |
| **img2img: "No images found"** | Upload your image to `input/` folder first (drag & drop in ComfyUI file browser) |
| **img2img: Output looks identical** | Increase denoise value in BasicScheduler (try 0.7–0.8) |
| **img2img: Output too different** | Decrease denoise value (try 0.3–0.5) |

## Known Limitations

- Free Colab sessions last ~12 hours, then reset — **save your images!**
- FLUX.1 dev is **non-commercial** — personal and research use only
- ~60–90 sec/image on T4 (vs ~15–25 sec for [FLUX.1 schnell](../flux-schnell-colab/))
- The public URL changes every session

## License

**FLUX.1 dev** is released under the [FLUX.1-dev Non-Commercial License](https://huggingface.co/black-forest-labs/FLUX.1-dev/blob/main/LICENSE.md). Free for personal and research use. For commercial applications, see [FLUX.1 pro](https://blackforestlabs.ai/).

## Resources

- [ComfyUI GitHub](https://github.com/comfyanonymous/ComfyUI)
- [ComfyUI-Manager](https://github.com/ltdrdata/ComfyUI-Manager)
- [FLUX.1 dev on Hugging Face](https://huggingface.co/black-forest-labs/FLUX.1-dev)
- [ComfyUI FLUX Examples](https://comfyanonymous.github.io/ComfyUI_examples/flux/)
- [r/StableDiffusion](https://reddit.com/r/StableDiffusion)

---
