# PC-FX_Info
Technical Information about the PC-FX


## FX-BMP Card-Edge pinout (on the FX-BMP cartridge)

Orientation:
When looking at the FX-BMP cartridge, the side which looks like the "top" (has the
branding identification) is rotated to face left for insertion into the PC-FX.
The "underside" (which faces right when inserted into the machine) is actually the
side of the PC Board on which the parts are mounted.

This right-facing "underside" also has silkscreen printing identifying a pin
numbering system.  The pin numbered "1" is at the lower-right when inserted into
the PC-FX, and the pin numbered "25" is on the same face, opposite side (upper-right
when inserted into PC-FX).  While there is no printing identifying pins 26 through
50, I will refer to them as numbered in the same way - the reverse-side of the board
from "pin 1" will be "pin 26", and the reverse of "pin 25" will be "pin 50".

Since a relatively common static RAM was used on the board, many of the electrical
signals were possible to be decoded.  However, an ASIC was mounted on the board,
governing many control signals.  As the FX-BMP is battery-backed, and the PC-FX can
detect when the batteries are low, it is not clear whether the ASIC has a memory-mapped
register for battery level, or whether analog comparators are used.

It is also not yet clear whether 5V or 3.3V logic is used for the SRAM and other signals.
Based on the part number of the SRAM - uPD431000AGW-70L - it is likely going to be a 5V bus.

| Pin | Description |
|-----|-------------|
| Pin 1 | GND |
| Pin 2 | A0 |
| Pin 3 | A1 |
| Pin 4 | A2 |
| Pin 5 | A3 |
| Pin 6 | A4 |
| Pin 7 | A5 |
| Pin 8 | A6 |
| Pin 9 | A7 |
| Pin 10 | Vdd |
| Pin 11 | Vdd |
| Pin 12 | A8 |
| Pin 13 | A9 |
| Pin 14 | A10 |
| Pin 15 | A11 |
| Pin 16 | A12 |
| Pin 17 | A13 |
| Pin 18 | A14 |
| Pin 19 | No Connection |
| Pin 20 | Vdd |
| Pin 21 | Vdd |
| Pin 22 | A15 |
| Pin 23 | A16 |
| Pin 24 | to CPLD, pin 11 (=A17 ?) |
| Pin 25 | GND |
| Pin 26 | GND |
| Pin 27 | D0 |
| Pin 28 | D1 |
| Pin 29 | D2 |
| Pin 30 | D3 |
| Pin 31 | D4 |
| Pin 32 | D5 |
| Pin 33 | D6 |
| Pin 34 | D7 |
| Pin 35 | GND |
| Pin 36 | to CPLD, pin 2 |
| Pin 37 | No Connection |
| Pin 38 | No Connection |
| Pin 39 | GND |
| Pin 40 | No Connection |
| Pin 41 | No Connection |
| Pin 42 | to CPLD, pin 3 |
| Pin 43 | GND |
| Pin 44 | GND |
| Pin 45 | to CPLD, pin 4 |
| Pin 46 | GND |
| Pin 47 | /WE |
| Pin 48 | No Connection |
| Pin 49 | to CPLD, pin 10 |
| Pin 50 | to CPLD, pin 12 |


Note: While GND at pins #1, 25, 26 appear to be ground plane, the other grounds may
be signals fed back to the PC-FX (such as "cart inserted"). 
