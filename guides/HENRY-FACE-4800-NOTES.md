# Henry Face 4800 Notes

This is the companion note for
[henry-face-motors-4800.src](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors-4800.src).

It carries the longer explanation that does not belong inside the
block source itself.

## Intent

The block source follows the `IDEAL Simplified` style:

- one load block
- one short title on line `0` of each block
- exact `16 x 64` block discipline
- compact definitions
- sparse comments in source

## What is confirmed

- Volatco / GA144 drives the parallel data bus through node `10007`.
- The external data lines `D00..D17` are written through `DATA R!`.
- Local notes indicate `D00` reaches the motor-driver path to `BI1A`.
- The motor driver is a `TB6612FNG` dual H-bridge.

## What is not yet confirmed

- which TB6612 channel drives eyes versus mouth
- which data bits drive `IN1`, `IN2`, `STBY`, and optional PWM
- whether `PWMA` and `PWMB` are hard-wired active

The unresolved mapping is intentionally localized in block `4802`.

## Block map

- `4800` load block
- `4801` basics and shared state
- `4802` bit operations and wiring constants
- `4803` standby and PWM words
- `4804` pin-level words
- `4805` drive semantics
- `4806` user actions
- `4807` cycle words and reset
- `4808` test helpers

## Relationship to the working source

[henry-face-motors.fth](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth)
remains the easier editing surface.

[henry-face-motors-4800.src](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors-4800.src)
is the block-oriented source intended to match professional
polyFORTH practice.

## Loading workflow

The source assumes the standard interactive setup described in
[task/README.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/task/README.md):

1. `BRIDGE LOAD`
2. `TALK`
3. `SPAN`
4. `0 10007 HOOK`

After that, the block code uses `DATA R!` through `FACE-BUS!`.

## Bench-first rule

Do not treat the action words as finalized hardware truth until the
bit map is verified on the bench.

Use:

- `FACE-OFF`
- `TEST-BIT`

to confirm mappings before trusting:

- `BLINK`
- `MOUTH`
- `FACE-RESET`
