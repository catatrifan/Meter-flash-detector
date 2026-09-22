# Meter Flash Detector — Test Plan

## How to use
Record actual results after each implementation milestone. Mark tests PASS, FAIL, or NOT TESTED. Physical-device tests must be performed by a human with an actual compatible meter and phone.

| ID | Test | Expected | Status |
|---|---|---|---|
| T01 | Camera starts on iPhone Safari | Rear camera starts and actual settings are shown | NOT TESTED |
| T02 | Known 1000 imp/kWh load at ~1000 W | Pulse interval ~3.6 s and power near 1000 W | NOT TESTED |
| T03 | Known 1000 imp/kWh load at ~500 W | Interval ~7.2 s and power near 500 W | NOT TESTED |
| T04 | Known 1000 imp/kWh load at ~2000 W | Interval ~1.8 s and power near 2000 W | NOT TESTED |
| T05 | 2000 imp/kWh at ~1000 W | Interval ~1.8 s and power near 1000 W | NOT TESTED |
| T06 | LED blocked/no genuine pulses | No false pulse count | NOT TESTED |
| T07 | Red reflection in/near ROI | No false pulse | NOT TESTED |
| T08 | Phone moves slightly | Detector recovers without sustained false pulses | NOT TESTED |
| T09 | Brightness changes/exposure shifts | No spurious pulse train; detector recovers | NOT TESTED |
| T10 | Torch on/off | Detector remains stable or clearly indicates instability | NOT TESTED |
| T11 | LED moves toward ROI edge | Behavior remains predictable; no unexplained false pulses | NOT TESTED |
| T12 | App backgrounded and resumed | Measurement is reset/interrupted safely; no corrupted interval | NOT TESTED |
| T13 | Start -> stop -> start | Fresh measurement state; no stale intervals | NOT TESTED |
| T14 | Reset during measurement | All measurement values and detector state reset | NOT TESTED |
| T15 | Meter constant changed during measurement | Prior data is not silently reinterpreted | NOT TESTED |
| T16 | Sustained bright red source | Not counted as repeated pulses | NOT TESTED |
| T17 | Long 10–30 minute run | No progressive detector lock-up or excessive performance degradation | NOT TESTED |
| T18 | Actual processed FPS observed | Processing cadence is known and documented | NOT TESTED |
| T19 | Network inspection during use | No camera frames or unnecessary personal data uploaded | NOT TESTED |
| T20 | Camera startup failure | Any acquired stream is cleaned up | NOT TESTED |
| T21 | Power formula unit check | 1000 imp/kWh with a 3.6 s interval gives ~1000 W; cumulative average uses millisecond factor 3,600,000,000 when elapsedMs is used directly | NOT TESTED |
| T22 | Impulse constant required | Field starts blank; Start without a value does not begin measurement and shows an adequate validation message | NOT TESTED |
| T23 | Pause/resume | Pause freezes measurement and timer; Start resumes without resetting prior readings | NOT TESTED |
| T24 | Measurement statistics | Pulse count, average interval, time since last pulse, peak/lowest instant power, and peak/lowest average power update from accepted pulses | NOT TESTED |
| T25 | Measurement timer | Start button displays active measurement duration as hh:mm:ss; paused time is excluded | NOT TESTED |
| T26 | Graph elapsed time | Graph x-axis displays elapsed measurement time rather than Older/Latest | NOT TESTED |

## Diagnostic trace recommended
For controlled debugging, capture timestamp, signal score, baseline, threshold, detector state, peak, pulse event, and interval. Use this trace to distinguish missed pulses from false positives.
