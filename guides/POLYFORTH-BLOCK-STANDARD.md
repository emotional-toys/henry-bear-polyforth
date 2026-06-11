# polyFORTH Block Standard

This guide defines the source format to use for professional block-oriented polyFORTH in this repository.

It is based on the local `IDEAL` example:

- [guides/IDEAL Simplified x86 v0.pdf](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/IDEAL%20Simplified%20x86%20v0.pdf)
- [guides/Ideal simplified README.md](/home/cartheur/ame/aiventure/aiventure-github/emotional-toys/henry-bear-polyforth/guides/Ideal%20simplified%20README.md)

## Purpose

Use this format when writing source intended for:

- polyFORTH block files
- agent training on professional FORTH style
- translation of higher-level logic into Volatco-targeted FORTH

## Hard format rules

### 1. Source is block-oriented

- One block is `1024` characters.
- One block is displayed as `16` lines.
- One line is `64` characters maximum.

That means a source file representing blocks must preserve:

- exact block numbering
- exact 16-line grouping
- no line longer than 64 characters

## 2. Each block has a number and a role

Follow the `IDEAL` pattern:

- one load block
- subsequent numbered source blocks
- one clear purpose per block

Example shape:

```text
4800  load block
4801  basic words
4802  state and constants
4803  low-level I/O
4804  semantic actions
4805  test words
```

## 3. Line 0 is the block title

Use line `0` as a short title comment.

Examples from the `IDEAL` style:

```forth
( IDEAL Simplified)
( Static, abstract things)
( Benefit of Experience)
( Stream of Intelligence)
```

For Henry-style work:

```forth
( Henry Face Basics)
( Henry Face Wiring)
( Henry Face Actions)
```

Keep the title short.

Comment syntax detail:

- `(` must be separated as a word
- `)` is only the delimiter and should not be preceded by a space

Preferred:

```forth
( Stream of Intelligence)
( Face Bus State)
( N --)
```

Avoid:

```forth
( Stream of Intelligence )
( Face Bus State )
( N -- )
```

## 4. Load blocks stay compact

Prefer a load block that names the block range clearly.

If supported by the environment, prefer `THRU`.

Example style:

```forth
( Henry Face Load Block)
4801 4806 THRU
```

If `THRU` is not desired in a given environment, a sequence of `LOAD`
words is acceptable, but `THRU` is the cleaner block idiom.

## 5. Use uppercase words

In this repo, source words should be uppercase.

Preferred:

```forth
: FACE-OFF ( --) 0 FACE-BUS! ;
```

## 6. Keep comments sparse and structural

The `IDEAL` example does not narrate every line.

Use comments for:

- block titles
- rare clarifications
- stack effects

Do not write long prose comments inside source blocks unless the block
exists specifically for explanation.

Preferred:

```forth
( Henry Face Actions)
: BLINK ( N --) CLAMP0 0 ?DO BLINK-ONCE LOOP ;
```

Avoid large narrative headers in the source itself.

## 7. Stack comments belong on definitions that need them

This is standard Brodie-style practice and matches the `IDEAL` sample.

Examples:

```forth
: PREDICT ( EXP -- RES) ...
: OTHEREXP ( CUR -- NEW) ...
: FACE-SET-BIT ( N --) ...
```

Rules:

- include stack effects on nontrivial words
- keep them short
- use them to clarify ordering, not to restate obvious code

## 8. Factor by responsibility

A professional block program is organized in layers.

For hardware control, the normal order is:

1. load block
2. constants and variables
3. low-level bit or port operations
4. semantic drive words
5. user-facing actions
6. test or bench helpers

That is close to the structure seen in the `IDEAL` example:

- abstract vocabulary first
- interactions next
- state next
- behavior next
- step logic last

## 9. Prefer short definitions

The `IDEAL` sample keeps most definitions compact.

Do the same:

- one line when it stays readable
- split over several lines when needed
- indent continuation lines

Example:

```forth
: MOUTH-CLOSE ( --)
  MOUTH-CLOSE-DRIVE
  MOUTH-RATE-MS @ MS-WAIT
  FACE-DRIVE-IDLE
;
```

## 10. Capture the `LIST` view as a document

For training and review, do not stop at the `.src` file.

Also generate a plain-text `LIST` document that shows the block source
exactly as a block system presents it.

Recommended companion artifacts:

- block source: `name-4800.src`
- list capture: `NAME-LIST-BLOCKS.txt`

Required structure of the list capture:

1. optional block index
2. one `480x LIST` section per block
3. lines numbered `0..15`
4. exact visible block text

Example:

```text
4800 LIST
0 ( Henry Face Load Block)
1 4801 4808 THRU
2
...
15
```

This step is part of the standard transformation, not an optional extra.
## 11. Keep block text dense, but not cryptic

Professional FORTH is compact. That does not mean careless.

Target:

- minimal wasted lines
- meaningful grouping
- readable names
- predictable factoring

## Recommended structure for Henry-style block programs

### Block 0: load block

Contains:

- block title
- `THRU` or equivalent loads

### Block 1: basics

Contains:

- `DECIMAL`
- timing variables
- shared state variables
- tiny general helpers

### Block 2: constants and wiring

Contains:

- bit constants
- hardware-mode constants
- named state cells

### Block 3: low-level operations

Contains:

- bit masks
- set/clear helpers
- output write words

### Block 4: hardware semantics

Contains:

- `STBY` control
- PWM control
- `IN1/IN2` control
- `COAST` and `BRAKE`

### Block 5: user actions

Contains:

- `EYES-OPEN`
- `EYES-CLOSE`
- `MOUTH-OPEN`
- `MOUTH-CLOSE`
- `BLINK`
- `FACE-RESET`

### Block 6: test helpers

Contains:

- `FACE-OFF`
- `TEST-BIT`
- small diagnostic words

## What agents should learn from this

An agent generating block-oriented source for this repo should:

1. start by choosing the block range
2. assign one responsibility per block
3. keep every line under 64 columns
4. place the block title on line 0
5. prefer `THRU` for load blocks when available
6. use uppercase names
7. omit the space before `)` in parenthetical comments
8. add stack comments only where they carry information
9. avoid prose-heavy comments in the source blocks
10. generate the `LIST` capture as part of the final output

## Standard versus working source

Use two layers if needed:

- a free-form working file such as `.fth`
- a canonical block file such as `.src`
- a plain-text `LIST` capture such as `*-LIST-BLOCKS.txt`

The block file is the publishable source of record for old-school
polyFORTH workflows.

## Review checklist

Before treating a block file as conformant, verify:

- every line is `<= 64` characters
- total lines are a multiple of `16`
- block numbers are correct
- block `0` is a load block
- block titles exist
- parenthetical comments omit the space before `)`
- source is factored by responsibility
- stack comments are present where needed
- comments are sparse and structural
- a corresponding `LIST` document exists

## Reference pattern from IDEAL

Observed local pattern:

- `4821` load block
- `4822` through `4826` source blocks
- line `0` titles
- compact definitions
- stack comments on selected words
- comments concentrated at block level

That is the model to imitate.
