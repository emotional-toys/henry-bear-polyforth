# Henry Face Motor Experimental Setup

This procedure is for identifying the Volatco-to-TB6612FNG wiring used by the Henry bear face system and validating the polyForth control block in [`henry-face-motors.fth`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth).

The current code assumes only what has been confirmed so far:

- Volatco parallel data signals `d00..d17` are driven through node `10007` using `DATA R!`.
- The motor driver is a `TB6612FNG`.
- Local notes indicate `d00` reaches the motor-driver path to `BI1A`.

Everything else is still under test and should be measured, not assumed.

## Objective

Establish the actual bit map from Volatco `DATA` bus lines to TB6612FNG control pins so the following words can be finalized:

- `EYES-IN1-BIT`
- `EYES-IN2-BIT`
- `MOUTH-IN1-BIT`
- `MOUTH-IN2-BIT`
- `STBY-BIT`
- optionally `EYES-PWM-BIT` and `MOUTH-PWM-BIT`

## Required hardware

- Volatco with development access enabled
- Henry face board with TB6612FNG attached
- stable bench power supply
- serial connection for the IDE / polyForth terminal
- voltmeter
- logic probe if available
- datasheet or board drawing for the TB6612FNG carrier / face board

## Required references

- [task/README.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/task/README.md)
- [henry-face-motors.fth](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth)
- Volatco docs for GA144 data bus and external headers:
  - https://volatco.github.io/
  - https://volatco.github.io/pages/db002/

## Safety constraints

- Begin with the face mechanics disconnected if possible, or with motor power limited.
- Use short pulses only while mapping signals.
- Never assume a line is `STBY`, `PWM`, or a motor direction input until measured.
- Return outputs to idle between tests using `FACE-OFF`.
- If the mechanism binds, stop immediately and remove motor power.

## Board preparation

1. Configure Volatco for development mode.
2. Install the no-boot jumper if your session requires manual IDE control.
3. Connect IDE serial access.
4. Power the board with a stable supply.
5. Keep the voltmeter ground tied to board ground.
6. Have the TB6612FNG pinout in front of you.

## Software preparation

Load the standard IDE path and attach to the bus-driving node as described in local notes:

1. `BRIDGE LOAD`
2. `TALK`
3. `SPAN`
4. `0 10007 HOOK`

At that point, node `10007` should be the active node for bus tests.

Load or paste the helper file:

1. load [`henry-face-motors.fth`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth)
2. run `FACE-OFF`
3. run `FACE-STATUS`

Expected result:

- `FACE-BUS=0`
- the current placeholder bit assignments are displayed

## Known starting assumption

The current file assumes:

- `EYES-IN1-BIT = 0`

That is not treated as final truth. It is simply the first candidate because the local hardware note says `d00` reaches the path to motor-driver pin `BI1A`.

## Measurement procedure

### Phase 1: confirm raw bus control

For each test:

1. run `FACE-OFF`
2. run `N TEST-BIT`
3. probe the expected data line or TB6612 input
4. record what changed
5. run `FACE-OFF` again

Start with:

- `0 TEST-BIT`
- `1 TEST-BIT`
- `2 TEST-BIT`
- `3 TEST-BIT`
- `4 TEST-BIT`
- `5 TEST-BIT`
- `6 TEST-BIT`

Record:

- which TB6612 input pin changes
- whether the line goes high cleanly
- whether any second line moves unexpectedly

### Phase 2: identify standby and PWM behavior

The TB6612FNG will not drive motors unless standby and enable conditions are satisfied.

Measure whether any of the tested bits correspond to:

- `STBY`
- `PWMA`
- `PWMB`

If `PWMA` and `PWMB` are tied high in hardware, note that and leave `USE-SOFTWARE-PWM` disabled in the Forth file.

If either PWM line is software-driven, update the corresponding bit constants.

### Phase 3: identify channel assignment

Determine which bridge channel drives which mechanism:

- eyes on channel A or B
- mouth on channel A or B

This can be done by:

1. enabling what you believe is `STBY`
2. driving one candidate `IN1` high and its paired `IN2` low
3. applying a very short pulse
4. observing whether eyes or mouth move

Then reverse the pair:

1. first direction: `IN1=1`, `IN2=0`
2. second direction: `IN1=0`, `IN2=1`

Record for each pair:

- mechanism affected
- direction of movement
- whether the pulse was strong enough
- whether the mechanism returned freely

## Suggested observation table

Use a table like this while probing:

| DATA bit | TB6612 pin observed | Voltage change | Function guess | Mechanism result |
|---|---|---:|---|---|
| d00 | BI1A | high | eyes or mouth IN1 | none / motion |
| d01 | ? | ? | ? | ? |
| d02 | ? | ? | ? | ? |
| d03 | ? | ? | ? | ? |
| d04 | ? | ? | ? | ? |
| d05 | ? | ? | ? | ? |
| d06 | ? | ? | ? | ? |

## Minimal command set during probing

Use these words from [`henry-face-motors.fth`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth):

- `FACE-OFF`
- `FACE-STATUS`
- `N TEST-BIT`

Do not use `BLINK`, `MOUTH`, or `FACE-RESET` as real motion tests until the bit map is confirmed.

## When the map is good enough

You are ready to replace the placeholders in `henry-face-motors.fth` when all of the following are known:

- exact bit for eyes direction input 1
- exact bit for eyes direction input 2
- exact bit for mouth direction input 1
- exact bit for mouth direction input 2
- exact standby behavior
- PWM wiring behavior

At that point the high-level words should work without structural changes:

- `BLINK`
- `MOUTH`
- `SMILE`
- `FACE-RESET`

## Current unresolved questions

- Is the `BI1A` note actually the eyes channel, or just the first reachable control input?
- What are the matching `IN2` lines for that same bridge?
- Is `STBY` tied high or software-controlled?
- Are `PWMA` and `PWMB` tied active or exposed on the bus?
- Are there any inverter or level-shifter stages that change polarity?

## Exit criteria

This experiment is complete when you can produce a verified mapping in this format:

- `EYES-IN1-BIT = n`
- `EYES-IN2-BIT = n`
- `MOUTH-IN1-BIT = n`
- `MOUTH-IN2-BIT = n`
- `STBY-BIT = n` or `STBY hard-wired high`
- `EYES-PWM-BIT = n` / `hard-wired high`
- `MOUTH-PWM-BIT = n` / `hard-wired high`

Once that table exists, the polyForth motor block can be finalized without guessing.
