# 63A average-current limit and 83% duty floor

## Goal
Implement the approved 63A average-current limiting scheme with a dedicated 83% high-duty reduction floor. Preserve all existing protections, the 15% global minimum, startup and hard braking, and the completed PA14 safety-switch removal / LED restoration. Direct source edits with Chinese GBK comments only; no Python edits, compilation, flashing, tests or static checks.

## Current Phase
Complete — implemented the approved 63A average-current limit and dedicated 83% high-duty reduction floor. The prior safety-switch/LED task is also complete.

## Next Step
Hand off the two source changes without compilation, flashing, static checks or tests.

### Phase 7: 63A average-current limit / 83% duty floor
**Status:** complete
- [x] Enable LIMIT_AVG_CURRENT_EN, set the target to 63A and add a dedicated 83% duty parameter.
- [x] Saturate average-current-driven PWM reduction at 83% only after reaching that high-duty range; retain normal startup, lower targets and peak-limit priority.
- [x] Retain the 15% global floor, original step timing and all protection thresholds.
- [x] Add Chinese GBK comments and hand off without compilation, flashing, static checks or tests.

Approved implementation plan: C:/Users/mym02/.claude/plans/enumerated-meandering-lemon.md. User explicitly chose 63A with original protections unchanged; sustained overload may still trip the existing faults. No Python edits were used.

Phase 7 write note: the combined two-file write was blocked before execution by the auto permission classifier. Smaller per-file edit requests subsequently succeeded for both files; no post-edit checking was performed.

### Phase 5: Remove safety switch and restore LED3
**Status:** complete
- [x] Remove PA14 switch initialization, polling, restart latch and dedicated brake integration.
- [x] Restore PA14 LED output, full scan map and normal display paths.
- [x] Preserve Chinese GBK comments and unrelated trigger/protection behavior.

### Phase 6: Source-only handoff
**Status:** complete
- [x] Report changes without performing static checks, tests, compilation or flashing.

## 2026-10-13 edit notes
- User declined creating a Python edit script and requested direct .c/.h modifications. No Python edit script was created or run; used inline byte-preserving source replacements.
- Initial header replacement expected one definition per LED conditional, but the first block contains two; stopped before any writes and corrected that edit anchor.
- One application-edit command had a replacement-delimiter syntax error and stopped before execution; corrected the delimiter and applied all three files successfully.
- All 12 intended source/header files were written successfully. No post-edit source reads, reviews, checks or firmware builds were performed.

## Historical request (superseded by Phase 5)
The previous PA14 low-active safety-switch polling and dedicated safety-brake implementation is being removed at the user's request. Earlier notes below describe its history only.

### Phase 4: Safety hard-brake stop
**Status:** complete
- [x] Reuse the existing hard-brake timing/output behavior for a runtime safety event.
- [x] Preserve the restart latch, original fault priority, and no-energizing boot-held safety behavior.
- [x] Add Chinese GBK comments; source edits only, no static checks/build/flash.

## Latest requirement (supersedes coast-only safety stop)
The user enabled hard braking for ON/OFF release and explicitly requests the same brake-to-stop behavior on the safety switch. Do not retain the old all-six-low coast behavior during a permitted braking interval; keep the no-restart lock and go all-low after the brake finishes or a fault requires it.

## Current recovery note
Permission now works. The stale config script stopped after partial header edits because the prior implementation was already on disk. The recovery script correct_current_sources.py successfully updated 13 source/header files in place, removed the duplicate definitions, changed PB0 to high-active/pull-down, and added Chinese comments using GBK bytes without re-encoding existing comments. No rollback of unrelated user changes.

### Phase 1: Inspect and resolve requirements
**Status:** complete
- [x] Check current diff and saved planning state.
- [x] Establish PA14 ownership, switch polarity, and ON/OFF behavior.
- [x] Resolve any hardware requirements not recoverable from source/screenshot.
- [x] Trace all motor output paths, ADC variants, and LED behavior.

### Phase 2: Implement GPIO polling and restart latch
**Status:** complete
- [x] Make focused encoding-preserving source edits.
- [x] Retain existing unrelated protections and changes.

### Phase 3: Handoff without checks
**Status:** complete
- [x] Record the applied edits for handoff; static checking, compilation and flashing were deliberately not performed, as requested.

## Decisions Made
- Screenshot supplies the agreed polling-only approach; no new external interrupt for PA14.
- Existing changes at entry are Keil UI and build outputs only; leave them untouched.

## Errors Encountered
- On permission recovery, update_config.py partially wrote headers before stopping because ENABLE_PWMOUT already called PwmOutputsEnable. Existing implementation was already on disk, so the saved no-source-changes assumption was stale. Adapting current source and merging only duplicate declarations/macros from the interrupted application; not rerunning the old scripts.
- Broad tool-location Glob under D:/ timed out; use the exact ARMCC directory from the existing Keil build log instead. No build was attempted.
- Initial reg_gpio.h lookup used HAL include path; located actual header with Glob under Device/MM32G0001/Include.
- Auto permission-classifier service (claude-sonnet-5[1m]) remained unavailable on resumption: Agent, temporary patch-script Write, and a normal byte-preserving source-edit Bash request were not executed. Firmware sources remain unchanged; wait for the service or a user-controlled permission-mode change.
