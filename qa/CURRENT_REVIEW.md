# Current QA Review

## Fresh independent QA review — 2026-09-23

### Scope
Fresh static review of the current `main` branch. The implementation was inspected directly rather than treating prior QA conclusions as authoritative. No physical iPhone/Safari + electricity-meter testing was performed.

### Overall status
**BLOCKED — NOT READY FOR CONTROLLED PHYSICAL TESTING.**

The current source is syntactically structured as a functioning single-page application, and several previously identified implementation issues are no longer present. However, the current implementation contains a measurement-state accounting defect and unresolved detector/frame-sampling risks that are sufficient to block controlled physical validation until clarified.

### Findings

#### QA-030 — HIGH — Accepted pulse count and complete-interval count are conflated

**Status: OPEN**  
**Evidence: STATIC REVIEW**

The current implementation uses `measurementPulseCount` for the number of complete valid intervals:

`if(validInterval){ measurementPulseCount++; ... }`

But the UI renders the same variable as **“Pulses detected”**:

`pulseCountEl.textContent=String(measurementPulseCount)`

The first accepted detector event establishes the first-pulse anchor but does not increment `measurementPulseCount`, because there is no preceding interval. Therefore the UI under-reports accepted pulses by one:

- after 1 accepted flash: **0 Pulses detected**
- after 2 accepted flashes: **1 Pulses detected**
- after 3 accepted flashes: **2 Pulses detected**

Meanwhile `flashCount` is incremented for every accepted detector event, confirming that the implementation itself distinguishes these concepts internally but does not expose them consistently.

**Why it matters:** This is not merely cosmetic. Pulse count is part of the measurement evidence presented to the user, and the incorrect count can make the user misinterpret the amount of evidence supporting the calculated average.

**Expected:** Maintain separate state:
- accepted/detected pulse count;
- complete valid interval count.

The cumulative average should continue to use complete valid intervals.

**Verification:** T24 with exactly one, two, and three accepted flashes.

---

#### QA-031 — HIGH — Invalid long intervals corrupt the interval anchor

**Status: OPEN**  
**Evidence: STATIC REVIEW**

`registerPulseForPower()` defines a valid interval as:

`interval!==null && interval>0.05 && interval<3600`

However, `lastPulseTime` is updated unconditionally at the end of the function:

`lastPulseTime=now`

This means a detected pulse after more than 3600 seconds is rejected for power calculation but still becomes the new timing anchor. The next detected pulse is then measured from that invalid pulse rather than from the last valid pulse.

The invalid event therefore changes subsequent interval semantics even though it was explicitly classified as invalid.

**Why it matters:** A long gap, stale detector event, or lifecycle/timing anomaly can cause the next interval and therefore instantaneous power to be calculated from an invalid anchor.

**Expected:** Invalid intervals should not silently become the timing anchor. Depending on the intended semantics, the implementation should either:
1. discard the invalid event completely and retain the last valid pulse anchor, or
2. explicitly reset the measurement/averaging window when an interval exceeds the supported range.

**Verification:** T21/T24-style controlled injection or a testable detector harness with an interval >3600 s followed by a valid interval. Confirm the invalid event cannot redefine the next interval.

---

#### QA-032 — MEDIUM — Duplicate helper definitions create ambiguous maintenance behavior

**Status: OPEN**  
**Evidence: STATIC REVIEW**

The current script defines these helpers twice:

- `formatPower()`
- `setLivePower()`
- `setAssessmentText()`
- `setInstantWaiting()`

The second declaration replaces the first declaration in the same script scope. The current pairs appear behaviorally equivalent enough that this does not currently create a demonstrated runtime defect, but it makes future changes dangerous: modifying the first definition would have no effect, while modifying only one copy can create misleading source behavior.

**Expected:** Keep one definition of each helper.

**Severity:** Medium maintenance/regression risk, not by itself a release blocker.

---

#### QA-033 — MEDIUM — Average-power assessment state has inconsistent pause semantics

**Status: OPEN**  
**Evidence: STATIC REVIEW**

The exact cumulative average uses active measurement elapsed time, but the provisional “Assessing” path has a separate `averageAssessmentElapsedMs` / `averageAssessmentStart` state machine.

On pause, `pauseAverageAssessment()` accumulates elapsed assessment time. On resume before the first pulse, `resumeAverageAssessment()` continues that assessment. However, once the first pulse is detected, `registerPulseForPower()` resets `averageAssessmentElapsedMs=0` and `averageAssessmentStart=now` when a prior estimate is frozen.

This creates two different timing windows inside the same “AVERAGE POWER (SINCE START)” area: the provisional assessment is initially based on measurement-start time, then abruptly switches to a first-pulse-relative assessment clock.

This may be intentional UX, but it is not clearly defined by the label “since start” and is difficult to reason about after pause/resume.

**Expected:** Define the provisional estimate's timing semantics explicitly. Ideally, the assessment should use the same active measurement clock and clearly distinguish “provisional estimate” from the exact since-first-pulse average.

**Severity:** Medium; verify before relying on the provisional display.

---

### Previously identified issues re-checked against current source

- Duplicate Pulse constant IDs: **NOT PRESENT in current source.**
- Per-frame canvas resizing: **FIXED** — dimensions are only assigned when they differ from 120×120.
- Pause duration entering exact cumulative average: **FIXED** — exact average uses active elapsed time and a first-pulse active-time anchor.
- Invalid intervals incrementing the exact interval count: **FIXED** for the count itself — only valid intervals increment `measurementPulseCount`. However, QA-031 identifies a separate remaining issue: invalid long events still replace `lastPulseTime`.
- Second accepted pulse producing the first exact complete-interval average: **IMPLEMENTED** in current source.
- Camera start function is declared `async`: **FIXED**.
- Literal escaped-newline regression: **NOT PRESENT** in the current source inspected.
- Default 4000 impulses/kWh: **INTENTIONAL current behavior**; the single visible control is read by the calculation.

### Core detector risks — independently re-confirmed

#### QA-025 / QA-026 remain OPEN

The current detector still:
- reduces the ROI to a 120×120 canvas;
- accepts any pixel with `red = r - max(g,b) > 8` and `r > 30`;
- averages only the strongest 5% of qualifying pixels, with a minimum of 3 pixels;
- has no spatial-neighbor/cluster requirement or minimum LED-like region requirement;
- processes frames through `requestAnimationFrame`.

These are implementation facts from the current source, not inherited conclusions. They leave two important unverified risks:

1. small bright red reflections/objects can influence the signal;
2. display refresh cadence may undersample camera frames, particularly when the requested camera rate exceeds display refresh cadence.

No claim about actual iPhone Safari performance or real meter detection is made because no physical test was performed.

### Additional current-source observations

- `flashCount` is incremented but is not used for the visible “Pulses detected” statistic; this contributes directly to QA-030.
- `pulseTimes` is maintained/reset but is not populated by the current implementation.
- `lastFrame` is declared but not used.
- `statusEl=null` and `setStatus()` is effectively a no-op, so camera errors and “FLASH DETECTED” status messages passed to `setStatus()` are not visibly surfaced through that mechanism.
- `METER_CONSTANT_KEY` is declared but no persistence/read/write path is present.
- `selectInitialMeterPreset()` and `updateAverageAssessment()` are defined but not called.
- There are repeated CSS declarations for several selectors. They are not currently shown to cause a functional defect, but they increase source ambiguity.

### Physical/runtime status

No physical testing was performed. Therefore these remain **NOT TESTED**:

- iPhone Safari camera startup and actual camera settings;
- known-load accuracy;
- false-positive resistance;
- exposure/white-balance changes;
- phone movement / ROI-edge behavior;
- torch behavior;
- background/foreground lifecycle;
- long-run performance;
- actual processed frame cadence;
- network/privacy behavior;
- camera startup failure cleanup.

### Current verdict
**BLOCKED — NOT READY FOR CONTROLLED PHYSICAL TESTING.**

The current source is materially different from earlier reviewed versions, so prior findings were not carried forward merely by history. The newly identified measurement-state issues (QA-030/031), together with the independently confirmed detector/frame-sampling risks (QA-025/026), should be resolved or explicitly accepted before controlled physical testing.
