# Plan: Latent-Space Rollout Inference Runner

**Status:** Draft / Proposal
**Depends on:** `NOAA-PSL/anemoi-core` latent-rollout model (`plan_latent_rollout.md` there; Phases 1–3)
**Goal:** Run inference WeatherMesh-style: encode initial conditions once, advance the state autoregressively in latent (hidden-mesh) space, and decode to grid space only at requested output lead times.

---

## 1. Current behavior

The default runner's forecast loop calls the full model (`predict_step`: encoder→processor→decoder) every step and rebuilds the grid-space input window from decoded output each step, recomputing grid-space dynamic forcings (solar, time embeddings) per step from checkpoint metadata.

## 2. Target behavior

```
input state ──encode()──► z ──process(z, f_t)──► z ──…──► decode(z_t) only at output times
```

- `z` (hidden-mesh latent + shard metadata) stays GPU-resident for the whole forecast.
- Per-step forcings are computed on hidden-mesh coordinates (from checkpoint metadata) and passed to `process`.
- `decode` is called only at lead times requested by the output configuration (e.g. every 6 h while the latent stepper may run at 1 h in a future extension).

## 3. Work plan

### Phase A — Runner (~1 week, after anemoi-core Phase 1)

1. New runner class (e.g. `LatentRolloutRunner`) registered alongside the default runner; selected automatically when the checkpoint metadata declares a latent-rollout model class, or explicitly via `runner: latent_rollout` in the config.
2. Forecast loop: `encode` once from the prepared input state; loop `process` per step; `decode` at output times; hand decoded fields to the existing post-processing/output pipeline unchanged.
3. Forcing computation: reuse the existing dynamic-forcings machinery but evaluate on hidden-mesh lat/lon (exposed via checkpoint metadata / the model's `node_attributes`).

### Phase B — Config & metadata (~few days)

1. Checkpoint metadata additions (written by anemoi-training): model type flag, hidden-mesh coordinates reference, latent Δt.
2. Config options: `output_frequency` decoupled from latent step frequency (forward-compatible with a high-frequency latent stepper).

### Phase C — Validation & performance (~1 week)

1. Parity test: latent runner vs. training-side rollout on the same checkpoint (bitwise not expected; statistical agreement).
2. Benchmarks: wall-clock and peak memory vs. default runner for 10-day forecasts with 6-hourly vs. daily output.
3. Precision handling (`out.to(batch.dtype)` analogues) and CPU-offload interaction.

## 4. Out of scope

- LAM boundary-forcing injection in latent space (tracked in anemoi-core plan, Phase 5).
- Ensemble/diffusion latent samplers.

## 5. Success criteria

- Correct forecasts matching the trained model's validation behavior.
- Reduced wall-clock for sparsely decoded long rollouts.
- No changes required to existing runners/outputs; feature is purely additive.
