# SPEC: fast-stable-diffusion Notebooks

- Created: 2026-04-27
- Last updated: 2026-04-30
- Target files:
  - `fast_stable_diffusion_AUTOMATIC1111.ipynb`
  - `fast_stable_diffusion_ComfyUI.ipynb`
  - `fast-DreamBooth.ipynb`
- Runtime: Google Colab (GPU required)

---

# AUTOMATIC1111 Notebook Spec

File: `fast_stable_diffusion_AUTOMATIC1111.ipynb`

## Overview

A notebook for launching the AUTOMATIC1111 Stable Diffusion WebUI on Google Colab.
Models and settings are stored on Google Drive and persist across sessions.

---

## Cell Structure

### Cell 1: Connect Google Drive

**Purpose**: Mount Google Drive and establish the base storage path.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Shared_Drive` | string | `""` | Shared drive name. Leave empty to use MyDrive. |

**Flow**:
1. Mount via `google.colab.drive.mount('/content/gdrive')`
2. If `Shared_Drive` is set, use `Shareddrives/` path
3. Set `mainpth` global variable (`MyDrive` or `Shareddrives/{name}`)

**Side effect**: Sets `mainpth` global variable.

---

### Cell 2: Install/Update AUTOMATIC1111 repo

**Purpose**: Clone the AUTOMATIC1111 WebUI repository to Google Drive (first run) or update it (subsequent runs).

**Dependencies**: `git`, `six`, `tqdm`, `requests`, `base64`

**Flow**:
1. If Google Drive is not connected, use local Colab storage temporarily
2. If the WebUI repo does not exist, run `git clone`
3. If it already exists, run `git reset --hard && git pull`

**Storage path**: `/content/gdrive/{mainpth}/sd/stable-diffusion-webui/`

---

### Cell 3: Requirements

**Purpose**: Install required packages and binaries.

**Key packages**:

| Package | Version | Notes |
|---------|---------|-------|
| `gradio` | Remove then reinstall | Python 3.12 path hardcoded |
| `wandb` | `0.15.12` | Pinned |
| `pydantic` | `1.10.2` | Pinned |
| `numpy` | `1.26` | Pinned |
| `scipy` | `1.15.3` | Pinned |
| `controlnet_aux` | Latest | `--no-deps` |
| `diffusers` | Latest | `--no-deps` |
| `libtcmalloc` | `gperftools-2.5` | Memory optimization (built on first run only) |

**Patches applied**:
- `warnings.py`: Suppress warning text output
- `pytorch_lightning/__init__.py`: Remove WandbLogger import
- `wandb/sdk/lib/retry.py`: Remove `ContextCancelledError` import
- `pydantic/typing.py`: Patch `recursive_guard` argument handling

**Warning**: Python path is hardcoded to `/usr/local/lib/python3.12/`. Must be updated if the Python version changes.

---

### Cell 4: Fix xformers (Added 2026-04-27)

**Purpose**: Reinstall xformers compatible with the current PyTorch version.

**Background**: The Colab PyTorch version was updated from `2.9.0 -> 2.10.0`, causing a binary mismatch with the pre-installed xformers.

**Fix**:
```
pip install xformers --pre --force-reinstall --no-deps -q
```

---

### Cell 5: Model Download/Load

**Purpose**: Download or specify a Stable Diffusion model.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Use_Temp_Storage` | boolean | `False` | `True`: Colab temp storage / `False`: Google Drive |
| `Model_Version` | select | `"SDXL"` | Preset model selection |
| `PATH_to_MODEL` | string | `""` | Full path to a custom model |
| `MODEL_LINK` | string | `""` | Civitai / HuggingFace / Google Drive link |

**Preset models**:

| Version | File | Source |
|---------|------|--------|
| `SDXL` | `sd_xl_base_1.0.safetensors` | HuggingFace |
| `1.5` | `v1-5-pruned-emaonly.safetensors` | HuggingFace |
| `v1.5 Inpainting` | `sd-v1-5-inpainting.ckpt` | HuggingFace |
| `V2.1-768px` | `v2-1_768-ema-pruned.safetensors` | HuggingFace |

**Supported download sources**: Civitai / Google Drive / HuggingFace / Any direct URL

---

### Cell 6: Download LoRA

**Purpose**: Download a LoRA model.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `LoRA_LINK` | string | `""` | Civitai / GDrive / other link. No-op if empty. |

**Storage path**: `.../models/Lora/`

---

### Cell 7: ControlNet

**Purpose**: Install and update the ControlNet extension and models.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `XL_Model` | select | `"None"` | XL model (None/All/Canny/Depth/Sketch/OpenPose/Recolor) |
| `v1_Model` | select | `"None"` | v1.x model |
| `v2_Model` | select | `"None"` | v2.x model |

**Extension**: `sd-webui-controlnet` (by Mikubill) installed to GDrive.

---

### Cell 8: Start Stable-Diffusion

**Purpose**: Launch the WebUI and expose a browser-accessible URL.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Ngrok_token` | string | `""` | ngrok token. If empty, uses Gradio share URL. |
| `User` | string | `""` | Gradio Basic Auth username (optional) |
| `Password` | string | `""` | Gradio Basic Auth password (optional) |

**Launch flags**:
```
--api --disable-safe-unpickle --enable-insecure-extension-access
--no-download-sd-model --no-half-vae --xformers
--disable-console-progressbars --skip-version-check
```

**Tunnel method**:
- No ngrok token: `--share` (Gradio share URL)
- With ngrok token: ngrok tunnel

---

## Known Issues / Limitations

| # | Issue | Impact | Status |
|---|-------|--------|--------|
| 1 | Python path hardcoded to `python3.12` | All patches fail if Python version changes | Workaround available (see below) |
| 2 | gdown v6 breaking change | All `gdown` calls fail | Critical — workaround available (see below) |
| 3 | `RuntimeError: dictionary changed size during iteration` | First model switch fails; succeeds on retry | A1111 internal issue — tolerated |

### Issue 1: Hardcoded Python path

Add the following to the top of the Requirements cell and replace `python3.12` with `$pyver`:

```python
import sys
pyver = f"python{sys.version_info.major}.{sys.version_info.minor}"
# Then use $pyver in all !sed / !rm / !wget shell commands
```

IPython `$variable` expansion is an official feature and works in Colab.

### Issue 2: gdown v6 breaking changes

gdown v6.0.0 was released in mid-April 2026. Colab may use the latest version automatically.

| Change | Detail |
|--------|--------|
| `get_url_from_gdrive_confirmation` | Deprecated in v5, removed in v6 |
| `gdown.download(..., fuzzy=True)` | `fuzzy` parameter removed (auto-detect now) |
| `!gdown --fuzzy ...` | `--fuzzy` CLI flag removed |
| `download()` return value | On failure: `None` -> raises `DownloadError` |

**Safest short-term fix**: Pin `gdown==5.2.1` in the Requirements cell.

---

## Verified Environments

| Date | PyTorch | CUDA | Python | Status |
|------|---------|------|--------|--------|
| 2026-04-27 | 2.10.0+cu128 | 12.8 | 3.12.13 | Working (with Fix xformers cell) |

---

# ComfyUI Notebook Spec

File: `fast_stable_diffusion_ComfyUI.ipynb`

## Overview

A notebook for launching ComfyUI (node-based Stable Diffusion GUI) on Google Colab.
Provides browser access via a SSH tunnel (pinggy.io) or ngrok.

---

## Cell Structure

### Cell 1: Connect Google Drive

Same spec as AUTOMATIC1111. Sets the `mainpth` variable.

---

### Cell 2: Install/Update ComfyUI repo

**Purpose**: Clone or update the ComfyUI repository on Google Drive.

**Flow**:
1. Clone `TheLastBen/diffusers` locally
2. `git clone` ComfyUI to `/content/gdrive/{mainpth}/ComfyUI` (first run) or `git pull` (subsequent)
3. Initialize cache directories; set `TRANSFORMERS_CACHE` and `TORCH_HOME`

**Storage path**: `/content/gdrive/{mainpth}/ComfyUI/`

---

### Cell 3: Requirements

**Purpose**: Install all required packages.

| Package | Version | Notes |
|---------|---------|-------|
| `diffusers` | Latest (`-U`) | |
| `accelerate` | Latest (`-U`) | |
| `transformers` | Latest (`-U`) | |
| `av` | Latest | |
| `comfyui-frontend-package` | Latest | |
| `comfyui-workflow-templates` | Latest | |
| `alembic` | Latest | |
| `comfy-aimdo` | Latest | Required by ComfyUI main.py |
| `simpleeval` | Latest | Required by nodes_math.py |
| `aiohttp` | Latest (`-U`) | Must be upgraded; older versions break FormData |
| `wandb` | `0.15.12` | Pinned |
| `pydantic` | `2.10.5` | Pinned (v2 — unlike AUTOMATIC1111 which uses v1) |
| `numpy` | `1.26` | Pinned |
| `scipy` | `1.15.3` | Pinned |
| `libtcmalloc` | `gperftools-2.5` | Memory optimization (built on first run only) |

**Patches applied**:
- `warnings.py`: Suppress warning text output
- `jax/_src/deprecations.py`: Suppress AttributeError raises
- `pydantic/typing.py`: Patch `recursive_guard` argument handling

**Differences from AUTOMATIC1111**:
- pydantic pinned to v2.10.5 (AUTOMATIC1111 uses v1.10.2)
- tensorflow packages removed via `rm -r`

**Warning**: Python path hardcoded to `/usr/local/lib/python3.12/` in multiple places.

---

### Cell 4: Fix xformers (Added 2026-04-27)

Same purpose as AUTOMATIC1111 cell 4. Reinstalls xformers for PyTorch 2.10.

```
pip install xformers --pre --force-reinstall --no-deps -q
```

---

### Cell 5: Model Download/Load

**Purpose**: Download or specify a model for ComfyUI.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Use_Temp_Storage` | boolean | `False` | `True`: Colab temp storage / `False`: Google Drive |
| `Model_Version` | select | `"SDXL"` | Preset model selection |
| `MODEL_LINK` | string | `""` | Civitai / HuggingFace / Google Drive link |

**Preset models**:

| Version | File | Source |
|---------|------|--------|
| `SDXL` | `sd_xl_base_1.0.safetensors` | HuggingFace / stabilityai |
| `1.5` | `v1-5-pruned-emaonly.safetensors` | HuggingFace / runwayml |
| `v1.5 Inpainting` | `sd-v1-5-inpainting.ckpt` | HuggingFace / runwayml |
| `flux` | `flux1-dev-fp8.safetensors` | HuggingFace / lllyasviel |

**Storage path**: `/content/gdrive/{mainpth}/ComfyUI/models/checkpoints/` (or `/content/temp_models/`)

**Supported download sources**: Civitai / Google Drive / HuggingFace / Any direct URL

---

### Cell 6: Download LoRA

**Purpose**: Download a LoRA model.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `LoRA_LINK` | string | `""` | Civitai / GDrive / other link. No-op if empty. |

**Storage path**: `/content/gdrive/{mainpth}/ComfyUI/models/loras/`

---

### Cell 7: Start ComfyUI

**Purpose**: Launch the ComfyUI server and expose a browser-accessible URL.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Ngrok_Token` | string | `""` | ngrok token. If empty, uses pinggy.io SSH tunnel. |

**Launch command**:
```
python -u main.py --listen --port 666 --enable-cors-header
```

**Tunnel behavior**:
- With ngrok token: pyngrok tunnel on port 666
- Without ngrok token: pinggy.io SSH tunnel (`ssh -p 443 -R 0:localhost:666 a.pinggy.io`)

**URL detection**: ComfyUI stdout is streamed line by line. The URL is printed only after
`"To see the GUI go to:"` appears in the output, ensuring ComfyUI is fully ready before
the link is shown.

**OpenGL fix**: `nodes_glsl.py` is fetched from the upstream repository and patched to
immediately return from `_check_opengl_availability()`, suppressing OpenGL-related
tracebacks on Colab (which has no GPU-accessible OpenGL context).

---

## Known Issues / Limitations

| # | Issue | Impact | Status |
|---|-------|--------|--------|
| 1 | Python path hardcoded to `python3.12` | Patches fail if Python version changes | Workaround available (see AUTOMATIC1111 spec) |
| 2 | gdown v6 breaking change | `gdown` calls fail | Critical — see AUTOMATIC1111 spec |

---

## Verified Environments

| Date | PyTorch | CUDA | Python | Status |
|------|---------|------|--------|--------|
| 2026-04-30 | 2.10.0+cu128 | 12.8 | 3.12.13 | Working — image generation confirmed (SDXL base 1.0) |

---

# DreamBooth Notebook Spec

File: `fast-DreamBooth.ipynb`

> **This notebook is not actively maintained in this fork.**
> The information below is provided for reference only.

## Overview

A notebook for fine-tuning Stable Diffusion models using the DreamBooth technique.
Supports SD 1.5 / v2.1. Requires approximately 10 training images of a subject.

---

## Cell Structure

### Cell 1: Connect Google Drive

Nearly identical to AUTOMATIC1111/ComfyUI. `mainpth` is fixed to `MyDrive` (no Shared Drive support).

---

### Cell 2: Dependencies

**Purpose**: Install all packages required for DreamBooth training.

| Package | Version | Notes |
|---------|---------|-------|
| `accelerate` | `0.12.0` | Pinned (specific version required) |
| `gradio` | `3.16.2` | Pinned (old version) |
| `wandb` | `0.15.12` | Pinned |
| `pydantic` | `1.10.2` | Pinned |
| `numpy` | `1.26` | Pinned |
| `scipy` | `1.15.3` | Pinned |
| `TheLastBen/diffusers` | main | git clone |
| `libtcmalloc` | `gperftools-2.5` | Memory optimization |

**Patches applied**:
- `diffusers/utils/dynamic_modules_utils.py`: Remove `cached_download` import
- `pytorch_lightning/loggers/__init__.py`: Remove `WandbLogger` import
- `wandb/sdk/lib/retry.py`: Remove `ContextCancelledError` references
- `pydantic/typing.py`: Patch `recursive_guard` argument handling

---

### Cell 3 (markdown): Model Download header

---

### Cell 4: Model Download

**Purpose**: Specify the base model for fine-tuning.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Model_Version` | select | `"1.5"` | Preset model |
| `Path_to_HuggingFace` | string | `""` | HuggingFace `profile/model` format |
| `MODEL_PATH` | string | `""` | Absolute path to a local model file |
| `MODEL_LINK` | string | `""` | Civitai / GDrive / other link |

**Preset models**:

| Version | Source | Notes |
|---------|--------|-------|
| `1.5` | `v1-5-pruned-emaonly.safetensors` / HuggingFace | Converted to diffusers format |
| `V2.1-512px` | HuggingFace / stabilityai/stable-diffusion-2-1-base | sparse checkout |
| `V2.1-768px` | HuggingFace / stabilityai/stable-diffusion-2-1 | sparse checkout |

**HuggingFace private model auth**: Write token to `/content/gdrive/MyDrive/Fast-Dreambooth/token.txt`.

**Storage path**: `/content/stable-diffusion-v1-5/` (or v2-512, v2-768, stable-diffusion-custom)

---

### Cell 5 (markdown): DreamBooth header

---

### Cell 6: Create/Load a Session

**Purpose**: Create or resume a training session.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Session_Name` | string | `""` | Session name (required; spaces converted to underscores) |
| `Session_Link_optional` | string | `""` | Import a session from another GDrive link |

**Session management**:
- `SESSION_DIR` = `/content/gdrive/MyDrive/Fast-Dreambooth/Sessions/{Session_Name}/`
- If an existing session is found, converts and loads the trained model in diffusers format
- For new sessions, creates the directory only

---

### Cell 7: Instance Images

**Purpose**: Upload and manage training instance images.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Remove_existing_instance_images` | boolean | `True` | Delete existing images before adding |
| `IMAGES_FOLDER_OPTIONAL` | string | `""` | Folder path (upload via widget if empty) |
| `Smart_Crop_images` | boolean | `True` | Enable automatic cropping |
| `Crop_size` | select | `512` | Crop size in pixels (512-1024) |

**Image naming convention**: `{identifier} (N).{ext}` (e.g., `phtmejhn (1).jpg`)

---

### Cell 8: Captions (optional)

**Purpose**: GUI for manually creating and editing per-image captions.

- Image selection UI via ipywidgets
- Text area for editing and saving captions per image
- Not required for subject/face training

---

### Cell 9: Training / Start DreamBooth

**Purpose**: Run DreamBooth fine-tuning.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Resume_Training` | boolean | `False` | Continue from previous checkpoint |
| `UNet_Training_Steps` | number | `1500` | UNet training steps (0 to disable) |
| `UNet_Learning_Rate` | select | `2e-6` | UNet learning rate |
| `Text_Encoder_Training_Steps` | number | `350` | Text encoder training steps (0 to disable) |
| `Text_Encoder_Learning_Rate` | select | `1e-6` | Text encoder learning rate |
| `Offset_Noise` | boolean | `False` | Recommended for style training |
| `External_Captions` | boolean | `False` | Use `.txt` captions |
| `Resolution` | select | `"512"` | Training resolution (512-1024) |
| `Save_Checkpoint_Every_n_Steps` | boolean | `False` | Save intermediate checkpoints |
| `Save_Checkpoint_Every` | number | `500` | Checkpoint interval in steps |
| `Start_saving_from_the_step` | number | `500` | Step to begin saving from |
| `Disconnect_after_training` | boolean | `False` | Auto-disconnect Colab after training |

**Training flow**:
1. Train UNet only, or UNet + text encoder
2. Intermediate results saved as diffusers format to `OUTPUT_DIR`
3. Final output: diffusers -> `.ckpt` saved to `SESSION_DIR`

**Output path**: `/content/gdrive/MyDrive/Fast-Dreambooth/Sessions/{Session_Name}/{Session_Name}.ckpt`

---

## Known Issues / Limitations

| # | Issue | Impact | Status |
|---|-------|--------|--------|
| 1 | Python path hardcoded to `python3.12` | Patches fail if Python version changes | Workaround available (see AUTOMATIC1111 spec) |
| 2 | `gradio==3.16.2` is very old | May be incompatible with newer Gradio APIs | Needs monitoring |
| 3 | `accelerate==0.12.0` is very old | Compatibility risk with newer PyTorch | Needs monitoring |
| 4 | gdown v6 breaking change | `gdown` calls fail | Critical — see AUTOMATIC1111 spec |
| 5 | `mainpth` fixed to MyDrive | Shared Drive not supported | By design |

---

## Verified Environments

| Date | Status |
|------|--------|
| 2026-04-27 | Analyzed (runtime testing not performed — not maintained) |
