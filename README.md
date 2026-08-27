# fast-stable-diffusion (Community Maintained Fork)

> **This is a community-maintained fork of [TheLastBen/fast-stable-diffusion](https://github.com/TheLastBen/fast-stable-diffusion).**
> The original repository is no longer actively updated. This fork keeps the notebooks working with the current Google Colab environment.

**Status: 🔧 Under active maintenance.**
AUTOMATIC1111 last verified end to end on Google Colab **2026-08-27** — a **150-image batch**
completed, every image delivered to the gallery. Current Colab runtime at that time: **Python 3.13**, PyTorch 2.10, CUDA 12.8.

---

## Open in Colab

| Notebook | Status | Open |
|----------|--------|------|
| **AUTOMATIC1111** | ✅ Maintained (verified 2026-08-27) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/markn333/fast-stable-diffusion/blob/main/fast_stable_diffusion_AUTOMATIC1111.ipynb) |
| **ComfyUI** | ⚠️ Maintained, but behind | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/markn333/fast-stable-diffusion/blob/main/fast_stable_diffusion_ComfyUI.ipynb) |
| **DreamBooth** | ❄️ Not Supported | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/markn333/fast-stable-diffusion/blob/main/fast-DreamBooth.ipynb) |

> ⚠️ **ComfyUI**: last verified 2026-04-30. The 2026-08 Colab recovery (BUG-0013 … BUG-0023) has
> only been applied to the AUTOMATIC1111 notebook so far, so ComfyUI may still hit the Python 3.13
> and dependency issues listed below.

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
| BUG-0016 | `ImportError: cannot import name 'IncEx' from 'pydantic.main'` at the very first import — FastAPI dropped Pydantic v1 support in 0.126.0, and the notebook installed it unpinned | Pinned `fastapi==0.125.0`, the last release carrying the v1 code path (verified in the wheel) |
| BUG-0017 | `creating model quickly: ValueError` on every start, then ~12 GB re-downloaded per session (CLIP 1.71 GB + OpenCLIP 10.2 GB) — an inherited `sed` made webui pass a model name alongside `state_dict={}`, which transformers rejects, dropping it to the slow load path | Reversed the `sed` in the Start cell (the file lives on Drive, so it must be actively undone) |
| BUG-0018 | `OSError: [Errno 107] Transport endpoint is not connected` from `import gradio` — not a gradio problem: the Google Drive FUSE mount dropped while the working directory was on it | Drive cell now verifies the mount and force-remounts a stale one; Start cell refuses to launch on a dead mount and says how to recover |
| BUG-0019 | `'CLIPTextModel' object has no attribute 'text_model'` — the WebUI serves its URL but the model never loads. Colab ships transformers 5.x, which removed the attribute `sd_hijack` reaches into and reshaped the `_load_pretrained_model` signature that webui monkeypatches by position | Pinned `transformers==4.49.0`, the newest 4.x that keeps both (A1111's own `4.30.2` needs tokenizers 0.13.x, which has no Python 3.13 wheel) |
| BUG-0020 | `RuntimeError: The shape of the 2D attn_mask is torch.Size([77, 77]), but should be (1, 1)` — open_clip 3.x switched its transformer to `batch_first`, while sgm still feeds LND tensors | Pinned `open_clip_torch==2.20.0` (A1111's own pin); also restored TheLastBen's `sd_disable_initialization` sed, which is the correct form under transformers 4.x |
| BUG-0021 | Gallery goes blank the moment generation completes, and Save fails with `JSONDecodeError` on `Arguments: ('', [], False, -1)` — the event result never reaches the browser over gradio's websocket queue (starlette/uvicorn have drifted far from what gradio 3.41.2 expects) | Confirmed by test matrix: results arrive over plain HTTP and never over gradio's websocket queue. `Disable_Gradio_Queue` defaults to on, passing `--no-gradio-queue`. Long multi-batch runs then need ngrok, since gradio.live 504s a synchronous POST that takes minutes |
| BUG-0022 | `<Queue ...> is bound to a different event loop` — uvicorn 0.50+ always picks its sans-I/O websocket implementation, which creates an `asyncio.Queue` on its own loop; gradio 3.41.2 touches it from another and the `RuntimeError` is swallowed by a bare `print(e)`, so no event result ever reaches the browser (root cause of BUG-0021: blank gallery, broken PNG Info, failing Save) | Pinned `uvicorn==0.49.0`, the last release defaulting to the legacy implementation |
| BUG-0023 | Root cause of BUG-0021: gradio's queue POSTs to its own `/api/predict` with a client built on **httpx's default 5-second timeout**, so every generation times out, is swallowed silently and is reported to the browser as `process_completed` with `success: False` — the page then correctly refuses to apply a failed result | Patched `queueing.py` to build the queue client with `timeout=None`, and made both silent handlers log |
| BUG-0024 | Ngrok tunnel connects but nothing is reachable through it — `failed to open private leg ... dial tcp [::1]:7860: connection refused`. `pyngrok` turns a bare port into `http://localhost:PORT` and the agent resolves it to the IPv6 loopback, where webui does not listen | Pass the address explicitly: `ngrok.connect('http://127.0.0.1:7860', ...)` |

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
