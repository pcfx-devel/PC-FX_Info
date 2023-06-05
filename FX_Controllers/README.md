# FX_Controllers

Information about the PC-FX controllers

## Overview

The PC-FX has two controller ports, and communicates via a serial protocol which sends 32
bits in a "scan". Scans are hardware-driven; no need to "bit-bang" the controllers.

Data can travel both inbound and outbound via these ports.

Internal bit representation of the data is inverted from external representation. That is
to say, if a button on a controller is not pressed, a pull-up resistor ensures that the
value is 'high' (logical '1'), but when the value is read via the port, it is reported as '0'.
Likewise, a pressed button is electrically 'low' (logical '0'), but reported to the program
as a '1'.


## Types of Devices

Within the 32-bit protocol, there is a 4-bit section (most-significant 4 bits) which
identifies the type of controller. This can be used by the program to determine how to
interpret the remainder of the data sent via the port.

### Joypad

The joypad is a fairly standard 6-button joypad from the time, but with a unique feature:
Two slide switches ('A' and 'B') whose position can be read by the host machine.

The joypad is identified by code 15 (0xF) in the top four bits of the data word returned at the data port.
Electrically, this is bit pattern 0000, but internally at the data port, the bit pattern is 1111.

The description of all of the bits is as follows:

| Bit # | 31 | 30 | 29 | 28 | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 19 | 18 | 17 | 16 |
|-------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
|       | 1  | 1  | 1  | 1  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  |
| Bit # | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-------|----|----|----|----|----|----|---|---|---|---|---|---|---|---|---|---|
|       | 0  | SW2 | 0 | SW1 | Left | Down | Right | Up | Run | Sel | VI | V | IV | III | II | I |


### Mouse

The mouse is a fairly-standard (for 1994) 2-button mouse with a cord and a ball which needs periodic cleaning.

The mouse is identified by code 13 (0xD) in the top four bits of the data word returned at the data port.
Electrically, this is bit pattern 0010, but internally at the data port, the bit pattern is 1101.

The X and Y values are repesneted as 8-bit signed numbers:
 - Values -127 though -1 are represented by 0x81 through 0xFF.
 - Values 1 through 127 are represented by 0x01 through 0x7F
 - 0x00 is used for no movement, and 0x80 is undefined

Horizontally, Left is negative, and Right is positive.
Vertically, Up is negative, and Down is positive.


| Bit # | 31 | 30 | 29 | 28 | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 19 | 18 | 17 | 16 |
|-------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
|       | 1  | 1  | 0  | 1  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | Left Btn | Right Btn |
| Bit # | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-------|----|----|----|----|----|----|---|---|---|---|---|---|---|---|---|---|
|       | X7 | X6 | X5 | X4 | X3 | X2 | X1 | X0 | Y7 | Y6 | Y5 | Y4 | Y3 | Y2 | Y1 | Y0 |


### Multitap

The multitap was a peripheral which was never sold at retail, despite being designed, and having clear
specifications in the developers manuals.

In the event that a multi-tap is present, a sequence of 5 joyport scans should have the multi-tap
identify itself as the last possible device in the list. For example, if the multi-tap could only
support 2 devices, those two devices would appear at scans #1 and #2, with the multi-tap self-identifying
on the third scan. The multi-tap was defined as supporting up to 4 devices, which means the maximum-sized
multi-tap would self-identify on scan #5.

The multitap is identified by code 14 (0xE) in the top four bits of the data word returned at the data port.
Electrically, this is bit pattern 0001, but internally at the data port, the bit pattern is 1110.

| Bit # | 31 | 30 | 29 | 28 | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 19 | 18 | 17 | 16 |
|-------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
|       | 1  | 1  | 1  | 0  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  |
| Bit # | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-------|----|----|----|----|----|----|---|---|---|---|---|---|---|---|---|---|
|       | -  | -  | -  | -  | -  | -  | - | - | - | - | - | - | - | - | - | - |


## Controller Pinouts

The electrical signals available at the ports are as follows:
(Both console-side and controller-cable are shown to prevent ambiguity)

<img src="https://github.com/pcfx-devel/PC-FX_Info/blob/main/FX_Controllers/images/console_port.jpg" width="366" height="400">
<img src="https://github.com/pcfx-devel/PC-FX_Info/blob/main/FX_Controllers/images/joypad_connector.jpg" width="366" height="400">


## Hardware Signalling


## Reading and Writing in Software

### I/O Ports

## Bugs and Memoranda

In testing the fxuploader, there were two behaviours which could be called "errata":

 1) When the direction of data changes from inbound (the PC-FX receives data) to
outbound (the PC-FX sends data), occasionally the lower halfword of the outbound data
is incorrect. In fact, it repeats the last inbound lower halfword.
   - We did not find any way to detect or prevent this issue, so the approach taken
was for the first word sent to be dummy data, to be ignored by the receiver.

 2) On older hardware, inbound data scans (PC-FX receives data, such as from a joypad)
may actually not trigger a scan. In such cases, reading the status register for transition
to "scan complete" will never succeed.
   - In fxloader, this showed as a 'hang'. The solution was to time out of the check
status loop, and re-trigger a scan.
   - This appears to be specific to some older hardware, as it was not reproducible on
a newer-manufactured system. It is not clear when during the system's lifetime this
issue was fixed.


## Special Thanks

Special Thanks to the following people for their contributions to this knowledgebase:

 - Jacques Gagnon for
[his initial breakdown of signalling](https://hackaday.io/project/170365-blueretro/log/191237-pc-fx-interface)
for the BlueRetro project
 - Martin Wendt for the groundwork on the [pcfx_uploader](https://github.com/enthusi/pcfx_uploader)
 - Alex Marshall (trap15) for the initial version of liberis
 - John Brandwood for his work maintaining the
[V810 gcc cross-compiler](https://github.com/jbrandwood/v810-gcc) and
[tools](https://github.com/jbrandwood/pcfxtools) to make all of this possible.
