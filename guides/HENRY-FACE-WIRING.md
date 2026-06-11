# Henry Face Wiring Guide

This guide is for bench bring-up of the Henry face motors with a
`TB6612FNG` driver.

Use one of these control paths:

1. `Raspberry Pi -> TB6612FNG`
2. `Volatco -> TXS0108 -> TB6612FNG`

The practical difference is simple:

- Raspberry Pi GPIO is `3.3V`
- Volatco I/O is `1.8V`
- Volatco therefore needs the `TXS0108`

## Parts

- `TB6612FNG` dual motor driver
- recycled Teddy Ruxpin face motors
- breadboard USB supply with `5V` and `3.3V` rails
- `TXS0108` for the Volatco setup
- separate `1.8V` source for the Volatco side of the `TXS0108`
- jumper wires
- multimeter

## Power plan

Use these rails for first bench tests:

- `5V -> TB6612 VM`
- `3.3V -> TB6612 VCC`
- `3.3V -> TXS0108 VCCB`
- `1.8V -> TXS0108 VCCA`
- all grounds common

Treat the recycled motors as `5V to 6V` motors until measured otherwise.

Do not do these things:

- do not connect Volatco `1.8V` directly to `TB6612 VCC`
- do not power the motors from Pi `3V3`
- do not apply `11.1V` directly to the Teddy motors
- do not assume the USB breadboard module can handle motor stall current

## TB6612FNG pin roles

Control inputs:

- `AIN1`
- `AIN2`
- `BIN1`
- `BIN2`
- `STBY`
- `PWMA`
- `PWMB`

Motor outputs:

- `A01`
- `A02`
- `B01`
- `B02`

Power pins:

- `VCC` logic supply
- `VM` motor supply
- `GND` common ground

## Motor mapping

Use this mapping first:

- `A01/A02 -> eyes motor`
- `B01/B02 -> mouth motor`

If a motor moves in the wrong direction, swap that motor's two wires.

## Raspberry Pi setup

### Pi control wiring

- `BCM17 -> AIN1`
- `BCM27 -> AIN2`
- `BCM22 -> BIN1`
- `BCM23 -> BIN2`
- `BCM24 -> STBY`

For first bring-up:

- tie `PWMA` high to `3.3V`
- tie `PWMB` high to `3.3V`

### Pi power wiring

- breadboard `5V -> TB6612 VM`
- Pi `3V3 -> TB6612 VCC`
- Pi `GND -> TB6612 GND`
- breadboard supply ground -> same common ground

### Pi step-by-step

1. Place the `TB6612FNG` on the breadboard.
2. Connect `VM`, `VCC`, and `GND`.
3. Tie `PWMA` and `PWMB` high.
4. Leave `STBY` low for the moment.
5. Connect the five Pi control lines.
6. Connect the eyes motor to `A01/A02`.
7. Connect the mouth motor to `B01/B02`.
8. Run the `gforth` harness in dry-run mode first.
9. Switch to real GPIO only after the pin map is confirmed.
10. Test with short pulses only.

## Volatco setup

### Volatco control path

Use this path:

`Volatco d-lines -> TXS0108 -> TB6612FNG`

Do not skip the `TXS0108`.

### TXS0108 wiring

- `VCCA -> 1.8V`
- `VCCB -> 3.3V`
- `OE -> enabled`
- `GND -> common ground`

Suggested channel use:

- `A1/B1 -> eyes in1`
- `A2/B2 -> eyes in2`
- `A3/B3 -> mouth in1`
- `A4/B4 -> mouth in2`
- `A5/B5 -> stby`

For first bring-up:

- tie `PWMA` high to `3.3V`
- tie `PWMB` high to `3.3V`

### Volatco power wiring

- breadboard `5V -> TB6612 VM`
- breadboard `3.3V -> TB6612 VCC`
- breadboard `3.3V -> TXS0108 VCCB`
- separate `1.8V -> TXS0108 VCCA`
- Volatco ground -> common ground
- `TXS0108` ground -> common ground
- `TB6612` ground -> common ground

### Volatco step-by-step

1. Place the `TB6612FNG` and `TXS0108` on the breadboard.
2. Connect the `TB6612` power rails.
3. Connect the `TXS0108` power rails.
4. Confirm all grounds are common.
5. Tie `PWMA` and `PWMB` high.
6. Leave `STBY` low for the moment.
7. Connect one shifted path first.
8. Start with `d00` if that is the easiest known line to probe.
9. Confirm:
   - the Volatco-side signal
   - the shifted `TXS0108` output
   - the matching `TB6612` input
10. Only after one path works, connect the remaining control lines.
11. Connect the eyes motor to `A01/A02`.
12. Connect the mouth motor to `B01/B02`.
13. Test with short pulses only.

## Next-session checklist

Use this exact order on the bench:

1. Verify the breadboard `5V` and `3.3V` rails with a meter.
2. Confirm the `1.8V` source for the Volatco setup.
3. Wire power first, with motors still disconnected.
4. Check for unexpected heating.
5. Wire control lines.
6. Keep `STBY` low until wiring is checked.
7. Connect the motors.
8. Run short test pulses.
9. Watch for supply sag, resets, or chatter.
10. Stop immediately if the driver gets hot or the supply collapses.

## Failure checks

Stop and re-check wiring if:

- the motor chatters but does not turn
- the USB supply resets or the rail droops
- the `TB6612FNG` gets hot quickly
- the Pi resets
- `STBY` low does not disable motion
- the Volatco signal does not appear correctly after the `TXS0108`

If the USB breadboard supply is unstable under motor load, keep it for
logic only and move `VM` to a stronger external `5V` supply.

## Local references

- [tests/henry-face-motors-gforth.fs](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/tests/henry-face-motors-gforth.fs)
- [task/README.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/task/README.md)
- [Volatco Documentation](https://volatco.github.io/)
- [DataBook 002 - The GA144A12](https://volatco.github.io/pages/db002/)
