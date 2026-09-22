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
