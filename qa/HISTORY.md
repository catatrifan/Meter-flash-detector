# QA History

## 2026-09-22 — Initial independent review
- Reviewed the current main-branch implementation.
- Established the QA protocol and device test plan.
- Recorded nine initial findings in CURRENT_REVIEW.md.
- No physical meter/iPhone accuracy tests have been performed by QA.
- Findings remain OPEN until implementation changes are independently verified.

## 2026-09-22 — Re-review after developer fixes
- Re-reviewed QA-001 through QA-009 against the current main branch.
- QA-001, QA-002, QA-003, and QA-007 are PARTIALLY FIXED.
- QA-004 and QA-005 are FIXED by static inspection but still require physical/device verification.
- QA-006 and QA-008 are FIXED by static inspection.
- QA-009 remains OPEN.
- Added QA-010 (small coherent red objects can pass the spatial filter), QA-011 (camera lifecycle cleanup), and QA-012 (pulse confidence vs measurement confidence).
- No physical iPhone/meter tests were performed in this review.
- Implementation is suitable for controlled physical testing, but not for claiming real-world measurement accuracy.

## 2026-09-22 — Re-review of latest deployed build
- Re-read the QA protocol, current review, test plan, and history, then inspected the latest main-branch index.html.
- Previous fixes for frame callbacks, backgrounding, camera-start cleanup, canvas sizing, and meter-constant reset remain present.
- QA-001, QA-002, QA-003, and QA-007 remain PARTIALLY FIXED.
- QA-004, QA-005, QA-006, and QA-008 remain FIXED by static inspection; physical verification is still outstanding where applicable.
- QA-009 remains STILL OPEN.
- Added QA-013: the new “Average Power · Last 5 Readings” display uses a median interval, not an arithmetic average.
- Added QA-014: the new prominent power displays do not by themselves establish measurement stability.
- No physical iPhone/meter testing was performed.

## 2026-09-22 — Latest QA review
- Re-reviewed current main branch and all prior findings.
- QA-009 is now FIXED by static inspection through the time-based power-history graph.
- QA-013 is FIXED by the new “Average power · Since Start” label/calculation.
- QA-002 and QA-011 improved but require verification; QA-010 remains open.
- Added QA-015: since-start average is biased by the incomplete first pulse interval.
- Added QA-016: page lifecycle restoration can leave camera controls inconsistent after pagehide/bfcache restoration.
- No physical iPhone/meter testing was performed.


## 2026-09-22 — User-reported UI regression re-review
- Inspected the current main-branch `index.html` after the user reported three issues: camera cannot start, power readers overlap the target, and literal escaped-newline text appears below Measurement history.
- Added QA-017 (BLOCKER): literal `\\n` sequences are present in HTML/CSS/JavaScript. The JavaScript occurrences make the script syntactically invalid, explaining the camera-start failure; the same malformed escaping explains the overlay styling failure and visible escaped-newline text.
- These issues were not identified in the previous static review.
- No physical iPhone/meter test was performed.


## 2026-09-22 — QA re-review after QA-017 fix
- Re-read the current QA protocol, review, test plan, history, and latest `index.html`.
- QA-017 is **VERIFIED by static inspection**: no literal \\n sequences remain and the embedded JavaScript passes syntax validation.
- QA-015 is fixed through first-pulse-based averaging using complete intervals only.
- QA-016 is fixed through pagehide/pageshow camera-control synchronization.
- Red detection thresholds were tightened; QA-010 remains open pending physical false-positive testing.
- No physical iPhone/meter testing was performed.
- Current status: suitable for controlled physical testing, but real-world measurement accuracy remains unverified.


## 2026-09-22 — QA review of average-power report
- Inspected the latest `index.html` after the user reported that Average Power is not working.
- Added QA-018: average power is only recalculated on accepted pulse events, can remain blank until the second valid pulse, and can remain stale between pulses. The valid-interval accounting also increments the pulse counter before interval validation.
- No physical iPhone/meter test was performed; the reported runtime symptom is treated as user-provided evidence, not independently reproduced.


## 2026-09-22 — QA-018 follow-up and diagnostic fix
- Re-checked the average-power calculation after the user reported that the average display shows “1” after the second pulse.
- Updated interval accounting so only valid pulse intervals advance the complete-interval count.
- Added calculation diagnostics showing latest interval, instantaneous power, average power, and meter constant.
- Static review still cannot reproduce the reported runtime value or establish whether “1” is a calculation, input/timing, or rendering issue.
- QA-018 remains QA VERIFYING pending browser/device testing.


## 2026-09-23 — Independent QA re-review
- Re-read the current QA protocol, current review, test plan, history, and latest `main` branch `index.html`.
- Found **QA-019 BLOCKER**: the current `index.html` contains no JavaScript at all (0 `<script>` tags).
- This prevents camera startup, pulse detection, measurement state, power calculation, reset, torch control, and lifecycle handlers from functioning.
- Also observed duplicate `</body>`, malformed document ending, and duplicate CSS declarations.
- This is a new regression more severe than the previously reported average-power issue.
- QA made **no application-code changes** during this review; only QA documentation was updated.
- Current verdict: **BLOCKED — NOT READY FOR CONTROLLED PHYSICAL TESTING.**


## 2026-09-23 — QA re-review after QA-019 regression
- QA-019 is now **FIXED by static inspection**: the application JavaScript has been restored.
- Added **QA-020 BLOCKER**: `startCamera` uses `await` but is not declared `async`, which makes the embedded JavaScript syntactically invalid.
- Added **QA-021 BLOCKER**: cumulative average power uses 3,600,000,000 instead of 3,600,000, producing a 1000× error.
- No application code was changed by QA.
- No physical iPhone/meter tests were performed.
- Current verdict: **BLOCKED — NOT READY FOR CONTROLLED PHYSICAL TESTING.**


## 2026-09-23 — Average-power unit alignment
- Corrected the actual defect in `updatePowerDisplay()`: it was using 3,600,000 while dividing by an elapsed value in milliseconds, making the displayed cumulative average 1000× too low.
- Aligned all power calculations around explicit named constants for seconds and milliseconds.
- Documented the unit convention in QA_PROTOCOL.md.
- Added T21 to TEST_PLAN.md for a known 3.6-second / 1000 imp/kWh calculation check.
- The earlier QA-021 finding that 3,600,000,000 was too large remains a documented invalid historical finding and is superseded by the unit-aligned implementation.
- No physical iPhone/meter test was performed.


## 2026-09-23 — Measurement controls redesign
- Replaced Meter constant/presets with an explicitly entered Impulse constant; no preset or persisted value is used.
- Start now requires a valid constant and reports a validation message otherwise.
- Replaced Stop with Pause and added Start-based resume behavior.
- Added active measurement timer (hh:mm:ss) inside Start, excluding paused time.
- Added pulse and reading statistics requested by the product UI.
- Changed graph x-axis labels from Older/Latest to elapsed measurement time.
- Added T22–T26 regression tests to TEST_PLAN.md.
- No physical-device tests were performed.
