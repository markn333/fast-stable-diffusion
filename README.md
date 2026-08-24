# fast-stable-diffusion (Community Maintained Fork)

> **This is a community-maintained fork of [TheLastBen/fast-stable-diffusion](https://github.com/TheLastBen/fast-stable-diffusion).**
> The original repository is no longer actively updated. This fork keeps the notebooks working with the current Google Colab environment (PyTorch 2.10+, Python 3.12, CUDA 12.8).

---

## Open in Colab

| Notebook | Status | Open |
|----------|--------|------|
| **AUTOMATIC1111** | ✅ Maintained | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/markn333/fast-stable-diffusion/blob/main/fast_stable_diffusion_AUTOMATIC1111.ipynb) |
| **ComfyUI** | ✅ Active Maintenance | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/markn333/fast-stable-diffusion/blob/main/fast_stable_diffusion_ComfyUI.ipynb) |
| **DreamBooth** | ❄️ Not Supported | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/markn333/fast-stable-diffusion/blob/main/fast-DreamBooth.ipynb) |

---

## What's Fixed

| Bug | Description | Fix |
|-----|-------------|-----|
| BUG-0001 | xformers incompatible with PyTorch 2.10.0 | Added "Fix xformers" cell |
| BUG-0002 | gdown v6 breaking change | Pinned `gdown==5.2.1` |
| BUG-0003 | `sd_emphasis.py` tensor device mismatch (`cuda:0` vs `meta`) | Added `.to(device, dtype)` cast |
| BUG-0004 | `load_state_dict` fails on meta tensor with PyTorch 2.10 | Added `assign=True` |
| BUG-0005 | CLIP `position_ids` becomes `HalfTensor`, breaks Embedding | Added `.long()` cast |
| BUG-0006 | ComfyUI crashes when `Ngrok_Token` is empty | Added `if Ngrok_Token != "":` guard |
| BUG-0007 | `ModuleNotFoundError: No module named 'comfy_aimdo'` | Added `comfy-aimdo` to Requirements |
| BUG-0008 | `simpleeval` missing + xformers built for PyTorch 2.9 | Added `simpleeval` to Requirements; added Fix xformers cell |
| BUG-0009 | ComfyUI URL inaccessible (tunnel + timing issues) | Switched to pinggy.io; stream stdout line-by-line to detect readiness |
| BUG-0010 | `FormData.__init__() got unexpected keyword argument 'default_to_multipart'` | Added `aiohttp -U` to Requirements |
| BUG-0011 | `sentencepiece` C extension and Python wrapper version mismatch after Colab base image update | Added `--force-reinstall sentencepiece` to Requirements |
| BUG-0012 | CLIP `position_ids` becomes `HalfTensor` in `transformers/modeling_clip.py`, breaks model load | Added `.long()` cast via `sed` patch in Requirements |
| BUG-0013 | Colab base image update (2026-08-24) cascaded into 10 `ModuleNotFoundError`s (`pyngrok`, `pytorch_lightning`, `kornia`, `open_clip`, `diskcache`, `git`, `piexif`, `pillow_avif`, `clip`) plus a `pytorch_lightning.utilities.distributed` API removal | Added all missing packages (incl. `clip-anytorch`, `dctorch`, `invisible-watermark`, `natsort`, `fairscale`, `fire`) to Requirements; `sed` patch for the `ddpm.py` import path |
| BUG-0014 | `AttributeError: module 'gradio.components' has no attribute 'IOComponent'` — bundled Gradio silently upgraded to 4.x | Pinned `gradio==3.41.2` in Requirements |
| BUG-0015 | Colab base image moved to **Python 3.13**: nine hardcoded `python3.12` paths became silent no-ops, and one bundled `pip install` aborted on `numpy==1.26` (no cp313 wheel), so `pydantic` stayed at 2.x (`AttributeError: __config__` → every script UI failed → webui exited) and `controlnet_aux` was never installed | Derived the interpreter paths from `sysconfig`; one `pip install` per line; `pydantic==1.10.22`; dropped the `numpy` pin; added a post-install package check |

See [`bugs/`](./bugs/) for full details on each fix.

---

## Environment

Tested on:
- Google Colab (2025–2026)
- Python 3.12
- PyTorch 2.10.0+cu128
- CUDA 12.8

---

## Original Project

Original work by [@TheLastBen](https://github.com/TheLastBen).  
Dreambooth paper: https://dreambooth.github.io/  
SD implementation by @XavierXiao: https://github.com/XavierXiao/Dreambooth-Stable-Diffusion  

This fork is published under the same [MIT License](./LICENSE).
