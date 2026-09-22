# Current QA Review

## Scope
Independent re-review of the current Meter Flash Detector implementation on the main branch after the developer's changes intended to address QA-001 through QA-009.

## Overall status
**READY FOR CONTROLLED PHYSICAL TESTING, NOT READY TO CLAIM REAL-WORLD MEASUREMENT ACCURACY.**

The implementation now addresses several static issues from the first review, notably camera-frame sampling, canvas resizing, backgrounding, camera-start cleanup, and meter-constant reset behavior. However, real-world reliability remains unverified, and the spatial red detector still has a meaningful false-positive risk.

## Previous findings

### QA-001 — HIGH — Camera sampling
**Status: PARTIALLY FIXED**  
**Evidence: STATIC REVIEW**

The implementation now uses requestVideoFrameCallback when available, with requestAnimationFrame as the fallback. This is a material improvement because the preferred path is tied to decoded video frames rather than display repaint cadence.

It is not VERIFIED because actual iPhone Safari behavior, callback cadence, dropped frames, and fast-pulse detection have not been physically tested.

**Remaining verification:** T01, T18, plus controlled fast-pulse testing.

### QA-002 — HIGH — Red metric vulnerable to reflections
**Status: PARTIALLY FIXED**  
**Evidence: STATIC REVIEW**

The implementation now requires neighboring red pixels and applies a clustered-pixel density threshold before returning a non-zero signal. This reduces isolated single-pixel noise.

However, the logic still does not require a sufficiently large or spatially coherent LED-sized region. A small bright red reflection containing three or more qualifying neighboring pixels can produce density = 1 and a positive signal. The density calculation is clustered / n, where n includes all qualifying pixels, so it is not an absolute area threshold.

This remains a meaningful false-positive risk.

**Remaining verification:** T06, T07, T11, T16.

### QA-003 — HIGH — Exposure/lighting changes
**Status: PARTIALLY FIXED**  
**Evidence: STATIC REVIEW**

The detector still uses an adaptive baseline and threshold, but the revised state machine adds explicit signal-shape handling and a timeout for signals lasting more than one second. This improves recovery from sustained/abnormal signals.

However, there is still no explicit normalization against global brightness/exposure changes, and no spatial comparison or exposure-change detection. Auto-exposure/white-balance changes can therefore still alter the red score and baseline enough to create or suppress detections.

**Remaining verification:** T09 and T10.

### QA-004 — HIGH — Backgrounding
**Status: FIXED by static inspection; physical verification still required**
**Evidence: STATIC REVIEW**

A visibilitychange handler now resets an active measurement when the page becomes hidden. This prevents a pre-background pulse and a post-resume pulse from being treated as one valid interval. Processing is also skipped while hidden and continued after resume.

The code path addresses the original interval-corruption mechanism.

Per protocol, this is not marked VERIFIED until the actual device lifecycle is tested.

**Required physical verification:** T12.

### QA-005 — MEDIUM — Camera startup cleanup
**Status: FIXED by static inspection; physical verification still useful**
**Evidence: STATIC REVIEW**

startCamera() now stops all acquired tracks in the catch path and clears stream, track, and video.srcObject.

This addresses the identified cleanup gap.

**Required verification:** T20.

### QA-006 — MEDIUM — Canvas resized every frame
**Status: FIXED by static inspection**
**Evidence: STATIC REVIEW**

The canvas dimensions are checked before assignment, so the 120x120 canvas is no longer resized on every analysis frame.

Runtime performance is still worth checking, but the original defect is addressed.

### QA-007 — MEDIUM — Measurement confidence
**Status: PARTIALLY FIXED**  
**Evidence: STATIC REVIEW**

The UI now distinguishes waiting, possible pulse, pulse shape, rejected noise, high-confidence detection, and paused states. It also displays pulse counts, interval count, median interval, and a provisional reading when only one interval exists.

However, high confidence is based on pulse peak/shape conditions, not on repeated interval consistency. A single accepted pulse can therefore be described as high confidence even though real measurement reliability has not been established.

The UI changes from Provisional to Power after two accepted intervals, which is a relatively small evidence base.

**Remaining verification:** T06–T18, especially stable and noisy conditions.

### QA-008 — MEDIUM — Meter constant changes
**Status: FIXED by static inspection**
**Evidence: STATIC REVIEW**

setMeterConstant() detects a changed value and calls resetMeasurement() when measurement history exists or measurement is active. This prevents old intervals from silently being reinterpreted under a new meter constant.

**Verification:** T15 should still be run as a regression test.

### QA-009 — LOW — Graph history sample based
**Status: STILL OPEN**  
**Evidence: STATIC REVIEW**

The history remains capped at 5000 samples. There is still no time-based history or time axis. The actual duration therefore depends on processing cadence.

This is not a measurement-correctness blocker.

**Verification:** T18.

## New findings

### QA-010 — HIGH — Spatial filter can accept very small coherent red objects
**Status: OPEN**  
**Evidence: STATIC REVIEW**

The new clustering logic reduces isolated-pixel noise but does not establish an absolute minimum LED-like area. Three neighboring qualifying pixels can make the density equal 1, allowing a tiny bright red reflection or red object to become a valid signal.

**Why it matters:** The detector's core failure mode is false pulse detection. A false pulse changes the measured interval and can produce a materially incorrect power value.

**Expected:** Require both an absolute/relative amount of qualifying red area and spatial coherence, ideally with a region size/shape appropriate to the expected LED.

**Verification:** T07 and T16 with the actual meter ROI.

### QA-011 — MEDIUM — No explicit camera stop/lifecycle cleanup on normal page exit
**Status: OPEN**  
**Evidence: STATIC REVIEW**

There is still no explicit stop-camera control and no page lifecycle cleanup. The visibility handler handles backgrounding but does not stop the camera. This is primarily a lifecycle/resource issue rather than a measurement calculation issue.

**Expected:** Provide a camera stop path and stop tracks when the camera is intentionally stopped or the page is being discarded where appropriate.

**Verification:** Browser/device lifecycle testing.

### QA-012 — MEDIUM — High confidence is not equivalent to measurement confidence
**Status: OPEN**  
**Evidence: STATIC REVIEW**

The detector sets Detection: high confidence immediately after one accepted pulse based on peak and timing conditions. That describes pulse-shape confidence, not confidence that the power measurement is correct.

**Expected:** Separate pulse detected from measurement stable/confident, with stability based on multiple accepted intervals and/or interval consistency.

**Verification:** T02–T05 and noisy/reflection tests.

## Regressions found

No clear regression from QA-001 through QA-009 was identified in static inspection.

One lifecycle limitation remains: processingStarted is never reset if the camera stream is intentionally stopped, so a future camera-stop/restart implementation will need to reset the processing-loop state.

## Physical testing still required

No physical iPhone/Safari + electricity-meter tests were available to this QA review. In particular, the following remain unverified:

- Actual requestVideoFrameCallback cadence on the target iPhone/Safari.
- Real pulse detection at different load levels.
- False positives from reflections/red objects.
- Exposure and white-balance changes.
- Torch behavior.
- Phone movement and ROI positioning.
- Background/foreground behavior.
- Long-run stability.
- Actual camera startup failure behavior.
- Network/privacy behavior in the deployed environment.

## Recommended priority tests

1. T01 — Camera starts on iPhone Safari.
2. T18 — Actual processed FPS/cadence.
3. T02, T03, T04, T05 — Known-load accuracy.
4. T06, T07, T16 — False-positive resistance.
5. T09, T10 — Exposure/torch changes.
6. T12 — Background/resume.
7. T17 — Long-run stability.
8. T19 — Network inspection.
9. T20 — Camera startup failure.

## QA conclusion

The developer has made substantial improvements and the implementation is now suitable for controlled physical testing.

It is not yet appropriate to claim accurate real-world power measurement. The highest-priority uncertainty is whether the revised red detector can reliably distinguish the meter LED from small bright red reflections while maintaining pulse detection under real camera exposure behavior.
