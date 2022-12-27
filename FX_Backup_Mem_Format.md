# FX_Backup_Mem_Format

An overview of the internal format of the backup memory storage areas on the PC-FX.

## Overview

The overall storage format of the backup memory area is FAT-12.  There are many variations of this
format, so in the following sections, this will be specified in more detail.

The high-level breakdown of the FAT filesystem includes the following elements:
 1. The Boot Sector (or reserved sectors), which includes an area called the BPB (BIOS Parameter Block),
which provides some specifics about the sizes and locations of other features in the filesystem.
 2. The FAT Region, which provides information on which allocation units (groups of sectors) are free,
which units are marked as unusable, and which are parts of files.
 3. The Root Directory Region, which is a fixed-size area containing entries for files and
subdirectories. While the root directory has a limit to the number of entries available,
subdirectories are not limited int eh same way, and are managed as files are - extensible chains of
allocation units.
 4. The Data Region, which is broken down into allocation units (groups of sectors), which
may be used for arbitrary storage by the game (when allocated and assembled into chains to create
files).

As for the actual save game entries on PC-FX, each game will create an entry in the root directory
with a characteristic name; this entry may refer to a single file storing the key information about
the game, or it may be a subdirectory which in turn may hold multiple files, each with its own stored
information about saved games. Most games which allow for multiple different save games will save
these as different files within the same game-specific folder.

### Layout Overview for Different Size Memories

| Type | Values: 32KB (Internal) | FX-BMP: 128KB | 256KB | 512KB | 1MB | 2MB | 4MB | 8MB |
|-----------|-------------:|--------:|--------:|-------:|------:|------:|------:|------:|
| FAT Offset | 0x0080 | 0x0080 | 0x0080 | 0x0080 | 0x0080 | 0x0080 | 0x0080 | 0x0080 |
| Root DIR Offset | 0x0200 | 0x0680 | 0x0C80 | 0x1800 | 0x3F80 | 0x4000 | 0x4080 | 0x4080 |
| Data Offset | 0x0A00 | 0x2600 | 0x2C00 | 0x3800 | 0x6000 | 0x6000 | 0x6000 | 0x6000 |
| Total Size | 0x8000 | 0x20000 | 0x40000 | 0x80000 | 0x100000 | 0x200000 | 0x400000 | 0x800000 |


## FAT Internals

In this section, the FAT filesystem's specifcs - as implemented on the PC-FX - will be explained.
As there are many possible (theoretical) sizes of FX-BMP cartridge, actual values for each will
be shown in the tables below.

### Boot Sector and BIOS Parameter Block

There is only one Boot Sector in a PC-FX save area.
The BIOS Parameter Block starts at offset 0x0B in the Boot Sector.

| Byte Offset | Use | Size (Bytes) | Values: 32KB (Internal) | FX-BMP: 128KB | 256KB | 512KB | 1MB | 2MB | 4MB | 8MB |
|-----------|-------|-------------:|------:|--------:|--------:|-------:|------:|------:|------:|------:|
| 0x0000 | ??? | 3 | 0x24 0x8A 0xDF | Same | Same | Same | Same | Same | Same | Same |
| 0x0003 | OEM Name | 8 | 'PCFXSram' | 'PCFXCard' | 'PCFXCard' | 'PCFXCard' | 'PCFXCard' | 'PCFXCard' | 'PCFXCard' | 'PCFXCard' |
| 0x000B | Bytes per sector | 2 | 0x0080 (128) | 0x0080 (128) | 0x0080 (128) | 0x0080 (128) | 0x0080 (128) | 0x0080 (128) | 0x0080 (128) | 0x0080 (128) |
| 0x000D | Sectors per cluster | 1 | 1 | 1 | 1 | 1 | 1 | 2 | 4 | 8 |
| 0x000E | Number of Reserved Sectors | 2 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 0x0010 | Number of File Allocation Tables | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 0x0011 | Maximum number of Root Directory entries | 2 | 0x40 (64) | 0xFC (252) | 0x100 (256) | 0x100 (256) | 0x104 (260) | 0x100 (256) | 0xFC (252) | 0xFC (252) |
| 0x0013 | Total number of Sectors | 2 | 0x40 (64) | 0x100 (256) | 0x200 (512) | 0x400 (1024) | 0x800 (2048) |  |  |  |
| 0x0015 | Media Descriptor | 1 | 0xF9 | 0xF9 | 0xF9 | 0xF9 | 0xF9 | 0xF9 | 0xF9 | 0xF9 |
| 0x0016 | Sectors per File Allocation Table | 2 | 0x03 (3) | 0x0C (12) | 0x17 (23) | 0x2F (47) | 0x7E (126) | 0x7F (127) | 0x80 (128) | 0x80 (128) |

### FAT Region

As this is a FAT-12 filesystem, each block (sector, in most cases) is represented by 12 bits, or one-and-a-half bytes.

Within memory, a 3-byte sequence with nybbles as so:\
```01 23 45```
Will mean that the first block's FAT value will be 0x301, and the second one's value will be 0x452

**Actual Values of FAT entries**

| Value | Description |
|-------|-------------|
| 0x000 | Free cluster - empty or available |
| 0x002 - 0xFEF | Data cluster - points to the next cluster in the sequence |
| 0xFF0 - 0xFFF | Treat as end-of-chain / end-of-file, or marked as unavailable for other reasons |

### Root Directory Region

 