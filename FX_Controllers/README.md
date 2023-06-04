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

### Mouse

### Multitap


## Controller Pinouts

The electrical signals available at the ports are as follows:

## Hardware Signalling

## Reading and Writing in Software

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

 - Jacques Gagnon for [his initial breakdown of signalling](https://hackaday.io/project/170365-blueretro/log/191237-pc-fx-interface) for the BlueRetro project
 - Martin Wendt for the groundwork on the [pcfx_uploader](https://github.com/enthusi/pcfx_uploader)
 - Alex Marshall (trap15) for the initial version of liberis
 - John Brandwood for his work maintaining the
[V810 gcc cross-compiler](https://github.com/jbrandwood/v810-gcc) and
[tools](https://github.com/jbrandwood/pcfxtools) to make all of this possible.
