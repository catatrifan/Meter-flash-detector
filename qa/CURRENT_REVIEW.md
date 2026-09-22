# Current QA Review

## Scope
Independent re-review of the current Meter Flash Detector implementation on the main branch after the developer's changes intended to address QA-001 through QA-009.

## Overall status
**READY FOR CONTROLLED PHYSICAL TESTING, NOT READY TO CLAIM REAL-WORLD MEASUREMENT ACCURACY.**

The latest build adds live instant/rolling power displays and changes the graph from red-signal history to detected power history. The earlier detector fixes remain present. No physical-device evidence is available to QA.

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

### QA-013 — MEDIUM — “Average power” display is actually a median-based calculation
**Status:** OPEN  
**Evidence:** STATIC REVIEW

The UI labels the top-right value “AVERAGE POWER · LAST 5 READINGS”, but the implementation sorts the last up-to-five pulse intervals and calculates the median interval, then converts that median interval to watts. That is not the arithmetic average of the last five power readings.

This may be an intentional robust estimator, but the label is factually misleading and can cause users to interpret the value incorrectly.

**Expected:** Either label it as median/robust power based on the last five intervals, or calculate the actual average of the last five power readings.

**Verification:** Compare the displayed value with a dataset containing unequal pulse intervals.

### QA-014 — MEDIUM — New live power UI does not establish measurement stability
**Status:** OPEN  
**Evidence:** STATIC REVIEW

The new “INSTANT POWER” display is populated after any accepted interval, while the “AVERAGE POWER · LAST 5 READINGS” display remains blank until five intervals have been collected. The UI therefore introduces a prominent power value before there is enough repeated evidence to establish stable measurement.

The existing lower panel still calls the two-interval value “Power”, rather than clearly distinguishing a provisional estimate from a stable measurement.

**Expected:** Keep provisional/instant values visually distinct from a stable reading, and define a clear stability criterion based on multiple intervals and consistency.

**Verification:** Start measurement and observe the UI after exactly two, three, four, and five accepted intervals.

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

No detector-state regression was identified in static inspection. The new power-history graph is a functional redesign rather than a direct regression, but its “average” label is inaccurate as described in QA-013.

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


## Latest re-review — 2026-09-22

### Previous findings status
- **QA-001:** PARTIALLY FIXED — requestVideoFrameCallback remains, but actual iPhone/Safari cadence is unverified.
- **QA-002:** PARTIALLY FIXED — minimum qualifying pixels increased to 8 and clustered pixels to 5, but a small coherent red object can still pass; physical false-positive testing is required.
- **QA-003:** PARTIALLY FIXED — no explicit exposure normalization/change detection; still requires T09/T10.
- **QA-004:** FIXED by static inspection — visibilitychange resets active measurement and pauses processing; device verification remains required.
- **QA-005:** FIXED by static inspection — startup catch stops acquired tracks.
- **QA-006:** FIXED by static inspection — analysis canvas is resized only when dimensions differ.
- **QA-007:** PARTIALLY FIXED — UI is more explicit, but measurement stability is still not rigorously established.
- **QA-008:** FIXED by static inspection — changing meter constant resets measurement state when needed.
- **QA-009:** FIXED by static inspection — the power-history graph is now time-based over retained pulse history, although it is capped to five minutes and still has only relative Older/Latest labels.
- **QA-010:** STILL OPEN — 8 qualifying pixels and 5 clustered pixels can still represent a very small red reflection/object.
- **QA-011:** FIXED by static inspection — pagehide now stops tracks and clears the stream. Restart behavior after page lifecycle restoration needs device/browser verification.
- **QA-012:** STILL OPEN — “high confidence” still means pulse-shape confidence, not measurement stability.
- **QA-013:** FIXED — the label is now “AVERAGE POWER · SINCE START”, and the implementation computes a count/elapsed-time average rather than the prior median-of-five label mismatch.
- **QA-014:** PARTIALLY FIXED — the UI now distinguishes “Provisional instant” from “Average power”, but average power can still be displayed after the first pulse and uses elapsed time from measurement start, including an incomplete first pulse interval.

### New findings

#### QA-015 — MEDIUM — Since-start average is biased by the incomplete first interval
**Status:** OPEN  
**Evidence:** STATIC REVIEW

The average calculation uses pulse count divided by elapsed time since measurement start. The first detected pulse represents an energy increment whose full interval began before measurement started unless measurement starts exactly at a pulse boundary. Therefore the displayed “average power since start” includes an incomplete first pulse interval. Shortly after starting, this can materially distort the average.

**Expected:** Either start the averaging window after the first pulse, or explicitly define and handle the incomplete first interval.

**Verification:** Start measurement at several points within a known pulse cycle and compare the displayed average over a controlled interval.

#### QA-016 — LOW — Page lifecycle restart is not fully wired for restoration
**Status:** OPEN  
**Evidence:** STATIC REVIEW

pagehide sets processingStarted=false, stops the stream, and clears the video source, but it does not re-enable the Start camera button or provide a pageshow handler that restores the camera UI state. If the page is restored from a browser lifecycle cache, the UI can remain in a disabled “Camera running” state while no stream exists.

**Expected:** On page restoration, camera state and controls should be synchronized or the page should force a clean restart state.

**Verification:** T01/device lifecycle testing including navigation away and back.

### Latest QA conclusion
**READY FOR CONTROLLED PHYSICAL TESTING; NOT READY TO CLAIM REAL-WORLD MEASUREMENT ACCURACY.**

The latest build resolves QA-009 statically and improves QA-002 and QA-011. The remaining major uncertainty is detector behavior with real meter LEDs, reflections, exposure changes, and actual camera cadence. QA-015 is a correctness issue in the new “Average power since Start” metric and should be addressed before treating that metric as a trustworthy average.


### QA-017 — BLOCKER — Literal \\n sequences break the page JavaScript and overlay/layout CSS
**Status: OPEN**  
**Evidence: STATIC REVIEW**

The current `index.html` contains literal backslash-n sequences (for example after the `.hint` CSS rule, between the live-power CSS rules, after the `Measurement history` div, and in JavaScript between statements) instead of actual line breaks. In the JavaScript, sequences such as `presets=[...];\\n  const instantPowerEl...` occur outside a string literal and make the script syntactically invalid. As a result, the event listeners and camera-start logic do not execute.

The same malformed escaping also affects the live-power CSS rules, so the overlay cards are not positioned/styled as intended, and the literal \\n text under Measurement history is rendered into the page.

**Why it matters:** This is a release-blocking regression: the camera cannot be started through the app's handler, the power-reader overlay can lose its intended positioning, and visible escaped-newline text appears in the UI.

**Expected:** `index.html` must contain actual newlines between HTML/CSS/JavaScript constructs, with no unintended literal `\\n` sequences outside string literals.

**Verification:** Load the deployed page in Safari and desktop browser console; confirm zero JavaScript syntax errors, Start camera handler works, both power cards remain inside the camera overlay without covering the target, and no literal \\n text is visible.


## Latest re-review — 2026-09-22

### QA-017 verification
- **QA-017:** VERIFIED by static review.
- The current `index.html` contains **0 literal \\n sequences**.
- The embedded JavaScript was independently syntax-checked and parses successfully.
- The live-power CSS is now present as valid CSS rules.
- The Measurement history markup no longer contains a literal escaped newline.
- This addresses the code-level causes of the previously reported camera-start failure, misplaced power overlay styling, and visible \\n text.

### Other previous findings
- **QA-015:** FIXED by static inspection. The averaging window now starts at the first detected pulse (`firstPulseTime`) and uses complete pulse intervals only, avoiding the incomplete first interval bias.
- **QA-016:** FIXED by static inspection. `pagehide` now restores the Start camera button/UI state, and a `pageshow` handler synchronizes the camera-start state after restoration.
- **QA-010:** STILL OPEN. Red detection was tightened to red-excess/ratio thresholds and minimum 10 qualifying / 7 clustered pixels, but static review cannot establish that this is sufficient against real reflections or red objects.
- **QA-001:** PARTIALLY FIXED — requestVideoFrameCallback remains; actual iPhone/Safari cadence is unverified.
- **QA-002:** PARTIALLY FIXED — stronger spatial/color thresholds reduce false positives; physical verification remains required.
- **QA-003:** PARTIALLY FIXED — exposure/white-balance behavior remains unverified and there is no explicit exposure-change normalization.
- **QA-004/005/006/008/009:** FIXED by static inspection; device/runtime verification remains where applicable.
- **QA-007/012/014:** PARTIALLY FIXED — the UI is clearer, but accepted pulse confidence is still not equivalent to demonstrated measurement stability.
- **QA-013:** FIXED.

### New regression check
No new static regression identified in the latest build. The latest commits specifically address the script escaping, cumulative average calculation, red detection thresholds, lifecycle UI state, and power updates.

### Physical/runtime status
No physical iPhone/Safari + electricity-meter testing was performed. Therefore camera startup, overlay positioning on the actual viewport, pulse detection, false-positive resistance, frame cadence, torch behavior, exposure changes, and real-world power accuracy remain unverified.

### Current verdict
**READY FOR CONTROLLED PHYSICAL TESTING; NOT READY TO CLAIM REAL-WORLD MEASUREMENT ACCURACY.**

The previous release-blocking QA-017 regression is statically fixed. The next priority is actual iPhone/Safari testing, especially T01, T02–T07, T09–T12, and T18.

### Latest re-review — 2026-09-22 — post power/detector fixes

#### Power calculation / display re-check
- **QA-013:** VERIFIED by static review. The previous last-five/median implementation is gone. The UI is labeled **AVERAGE POWER · SINCE START**.
- **QA-014:** PARTIALLY FIXED. The UI exposes separate instant and cumulative-average values. A stable real-world measurement criterion is still not established and requires physical testing.
- **QA-015:** VERIFIED by static review. The averaging window begins at `firstPulseTime`, which is set on the first accepted pulse. The average uses `measurementPulseCount - 1` complete intervals over the elapsed time from the first to the latest pulse.
- **Power update path:** VERIFIED by static inspection in commit `46d862e5a5976d3c6540dd1d7c8bc8a28faacf56`. Every accepted pulse increments `measurementPulseCount`; every accepted second-or-later pulse calculates instant power from the latest interval and cumulative average power from all complete intervals since the first pulse; both overlay values are updated explicitly.

This addresses the code-level cause of the previously reported condition where accepted pulses could exist without the power displays being populated. Runtime verification on the deployed build is still required.

#### Detector re-check
- **QA-010:** STILL OPEN. The detector now requires stronger red excess, red-channel minimum, red-ratio threshold, minimum qualifying area, clustered area, and higher cluster density. This is a material reduction in reflection risk, but only physical tests T06, T07, and T16 can establish whether real reflections still trigger pulses.
- **QA-002:** PARTIALLY FIXED. Stronger color and spatial thresholds are present; physical verification remains required.

#### Runtime verification required
No physical iPhone/Safari test was performed during this re-review. The following must still be tested on the deployed build:
- T01 camera startup
- T02–T05 known-load power readings
- T06/T07/T16 false-positive resistance
- T09/T10 exposure and torch changes
- T12 lifecycle/backgrounding
- T18 actual frame cadence

#### Current verdict
**READY FOR CONTROLLED PHYSICAL TESTING; NOT READY TO CLAIM REAL-WORLD MEASUREMENT ACCURACY.**

The current code passes the static re-check for the cumulative-average calculation and power-display update path. If the deployed build still shows no instant/average power while the pulse counter increments, that is now a deployment/runtime regression rather than the calculation logic identified in the source review.


### QA-018 — MEDIUM — Average power is pulse-event driven and can remain stale/blank
**Status: OPEN**  
**Evidence: STATIC REVIEW + user-reported runtime symptom**

The current average-power calculation is only refreshed when `registerPulseForPower()` runs, i.e. when an accepted pulse is detected. There is no timer/frame-loop update for the average between pulse events. The top “AVERAGE POWER · SINCE START” value therefore remains — until a second valid pulse is accepted, and after that it remains frozen at the last accepted pulse until the next accepted pulse.

The implementation also increments `measurementPulseCount` before validating the interval. If a second detected event produces an interval outside the accepted 0.05–3600 s range, the count changes but no average is calculated from that event, which can leave the UI inconsistent with the actual valid interval count.

**Why it matters:** The UI presents an average-power measurement, but the value may appear non-functional or stale during a measurement. This is particularly noticeable at low loads, where pulse intervals are long.

**Expected:** Define the average window in terms of valid complete pulse intervals and update the displayed average consistently from that valid state. At minimum, the UI should explicitly show that it is waiting for the next valid pulse rather than appearing broken; ideally the average should be recomputed periodically from the first valid pulse through the current time when that definition is appropriate.

**Verification:** On a known stable load, start measurement between pulses and observe the average card before the first pulse, after the first pulse, after the second pulse, and while waiting for the third. Test a case where an interval is rejected as invalid.


### QA-018 follow-up — 2026-09-22
- Re-checked the current power path after the user-reported “average power shows 1” symptom.
- Commit `bec86cda2bf20d8e4f9bfd56fdee8cb25eb61216` changes interval accounting so only **valid** pulse intervals contribute to the complete-interval count.
- Commit `83d299fd6bf7352e5d109e852136f65fe91245f2` adds explicit diagnostics to the power-detail line: latest interval, instant power, average power, and meter constant.
- Static review does **not** prove why the deployed UI showed “1”; the next physical/runtime test must compare the displayed value with the displayed interval and meter constant.
- QA-018 remains **QA VERIFYING** pending browser/device reproduction.


## Latest independent QA re-review — 2026-09-23

### QA-019 — BLOCKER — Current index.html contains no application JavaScript
**Status: OPEN**  
**Evidence: STATIC REVIEW**

The current `main` branch `index.html` contains **zero `<script>` tags and zero `</script>` tags**. The entire application JavaScript is absent from the current file.

This means the current source cannot attach the Start camera, measurement, reset, torch, detector, or power-calculation event handlers. The UI is therefore effectively static HTML/CSS.

This directly explains why core behavior such as camera startup and average-power calculation cannot work from this source state.

Additional structural regression observed in the same file:
- There are **two closing `</body>` tags**.
- The document ends with `</body></body></html>`.
- The measurement-action block appears outside the intended camera container because of the surrounding closing `</div>` structure.
- There are duplicate CSS declarations for several selectors, including `.live-power`, `.power-card`, `.status`, `.grid`, `.card`, `.settings`, `.power-display`, and `.measurement-actions`. These are not themselves the primary blocker, but increase regression risk.

**Expected:** `index.html` must contain the complete application JavaScript and valid document structure. The Start camera, measurement, detector, power calculation, lifecycle, and torch handlers must be present and executable.

**Verification:** Static inspection should confirm the application script exists and parses. Browser verification should then confirm Start camera, measurement controls, pulse detection, and power displays function.

### Impact on previous findings
- **QA-018:** Cannot be meaningfully runtime-verified against this source while the application JavaScript is absent.
- **QA-017:** The previously verified escaped-newline issue remains absent, but the current build has a new and more severe structural regression.
- Previous static fixes in JavaScript cannot be considered present in the current source because the JavaScript itself is missing.

### Current verdict
**BLOCKED — NOT READY FOR CONTROLLED PHYSICAL TESTING.**

The current repository state must first restore the application JavaScript and valid document structure. No application code was changed by QA during this review.


## Latest independent QA re-review — 2026-09-23 (post QA-019)

### QA-019 re-check
**Status: FIXED by static inspection**

The current `index.html` now contains one application `<script>` block and the previously missing JavaScript has been restored. The document has one closing `</body>` tag and no literal escaped-newline sequences.

### QA-020 — BLOCKER — Camera startup function uses await in a non-async function
**Status: OPEN**  
**Evidence: STATIC REVIEW**

The current source declares `startCamera(){...}` rather than `async function startCamera(){...}`, but the function contains `await navigator.mediaDevices.getUserMedia(...)` and `await video.play()`.

Because `await` is used inside a non-async function, the JavaScript parser will reject the entire script. Consequently none of the application event listeners or camera/measurement logic can execute.

**Expected:** Declare `startCamera` as an async function (or otherwise remove await usage).

**Verification:** Run the embedded script through a JavaScript parser and load the page in a browser console; confirm no syntax error and that Start camera attaches and executes.

### QA-021 — BLOCKER — Average-power calculation is off by 1000×
**Status: OPEN**  
**Evidence: STATIC REVIEW**

Both average-power calculations currently use:

`completeIntervals * 3600000000 / (elapsedMs * meterConstant)`

The instantaneous calculation correctly uses `3600000 / (interval * meterConstant)`. The cumulative average should use the same energy constant of 3,600,000, not 3,600,000,000.

For example, with a 1000 impulses/kWh meter and a 3.6-second pulse interval, the expected power is approximately 1000 W. The current average formula produces approximately **1,000,000 W** for the same interval.

This is a direct measurement-correctness blocker.

**Expected:** Use 3,600,000 in the cumulative-average formula, consistent with the instantaneous-power formula.

**Verification:** T02–T05, plus a static formula check against known intervals.

### Additional observation
The current source has duplicate CSS declarations and some duplicated UI assignments, but these are lower priority than QA-020 and QA-021. No physical-device tests have been performed.

### Current verdict
**BLOCKED — NOT READY FOR CONTROLLED PHYSICAL TESTING.**

The missing-JavaScript regression (QA-019) is now statically resolved, but the restored script currently contains two new release-blocking defects: a JavaScript syntax error in camera startup and a 1000× cumulative-average calculation error.


### QA-021 correction — 2026-09-23
**Status: REJECTED / INVALID FINDING**

QA-021 incorrectly concluded that 3,600,000,000 is 1000× too large for the cumulative average. The implementation's elapsedMs value is explicitly measured in milliseconds (performance.now() - firstPulseTime). Therefore the cumulative formula must use 3,600,000,000 when dividing directly by elapsed milliseconds. The equivalent seconds-based formula uses 3,600,000 after converting elapsed milliseconds to seconds.

Correct relationship:
- Instant power: 3,600,000 / (intervalSeconds × meterConstant).
- Cumulative average: (completeIntervals × 3,600,000,000) / (elapsedMs × meterConstant).

Example: 1000 impulses/kWh and a 3.6-second interval gives approximately 1000 W. QA must not recommend replacing 3,600,000,000 with 3,600,000 unless elapsedMs is first converted to seconds.

This correction supersedes the previous QA-021 conclusion. Future reviews should verify units/dimensions before recommending a change to this formula.

### Power calculation unit alignment — 2026-09-23

#### QA-021 — INVALID FINDING — superseded and explicitly documented
The previous QA-021 claim that the cumulative-average factor 3,600,000,000 was 1000× too large was incorrect. The source uses `elapsedMs`, which is milliseconds. The repository now makes the unit convention explicit with named constants:
- `POWER_FACTOR_SECONDS = 3,600,000` for interval values expressed in seconds.
- `POWER_FACTOR_MILLISECONDS = 3,600,000,000` for elapsed values expressed directly in milliseconds.

The cumulative-average formula is therefore:
`completeIntervals × 3,600,000,000 / (elapsedMs × meterConstant)`.

The equivalent seconds-based formula is:
`completeIntervals × 3,600,000 / (elapsedSeconds × meterConstant)`.

The application code has been aligned so all power calculations use the named constants rather than repeated magic numbers. A regression test (T21) has been added to TEST_PLAN.md. This entry supersedes the earlier QA-021 recommendation to change the factor to 3,600,000.

#### Current status
Static code alignment is complete. Physical/runtime verification of known-load readings remains outstanding.
