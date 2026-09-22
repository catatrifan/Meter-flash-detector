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
