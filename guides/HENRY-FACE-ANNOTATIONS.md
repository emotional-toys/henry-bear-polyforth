# Henry Face Annotations

This file is the explanatory companion to:

- [henry-face-motors-4800.src](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors-4800.src)
- [guides/HENRY-FACE-LIST-BLOCKS.txt](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/HENRY-FACE-LIST-BLOCKS.txt)

It exists because the block source itself is constrained by the `16 x 64`
format and should remain compact. Long transform commentary belongs here,
not inside the source blocks.

## Use

Read this file by block number while reviewing the block source or the
`LIST` capture.

The intent is similar to the explanatory prose surrounding the `IDEAL`
example, but separated cleanly from the source text.

## 4800

- Load block for the Henry face motor package.
- Uses `THRU` to load blocks `4801` through `4808`.
- This mirrors the compact load-block style seen in the `IDEAL` example.

## 4801

- Establishes numeric mode with `DECIMAL`.
- Defines timing variables for blink and mouth motion.
- Defines `FACE-BUS`, the software shadow of the output register.
- Introduces the basic write path:
  - `FACE-BUS!`
  - `FACE-BUS@`
- This block comes from the transformation step that separates shared
  runtime state from behavior.

## 4802

- Defines the low-level bit helpers:
  - `BIT-MASK`
  - `BIT-SET`
  - `BIT-CLEAR`
  - `FACE-SET-BIT`
  - `FACE-CLEAR-BIT`
- Holds the unresolved wiring constants:
  - `EYES-IN1-BIT`
  - `EYES-IN2-BIT`
  - `MOUTH-IN1-BIT`
  - `MOUTH-IN2-BIT`
  - `STBY-BIT`
  - PWM-related bits
- This is a deliberate transformation choice: uncertain hardware mapping is
  localized here rather than spread through behavior words.

## 4803

- Defines standby and PWM support words.
- Separates enable behavior from direction behavior.
- This block corresponds to the “driver policy” layer in the transform:
  the program decides whether PWM is software-controlled or effectively
  hard-wired active.

## 4804

- Names the individual control lines at the pin or signal level.
- Keeps “what signal is toggled” distinct from “what the face does.”
- This factoring makes later hardware correction safer:
  if a bit assignment changes, semantic words do not have to be rewritten.

## 4805

- Defines drive semantics:
  - coast
  - idle
  - directional drive for eyes
  - directional drive for mouth
- This is the first block where the code starts to describe motor behavior
  rather than signal-level operations.

## 4806

- Defines user-facing motion words:
  - `EYES-OPEN`
  - `EYES-CLOSE`
  - `MOUTH-OPEN`
  - `MOUTH-CLOSE`
- These words combine:
  - a drive word
  - a timing wait
  - return to idle
- This is the closest equivalent to named hardware methods in the original
  C#-style design, but factored into FORTH words.

## 4807

- Defines repeated or composite actions:
  - `BLINK-ONCE`
  - `MOUTH-ONCE`
  - `FACE-RESET`
- This block captures the translation from imperative control flow into
  reusable FORTH gesture words.
- `FACE-RESET` remains partly assumption-dependent because the true physical
  neutral positions still depend on bench verification.

## 4808

- Defines operator or bench-facing helper words:
  - `BLINK`
  - `MOUTH`
  - `FACE-OFF`
  - `TEST-BIT`
- `TEST-BIT` is especially important during bring-up because it allows a
  single output path to be exercised without invoking a larger gesture.

## Transform notes

### Why the source is split this way

The block structure follows the same general discipline seen in the local
`IDEAL` reference:

- load block first
- foundational data next
- semantic behavior later
- top-level or test behavior last

That is more valuable for training than a direct line-by-line port from C#.

### Why the comments are sparse in the source

The source blocks are not the right place for long explanations because:

- block lines are limited in width
- block count is limited by readability
- the old-school source style expects density

So the transform intentionally keeps the source terse and moves explanation
into this companion file.

### What remains unresolved

The following still depend on hardware confirmation:

- exact bit mapping to TB6612 inputs
- whether PWM lines are truly software-driven
- whether the current assumed eye and mouth directions match the actual
  mechanism

Those uncertainties are isolated in the source so the rest of the program
structure can stay stable.

## Relationship to other training files

- [guides/POLYFORTH-BLOCK-STANDARD.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/POLYFORTH-BLOCK-STANDARD.md)
  defines the formatting and structural rules.
- [guides/CSHARP-TO-POLYFORTH.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/CSHARP-TO-POLYFORTH.md)
  defines the translation method.
- [guides/HENRY-FACE-LIST-BLOCKS.txt](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/HENRY-FACE-LIST-BLOCKS.txt)
  gives the final block listing.

Together, these files provide:

1. transform method
2. source standard
3. canonical block source
4. exact list capture
5. explanatory annotations
