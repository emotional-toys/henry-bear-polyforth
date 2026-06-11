# Starting FORTH for Agents

This file is an agent-readable training guide derived from [`guides/Starting-FORTH.pdf`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/Starting-FORTH.pdf).

It is not a verbatim transcription of the book. It is a compact study guide for work in this repository, especially polyFORTH on Volatco / GA144 hardware.

## Purpose

Use this guide when an agent needs to:

- read or write simple FORTH or polyFORTH code
- reason about stack effects
- define new words in the style expected by this repo
- understand the difference between standard FORTH ideas and repo-specific hardware control
- plan experiments without pretending unknown hardware details are known

## Repo-specific assumptions

- Word names should be uppercase in this repo.
- Comments in FORTH code should use Brodie-style parenthetical comments: `( ... )`.
- Hardware-facing code should isolate unknown wiring in a small set of constants or words.
- For Volatco work, prefer simple interactive words that can be tested from the terminal.
- Do not infer exact hardware mappings unless the repository or measurement notes confirm them.

## Source map

The PDF covers these broad topics:

1. Fundamental FORTH
2. Arithmetic and stack use
3. Editor and loading workflow
4. Conditionals
5. Arithmetic technique and fixed-point thinking
6. Loops
7. Number representations
8. Variables, constants, arrays
9. Dictionary internals and execution model
10. I/O and string handling
11. Defining words and compiler control
12. Worked examples

For this repository, chapters 1, 2, 4, 6, 8, 9, and 11 are the most relevant.

## Mental model

FORTH is built around:

- a data stack
- small words with explicit stack effects
- interactive execution at the terminal
- incremental extension of the language

The practical rule is:

- write tiny words
- test them immediately
- factor repeated logic into named words
- keep stack discipline obvious

## Core syntax model

### Numbers

Typing a number pushes it on the stack.

Example:

```forth
3 4
```

Result:

- stack contains `3 4`

### Words

Typing a word executes it immediately in interpret mode.

Example:

```forth
3 4 +
```

Result:

- stack contains `7`

### Colon definitions

New words are defined with `:` and ended with `;`.

Example:

```forth
: DOUBLE 2 * ;
```

Usage:

```forth
5 DOUBLE
```

Result:

- stack contains `10`

## Stack-first reasoning

Every word should be understood by its stack effect.

Examples:

```text
+        ( A B -- SUM )
DUP      ( A -- A A )
DROP     ( A -- )
SWAP     ( A B -- B A )
OVER     ( A B -- A B A )
!        ( N ADDR -- )
@        ( ADDR -- N )
```

When writing code:

1. state the intended stack effect
2. walk the stack after each word
3. avoid hidden stack coupling across long definitions

## Minimal word set to know first

### Stack manipulation

- `DUP`
- `DROP`
- `SWAP`
- `OVER`
- `ROT`

### Arithmetic

- `+`
- `-`
- `*`
- `/`
- `MOD`
- `/MOD`

### Comparisons and logic

- `=`
- `0=`
- `<`
- `>`
- `AND`
- `OR`
- `INVERT`

### Memory

- `VARIABLE`
- `CONSTANT`
- `@`
- `!`

### Control flow

- `IF ... THEN`
- `IF ... ELSE ... THEN`
- `DO ... LOOP`
- `BEGIN ... UNTIL`
- `BEGIN ... WHILE ... REPEAT`

### Definition and inspection

- `:`
- `;`
- `."`
- `.`
- `CR`

## Style rules for this repo

### 1. Use uppercase names

Preferred:

```forth
: FACE-OFF 0 FACE-BUS! ;
```

Not preferred here:

```forth
: face-off 0 face-bus! ;
```

### 2. Put hardware uncertainty in one place

Preferred:

```forth
0 CONSTANT EYES-IN1-BIT
1 CONSTANT EYES-IN2-BIT
```

Then use those constants elsewhere.

### 3. Keep hardware actions factored

Preferred layering:

1. raw bit writes
2. pin-level words
3. motor-direction words
4. user-facing words like `BLINK`

### 4. Use explicit stack comments on nontrivial words

Example:

```forth
: FACE-SET-BIT ( N -- )
  FACE-BUS@ SWAP BIT-SET FACE-BUS!
;
```

For ordinary source comments, prefer parenthetical comments as taught in *Starting FORTH*:

```forth
( BUS SHADOW SO WE CAN SET AND CLEAR INDIVIDUAL CONTROL BITS. )
```

### 5. Prefer terminal-testable words

Good examples:

- `FACE-OFF`
- `FACE-STATUS`
- `TEST-BIT`

These are easier to validate than large opaque sequences.

## Control-flow patterns

### Conditional action

```forth
: ABS ( N -- U )
  DUP 0< IF NEGATE THEN
;
```

### Fixed-count loop

```forth
: BLINK ( N -- )
  0 ?DO
    BLINK-ONCE
  LOOP
;
```

### Repeated polling

```forth
: WAIT-UNTIL-READY ( -- )
  BEGIN
    READY?
  UNTIL
;
```

For hardware work, always include an exit condition you trust.

## Variables and state

Use `VARIABLE` for mutable runtime state.

Example:

```forth
0 VARIABLE FACE-BUS
```

Store with `!` and fetch with `@`:

```forth
5 FACE-BUS !
FACE-BUS @ .
```

Use `CONSTANT` for fixed identifiers such as bit numbers:

```forth
4 CONSTANT STBY-BIT
```

## Hardware-control pattern for this repo

The current repo uses this structure in [`henry-face-motors.fth`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth):

1. maintain a software shadow of the output bus
2. update individual bits in the shadow
3. write the whole value to `DATA R!`
4. build semantic words on top

Pattern:

```forth
: FACE-BUS! ( U -- )
  DUP FACE-BUS !
  DATA R!
;

: FACE-SET-BIT ( N -- )
  FACE-BUS@ SWAP BIT-SET FACE-BUS!
;
```

This is preferable to scattering raw `DATA R!` writes throughout the code.

## polyFORTH and standard FORTH

The source material includes both general FORTH concepts and polyFORTH-specific capabilities.

For this repo:

- prefer standard stack and control-flow idioms first
- isolate polyFORTH- or platform-specific words behind small wrappers
- document assumptions near the hardware-facing code

Examples of platform-specific ideas in this repo:

- `DATA R!`
- `HOOK`
- `BRIDGE LOAD`
- `TALK`
- `SPAN`

These are not generic FORTH concepts. They belong to the Volatco / toolchain environment.

## Agent behavior rules

When generating FORTH in this repo, an agent should:

1. write uppercase word names
2. add stack comments for nontrivial definitions
3. prefer short definitions over long monoliths
4. avoid assuming hardware polarity or pin mapping without evidence
5. provide bench-test helper words when hardware is uncertain
6. keep timing values adjustable through variables

## Common mistakes

### Mistake: infix thinking

Wrong mental model:

```text
3 + 4
```

FORTH uses postfix:

```forth
3 4 +
```

### Mistake: losing track of stack contents

Bad practice:

- writing a long definition without stack comments
- assuming intermediate values still exist

Fix:

- annotate stack effects
- factor into smaller words

### Mistake: mixing raw bus writes with semantic behavior

Bad pattern:

```forth
: BLINK-ONCE
  1 DATA R!
  0 DATA R!
;
```

Better pattern:

```forth
: EYES-CLOSE ( -- ) ... ;
: EYES-OPEN  ( -- ) ... ;
: BLINK-ONCE ( -- )
  EYES-CLOSE
  EYES-OPEN
;
```

## Recommended learning order for agents

1. Learn postfix evaluation and stack effects.
2. Learn colon definitions and factoring.
3. Learn variables and constants.
4. Learn control flow.
5. Learn repo-specific hardware words.
6. Only then generate motor-control or bus-control code.

## Practical exercises for this repo

An agent should be able to produce and explain these safely:

### Exercise 1: constant and variable

```forth
0 VARIABLE COUNTER
5 CONSTANT LIMIT
```

### Exercise 2: increment a variable

```forth
: INC-COUNTER ( -- )
  COUNTER @ 1+ COUNTER !
;
```

### Exercise 3: simple loop

```forth
: N-STARS ( N -- )
  0 ?DO
    [CHAR] * EMIT
  LOOP
;
```

### Exercise 4: hardware wrapper

```forth
: MOTOR-IDLE ( -- )
  FACE-DRIVE-IDLE
;
```

## Relevance to Henry face work

For Henry face control, the important FORTH lessons are:

- define motor behaviors as words
- hide raw bus operations behind stable names
- keep timings adjustable
- create small probe words for uncertain hardware
- validate incrementally at the terminal

This is more important than reproducing textbook examples exactly.

## Suggested follow-up docs

If more training material is needed, the next useful derived files would be:

- `guides/POLYFORTH-CHEATSHEET.md`
- `guides/VOLATCO-INTERACTIVE-WORKFLOW.md`
- `guides/STACK-EFFECTS-REFERENCE.md`

## Provenance

Derived from:

- [`guides/Starting-FORTH.pdf`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/Starting-FORTH.pdf)

Related local files:

- [`henry-face-motors.fth`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth)
- [`task/EXPERIMENTAL-SETUP.md`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/task/EXPERIMENTAL-SETUP.md)
- [`task/README.md`](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/task/README.md)
