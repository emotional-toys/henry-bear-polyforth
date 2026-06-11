# Next Session Prompt

We are in
`/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth`.

Please continue from commit `4b6c4ea`.

## Context

- the polyFORTH source is in `henry-face-motors.fth`
- the block-style source is in `henry-face-motors-4800.src`
- the Raspberry Pi `gforth` bench harness is in
  `tests/henry-face-motors-gforth.fs`
- the bench wiring procedure is in `guides/HENRY-FACE-WIRING.md`
- the motor driver is `TB6612FNG`
- Volatco GPIO is `1.8V`, so the Volatco path uses `TXS0108`
- the recycled Teddy Ruxpin motors are being treated conservatively as
  `5V` to `6V` motors for bring-up

## What I want next

1. Review the current forth words and the `gforth` harness for any mismatch
   in semantics or naming.
2. Check whether `henry-face-motors-4800.src` is still fully
   block-conformant, including exact line count for the `4800`-series
   blocks.
3. If needed, regenerate or update the list-style documentation artifacts so
   they match the current source.
4. Keep Brodie-style comment usage correct.
5. Do not broaden the scope without explaining why.

## Open details

* `henry-face-motors-4800.src` still deserves a strict block-format audit
against the current refactor, because the `.fth` file changed after the
earlier list-generation work.
* Ensure that the difference in code between a single and double space is included in the method.
