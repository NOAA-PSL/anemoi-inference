---
name: anemoi
description: Coding agent profile for Anemoi latent-rollout development
model: claude-sonnet-5.5
---

You are working on the Anemoi machine-learning weather forecasting framework
(fork of ecmwf/anemoi-core and ecmwf/anemoi-inference).

## Style and verbosity

Match the existing Anemoi code base exactly:

- Apache 2.0 / ECMWF copyright header block at the top of every new Python file
  (copy verbatim from an existing file, updating the year).
- NumPy-style docstrings with Parameters/Returns sections.
- Full type annotations (`from __future__ import annotations` where used elsewhere);
  style matching neighboring code.
- Concise inline comments only where intent is non-obvious; no narrative comments.
- Configuration-driven design: new runners/components are registered and selectable
  via config, never hard-coded.
- Keep changes additive: new classes/modules/configs rather than modifying existing
  runners, to remain upstreamable to ecmwf.
- Tests mirror existing patterns in `tests/` (pytest, shape assertions).
- Logging via module-level `LOG = logging.getLogger(__name__)` matching repo convention.
- Follow the repo's pre-commit config (ruff, line length, import order); run it
  before committing.

## Context

The current work implements a latent-space rollout inference runner
(encode-once → GPU-resident latent AR steps → decode-on-demand). The authoritative
plan is `plan_latent_rollout.md` on the `feature/latent-rollout-plan` branch;
consult it before making design decisions. The companion model/training work lives
in NOAA-PSL/anemoi-core.
