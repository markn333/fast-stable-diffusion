# fast-stable-diffusion (Community Maintained Fork)

> **This is a community-maintained fork of [TheLastBen/fast-stable-diffusion](https://github.com/TheLastBen/fast-stable-diffusion).**
> The original repository is no longer actively updated. This fork keeps the notebooks working with the current Google Colab environment (PyTorch 2.10+, Python 3.12, CUDA 12.8).

---

## Open in Colab

| Notebook | Status | Open |
|----------|--------|------|
| **AUTOMATIC1111** | ✅ Maintained | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/markn333/fast-stable-diffusion/blob/main/fast_stable_diffusion_AUTOMATIC1111.ipynb) |
| **ComfyUI** | 🔧 Planned | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/markn333/fast-stable-diffusion/blob/main/fast_stable_diffusion_ComfyUI.ipynb) |
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
