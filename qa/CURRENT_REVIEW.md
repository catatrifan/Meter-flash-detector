# Current QA Review

## Fresh independent QA review — 2026-09-23

### Scope
Fresh static review of the current `main` branch. The application source was inspected directly and checked independently against the previous QA findings. No physical iPhone/Safari + electricity-meter testing was performed.

### Overall status
**BLOCKED — NOT READY FOR CONTROLLED PHYSICAL TESTING.**

The current source now passes the previously identified code-level measurement-state fixes. The remaining release-relevant risks are the detector's weak spatial red-signal discrimination and use of `requestAnimationFrame` for camera processing. Those require controlled runtime/physical validation and/or explicit approval before changing the detector behavior.

### Current-source verification

- **JavaScript syntax:** PASS — embedded script parses successfully.
- **Application script:** PRESENT — exactly one `<script>` block.
- **Document structure:** PASS — one `<body>` and one `</body>`.
- **Literal escaped-newline regression:** ABSENT.
- **Pulse-constant controls:** PASS — one `meterPreset`, one `meterConstant`, one `customMeterRow`.
- **Pulse count:** FIXED — visible “Pulses detected” uses `acceptedPulseCount`; complete intervals remain separately tracked by `measurementPulseCount`.
- **Long invalid interval handling:** FIXED — an interval >=3600 s starts a new timing/averaging window instead of being used as a normal interval.
- **Duplicate helper definitions:** FIXED — `formatPower`, `setLivePower`, `setAssessmentText`, and `setInstantWaiting` each have one definition.
- **Canvas allocation:** FIXED — the 120×120 dimensions are assigned only when they differ, not on every frame.
- **Pause/resume exact-average accounting:** FIXED — exact average uses active elapsed time and a first-pulse active-time anchor.
- **Cumulative-average units:** PASS — millisecond-based factor `3,600,000,000` is dimensionally correct.
- **Camera startup syntax:** FIXED — `startCamera` is async.

### Remaining findings

#### QA-025 — HIGH — Weak spatial discrimination in red detector
**Status: OPEN**  
**Evidence: STATIC REVIEW**

The detector still classifies pixels using `red = r - max(g,b) > 8` and `r > 30`, then averages the strongest 5% of qualifying pixels. It does not require a sufficiently large, spatially coherent LED-like region.

This leaves a static false-positive risk from small bright red reflections or unrelated red objects within the ROI. Actual false-positive frequency cannot be established without physical testing.

**Required verification:** T06, T07, T11, T16.

#### QA-026 — HIGH — Camera processing uses requestAnimationFrame
**Status: OPEN**  
**Evidence: STATIC REVIEW**

The processing loop uses `requestAnimationFrame` rather than `requestVideoFrameCallback`. At camera rates above display refresh, this can process fewer decoded frames than the camera produces and can reduce detection opportunities for short flashes.

This is an implementation risk, not proof of missed pulses on the target device.

**Required verification:** T01 and T18, plus controlled fast-pulse testing.

### QA-033 — MEDIUM — Provisional assessment timing
**Status: DESIGN ACCEPTED / NO CODE DEFECT CONFIRMED**

The provisional “Assessing” state intentionally uses a separate assessment clock and changes its reference point at the first detected pulse. The exact cumulative average is separately defined from the first pulse using active elapsed time. This is a product/UX design choice rather than a confirmed calculation defect.

### Additional observations

- `statusEl=null` makes `setStatus()` a no-op, so messages passed to that function are not surfaced through a status element. This is a UX/diagnostic weakness but not currently a measurement blocker.
- `METER_CONSTANT_KEY`, `selectInitialMeterPreset()`, `updateAverageAssessment()`, `pulseTimes`, and `lastFrame` are unused. These are cleanup opportunities, not release blockers.
- Repeated CSS declarations remain, increasing maintenance ambiguity but not currently demonstrating a functional defect.

### Physical/runtime status

No physical testing was performed. The following therefore remain **NOT TESTED**: actual iPhone Safari camera startup/settings, known-load accuracy, false-positive resistance, exposure/white-balance changes, movement/ROI-edge behavior, torch behavior, lifecycle behavior, long-run performance, actual processed frame cadence, network/privacy behavior, and camera-start failure cleanup.

### Current verdict
**BLOCKED — NOT READY FOR CONTROLLED PHYSICAL TESTING.**

The current code-level findings QA-030 through QA-032 are fixed. The remaining blockers are QA-025 and QA-026, which need controlled runtime/physical validation or separately approved implementation changes.
