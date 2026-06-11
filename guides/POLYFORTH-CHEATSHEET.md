# polyFORTH Cheat Sheet

This is a compact working reference for polyFORTH in the Henry bear Volatco environment.

Use it together with:

- [guides/STARTING-FORTH-AGENT.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/STARTING-FORTH-AGENT.md)
- [henry-face-motors.fth](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth)
- [task/EXPERIMENTAL-SETUP.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/task/EXPERIMENTAL-SETUP.md)

## Conventions for this repo

- write words in uppercase
- use Brodie-style parenthetical comments: `( ... )`
- keep words small
- include stack comments on nontrivial words
- isolate hardware mapping in constants
- prefer interactive test words over opaque routines

## Basic FORTH model

- numbers push values onto the stack
- words consume and produce stack values
- FORTH uses postfix notation

Example:

```forth
3 4 +
```

Result:

- stack contains `7`

## Defining words

```forth
: DOUBLE ( N -- 2N )
  2 *
;
```

Use:

```forth
5 DOUBLE
```

Comment form:

```forth
( THIS IS A SOURCE COMMENT )
```

## Core stack words

```text
DUP    ( A -- A A )
DROP   ( A -- )
SWAP   ( A B -- B A )
OVER   ( A B -- A B A )
ROT    ( A B C -- B C A )
```

## Core arithmetic words

```text
+      ( A B -- SUM )
-      ( A B -- DIFF )
*      ( A B -- PROD )
/      ( A B -- QUOT )
MOD    ( A B -- REM )
/MOD   ( A B -- REM QUOT )
1+     ( N -- N+1 )
1-     ( N -- N-1 )
2*     ( N -- 2N )
2/     ( N -- N/2 )
```

## Comparison and logic words

```text
=      ( A B -- FLAG )
0=     ( A -- FLAG )
<      ( A B -- FLAG )
>      ( A B -- FLAG )
0<     ( A -- FLAG )
AND    ( A B -- A&B )
OR     ( A B -- A|B )
INVERT ( A -- ~A )
```

In classic FORTH, a true flag is usually nonzero and often `-1`.

## Memory words

### Variables

```forth
0 VARIABLE FACE-BUS
```

Fetch and store:

```forth
FACE-BUS @
5 FACE-BUS !
```

### Constants

```forth
4 CONSTANT STBY-BIT
```

## Printing and terminal output

```text
.      ( N -- )      print number
CR     ( -- )        newline
."     ( -- )        print string until closing quote
EMIT   ( C -- )      print character
```

Example:

```forth
: HELLO ( -- )
  CR ." HELLO"
;
```

## Control flow

### IF / THEN

```forth
: ABS ( N -- U )
  DUP 0< IF NEGATE THEN
;
```

### IF / ELSE / THEN

```forth
: SIGN-NAME ( N -- )
  0< IF
    ." NEG"
  ELSE
    ." POS"
  THEN
;
```

### Counted loop

```forth
: N-DOTS ( N -- )
  0 ?DO
    [CHAR] . EMIT
  LOOP
;
```

### Indefinite loop

```forth
: WAIT-READY ( -- )
  BEGIN
    READY?
  UNTIL
;
```

## Stack-discipline rule

Before finalizing a word:

1. state the stack effect
2. simulate the stack after each token
3. test the word interactively

Bad:

```forth
: X 1 2 SWAP DROP + ;
```

Good:

```forth
: ADD-ONE ( N -- N' )
  1 +
;
```

## Interactive workflow on Volatco

The local notes use this sequence to reach the bus-driving node:

```forth
BRIDGE LOAD
TALK
SPAN
0 10007 HOOK
```

Meaning in practice:

- `BRIDGE LOAD` loads the bridge support
- `TALK` connects to the host chip
- `SPAN` appends the target chip
- `0 10007 HOOK` attaches to the node that drives the data bus

After that, `DATA R!` is used to write the parallel data bus.

## Bus-control pattern used in this repo

From [henry-face-motors.fth](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth):

```forth
0 VARIABLE FACE-BUS

: FACE-BUS! ( U -- )
  DUP FACE-BUS !
  DATA R!
;
```

Reason:

- keep a software shadow of output state
- avoid losing unrelated bits when toggling one signal

## Bit operations

Used to set and clear individual output bits:

```forth
: BIT-MASK ( N -- U )
  1 SWAP LSHIFT
;

: BIT-SET ( U N -- U' )
  BIT-MASK OR
;

: BIT-CLEAR ( U N -- U' )
  BIT-MASK INVERT AND
;
```

Typical wrapper:

```forth
: FACE-SET-BIT ( N -- )
  FACE-BUS@ SWAP BIT-SET FACE-BUS!
;
```

## Henry face words

Current high-level words:

- `BLINK`
- `MOUTH`
- `SMILE`
- `FACE-RESET`
- `FACE-OFF`
- `FACE-STATUS`
- `TEST-BIT`

### Safe probing sequence

```forth
FACE-OFF
FACE-STATUS
0 TEST-BIT
FACE-OFF
1 TEST-BIT
FACE-OFF
```

Use this while measuring which `DATA` bit reaches which TB6612FNG input.

## TB6612FNG logic reference

For one motor channel:

```text
IN1 IN2
0   0   COAST
1   0   DRIVE ONE DIRECTION
0   1   DRIVE OPPOSITE DIRECTION
1   1   BRAKE
```

The repo currently models:

- one channel for eyes
- one channel for mouth
- optional software control of `STBY` and PWM

## Timing variables

In [henry-face-motors.fth](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth):

```forth
250 VARIABLE BLINK-RATE-MS
250 VARIABLE BLINK-DELAY-MS
140 VARIABLE MOUTH-RATE-MS
120 VARIABLE MOUTH-DELAY-MS
```

Change interactively:

```forth
200 BLINK-RATE-MS !
100 MOUTH-DELAY-MS !
```

## Useful development patterns

### Small probe word

```forth
: TEST-BIT ( N -- )
  FACE-OFF
  TB6612-STBY-ON
  FACE-SET-BIT
;
```

### Semantic wrapper

```forth
: EYES-OPEN ( -- )
  EYES-OPEN-DRIVE
  BLINK-RATE-MS @ MS-WAIT
  FACE-DRIVE-IDLE
;
```

### Counted action

```forth
: BLINK ( N -- )
  CLAMP-NONNEGATIVE 0 ?DO
    BLINK-ONCE
  LOOP
;
```

## What not to do

### Do not scatter raw hardware writes

Avoid:

```forth
: BAD-BLINK
  1 DATA R!
  0 DATA R!
;
```

Prefer:

```forth
: GOOD-BLINK
  EYES-CLOSE
  EYES-OPEN
;
```

### Do not assume unknown mappings

If a pin map is unverified, leave it as a constant placeholder and document it.

### Do not hide stack effects

If a word is not obvious, annotate it.

## Quick reference

### Definition

```forth
: NAME ( IN -- OUT )
  ...
;
```

### Variable

```forth
0 VARIABLE NAME
```

### Constant

```forth
5 CONSTANT NAME
```

### Fetch and store

```forth
NAME @
10 NAME !
```

### Conditional

```forth
COND IF ... THEN
COND IF ... ELSE ... THEN
```

### Loop

```forth
0 ?DO ... LOOP
BEGIN ... UNTIL
```

## Bench checklist

Before testing hardware:

1. attach to the correct node
2. clear outputs with `FACE-OFF`
3. probe one bit at a time
4. record exact observations
5. update only the confirmed constants

## Related files

- [guides/STARTING-FORTH-AGENT.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/STARTING-FORTH-AGENT.md)
- [henry-face-motors.fth](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/henry-face-motors.fth)
- [task/EXPERIMENTAL-SETUP.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/task/EXPERIMENTAL-SETUP.md)
- [task/README.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/task/README.md)
