# PROGRESS: fast-stable-diffusion

Last Updated: 2026-08-25

---

## Current Phase: 🔧 Maintenance Mode

---

## Progress Summary

| Phase | Task | Status |
|-------|------|--------|
| Phase 1 | Notebook analysis, SPEC.md creation, bug fixes | ✅ Complete |
| Phase 2 | Standalone repository setup | ✅ Complete |
| Phase 3 | Ongoing maintenance | 🔧 Active Maintenance |

---

## Work Log

### 2026-04-27

- ✅ Project created (REQUEST.md, PLAN.md, PROJECT.md)
- ✅ Fixed xformers issue in `fast_stable_diffusion_AUTOMATIC1111.ipynb`
  - Updated for PyTorch 2.10.0 compatibility (added Fix xformers cell)
  - Committed and pushed to GitHub (`8308f00`)
- ✅ Completed full cell analysis of `fast_stable_diffusion_AUTOMATIC1111.ipynb` → SPEC.md created
- ✅ Created bugs/BUG-0001.md (xformers fix record)
- ✅ Pinned gdown==5.2.1 (BUG-0002 / T-009) → commit `822d26c`
- ✅ Added Colab environment check cell to all 3 notebooks (T-007) → commit `3b14375`
- ✅ Replaced 20 hardcoded python3.12 paths with dynamic pyver variable (T-008) → commit `2cdfc95`
- ✅ Completed all Phase 1 tasks (T-000 to T-009)

### 2026-04-28

- ✅ BUG-0003: Fixed device mismatch in sd_emphasis.py → commit `b7afb7a`
- ✅ BUG-0004: Fixed meta tensor issue in sd_disable_initialization.py (assign=True) → commit `adc2c59`
- ✅ BUG-0005: Fixed CLIP position_ids HalfTensor → position_ids.long() → commit `d8d750c`

### 2026-04-30

- ✅ Verified BUG-0005 fix (image generation confirmed working on SDXL + Illustrious models)
- ✅ Removed duplicate xformers cell (id=NONE) → commit `214e5e9`
- ✅ BUG-0006: ComfyUI ngrok guard missing → commit `41b37af`
- ✅ BUG-0007: comfy-aimdo module not found → commit `f2d9729`
- ✅ BUG-0008: simpleeval missing + xformers fix cell for ComfyUI → commit `c162ced`
- ✅ BUG-0009: ComfyUI URL not accessible (tunnel + timing fixes) → commits `14cfd1f`〜`8c16b59`
- ✅ BUG-0010: aiohttp version mismatch (FormData error) → commit `b4e4af2`
- ✅ T-010: ComfyUI PyTorch 2.10 full verification complete — image generation confirmed ✅
- ✅ T-011: xformers fix cell added to ComfyUI notebook
- ✅ T-012: gdown==5.2.1 pin confirmed in ComfyUI Requirements cell
- ✅ README.md updated: ComfyUI status, BUG-0006~0010 added
- ✅ SPEC.md fully translated to English and published
- ✅ Mothership internal files removed from git tracking (gitignore)
- ✅ Notebook title URLs updated from TheLastBen → markn333
- ✅ bugs/BUG-0008.md, BUG-0009.md, BUG-0010.md created
- ✅ X post draft prepared (T-013 — pending manual post by markn3)

### 2026-07-25

- ✅ BUG-0011: `sentencepiece` C extension / Python wrapper version mismatch after Colab base image update
  - Added `--force-reinstall sentencepiece` to AUTOMATIC1111 Requirements cell → commits `ae207aa`, `ea729a2`, `0ebff67`
- ✅ README.md What's Fixed table updated (BUG-0011 added)
- ✅ BUG-0012: CLIP `position_ids` HalfTensor in `transformers/modeling_clip.py` — added `.long()` sed patch to Requirements cell

### 2026-08-24 – 2026-08-25

- ✅ BUG-0013 (consolidated): Colab base image update removed `pyngrok`, `pytorch_lightning`, `kornia`, `open_clip`, `diskcache`, `git`, `piexif`, `pillow_avif`, `clip` and broke `pytorch_lightning.utilities.distributed` (v2.x API removal) — added all missing packages to Requirements cell; `sed` patch for `ddpm.py` import path
- ✅ Proactively checked k-diffusion/sgm upstream `requirements.txt` — swapped `openai-clip` for `clip-anytorch`; added `dctorch`, `invisible-watermark`, `natsort`, `fairscale`, `fire` to Requirements cell
- ✅ Consolidated former BUG-0013~BUG-0022 (individually filed) into a single BUG-0013 record since they share one root cause (2026-08-24 Colab base image update)
- ✅ BUG-0014: bundled Gradio silently upgraded to 4.x, removing `gr.components.IOComponent` used by webui's `gradio_extensons.py` — pinned `gradio==3.41.2` in Requirements cell

### 2026-08-25

- ✅ Imported into Claude Mothership (managed-project path: existing PROJECT/PLAN/PROGRESS kept as-is). Copilot → Claude environment confirmed already converted; added `CLAUDE.md` / `.claude/` to `.gitignore` so the charter stays out of the public repo (commit `03f610c`)
- ✅ BUG-0015: Colab base image moved to **Python 3.13**. Two silent defects turned that into a startup crash:
  - nine hardcoded `python3.12` paths in the Requirements cell became no-ops (including the BUG-0005/0012 `position_ids.long()` patch, whose target line still exists in current transformers)
  - one bundled `pip install` aborted on `numpy==1.26` (no cp313 wheel, no pure-python fallback), so `pydantic`, `controlnet_aux`, `wandb` and `scipy` were all skipped
  - → pydantic stayed at 2.x → `modules/api/models.py` `__config__` AttributeError → every script UI failed → `ui.py:476` got `steps = None` → gradio `set_event_trigger` crashed → webui exited
  - Fix: derive `sitepkg`/`stdlib` from `sysconfig`, one `pip install` per line, `pydantic==1.10.22` (first 1.10.x with cp313 wheels), dropped the `numpy` pin, and added a post-install package check that prints what is missing
- ✅ BUG-0016 (found by the first Colab run after the BUG-0015 fix): `ImportError: cannot import name 'IncEx' from 'pydantic.main'` at the very first import. The pydantic downgrade succeeded (the error names the cp313 `.so` of 1.10.22), but **FastAPI removed pydantic v1 support in 0.126.0** and the notebook installed fastapi unpinned, so Colab had 0.141.x
  - Fix: pinned `fastapi==0.125.0` — the last release that still carries the v1 code path, confirmed by unpacking the wheel (`_compat/v1.py`, and a `types.py` that defines `IncEx` locally). Chosen over A1111's own `fastapi==0.94.0` because that drags `starlette<0.27`, a 2023 stack never run on Python 3.13, whereas the previous Colab run proved modern fastapi/starlette import fine there
  - Audited every package in the stack against its PyPI `requires_dist`: **fastapi was the only remaining hard pydantic-v2 requirement**. `gradio` and `wandb` also demand v2 at their latest versions but are pinned (wandb additionally `--no-deps`); `transformers` needs v2 only under extras that are not installed. Also re-checked every pin in the notebook for Python 3.13 installability — none now requires a source build. Details in bugs/BUG-0016.md
- ⚠️ Not yet verified on Colab (T-015). `sd_disable_initialization.py` `state_dict` ValueError left alone pending an A/B run (T-014)
