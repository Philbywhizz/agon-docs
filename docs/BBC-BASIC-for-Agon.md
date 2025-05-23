# What is BBC BASIC for Agon?

The original version of BBC BASIC was written by Sophie Wilson at Acorn in 1981 for the BBC Micro range of computers, and was designed to support the UK Computer Literacy Project. R.T.Russell was involved in the specification of BBC Basic, and wrote his own Z80 version that was subsequently ported to a number of Z80 based machines. [I highly recommend reading his account of this on his website for more details](http://www.bbcbasic.co.uk/bbcbasic/history.html).

As an aside, R.T.Russell still supports BBC Basic, and has ported it for a number of modern platforms, including Android, Windows, and SDL, which are [available from his website here](https://www.bbcbasic.co.uk/index.html).

BBC BASIC for Agon is a port of his BBC BASIC for Z80, which is now open source, with a number of modifications to make it run on Agon.

Please note that the information in this file is currently out of date.  The original version of BASIC described here is based on Dean Belfield's Agon port of R.T.Russell's original Z80 BBC BASIC.  This version is broadly the same as Acorn's BBC BASIC version 4 for the BBC Master series of computers, but includes an inbuilt Z80 Assembler instead of 6502.  Since that time, Dean Belfield made a new version based on that work which makes use of the Agon's eZ80 processor, allowing access to all 512KB of RAM, compared to the 64KB of the original Z80 version, and extending the Assembler to support the new eZ80 opcodes.  Richard Russell also revisited his original Z80 version of BASIC and extended it to support most features of Acorn's BBC BASIC V, and Dean Belfield has since ported this to the Agon as well.  The BASIC V for Agon currently only supports Z80 mode.

# Implementation on the Agon Light

BBC BASIC for Z80 runs in Z80 mode, that is within a 64K segment. The interpreter takes around 16K of RAM, leaving around 48K available for user programs and data.

If you are not familiar with the BASIC programming language, or need a refresher on BBC BASIC, please refer to the [official BBC BASIC for Agon documentation here](https://oldpatientsea.github.io/agon-bbc-basic-manual/0.1/index.html).

(Please note that this documentation is also slightly outdated and contains some inaccuracies.  For example it has incorrect information about PLOT commands.  At some point we may merge the Agon BBC BASIC documentation in with the community documentation site.)

To run, load into memory and run as follows from the MOS command prompt:

```
LOAD bbcbasic.bin
RUN
```

It is possible to automatically `CHAIN` (load and run) a BBC BASIC program by passing the filename as a parameter:
```
LOAD bbcbasic.bin
RUN . /path/to/file.bas
```

Note that passing a . as the first parameter of RUN is informing MOS to use the default value there (&40000)

BBC BASIC needs a full 64K segment, so cannot be run from the MOS folder as a star command.


If you are running MOS 2.2.0 you can load and run BBC BASIC from the MOS command prompt using:

```
bbcbasic
```

This assumes that your copy of BASIC is called `bbcbasic.bin` and is either in the root of the SD card, or in the `bin` folder.

Automatically "chaining" a BBC BASIC program can be done by passing the filename as a parameter:

```
bbcbasic /path/to/file.bas
```


# Summary of Agon Light Specific Changes

## Star Commands

The following * commands are supported

### BYE

Syntax: `*BYE`

Exit BASIC and return to MOS.

### EDIT

Syntax: `*EDIT linenum`

Pull a line into the editor for editing.

### FX

Syntax: `*FX osbyte, params`

Execute an OSBYTE command.

The only OSBYTE commands supported at the moment are:

- 19: Wait for vertical blank retrace

And from MOS 1.03 or above

- 11: Set keyboard repeat delay in ms (250, 500, 750 or 1000)
- 12: Set keyboard repeat rate ins ms (between 33 and 500ms)
- 118: Set keyboard LED (Bit 0: Scroll Lock, Bit 1: Caps Lock, Bit 2: Num Lock) - does not currently change status, just the LED

### VERSION

Syntax: `*VERSION`

Display the current version of BBC BASIC

### MOS commands

In addition, any of the MOS commands can be called by prefixing them with a *

See the MOS documentation for more details

## BASIC

The following statements are not currently implemented:

- ENVELOPE
- ADVAL

The following statements differ from the BBC Basic standard:

### REM

REM does not tokenise any statements within comments. This is to bring it inline with string literals for internationalisation.

### LOAD
### SAVE

The following file extensions are supported:

- `.BBC`: LOAD and SAVE in BBC BASIC for Z80 tokenised format
- `.BAS`: LOAD and SAVE in plain text format (also `.TXT` and `.ASC`)

If a file extension is omitted, ".BBC" is assumed.

### MODE

The modes differ from those on the BBC series of microcomputers. The full list can be found [here](vdp/Screen-Modes.md) in the VDP documentation.

### COLOUR

Syntax: `COLOUR c`

Change the the current text output colour

- If c is between 0 and 63, the foreground text colour will be set
- If c is between 128 and 191, the background text colour will be set

Syntax: `COLOUR l,p`

Set the logical colour l to the physical colour p

Syntax: `COLOUR l,r,g,b`

### GCOL

Syntax: `GCOL mode,c`

Set the graphics colour `c`, and the "mode" of graphics paint operations.

Colour values are interpreted as per the COLOUR command, i.e. values below 128 will set the foreground colour, and values above 128 set the background colour.

Versions of the VDP earlier than 1.04 only supported mode 0, with all painting operations just setting on-screen pixels.

VDP 1.04 introduced _partial_ support for mode 4, which inverts the pixel.  Mode 4 would only apply to straight line drawing operations.  The mode would affect all applicable plot operations.

As of Console8 VDP 2.6.0, all 8 of the basic modes are supported for all currently supported plot operations.  Separate plot modes are now tracked for foreground and background colours, and the mode is applied to the graphics operation.

The full array of available modes is as follows:

| Mode | Effect |
| ---- | ------ |
| 0 | Set on-screen pixel to target colour value |
| 1 | OR value with the on-screen pixel |
| 2 | AND value with the on-screen pixel |
| 3 | EOR value with the on-screen pixel |
| 4 | Invert the on-screen pixel |
| 5 | No operation |
| 6 | AND the inverse of the specified colour with the on-screen pixel |
| 7 | OR the inverse of the specified colour with the on-screen pixel

### POINT

Syntax: `POINT(x,y)`

This returns the physical colour index of the colour at pixel position (x, y)

### PLOT

Syntax: `PLOT mode,x,y`

For information on the various PLOT modes, please see the [VDP PLOT command documentation](vdp/PLOT-Commands.md)

### GET$

Syntax: `GET$(x,y)`

Returns the ASCII character at position x,y

### GET

Syntax: `GET(x,y)` (from BBC BASIC 1.05)

As GET$, but returns the ASCII code of the character at position x, y

Syntax: `GET(p)`

Read and return the value of Z80 port p

### SOUND

Syntax: `SOUND channel,volume,pitch,duration`

Play a sound through the Agon Light buzzer and audio output jack

- `Channel`: 0 to 2
- `Volume`: 0 (off) to -15 (full volume)
- `Pitch`: 0 to 255
- `Duration`: -1 to 254 (duration in 20ths of a second, -1 = play forever)

### TIME$

Access the ESP32 RTC data

Example:

```
  10 REM CLOCK
  20 :
  30 CLS
  40 PRINT TAB(2,2); TIME$
  50 GOTO 40
```

NB: This is a virtual string variable; at the moment only getting the time works. Setting is not yet implemented.

### VDU

The VDU commands on the Agon Light will be familiar to those who have coded on Acorn machines. Please read the [[VDP]] documentation for details on what VDU commands are supported.

## Inline Assembler

BBC BASIC for Z80, like its 6502 counterpart, includes an inline assembler. For instructions on usage, please refer to the [original documentation](https://www.bbcbasic.co.uk/bbcbasic/mancpm/bbc3.html#introduction).

In addition to the standard set of Z80 instructions, the following eZ80 instructions have been added

- `MLT`

The assembler will only compile 8-bit Z80 code and there are currently no plans for extending the instruction set much further in this version.

## Integration with MOS

For the most part, the MOS is transparent to BASIC; most of the operations via the MOS and VDP are accessed via normal BBC BASIC statements, with the following exceptions:

### Accessing the MOS SysVars

MOS has a small area of memory for system state variables (sysvars) which lives in an area of RAM outside of the 64K segment in which BBC BASIC runs. To access these, you will need to do an OSBYTE call

Example: Print the least significant byte of the internal clock counter
```
10 L%=&00 : REM The sysvar to fetch
20 A%=&A0 : REM The OSBYTE number
30 PRINT USR(&FFF4)
```

Documentation for the full list of sysvars can be found in the [MOS API documentation](mos/API.md#sysvars).

### Running star commands with variables

The star command parser does not use the same evaluator as BBC BASIC, so whilst commands can be run in BASIC, variable names are treated as literals.

Example: This will not work
```
10 INPUT "Filename";f$
20 INPUT "Load Address";addr%
30 *LOAD f$ addr%
```

To do this correctly, you must call the star command indirectly using the OSCLI command

Example: This will work
```
30 OSCLI("LOAD " + f$ + " " + STR$(addr%))
```