# TODO: fast-stable-diffusion

Last Updated: 2026-08-25

---

## Notebook Support Policy

| Notebook | Status | Notes |
|----------|--------|-------|
| AUTOMATIC1111 | ✅ Active Maintenance | Primary target |
| ComfyUI | ✅ Active Maintenance | PyTorch 2.10 verified, image generation confirmed |
| DreamBooth | ❄️ Not Supported | Frozen. No maintenance. |

---

## In Progress

| # | Task | Priority | Notes |
|---|------|----------|-------|
| T-015 | Verify BUG-0022 on Colab | High | Pinning `uvicorn==0.49.0` should restore the gallery, PNG Info and Save in one go. Turn `Test_Disable_Extensions` back off and leave `Disable_Gradio_Queue` off. |


---

## Pending

| # | Task | Priority | Notes |
|---|------|----------|-------|
| T-013 | Post on X (Twitter) to announce fork | Low | Draft ready. markn3 to post manually. |

---

## Completed

| # | Task | Completed | Notes |
|---|------|-----------|-------|
| T-014 | Resolve the `sd_disable_initialization.py` sed | 2026-08-25 | Settled by source inspection instead of an A/B run: transformers' check is `state_dict is not None and pretrained_model_name_or_path is not None`, and A1111 passes `None` by design. Reversed the sed. BUG-0017 |
| T-010 | ComfyUI: verify compatibility with PyTorch 2.10 | 2026-04-30 | Verified. Image generation confirmed. BUG-0006〜0010 fixed |
| T-011 | ComfyUI: add xformers fix cell | 2026-04-30 | Done. `--pre` flag added |
| T-012 | ComfyUI: verify gdown==5.2.1 pin | 2026-04-30 | Confirmed in Requirements cell |
| T-008 | Fix hardcoded Python paths | 2026-04-27 | 20 locations across 3 notebooks. commit `2cdfc95` |
| T-007 | Implement Colab environment check cell | 2026-04-27 | All 3 notebooks. commit `3b14375` |
| T-009 | gdown v6 compatibility fix | 2026-04-27 | Short-term fix (gdown==5.2.1 pin) complete. commit `822d26c` |
| T-006 | Investigate gdown v6 breaking changes | 2026-04-27 | Documented in SPEC.md known issues #2 |
| T-005 | Plan fix for hardcoded Python path issue | 2026-04-27 | Solution documented in SPEC.md. Implemented in T-008 |
| T-004 | Document branch strategy and commit conventions | 2026-04-27 | Documented in PROJECT.md |
| T-003 | Organize upstream diff | 2026-04-27 | Documented in PROJECT.md |
| T-002 | Analyze DreamBooth notebook and update SPEC | 2026-04-27 | SPEC.md updated |
| T-001 | Analyze ComfyUI notebook and update SPEC | 2026-04-27 | SPEC.md updated |
| T-000 | Fix xformers PyTorch 2.10.0 incompatibility | 2026-04-27 | BUG-0001 / commit `8308f00` |
