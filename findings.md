# Findings

## 63A average-current limiting (approved and implemented)
- Target is 63A; preserve every original protection threshold and timer. Continuous high load can still trip the existing average OCP thresholds, which are also scaled by gears_level.
- Current actual duty target is fixed at 100%; analog proportional throttle and gear-duty scaling branches are inactive. This change does not reactivate them.
- Dedicated 83% floor applies only to average-limit-driven downward adjustment when current actual duty and the normal ramp target are both already >=83%; global 15% minimum and startup remain unchanged.
- The existing peak-limit decision takes priority. At the average-limit floor, the existing over-current status remains set while current is high; no protection is suppressed.
- Source edits completed in USER/parameter.h and MOTOR_CONTROL/motor_control.c, with Chinese GBK comments and no post-edit verification, as requested.

## 2026-10-13: Remove switch / restore LED3
- Completed source-only removal/restoration in 12 files. No post-edit static checking or hardware validation was performed, per the user.
- Actual edits were direct byte-preserving .c/.h replacements, not Python. Original GBK bytes and line endings outside changed spans were retained.
- Current request supersedes all prior PA14 safety-switch requirements.
- Safety uses PA14, not PA4 EN. PA4 over-current/emergency fault handling must remain.
- Integration includes USER board/parameter/LED/main/ISR/user state, DRIVE PWM helpers, and MOTOR_CONTROL startup/brake handling.
- `LED_PA14_EN=0` plus a compile-time error prevents LED restoration; fault-only mapping and early return also suppress original displays.
- Source reads are only for edit anchors. No post-edit checking, compilation or flashing.
- Current `ON_OFF_DIGITAL_EN` is 0 (ADC mode); preserve this actual setting rather than relying on historical notes. Keep the debounce and hard-brake settings unchanged.
- LED original six-way map, full scan and SOC/gear handler are still present under `LED_PA14_EN`; remove safety-only conditionals to restore them.
- PWM removal includes repeated polling guards, `Safety_Brake_Motor`, and the safety-aware `PwmOutputsEnable` wrapper; retain ordinary `Brake_Motor` output logic and hardware break configuration.
- User trigger debounce should remain intact; remove only safety restart counters/latch, latch-specific critical section and user-state gating. Preserve real-time and latched fault checks.
- Motor state machine should retain normal MOTOR_HBRAKE timing/output and MOTOR_BRAKE2STOP, deleting only the dedicated safety branches and MOS-test polling.
- GPIO boot currently leaves PWM pins in safety GPIO-low mode until main remuxes them. Restore normal `Bsp_Pwm_Init()` in `Bsp_Gpio_Init()` and remove the safety-only GPIO-low helper; boot MOS self-test becomes unconditional again after offset calibration.
- PA14 must no longer be wired as a grounding switch once LED drive is restored; communicated the hardware consequence to the user.
- ADC/TIM14 ISR safety wrappers can be removed without changing ADC capture, over-current detection or PA4 EXTI handler.
- Header/global declarations for safety latch, brake activity, IPD activity and GPIO-low helper must be removed along with their implementations. Main IPD returns to its normal flag-driven call.

## Safety hard-brake synchronization
- Latest user request supersedes the previous coast/all-six-low stopping interval. ON/OFF release now uses enabled `HBrake_En=1`; shared hard-brake hold is `HBrake_Time_MS_CNT` (400 ms), with the existing brake-to-stop state afterward.
- Current `motor_safety_poll()` forces MOTOR_STOP, MOE-off and GPIO-low on every call, and every `Brake_Motor()` entry is guarded, so simply changing the state is insufficient.
- Reuse the existing MOTOR_HBRAKE / MOTOR_BRAKE2STOP states and counters; add a tightly scoped, fixed-brake-only driver entry while keeping all normal driver callbacks/core ISRs blocked by the latch.
- Safety event during a non-STOP motor state requests braking once. Boot-held or already-stopped safety does not energize a bridge. Existing current faults and fault latches cancel safety braking and keep all outputs off.
- Track main's blocking IPD execution separately. Clamp outputs during in-flight IPD and do not energize the brake until the binary routine returns; its pulse callbacks remain blocked.
- Do not clear the restart latch until the brake is finished and a valid release/repress occurs. Pressing during braking consumes that attempt instead of deferring a restart.
- Continue GBK Chinese comments and source-only modifications; no static checker/tests/build/flash.

## Resumption after write permission restored
- Actual source already has the earlier safety polling and low-active PB0 code. Failed old-script application inserted duplicate definitions into parameter.h, board.h, user_control.h before stopping at drv_pwm.h. Merge these duplicates in place, not a baseline rollback.
- Targeted edits now: PB0 pressed uses Bit_SET / ON_OFF_PRESSED(), GPIO_Mode_IPD, coherent Chinese GBK comments on the prior safety/ADC/LED changes. PA14 remains low-active as agreed; remove the conflicting duplicate active-high SAFETY_PRESSED definition.
- Reads are limited to obtaining current edit anchors; no static analysis, test, compile, or firmware flashing is requested/performed.

## Latest user correction
- PB0 ON/OFF is HIGH when pressed, LOW when released (supersedes all earlier low-active PB0 notes). Digital input uses pull-down; ADC fallback remains >600 ON / <550 OFF.
- Add Chinese comments to the implementation changes, encoded as GBK/CP936.
- User explicitly requests no static checking and no compilation. Do not run further audits/tests/builds; make source edits only.
- Source diff on resumption is empty: previous attempted firmware writes did not execute. Implement the agreed feature set using the corrected polarity.

## Requirements clarified by second screenshot (2026-10-10)
- PB0 default digital input: low = pressed, high = released, about 10 ms debounce; macro fallback to current ADC >600 ON / <550 OFF. Remove PB0 from ADC scan in digital mode and update subsequent channel ranks. PA7 fly-machine boot check stays unchanged.
- PA14 exclusively safety input: LOW means pressed/stop. No EXTI14 per later user override; poll periodically. Existing PA4 protection interrupt remains.
- Safety trip removes PWM drive, calls Stop_Motor and latches; hold all six PWM pins as low GPIO outputs even if binary core attempts output re-enable. Cover IPD, commutation, error recovery, wakeup, and boot self-test.
- After a safety event, merely releasing safety never restarts. With safety released, confirm ON/OFF released, then accept fresh press only when original faults permit.
- At power-on with safety pressed, skip bridge-energizing MOS self-test.
- Macro-disable all PA14 LED drive/scan paths; keep available PA1/PA13 fault blinking only. Cancel SOC and gear LED display, retain gear control.
- Do not compile either variant or flash; only static verification. Do not claim memory usage/bench tests.
- PA14 is also SWCLK; depressed switch may interfere with SWD. Motor coasts after drive removal; not a mechanical instant brake.

## Implementation decisions
- Snapshot of tracked source and SHA256 baseline saved outside repo: `C:/Users/mym02/AppData/Local/Temp/nx32g0001-safety-871q1ezf/`.
- Read-only FromElf disassembly of the existing AXF confirms IPD disables interrupts around each ~60 us pulse and calls the six app-side HOnLOn helpers. It re-enables interrupts between pulses. Guard all driver energizing entries, plus TIM1/ADC/TIM14 and the main IPD call/return. Polling latency can exceed one PWM period during masked code; no hard real-time bound is claimed.
- GPIO low clamp detaches all six PA0/5/6/8/9/10 PWM pins from AF. Only a confirmed new press after safety-high/released debounce may restore AF, with MOE/CCER off during remux.
- Restart state/counters are separate volatile values; short PRIMASK-preserving critical sections serialize rearm with ISR trips. Never clear existing faults as part of safety release.
- Keep ordinary ADC result capture and fault recovery active while safety-latched; skip motor-core ISR processing, not the entire ADC IRQ.
- Map existing alarm blink to the two physical PA1/PA13 directions, avoiding the old logical-to-physical map that routes alarms through PA14.

## Motor output paths inspected
- TIM1 update calls app state machine; ADC and TIM14 IRQs call binary core directly. Main calls blocking `ipd_detect()` when flagged.
- `Stop_Motor()` only zeros CCR1..3 and clears CCER enable bits. It does not prevent library MOE re-enable. Need GPIO-low clamp while latched.
- Driver has commutation, IPD, individual MOS self-test, brake and start helpers writing CCER. All runtime pin remux in visible source is `Bsp_Pwm_Init()`.
- Current calibration and MOS test run before NVIC initialization. MOS test loops need explicit polling because TIM1 IRQ is disabled there.
- Fault recovery and wakeup explicitly enable TIM1 MOE in three `user_state_control()` branches; route through a safety-aware output-enable wrapper.
- ADC scan uses rank 0..7; calibration/MOS-test restore it through `drv_adc_init()`. Binary core takes IPEAK channel, not scan rank, as runtime parameter.
- Core struct ABI cannot change; add separate volatile safety state rather than editing library structs.

## Request recovered from screenshot
- User requests normal GPIO polling for the safety switch; no external interrupt and no compilation.
- Previous response specifies PA14, a stop latch, and restart only after ON/OFF release and repress.
- Switch polarity and other preceding requirements are not visible in the screenshot.

## Initial repository observations
- No previous planning files found; resolver returned legacy root.
- No pending source changes; only existing Keil UI and generated artifacts differ from HEAD.
- `USER/led.h` currently assigns LED_PIN3 to GPIO_Pin_14: must investigate before repurposing.
## Confirmed wiring conflict and required clarification
- `USER/led.c:35-44` configures PA13/PA14 AF7 and PA1/PA13/PA14 as push-pull LED outputs.
- `LED_Scan()` and LED3–LED6 bright helpers repeatedly reconfigure PA14 as input/output and write its output level. Merely adding safety GPIO initialization would not work.
- Need actual PA14 safe/unsafe polarity and whether the new hardware removes or relocates the original multiplexed LED wiring. Neither can be inferred safely from the screenshot or source.
- `motor_on_off_control()` reads `user_list.on_start_adc` (PB0) with two thresholds and >10-sample debounce.
- `user_state_control()` currently has no safety latch. Existing error-stop release handling is separate and must be preserved.
- The 1 ms main loop calls trigger processing before fault handling and user state; final motor enable is currently only gated by `gToolEn`.
- Existing EXTI4_15 handler handles PA4 EN. Do not confuse this with the requested PA14 safety switch.
- `Bsp_Gpio_Init()` calls `led_init()` and initializes PA4 EN, but has no PA14 safety input initialization.
- ADC trigger input is PB0/ADC channel 1 (`ON_START_PIN`); PA7 is the speed ADC. The old ON_OFF_AD aliases also point to PA7 but must not be assumed to define the actual trigger logic.
