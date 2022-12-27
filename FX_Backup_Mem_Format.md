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


## FAT Internals

In this section, the FAT filesystem's specifcs - as implemented on the PC-FX - will be explained.
As there are many possible (theoretical) sizes of FX-BMP cartridge, actual values for each will
be shown in the tables below.

### Boot Sector and BIOS Parameter Block

There is only one Boot Sector in a PC-FX save area.

| Byte Offset | Use | Size (Bytes) | Values: 32KB (Internal) | FX-BMP: 128KB | 256KB | 512KB | 1MB | 2MB |
|-----------|-------|-------------:|------:|--------:|--------:|-------:|------:|------:|
| 0x0000 | ??? | 3 | 0x24 0x8A 0xDF | Same | Same | Same | Same | Same |
| 0x0003 | OEM Name | 8 | 'PCFXSram' | 'PCFXCard' | 'PCFXCard' | 'PCFXCard' | 'PCFXCard' | 'PCFXCard' |

### FAT Region

### Root Directory Region

## 
