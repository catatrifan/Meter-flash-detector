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


---

## Detector fix — 2026-09-23

### QA-025 follow-up — spatial/color detector strengthened
**Status: STATICALLY IMPROVED; PHYSICAL VERIFICATION REQUIRED**

The detector was updated specifically to address false positives from skin and diffuse warm/red regions.

Changes:
- RGB samples are aggregated into 2×2 blocks before classification, reducing single-pixel sensor noise.
- Candidate pixels now require stronger red dominance: red must exceed green and blue by multiplicative margins.
- Candidates must also meet a minimum saturation and red-excess threshold.
- Candidate pixels are grouped into 8-connected spatial components.
- Very small components are rejected.
- Very large/diffuse components are rejected.
- Component compactness is required, favoring a concentrated LED-like region rather than a broad skin-colored area.
- The signal combines component mean red strength, peak strength, and area concentration.
- The existing baseline-relative rise/peak/fall state machine was not changed.

### Static regression checks
- Embedded JavaScript syntax check: **PASS**.
- Synthetic compact saturated-red LED-like cluster: **detected**.
- Synthetic broad skin-colored region: **rejected**.
- Synthetic warm orange region: **rejected**.
- Synthetic compact saturated-red reflection: **detected by the signal stage**, as expected from a color/spatial-only detector; the existing temporal pulse state machine remains responsible for rejecting sustained/static sources.

These are code-level/synthetic checks only. They are **not physical camera tests**.

### Physical tests still required
T06, T07, T08, T09, T10, T11 and T16 should be rerun on the target iPhone/Safari + meter setup, with a specific regression case for a hand/finger or visible skin inside the ROI.

The detector was intentionally changed only in its signal-analysis stage. Power formulas, pulse/interval accounting, measurement timer, pause/resume behavior, and meter-constant handling were not modified by this detector fix.
