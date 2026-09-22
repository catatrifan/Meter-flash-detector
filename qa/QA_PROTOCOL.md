# Meter Flash Detector — QA Protocol

## Purpose
Independently verify the Meter Flash Detector against its product requirements and identify defects before release. Treat the implementation as untrusted until verified.

## Independence
- Review the actual current repository, not a prior summary.
- Do not assume previous AI-generated code is correct.
- Attempt to falsify behavior and requirements.
- Re-check previous QA findings independently.
- Never mark a finding VERIFIED unless the fix has been independently checked.
- Never claim physical-device behavior was tested unless the tester actually performed that test.

## Scope
Review:
1. Pulse detection correctness: false positives, false negatives, double counts, missed pulses.
2. Power calculation and meter-constant handling.
3. Camera/frame sampling and browser compatibility, especially iPhone Safari.
4. Lighting, reflections, exposure, autofocus, movement, and ROI behavior.
5. State transitions, start/stop/reset, backgrounding, camera lifecycle.
6. Performance and resource use.
7. UX clarity and whether displayed measurements have enough evidence.
8. Privacy: no unnecessary frame upload, storage, analytics, or third-party runtime dependency.
9. Regression risk from every implementation change.

## Severity
- BLOCKER: prevents trustworthy measurement or creates a serious safety/privacy/data-integrity issue.
- HIGH: materially affects correctness or makes a core feature unreliable.
- MEDIUM: meaningful defect with a workaround or limited impact.
- LOW: minor defect, polish, or maintainability issue.

## Finding format
For every finding include:
- ID
- Severity
- Status
- What is wrong
- Exact file/function/area
- Why it matters
- Reproduction or reasoning
- Expected behavior
- Recommended fix
- Verification method

## Evidence rules
Separate:
- STATIC REVIEW: code/repository evidence.
- BROWSER TEST: reproducible browser behavior.
- PHYSICAL DEVICE TEST: actual phone/meter testing.
Do not turn static reasoning into a claim of real-world accuracy.

## Acceptance principle
The app should not present a precise-looking power value as trustworthy unless the detector has sufficient evidence that genuine meter pulses are being observed consistently.

## Regression rule
Every confirmed defect should lead to a regression test, documented either in TEST_PLAN.md or an automated test where practical.

## Review states
OPEN -> DEVELOPER FIXED -> QA VERIFYING -> VERIFIED.
Only QA can mark a finding VERIFIED.

## Current technical risks to pay special attention to
- requestAnimationFrame may not sample every camera frame; requestVideoFrameCallback should be considered.
- Red intensity should be spatial/area-aware enough to resist reflections and isolated bright red pixels.
- Adaptive baseline must tolerate exposure/lighting changes without creating false pulses.
- Page visibility/backgrounding must not corrupt pulse intervals or detector state.
- Camera startup failures must clean up acquired streams.
- Canvas should not be resized every analysis frame.
- Measurement UI should expose enough evidence to judge reliability.
- Changing the meter constant during a measurement should not silently reinterpret prior intervals.

## Power calculation unit convention
- `performance.now()` returns milliseconds, so `elapsedMs` and `firstPulseTime` differences are in milliseconds.
- Instant power uses interval seconds: `3600000 / (intervalSeconds × meterConstant)`.
- Cumulative average uses elapsed milliseconds directly: `completeIntervals × 3600000000 / (elapsedMs × meterConstant)`.
- The two constants are mathematically equivalent because `3600000000 = 3600000 × 1000`.
- QA must verify units before proposing a change to either factor. Replacing 3,600,000,000 with 3,600,000 without first converting milliseconds to seconds introduces a 1000× error.
