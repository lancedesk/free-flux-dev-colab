# Free FLUX.1 dev on Google Colab (ComfyUI) — v1

Run [Black Forest Labs' FLUX.1 dev](https://huggingface.co/black-forest-labs/FLUX.1-dev) — a 12B parameter Diffusion Transformer — for free on Google Colab T4 GPU via [ComfyUI](https://github.com/comfyanonymous/ComfyUI). No local GPU required.

**v1:** Text-to-Image workflow only.

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
- **ComfyUI-Manager** — install extra nodes from within the UI
- **ComfyUI-GGUF** — custom node for loading GGUF quantized models (pre-installed)
- **Pre-built workflow** — `flux_dev_default.json` (text-to-image)

## Quick Start

1. Click the **Open in Colab** badge above, or go to [Google Colab](https://colab.research.google.com/) → **File → Upload notebook** → select `flux_dev_colab.ipynb`
2. **Runtime → Change runtime type** → Hardware accelerator: **T4 GPU** → **Save**
3. **Runtime → Run all** — ~6 minutes total
4. **Click the `trycloudflare.com` URL** in Cell 3 output → ComfyUI opens!
5. **Load the workflow:** Click **Load** (right sidebar) → select `flux_dev_default.json`
6. Click **Queue Prompt** → wait ~60-90 sec

## License

**FLUX.1 dev** is released under the [FLUX.1-dev Non-Commercial License](https://huggingface.co/black-forest-labs/FLUX.1-dev/blob/main/LICENSE.md). Free for personal and research use.

---

**Made with ❤️ — Star the repo if this saved you time!**
