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
- ✅ **Colab run confirms BUG-0015 + BUG-0016 are fixed**: the WebUI reached `Running on public URL: https://…gradio.live` / `Connected`, ControlNet v1.1.455 loaded and registered its UI callback, and not one `Error creating UI` appeared. Startup time 522.2s (create ui 156.2s, load scripts 135.2s). Image generation still unconfirmed (T-015)
- ✅ BUG-0017 (T-014, closed): that same run showed `creating model quickly: ValueError` followed by 1.71 GB of CLIP and **10.2 GB of OpenCLIP** being downloaded — ~12 GB per session
  - Settled by reading the sources rather than an A/B run. transformers rejects exactly `state_dict is not None and pretrained_model_name_or_path is not None`; A1111 passes `None` with `config=<name>, state_dict={}` **on purpose** so the text encoder is built from config without fetching weights. TheLastBen's inherited `sed` replaced that `None` with the model name
  - The cost is structural: `sd_models.py` catches the failure and retries **outside** `DisableInitialization`, which is also the context that neutralises open_clip (`pretrained=None`) — so one `sed` on one argument pulls down both encoders
  - Fix: the Start cell now **reverses** the sed rather than dropping it. `sd_disable_initialization.py` lives on Google Drive and earlier sessions already rewrote it in place, so removing the line would have left every existing install patched. Path also moved from a hardcoded `/content/gdrive/MyDrive/…` to `$mainpth`/`$blsaphemy`, which silently missed Shared Drive users
  - Risk is bounded: the surrounding `except Exception` means a failure just takes the slow path — today's behaviour
- ✅ BUG-0018: the next run died with `OSError: [Errno 107] Transport endpoint is not connected` inside `import gradio`. **Not a dependency problem** — errno 107 is `ENOTCONN` from FUSE: the Google Drive mount dropped, and since the Start cell `%cd`s into the Drive path, the process's working directory was on it. `os.getcwd()` in `gradio/deploy_space.py` was simply the first syscall to touch it, so the traceback blames gradio
  - A dead FUSE mount is not an unmounted one: `/content/gdrive` still exists and `drive.mount()` still reports success; only a real access reveals it. Plausibly provoked by the ~12 GB the previous session pushed through the mount (BUG-0017)
  - Mitigation — the mount cannot be made reliable from inside the notebook, so fail clearly and self-heal: the Drive cell now probes it with `os.listdir` and force-remounts a stale mount; the Start cell steps off the mount (`os.chdir('/content')`, since even printing can fail when the cwd is dead) and refuses to launch with a recovery instruction instead of a cryptic errno
  - Guard not yet exercised on Colab

### 2026-08-26

- ✅ **BUG-0017 confirmed working**: the run shows `CLIPTextModel_from_pretrained(None, ...)` — the reverse sed landed. Startup reached the public URL again (530.7s)
- ✅ BUG-0019: the WebUI serves its URL but **the model never loads**. `Stable diffusion model failed to load`, fatally at `sd_hijack.py:218` → `AttributeError: 'CLIPTextModel' object has no attribute 'text_model'`
  - Root cause: **Colab's base image now ships transformers 5.x**, and webui is built against 4.x. Two collisions: (a) v5 removed `CLIPTextModel.text_model`, which `sd_hijack` needs to install the embedding hijack — fatal; (b) webui monkeypatches `_load_pretrained_model` **by argument position**, and v5 made the 4th positional arg `load_config`, so the patch wrote `'/'` into it → `'str' object has no attribute 'hf_quantizer'`
  - Fix: pinned `transformers==4.49.0`. Chosen by unpacking wheels, not release notes: it is the **last** version whose 4th positional arg is still `resolved_archive_file` (4.50.3 / 4.51.3 / 4.52.4 / 4.53.3 all moved it), it still defines `self.text_model = CLIPTextTransformer(`, it is called positionally so the patch lands, and its `tokenizers<0.22,>=0.21` resolves to a **cp39-abi3** wheel that covers Python 3.13
  - A1111's own `transformers==4.30.2` is not an option: tokenizers 0.13.3 ships only cp37–cp311 wheels, so pip would have to build it from Rust on 3.13
  - Installed **with** deps deliberately — transformers 4.x needs `huggingface-hub<1.0` while Colab carries hub 1.x
  - Expected side effect: with the argument alignment restored, the fast path works again, so `DisableInitialization` stays active and BUG-0017's intended ~12 GB saving should finally materialise — it was correct but masked by this
- ✅ **BUG-0019 confirmed working**: `CLIPTextModel.text_model` no longer raises and the traceback now comes from transformers 4.x code paths. Startup 472.0s
- ✅ BUG-0020: the model now builds and loads weights, then dies computing the empty prompt — `RuntimeError: The shape of the 2D attn_mask is torch.Size([77, 77]), but should be (1, 1)`
  - Cause: `open_clip_torch` was unpinned, so Colab took **3.3.0**, which switched the transformer to `batch_first`. sgm's `text_transformer_forward` still hands over LND tensors with a `[77,77]` causal mask, so the sequence axis lands where the batch axis is expected and PyTorch demands a `(1,1)` mask. Counted in the wheels: 2.20.0 and 2.24.0 have 0 `batch_first` references and 9 `permute(1, 0, 2)`; 3.3.0 has 36 and 3
  - Fix: pinned `open_clip_torch==2.20.0` — A1111's own pin, pure-python wheel, deps all already satisfied (`protobuf<4` agrees with the existing `protobuf==3.20.0`)
- ↩️ **BUG-0017's fix reverted (by BUG-0020)**. Reversing TheLastBen's sed was right for transformers 5.x, but BUG-0019 pinned transformers to 4.49.0 and under 4.x the premise inverts: the 5.x `state_dict cannot be passed together with a model name` check does not exist, and 4.49 resolves the config from `pretrained_model_name_or_path` rather than from `config` — so webui's `None` requests `https://huggingface.co/None/...` → 401 and the fast path dies anyway (seen in this run as `OSError: None is not a local folder`). The sed is restored, and the cell now records that it is **coupled to the transformers pin**: change one, re-evaluate the other
  - What survives from BUG-0017: the `$mainpth`/`$blsaphemy` path fix, and the point that a sed against a file on Drive is persistent state
- ⚠️ Four bugs in a row now share one shape: **the notebook left unpinned what A1111 pins**. Remaining unpinned entries from `requirements_versions.txt` — `kornia==0.6.7`, `einops==0.4.1`, `safetensors==0.4.2`, `pytorch_lightning==1.9.4` — are the next candidates
- ✅ **BUG-0020 confirmed working — the model loads and images generate.** `Model loaded in 266.9s`, twenty batches of `100% 20/20`, PNGs written to Drive under `outputs/txt2img-images/`. Startup 473.9s
- ✅ BUG-0021: generation works but **nothing comes back to the page** — the gallery blanks the instant generation completes, and Save fails with `JSONDecodeError`
  - The Save traceback gave it away without any theorising: `*** Arguments: ('', [], False, -1)` are the bound inputs `[generation_info, result_gallery, ...]`, so `generation_info` is `''` and **the gallery is genuinely `[]`**. Both are txt2img outputs, so the event result never arrived in the browser — the components still held their initial values and the browser sent those back
  - The blank gallery is the same fact: A1111 draws the live preview as an overlay on top of the gallery and removes it at the end, revealing a gallery that was never populated. One cause, both symptoms, and it is **transport**, not generation
  - gradio 3.41.2 delivers results over a websocket (`@app.websocket("/queue/join")`, `starlette.websockets.WebSocketState`). The surrounding stack has drifted: starlette **0.40–0.50** via our `fastapi==0.125.0` where A1111 pins `fastapi==0.94.0` (starlette 0.26.x); gradio pins `websockets<12.0` while uvicorn 0.52.4 unconditionally selects its sans-I/O implementation. A dropped websocket message raises nothing, which is why completion was silent
  - Workaround: A1111's own `--no-gradio-queue` ("causes the webpage to use http requests instead of websockets; was the default in earlier versions"), exposed as a `Disable_Gradio_Queue` notebook parameter defaulting to on. A1111's progress bar and live preview come from its own `/internal/progress` HTTP endpoint, so nothing is lost. Unticking the box is the single-variable A/B that confirms or clears the websocket
  - Also in that log and **not** a bug: `xdg-open: no method available for opening 'outputs/txt2img-images'` — that is the Open folder button asking a headless VM for a file manager
- ❌ **BUG-0021 A/B negative**: with `--no-gradio-queue` the failure is byte-identical (`Arguments: ('', [], False, -1)`), so results fail to arrive over plain HTTP too. **The websocket transport is cleared** — and with it starlette, uvicorn and the `websockets` pin question; the BUG-0016 `fastapi==0.125.0` choice is not implicated here after all
  - Generation stays healthy (`Model loaded in 518.6s`, eight batches of `100% 20/20`), and txt2img produces no `*** Error completing request` of its own — only the Save button does, and only because the page handed it empty values
  - Next candidate: a third-party extension breaking the page's JS. This install runs ADetailer, sd-webui-controlnet, Civitai Helper, Tag Autocomplete and WAI-NSFW-illustrious-character-select — the last of which already threw during UI construction earlier in this saga. A JS error there stops the gallery updating while leaving the server silent, which is exactly this bug's shape
  - Added `Test_Disable_Extensions` (`--disable-extra-extensions`) to the Start cell, and made it **print the launch flags** before starting — an A/B is only worth anything if the flag actually reached the launch line
- 🔎 **BUG-0021: the browser console names it** — `/run/predict:1 Failed to load resource: the server responded with a status of 504`. With `--no-gradio-queue` gradio submits a generation as one long synchronous POST; that run was eight batches at ~43 s each (~6 minutes) and the gradio.live tunnel cut it off. The server never learns the client gave up, which is why it generates, saves and stays silent; the page keeps its initial values and Save posts them back as `('', [], False, -1)`. The live preview survived because `/internal/progress` is short polling
  - ↩️ **Previous A/B withdrawn.** "The websocket is not the cause" was wrong: both paths fail, but for different reasons — queue-off fails with a 504 that is an artefact of the workaround itself. Identical symptoms, different mechanisms; nothing about starlette, uvicorn, `websockets` or the BUG-0016 fastapi pin was actually cleared
  - `Disable_Gradio_Queue` default reverted to **False** — `--no-gradio-queue` cannot work for multi-batch runs behind a tunnel, since the queue exists precisely to avoid holding one HTTP request open for a whole generation
  - **Duration is the variable never controlled**: every failing run so far has been a long multi-batch job. Next test is one image at low steps (~25 s) with the queue on
- 🎯 **BUG-0022 — root cause of BUG-0021 found.** Launching with `--disable-extra-extensions` removed the extension chatter and exposed a line that had been printing all along: `<Queue at 0x… maxsize=0 tasks=2> is bound to a different event loop`, once per connection
  - **uvicorn 0.50+ always selects its sans-I/O websocket implementation** — `auto.py` is not a version check, it picks sansio whenever `websockets` imports — and that implementation creates a per-connection `asyncio.Queue` **on uvicorn's own event loop**. gradio 3.41.2 starts the server in a thread and runs its queue coroutine on a different loop, so touching that Queue raises `RuntimeError`
  - **gradio swallows it**: `gradio/queueing.py:464` is a bare `print(e)`. One line of console output, no traceback, no error to the client, and a result that is simply never delivered. That is the whole of BUG-0021 — blank gallery, broken PNG Info, Save receiving `('', [], False, -1)` — while clicks in the other direction kept working
  - Fix: pinned **`uvicorn==0.49.0`**, the last release defaulting to the legacy implementation (0.50.0 is the first to switch; found by bisecting every release between 0.36.0 and 0.52.4)
  - **Settled by experiment, not by reading wheels.** Wheel inspection had suggested the legacy implementation could not even import against websockets 11; installing both combinations in a throwaway venv showed 0.52.4 → sansio with a per-connection Queue, 0.49.0 → legacy without one. The archaeology was wrong and the experiment took seconds
  - Extensions were never the cause — but disabling them is what made the bug findable
- ⚠️ **BUG-0022 not confirmed.** The run after pinning `uvicorn==0.49.0` fails identically, and it is unknown whether the pin was installed — the pasted log starts at the Start cell, and re-running Requirements costs ten minutes. Also, reviewing the logs honestly: the `<Queue ...> is bound to a different event loop` lines appeared **only** in the `--disable-extra-extensions` run, while the failure is present in every run, so "extensions buried them" is not supported. The loop-bound defect is real and was reproduced locally, but its role here is now uncertain
  - The Requirements cell now **always prints the resolved versions** of every pinned package, so no future log can leave "did Requirements even run?" open
  - **Gap found in the test matrix**: queue-off has only ever been tried on a run long enough to hit the tunnel's 504, so HTTP delivery has never had a fair trial. Short generation + `Disable_Gradio_Queue` on is the missing cell — it separates "websocket-specific fault" from "fault upstream of transport"
- ❌ **BUG-0022 ruled out.** The Start cell's new stack line confirms `uvicorn==0.49.0` is installed and the failure is unchanged, so the sans-I/O websocket implementation does not explain BUG-0021. The pin is kept — the loop-bound defect is real and reproduced locally — but it is not the cause
- 🔎 **The ASGI layer becomes visible for the first time**: `starlette==0.50.0`, `anyio==4.14.2`. Nobody has ever pinned either, and they are the layer gradio's queue actually runs on (`run_coro_in_background`, blocking portals, the websocket state machine) — roughly twenty-four minor versions ahead of the code driving them. They arrived via the BUG-0016 `fastapi==0.125.0` choice, whose only selection criterion was pydantic-v1 support. Now the largest unexamined variable in the stack
- ⏸️ The one-checkbox test (**queue off + short generation**) is still unfilled — `Launch flags: --share` shows the box was off again. It stays the cheapest decisive experiment: no reinstall, ~30 s, and it separates a websocket-specific fault from one upstream of transport before anything else is changed
- ✅ **BUG-0021 diagnosed.** The missing test-matrix cell finally ran — `Launch flags: --share --no-gradio-queue`, one 49 s generation — and **the image appears in the gallery**, with no `*** Error completing request`
  - **gradio's websocket queue is the fault.** Plain HTTP delivery works; nothing upstream of transport is broken. Blank gallery, dead PNG Info and `Save` receiving `('', [], False, -1)` were all one undelivered result, specific to the queue path
  - Shipped: `Disable_Gradio_Queue` now defaults to **True** — strictly better, since with the queue on nothing works at all. The trade-off is stated in the cell: each generation becomes one long synchronous POST and gradio.live 504s once a run takes minutes, so **long multi-batch jobs need `Ngrok_token`** until the queue is fixed
  - Still unexplained: why the queue fails. Ruled out — generation, saving, extensions, duration, transport being dead either way, uvicorn's websocket implementation, and the fastapi/transformers/open_clip pins. Remaining suspect: `starlette==0.50.0` / **`anyio==4.14.2`**, against the starlette 0.26/0.27 and anyio 3.x gradio 3.41.2 was written for — its queue is built on exactly the primitives anyio 4 reworked
  - Next single-variable experiment, now cheap (warm restart was 31 s): `!pip install anyio==3.7.1 -q`, untick the switch, generate once. Every starlette in range including 0.50.0 declares `anyio<5,>=3.6.2`, so anyio moves alone
- 📌 **Key data point from the user: batch 50 worked over `--share` a few days ago**, before the 2026-08-24 Colab image update — roughly 35 minutes of generation. That reframes the 504 entirely: **with the queue working, a generation is not a long HTTP request** (the browser joins the queue and the result arrives over the websocket, so duration is irrelevant). `--no-gradio-queue` is what turns it into one synchronous POST held open for the whole run, and that is what gradio.live refuses
  - So the 504 is not a gradio.live limitation that ngrok can be swapped in to dodge — it is a direct consequence of the workaround, and it caps runs at a few minutes
  - **Fixing the queue is the fix, not changing tunnels.** It restores batch 50 on the tunnel that already worked. ngrok is still worth a try, but holding a 22-minute request open is a poor bet for any proxy
  - Timeline sharpened: the queue worked before 2026-08-24 and not since — the same event behind BUG-0013 / 0015 / 0019, and exactly when `starlette` and `anyio` would have moved. The `anyio==3.7.1` experiment is promoted to **the** next step
- ❌ **anyio ruled out.** `anyio==3.7.1` confirmed in the launch line with the queue re-enabled — identical failure (`Arguments: ('', [], False, -1)`). anyio 4's reworked primitives are not the cause
- 🧪 **Next pin committed: `fastapi==0.103.2`**, which brings **starlette 0.50.0 → 0.27.0**. starlette is never installed directly — it arrives with fastapi — so the fastapi pin is the only lever. 0.103.2 still accepts pydantic v1, so BUG-0016 stays satisfied; verified locally that it resolves to starlette 0.27.0 and imports cleanly. With anyio 3.7.1 already pinned this is the exact ASGI layer gradio 3.41.2 shipped against
  - If this comes back negative the ASGI layer is cleared end to end, and the remaining explanation lives inside gradio 3.41.2 under Python 3.13 — not something the notebook can pin its way out of. The honest answer then is the shipped workaround: `--no-gradio-queue` for ordinary runs, ngrok for long batches
- 🎯 **BUG-0023 — root cause of BUG-0021 found and fixed.** Instrumenting `send_message` produced no output during a failing generation, so the send had succeeded; what was sent was `process_completed` with **`success: False`**
  - **gradio's queue does not run predictions in-process** — it POSTs to its own `/api/predict` with `httpx.AsyncClient(verify=ssl_verify)`, which carries httpx's **default 5-second timeout** (verified by installing httpx 0.24.1 and asking it). A 48-second generation raises `ReadTimeout`; `AsyncRequest` swallows it by design — *"Exceptions are handled silently"* is in its own docstring — and the queue reports the job as failed. The browser then correctly refuses to apply a failed result, so every component keeps its initial value, which is exactly the `('', [], False, -1)` that `Save` had been sending back all along
  - The external websocket probe had succeeded precisely because its event returned in ~0.6 s, **inside the five-second window**. Server and browser were both behaving correctly the whole time
  - Fix: the Start cell patches `queueing.py` to build the queue client with `timeout=None`, and makes both silent handlers (`call_prediction`, `send_message`) log their exception. All three patches verified against the real 3.41.2 wheel — patterns match and the result compiles
  - Expected: gallery, PNG Info and Save all recover, `Disable_Gradio_Queue` can go back off, and with the queue restored a generation is no longer one long HTTP request — so **the 504 ceiling lifts and batch 50 should run again**
- 🧩 **Why the queue used to work — this looks self-inflicted.** A five-second timeout would have broken every generation ever, yet batch 50 ran days ago. The internal HTTP call is **specific to gradio 3.41.x**: 3.44.4 and 3.50.2 removed it and run the prediction in-process instead (upstream's source now says it decodes the result *"emulating the HTTP client behavior"*). Both later versions still have `IOComponent` and `pil_to_temp_file`, the two internals webui reaches into
  - The notebook never pinned gradio until the **BUG-0014 fix on 08-25** — made during this same investigation. Before that it deleted gradio and used whatever TheLastBen's bundle or Colab's image provided. If that was any 3.44–3.50 build, there was no HTTP call, no httpx client and no five-second ceiling. Pinning 3.41.2 would then have introduced BUG-0023
  - Not proven — the gradio version present before 08-24 was never recorded — but consistent with every observation, and it yields a second fix: **pin `gradio==3.50.2`** (upstream's own fix, no monkey-patching) as the fallback if the `timeout=None` patch does not hold
- 🎉 **BUG-0021 / BUG-0023 RESOLVED — verified on Colab 2026-08-27.** With `queueing.py` patched to build the queue client with `timeout=None`, all three switches off and the queue enabled, **the image appears in the gallery**
  - Cleanup: `Disable_Gradio_Queue` back to **False** (it is now only an escape hatch — turning it on reintroduces the 504 ceiling); the launch warning, which had been asserting the opposite of the truth, now fires on the *disabled* case and explains the trade-off; both diagnostics stay in permanently
  - Remaining: confirm **batch 50**, the run that started this. With the queue restored a generation is no longer one long HTTP request, so the 504 ceiling should be gone
- 📝 Side observation: the long-standing annoyance where the in-progress output folder could not be opened from Windows Explorer (Google Drive for Desktop) also cleared up. No verified mechanism — the plausible reading is simply less Drive churn now that generations are not being repeated and retried against a broken delivery path. Recorded in case it comes back.
- 🏁 **Batch 30 completed end to end (2026-08-27).** ~22 minutes, every image delivered to the gallery, no 504 and no `call_prediction FAILED`. **T-015 closed.** The 504 ceiling was an artefact of `--no-gradio-queue`, and it is gone now that the queue works
  - This closes the run that began with the 2026-08-24 Colab base image update: **BUG-0013, 0014, 0015, 0016, 0017, 0018, 0019, 0020, 0021, 0022, 0023** — from "the WebUI will not start" to "batch 30 completes"
- 🧹 Cleanup: retired two of the three diagnostic switches. **`Disable_Gradio_Queue` removed** — with the queue working it can only make things worse, converting every generation into one synchronous POST and reinstating the 504 ceiling; a switch whose only effect is to reintroduce a fixed bug is a trap. **`Test_Stock_Gradio` removed** — ruled out, force-reinstalls gradio each launch, and breaks the Ngrok path which needs TheLastBen's `blocks.py`. **`Test_Disable_Extensions` kept** — cheap, conflicts with nothing, and still the right first move on any future report. The three `queueing.py` patches stay: they are the fix and its instrumentation, not switches
- 📌 Decision (2026-08-27): **the 2026-08 recovery will not be ported to the ComfyUI notebook.** The README now says plainly that ComfyUI is behind and gives its last verified date (2026-04-30) — honest, zero maintenance, and revisitable if that notebook is ever actually needed. Recorded as T-016 so it does not get re-opened by drift
- 🏆 **Batch 150 completed (2026-08-27)** — on the order of two hours of continuous generation, every image delivered to the gallery, no 504 and no `call_prediction FAILED`. That is **past the batch-50 the notebook managed before the 2026-08-24 breakage**, so the queue is not merely restored to where it was
- 🐛 **BUG-0024 — found by testing the ngrok path because the repo is public.** The tunnel came up and printed its URL, but `failed to open private leg ... dial tcp [::1]:7860: connect: connection refused`, and an external `GET /config` returned 502
  - `pyngrok/ngrok.py:293` turns a bare port into `http://localhost:PORT`, and the agent resolves `localhost` to the **IPv6 loopback**. webui binds IPv4 only, so nothing is ever on `::1` — not a startup race, a permanent defect. Fixed by naming the address: `ngrok.connect('http://127.0.0.1:7860', ...)`
  - The probe also settled BUG-0021's open ngrok question: the tunnel returned a plain **502**, not ngrok's browser interstitial, so non-browser User-Agents pass straight through and the queue's internal `POST /api/predict` will not be served an HTML warning page
  - The `Ngrok URL:` line had been added earlier in this same session and never run once. Shipped-but-unexecuted code is not "probably fine"
- ✅ **BUG-0024 confirmed fixed (2026-08-27).** After pointing the tunnel at `127.0.0.1`, the run shows no `failed to open private leg`, and `Running on local URL: https://…ngrok-free.dev` / `✔ Connected` — so the agent reaches webui and the `blocks.py` host substitution works. The run then ended with `Interrupted with signal 2` (the cell was stopped), so **generating through the ngrok tunnel is still unconfirmed**; only the tunnel is
  - The trailing `RuntimeError: reentrant call inside <_io.BufferedWriter>` is upstream noise: a second SIGINT arrived while A1111's `sigint_handler` was mid-`print`. Nothing to chase
- ✅ **ngrok path fully verified (2026-08-27)** — tunnel, WebUI and image generation all work through it. BUG-0024 closed, and BUG-0021's two open ngrok concerns settled: no browser interstitial for non-browser User-Agents, and the queue's internal call is unaffected. The earlier `Interrupted with signal 2` was Colab reclaiming the runtime, not a fault in this path
  - README now states both tunnels are verified
