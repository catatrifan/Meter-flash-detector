# Current QA Review

## Scope
Static review of the current Meter Flash Detector implementation on the main branch.

## Overall status
**CHANGES_REQUIRED before claiming reliable real-world measurement accuracy.**

The core power formula and general pulse-state-machine approach are plausible, but several implementation risks can materially affect measurement correctness.

## Findings

### QA-001 — HIGH — Camera sampling is tied to requestAnimationFrame
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** Main analysis loop.

The detector processes frames from requestAnimationFrame. This follows display repaint cadence rather than guaranteeing one callback per decoded camera frame. A genuine LED pulse may therefore be missed, and a missed pulse can produce an incorrect interval and power value.

**Expected:** Prefer requestVideoFrameCallback when available, with a documented fallback.

**Verification:** Compare processed-frame cadence with actual camera frame cadence and test fast pulses.

### QA-002 — HIGH — Red signal metric is vulnerable to isolated reflections
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** Red-intensity calculation.

The current metric averages red excess only across pixels that pass a red threshold. A small number of very bright red pixels can therefore produce a strong score without requiring a spatially coherent LED region.

**Expected:** Use area-aware/spatial evidence such as qualifying-pixel fraction, total red energy, and/or connected-region validation.

**Verification:** Test with the LED blocked while introducing red reflections or bright red objects near the ROI.

### QA-003 — HIGH — Exposure/lighting changes can corrupt detector state
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** Adaptive baseline and armed/re-arm state.

The baseline adapts slowly and detector state depends on the baseline. Camera auto-exposure/white-balance changes or gradual lighting shifts can resemble signal transitions or prevent clean re-arming.

**Expected:** Detector should distinguish genuine short LED pulses from global illumination changes and recover after exposure shifts.

**Verification:** Force controlled brightness/exposure changes and inspect signal/baseline/threshold traces.

### QA-004 — HIGH — Backgrounding can corrupt intervals
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** Measurement lifecycle.

The page does not explicitly handle visibility changes. requestAnimationFrame may pause while the page is backgrounded, and the next pulse interval can therefore include time during which no frames were processed.

**Expected:** Backgrounding should interrupt/reset the active measurement safely, then require a fresh stable pulse sequence.

**Verification:** Start measurement, background/lock the phone, resume, and confirm no invalid interval is used.

### QA-005 — MEDIUM — Camera startup cleanup is incomplete
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** Camera startup error path.

If a media stream is acquired and a later startup step fails, the acquired tracks should be explicitly stopped.

**Expected:** Every failure path after getUserMedia must clean up the stream.

**Verification:** Simulate/reproduce startup failure and confirm tracks are stopped.

### QA-006 — MEDIUM — Canvas is resized during every analysis
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** analyze().

The analysis canvas is resized to 120x120 on every frame. This is unnecessary work and may cause repeated allocation/state reset.

**Expected:** Resize the canvas only when dimensions change.

**Verification:** Inspect runtime performance and confirm the canvas size is stable between frames.

### QA-007 — MEDIUM — Measurement confidence is not sufficiently visible
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** Measurement UX.

The UI can show a power value without exposing enough evidence to judge whether the detector has observed a stable pulse sequence.

**Expected:** Show at least pulse count and/or accepted interval information and a clear signal/measurement stability state.

**Verification:** Observe the UI with no LED, intermittent pulses, and stable pulses.

### QA-008 — MEDIUM — Meter-constant changes can reinterpret existing measurement history
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** Meter constant control.

Changing the impulses/kWh value while an active measurement contains prior intervals can cause the displayed power to be recalculated under a different constant without clearly resetting the measurement.

**Expected:** Changing the constant should reset measurement history or clearly start a new measurement context.

**Verification:** Record a stable measurement, change the constant, and confirm old intervals are not silently reused.

### QA-009 — LOW — Graph history is sample-count based
**Status:** OPEN  
**Evidence:** STATIC REVIEW  
**Area:** Graph history.

A fixed 5000-sample history represents different real-world durations at different processing rates and has no time axis.

**Expected:** Prefer a time-based history and show elapsed time or a meaningful time axis.

**Verification:** Compare graph duration at different processing rates.

## Physical tests still required
No static code review can establish real-world detection accuracy. T01–T20 in TEST_PLAN.md should be executed on the intended iPhone/Safari + meter setup.
