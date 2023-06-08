# FX_Controllers

Information about the PC-FX controllers

## Contents

 1. [Overview](#overview)
 2. [Types of Devices](#types-of-devices)
    1. [Joypad](#joypad)
    2. [Mouse](#mouse)
    3. [Multitap](#multitap)
 3. [Controller Pinouts](#controller-pinouts)
 4. [Hardware Signalling](#hardware-signalling)
 5. [Reading and Writing in Software](#reading-and-writing-in-software)
 6. [Bugs and Memoranda](#bugs-and-memoranda)


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
Electrically, this is bit pattern ``0000``, but internally at the data port, the bit pattern is ``1111``.

The description of all of the bits is as follows:

| Bit # | 31 | 30 | 29 | 28 | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 19 | 18 | 17 | 16 |
|:-----:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|       | 1  | 1  | 1  | 1  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  |
| **Bit #** | **15** | **14** | **13** | **12** | **11** | **10** | **9** | **8** | **7** | **6** | **5** | **4** | **3** | **2** | **1** | **0** |
|       | 0  | SW2 | 0 | SW1 | Lft | Dn | Rt | Up | Run | Sel | VI | V | IV | III | II | I |


### Mouse

The mouse is a fairly-standard (for 1994) 2-button mouse with a cord and a ball which needs periodic cleaning.

The mouse is identified by code 13 (0xD) in the top four bits of the data word returned at the data port.
Electrically, this is bit pattern ``0010``, but internally at the data port, the bit pattern is ``1101``.

The X and Y values are repesneted as 8-bit signed numbers:
 - Values -127 though -1 are represented by 0x81 through 0xFF.
 - Values 1 through 127 are represented by 0x01 through 0x7F
 - 0x00 is used for no movement
 - 0x80 is undefined

Horizontally, Left is negative, and Right is positive.
Vertically, Up is negative, and Down is positive.


| Bit # | 31 | 30 | 29 | 28 | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 19 | 18 | 17 | 16 |
|:-----:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|       | 1  | 1  | 0  | 1  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | L Btn | R Btn |
| **Bit #** | **15** | **14** | **13** | **12** | **11** | **10** | **9** | **8** | **7** | **6** | **5** | **4** | **3** | **2** | **1** | **0** |
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
Electrically, this is bit pattern ``0001``, but internally at the data port, the bit pattern is ``1110``.

| Bit # | 31 | 30 | 29 | 28 | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 19 | 18 | 17 | 16 |
|:-----:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|       | 1  | 1  | 1  | 0  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  | -  |
| **Bit #** | **15** | **14** | **13** | **12** | **11** | **10** | **9** | **8** | **7** | **6** | **5** | **4** | **3** | **2** | **1** | **0** |
|       | -  | -  | -  | -  | -  | -  | - | - | - | - | - | - | - | - | - | - |


## Controller Pinouts

The electrical signals available at the ports are as follows:
(Both console-side and controller-cable are shown to prevent ambiguity)

<img src="https://github.com/pcfx-devel/PC-FX_Info/blob/main/FX_Controllers/images/console_port.jpg" width="366" height="400">
<img src="https://github.com/pcfx-devel/PC-FX_Info/blob/main/FX_Controllers/images/joypad_connector.jpg" width="366" height="400">


## Hardware Signalling

The signalling protocol on the PC-FX is serial, resembling SPI somewhat, with the PC-FX driving the CLK and other control lines.
Although the general use of a joyport is to accept data from the outside world (such as from a joypad), the joyports on the PC-FX
are capable of bidirectional data transfer, with the direction controlled by the R/W line.

When a 'scan' (data transfer) takes place, the following sequence occurs:
  1. The R/W line is set to indicate data direction:
     - LOW = "Read" (data is outbound from peripheral, and inbound to PC-FX)
     - HIGH = "Write" (data is outbound from PC-FX, and inbound to peripheral)
  2. The LATCH line descends from it normal HIGH level to LOW for roughly 3 microseconds, returning to HIGH, to indicate "start of scan".
     - If CLK transitions for a cycle while LATCH is low, this is to indicate that the multitap joypad counter is to be reset, and the first joypad is to return its data in this scan.
     - Actual data is only transferred when LATCH has returned to HIGH
  3. CLK cycles for 32 cycles at roughly 3 microseconds per cycle, sending/requesting data synchronously with the CLK signal.
     - inbound data from peripheral should be set when CLK is HIGH - including immediately as LATCH returns to HIGH - to be read by the PC-FX on transition to LOW
     - outbound data from PC-FX is also set when CLK is HIGH - including immediately as LATCH returns to HIGH - to be read by the peripheral on transition to LOW

Data sequence is least-significant bit first, and most-significant bit last, with the controller-specific bit-fields above describing
the meanings of the various bits. Keep in mind that electrical data values are inverted as compared with their internal representation
at the software ports.  For example, an internal '0' would be represented externally on the joyport as a '1'.

### Logic Traces

#### Inbound Data

<img src="https://github.com/pcfx-devel/PC-FX_Info/blob/main/FX_Controllers/images/Inbound_data_logic_trace.png">

#### Outbound Data

<img src="https://github.com/pcfx-devel/PC-FX_Info/blob/main/FX_Controllers/images/Outbound_data_logic_trace.png">

## Reading and Writing in Software

For each of the two joyports, there are separate control and data ports to handle I/O.

The status port is read in order to determine scanning status.\
The control port (same address as the status port) is written to, in order to trigger a scan.
As the joyport can perform bidirectional I/O, the data port is also read/write.

### I/O Ports

| Port | Port Address | Memory Address |
|:----:|:------------:|:--------------:|
| Port 0 Control (write) | 0x00000000 | 0x80000000 |
| Port 0 Status (read) | 0x00000000 | 0x80000000 |
| Port 0 Data (Low) | 0x00000040 | 0x80000040 |
| Port 0 Data (High) | 0x00000042 | 0x80000042 |
| Port 1 Control (write) | 0x00000080 | 0x80000080 |
| Port 1 Status (read) | 0x00000080 | 0x80000080 |
| Port 1 Data (Low) | 0x000000C0 | 0x800000C0 |
| Port 1 Data (High) | 0x000000C2 | 0x800000C2 |

#### Control Port (When Written)

| Bit # | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|:-----:|:--:|:--:|:--:|:--:|:--:|:--:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Purpose | -  | -  | -  | -  | -  | -  | - | - | - | - | - | - | - | KOIOS | KOMOD | KOTRG |
| Initial Value | -  | -  | -  | -  | -  | -  | - | - | - | - | - | - | - | 1 | 1 | 0 |

**KOIOS** - I/O Setting (Direction of transfer):\
1: Input\
0: Output

**KOMOD** - Mode Setting:\
1: Multi-tap Clear (First Scan of any read cycle should set this)\
0: No Multi-tap Clear

**KOTRG** - Transfer Start:\
1: Initiate Transfer\
0: Do Not Start Transfer (Do Not Use)

#### Status Port (When Read)

| Bit # | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|:-----:|:--:|:--:|:--:|:--:|:--:|:--:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|       | -  | -  | -  | -  | -  | -  | - | - | - | - | - | - | KOEND | KOIOS | KOMOD | KOTRG |
| Initial Value | -  | -  | -  | -  | -  | -  | - | - | - | - | - | - | 0 | 1 | 1 | 0 |

**KOEND** - Transfer End State\
1: Transfer complete (if interrupt not masked, generate interrupt; read will reset port)\
0: Not Complete, or ready for next transfer

**KOIOS** - I/O Setting (Direction of transfer):\
1: Input\
0: Output

**KOMOD** - Mode Setting:\
1: Multi-tap Clear (i.e. First Scan of a scan cycle)\
0: No Multi-tap Clear

**KOTRG** - Transfer Start:\
1: Transfer in Progress\
0: Transfer Complete - Note that KOEND should be read in order to reset


#### Data Port

The data port (low) and data port (high) are described by the developer documents as being two separate 16-bit ports.
While it may be possible to combine them into a single 32-bit read or write, this appears to be unsupported, and may
also have different timing characteristics than expected due to wait states which may be required by the I/O controller.


## Bugs and Memoranda

In testing the fxuploader, there were two behaviours which could be called "errata":

 1. When the direction of data changes from inbound (the PC-FX receives data) to
outbound (the PC-FX sends data), occasionally the lower halfword of the outbound data
is incorrect. In fact, it repeats the last inbound lower halfword.
    - We did not find any way to detect or prevent this issue, so the approach taken
was for the first word sent to be dummy data, to be ignored by the receiver.

 2. On older hardware, inbound data scans (PC-FX receives data, such as from a joypad)
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
