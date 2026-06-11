# C# to polyFORTH Translation Guide

This guide explains how to turn structured C# logic into block-oriented
polyFORTH suitable for this repository.

Use it together with:

- [guides/POLYFORTH-BLOCK-STANDARD.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/POLYFORTH-BLOCK-STANDARD.md)
- [guides/POLYFORTH-CHEATSHEET.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/POLYFORTH-CHEATSHEET.md)
- [guides/STARTING-FORTH-AGENT.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/STARTING-FORTH-AGENT.md)

## Purpose

Agents should not translate C# to FORTH mechanically line by line.

They should translate:

- state
- decisions
- loops
- hardware actions
- observable behavior

into small FORTH words organized by block.

## Core rule

Translate intent, not syntax.

C# example:

```csharp
if (value < 0)
{
    value = 0;
}
```

FORTH version:

```forth
: CLAMP0 ( N -- N' ) DUP 0< IF DROP 0 THEN ;
```

The FORTH word is smaller, clearer, and reusable.

## Translation strategy

### 1. Identify state

In C#, mutable state is often:

- fields
- properties
- local variables that persist through a routine

In FORTH, that usually becomes:

- `VARIABLE`
- `CONSTANT`
- stack values for transient data

Example:

```csharp
private static int mouthDelayMs = 120;
```

becomes:

```forth
120 VARIABLE MOUTH-DELAY-MS
```

## 2. Identify hardware abstractions

C# often hides hardware actions inside methods:

```csharp
OpenEyes();
CloseMouth();
SafeReset();
```

Those should become semantic FORTH words first:

```forth
: EYES-OPEN ( -- ) ... ;
: MOUTH-CLOSE ( -- ) ... ;
: FACE-RESET ( -- ) ... ;
```

Do not translate directly into raw bus writes at every call site.

## 3. Pull out reusable logic

C#:

```csharp
for (var i = 0; i < cycles; i++)
{
    BlinkOnce();
}
```

FORTH:

```forth
: BLINK ( N -- ) CLAMP0 0 ?DO BLINK-ONCE LOOP ;
```

## 4. Replace nested imperative flow with factored words

C# often nests operations in one method.

FORTH should split them:

```text
low-level bit operations
direction words
single motion words
repeated behavior words
test words
```

This keeps stack flow manageable.

## 5. Distinguish compile-time names from runtime state

C# constants and enums often become:

- `CONSTANT`
- named words

Runtime counters and latches become:

- `VARIABLE`

Example:

```csharp
const int StbyBit = 4;
```

becomes:

```forth
4 CONSTANT STBY-BIT
```

## Pattern mapping

### C# field

```csharp
private static int blinkDelayMs = 250;
```

FORTH:

```forth
250 VARIABLE BLINK-DELAY-MS
```

### C# helper method

```csharp
private static void FaceOff()
{
    WriteBus(0);
}
```

FORTH:

```forth
: FACE-OFF ( -- ) 0 FACE-BUS! ;
```

### C# conditional

```csharp
if (useSoftwarePwm)
{
    SetBit(eyesPwmBit);
}
```

FORTH:

```forth
: EYES-PWM-ON ( -- )
  USE-SOFTWARE-PWM 0< IF EXIT THEN
  EYES-PWM-BIT FACE-SET-BIT
;
```

### C# counted loop

```csharp
for (var i = 0; i < cycles; i++)
{
    MouthOnce();
}
```

FORTH:

```forth
: MOUTH ( N -- ) CLAMP0 0 ?DO MOUTH-ONCE LOOP ;
```

## Translation workflow for agents

### Step 1: describe the C# behavior in plain language

Example:

- maintain a bus shadow
- set and clear named bits
- drive eye motor one direction for a pulse
- return to idle
- repeat for blink cycles

### Step 2: choose FORTH layers

Typical Henry layering:

1. variables and constants
2. bit helpers
3. driver helpers
4. semantic motor words
5. user-facing actions

### Step 3: assign blocks

Fit the layers into numbered blocks.

### Step 4: write stack effects before code

Every nontrivial word should start from its stack contract.

### Step 5: compress only after correctness

First make the factoring right.
Then tighten the layout to fit `16 x 64` block format.

## What not to do

### Do not preserve C# structure blindly

Bad:

- one huge FORTH definition that mirrors one huge C# method

Better:

- many small FORTH words named after behavior

### Do not keep temporary named variables when stack values suffice

C# requires more local names than FORTH does.

### Do not erase semantics

Bad:

```forth
: X1 ... ;
: X2 ... ;
```

Better:

```forth
: EYES-OPEN-DRIVE ... ;
: MOUTH-CLOSE-DRIVE ... ;
```

### Do not mix uncertain hardware mapping into action words

Put uncertain mappings in constants and low-level words.

## Henry example

C# intent:

- blink a number of cycles
- each cycle closes the eyes, waits, opens the eyes, waits

FORTH translation:

```forth
: BLINK-ONCE ( -- )
  EYES-CLOSE
  BLINK-DELAY-MS @ MS-WAIT
  EYES-OPEN
  BLINK-DELAY-MS @ MS-WAIT
;

: BLINK ( N -- ) CLAMP0 0 ?DO BLINK-ONCE LOOP ;
```

That is a proper semantic translation, not a token-for-token port.

## Training rules for agents

When translating C# to polyFORTH here, the agent should:

1. identify semantics first
2. derive stack effects
3. create named low-level words before behavior words
4. keep hardware uncertainty localized
5. produce block-oriented source that follows the local standard
6. verify 64-column lines in block source

## Output expectation

For serious work, the agent should ideally produce:

1. a readable working `.fth` file
2. a canonical `.src` block file
3. short notes on unresolved hardware assumptions

That matches the repo’s actual needs better than a single mixed-format
artifact.
