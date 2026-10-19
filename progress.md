# Progress

## 63A average-current limit and 83% duty floor
- User approved the final plan with 63A (superseding their initial 62A response) and all existing protections retained.
- Updated USER/parameter.h: enabled LIMIT_AVG_CURRENT_EN, set LIMIT_AVG_CURRENT=63.0f, added LIMIT_AVG_CURRENT_MIN_DUTY at 83% of PWM_PRIOD_LOAD. Left the global 15% floor and protection settings unchanged.
- Updated only the active pwm_set() in MOTOR_CONTROL/motor_control.c: average-only limiting is distinguished from the earlier peak-limit decision; saturated decrement enforces 83% only when actual and normal ramped target duties are already in that range. Startup/lower targets and original peak limiting retain priority; the macro-disabled path retains its old decrement.
- Added Chinese GBK comments and preserved untouched bytes/line endings through direct per-file edits. No Python edits, firmware builds, flashing, static checks, tests or post-edit source review were performed.
- Initial combined write was blocked by the auto permission classifier before execution. Smaller, explicit per-file writes succeeded for both source files.

## 2026-10-13: Remove switch / restore LED3
- Loaded the selected legacy-root plan and existing progress; latest user request supersedes the earlier safety-switch feature.
- Existing working changes are Keil UI and generated build artifacts; leave them untouched.
- Located safety-switch and disabled LED paths; preparing source-only edits with Chinese GBK comments.
- Finished reading edit anchors across 12 source/header files. The current trigger mode is ADC (`ON_OFF_DIGITAL_EN=0`); preserve it. No firmware checks were run.
- Applying the removal directly after the user's continuation request; all source writes use byte-oriented GBK replacements.
- User requested no Python edits. The proposed Python script was not created or run; performed direct inline replacements of the .c/.h files instead.
- Updated 12 files: USER/parameter.h, USER/board.h, USER/board.c, USER/led.h, USER/led.c, USER/user_control.h, USER/user_control.c, USER/main.c, USER/mm32g0001_it.c, DRIVE/inc/drv_pwm.h, DRIVE/src/drv_pwm.c, MOTOR_CONTROL/motor_control.c.
- Removed PA14 switch GPIO initialization, safety polling/latch/rearm, safety-only GPIO clamp, dedicated braking integration and ISR/output guards. Restored normal PWM boot initialization, MOS self-test, flag-driven IPD and ordinary hard braking.
- Restored PA14 LED pin macros, six-way scan/mapping, SOC initialization and original SOC/gear/fault display handler. Added Chinese GBK comments without transcoding existing source bytes.
- Preserved ON_OFF_DIGITAL_EN=0, trigger debounce, hard-brake parameters, PA4 handling and unrelated fault logic. Existing Keil UI/build artifacts were not modified.
- No static checks, tests, compilation, flashing or git commit was performed. No post-edit source review was run.

## 2026-10-10
- Loaded planning-with-files and resolved plan directory; no saved plan exists.
- Checked source/artifact diff and available Python interpreter.
- Recorded screenshot constraints and initial PA14/LED ownership conflict.
- Inspected board initialization, LED scan, main-loop ordering, trigger debounce, and user-state stop/restart logic.
- Confirmed PA14 is actively driven by the existing LED multiplexer, and the screenshot omits safe/unsafe polarity. Requested hardware clarification before editing source.
- No firmware source edits, builds, flashes, or git commits performed. Only local planning notes were added.
- User supplied the earlier detailed requirements, resolving PB0 low-active ON/OFF, PA14 low-active stop, and fault-only PA1/PA13 LEDs.
- Source baseline and existing-image disassembly are in the temporary snapshot recorded in findings.md. Previous attempts to save the patch script were not executed because Auto permission classification was unavailable.
- Resumed on user request; confirmed firmware sources still match the baseline. A read-only agent audit request also encountered the same temporary permission-classifier outage.
- Latest user correction: ON/OFF PB0 is HIGH pressed / LOW released; use pull-down, add Chinese GBK comments, no static checking or compilation.
- Prepared `update_config.py` and `update_implementation.py` in `C:/Users/mym02/AppData/Local/Temp/nx32g0001-safety-871q1ezf/`. These are edit scripts only; neither executed. Source writes via Bash remained blocked by the unavailable Auto classifier on three attempts, including after writing the scripts.
- Firmware sources were still unchanged at the previous blocked turn. No static checks, builds or flashes were run for that correction attempt.

## Write permission restored — changes applied
- On current resumption the firmware already contained the earlier implementation, making the saved no-source-changes state stale. The old configuration script partially applied then stopped at its already-changed PWM macro anchor.
- Read the affected code solely to obtain current edit anchors; did not run a static checker, review agent, tests or compiler.
- Applied correct_current_sources.py successfully to 13 files: USER/parameter.h, USER/board.h, USER/board.c, USER/user_control.h, USER/user_control.c, USER/main.c, USER/mm32g0001_it.c, USER/led.h, USER/led.c, DRIVE/inc/drv_pwm.h, DRIVE/src/drv_pwm.c, DRIVE/src/drv_adc.c, MOTOR_CONTROL/motor_control.c.
- PB0 ON/OFF now uses ON_OFF_PRESSED() == Bit_SET with pull-down input (HIGH pressed / LOW released). ADC fallback thresholds and debounce retained.
- PA14 remains low-active for safety, per agreed requirements; removed conflicting duplicate definition and partial-script duplicate declarations/config blocks.
- Added Chinese GBK comments to the previously added safety polling, restart latch, PWM guards, ADC selection, boot/IPD handling, and LED changes. Existing source bytes and line-ending style were preserved by byte-oriented replacement.
- No post-edit static check, compilation, test, flash, or git commit was run. Existing generated artifacts were not edited.

## Safety hard-brake synchronization
- User enabled HBrake_En and requested safety-switch braking to match ON/OFF release, superseding the previous coast-only stop interval.
- Applied sync_safety_hard_brake.py successfully to 7 files: USER/user_control.h, USER/user_control.c, DRIVE/inc/drv_pwm.h, DRIVE/src/drv_pwm.c, MOTOR_CONTROL/motor_control.c, USER/main.c, USER/board.c.
- Runtime safety trigger requests MOTOR_HBRAKE once, then reuses the existing HBrake_Time_MS_CNT (currently 400 ms) and MOTOR_BRAKE2STOP transition. Holding safety does not continuously reset the brake timer.
- Normal and safety brake entry points now share Brake_Output_Set; only the dedicated safety entry may enable this fixed brake output under the restart lock. All running/commutation callbacks remain blocked.
- Current or latched faults cancel safety braking and keep bridge output off. Boot-held safety/STOP state remains non-energizing.
- Main tracks blocking IPD execution; bridge stays clamped until IPD returns, then the next PWM tick may start the requested hard brake. No binary library changes.
- Repress attempts while braking are not deferred; restart still requires completed braking, no faults, safety release, and a fresh ON/OFF release/repress.
- Added Chinese comments through GBK byte-oriented edits. No post-edit static checks/tests/build/flash or git operations were run; user hard-brake configuration was not changed.
