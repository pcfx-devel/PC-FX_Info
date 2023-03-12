# FX_Backup_Mem_Format

An overview of the internal format of the backup memory storage areas on the PC-FX.

## Overview

The overall storage format of the backup memory area is FAT.  There are many variations of this
format, so in the following sections, this will be specified in more detail.

Notably, FAT-12 is in use for storage areas of 512KB and smaller, whereas FAT-16 is in use for
storage areas of 1MB or larger.

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

### Re-Formatting

Note that the boot sector below has specific values which indicate overall size of the FX-BMP unit;
although the format process normally detects the size of the module, it will NOT resize on a reformat,
so be careful in certain situations !

For example, if you restore a 128KB image onto a 512KB-capable BMP device and later try to reformat it,
the original boot sector entries will remain, and the unit will continue to appear to be a 128KB device.
In such a case, you will need to erase the boot sector (i.e. load 0x00's or seomthing similar) into the
boot sector, and then erase it.


### Layout Overview for Different Size Memories

| Type | Values: 32KB (Internal) | FX-BMP: 128KB | 256KB | 512KB | 1MB | 2MB | 4MB | 8MB |
|-----------|-------------:|--------:|--------:|-------:|------:|------:|------:|------:|
| FAT Offset | 0x0080 | 0x0080 | 0x0080 | 0x0080 | 0x0080 | 0x0080 | 0x0080 | 0x0080 |
| FAT Type | FAT-12 | FAT-12 | FAT-12 | FAT-12 | FAT-16 | FAT-16 | FAT-16 | FAT-16 |
| FAT Table Entries | 236 | 948 | 1960 | 3984 | 8000 | 8096 | 8144 | 8168 |
| Root DIR Offset | 0x0200 | 0x0680 | 0x0C80 | 0x1800 | 0x3F80 | 0x4000 | 0x4080 | 0x4080 |
| Root DIR Entries | 64 | 252 | 256 | 256 | 260 | 256 | 252 | 252 |
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
| 0x0013 | Total number of Sectors | 2 | 0x100 (256) | 0x400 (1024) | 0x800 (2048) | 0x1000 (4096) | 0x2000 (8192) | 0x4000 (16384) | 0x8000 (32768) | 0x10000 (65536) |
| 0x0015 | Media Descriptor | 1 | 0xF9 | 0xF9 | 0xF9 | 0xF9 | 0xF9 | 0xF9 | 0xF9 | 0xF9 |
| 0x0016 | Sectors per File Allocation Table | 2 | 0x03 (3) | 0x0C (12) | 0x17 (23) | 0x2F (47) | 0x7E (126) | 0x7F (127) | 0x80 (128) | 0x80 (128) |
| 0x0018 | "Sectors per Track"(*1) | 2 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 0x001A | "Number of Heads"(*1) | 2 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 0x001C | "Number of Hidden Sectors Preceding this FAT Volume"(*1) | 4 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 0x0020 | "Total Logical Sectors"(*1) | 4 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 0x0024 - 0x0027 | Appears to be unused | 4 | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) |
| 0x0028 - 0x003F | Boot information | 0x18 | (not eligible) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) |
| 0x0040 - 0x007F | Appears to be unused | 0x40 | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) | (uninitialized) |

**NOTE:**\
(*1) - These are the descriptions as per DOS 3.31 BPB - but it is not clear whether they have any such purpose here.

### FAT Region

As this is a FAT-12 filesystem, each block (sector, in most cases) is represented by 12 bits, or one-and-a-half bytes, meaning that two FAT entries
consumes 3 consecutive bytes; entries are aligned as such within the block.

As such, a 3-byte FAT sequence with nybbles as so:\
```01 23 45```\
Will mean that the first block's FAT value will be 0x301, and the second one's value will be 0x452

**Actual Values of FAT entries**

| Value | Description |
|-------|-------------|
| 0x000 | Free cluster - empty or available |
| 0x002 - 0xFEF | Data cluster - points to the next cluster in the sequence |
| 0xFF0 - 0xFFF | Treat as end-of-chain / end-of-file, or marked as unavailable for other reasons |

**NOTE:**\
The first 2 FAT entries are reserved, and contain special values: 0xFF9, and 0xFFF. The first value after these refers to the first block in
the Data Area.


### Root Directory Region

Each directory entry is a fixed-size 0x20 (32) bytes in length, and filename are based on MS-DOS type 8.3 filenames (filename + extension).\
However, the PC-FX allows the use of SJIS and longer filenames (up to 16 characters) by repurposing part of the directory entry data (see below).

| Byte Offset | Use | Size (Bytes) | Values |
|-----------|-------|-------------:|------|
| 0x00 | First 8 characters of filename (*1); Space-padded if short | 8 | |
| 0x08 | File Extension | 3 | Appears to be a code based on developer company |
| 0x0B | File attribute | 1 | 0x10 if subdirectory |
| 0x0C | Final (up to) 9 characters of filename; Zero-terminated if short | 9 | |
| 0x15 | Zero-terminator | 1 | 0x00 |
| 0x16 | "Last Modified Time" (*2) | 2 |  |
| 0x18 | "Last Modified Date" (*2) | 2 |  |
| 0x1A | First used cluster number | 2 |  |
| 0x1C | "File Size in Bytes" (*3) | 4 | 0x00 |

**NOTE:**\
(*1) - If the first character is 0xE5, the entry has been deleted and is no longer relevant.  Entries may exist for '.' and '..' (directory pointers).  If the first character ix 0x00, it marks the end of the list.\
(*2) - These are the descriptions as per DOS 3.31 FAT - but it is not clear whether they have any such purpose here. These are mostly unused, but are used by "J.B Harold Blue Chicago Blues"\
(*3) - This is the descriptions as per DOS 3.31 FAT - but it appears to be unused here, appearing as zeroes in all reviewed cases.


## Bootable Carts

For a cart to be bootable, the PC-FX checks for a key word, and if found, it copies data from the cartridge into main memory, and
executes it.  Code will not run directly from the FX-BMP due to the addressing arrangements/bus width.

Internal memory cannot be booted from, and the PC-FXGA doesn't support booting from backup.

When auto-boot is in place, the PC-FX boot cycle displays the splash screen, and goes directly into the cartridge boot code instead
of entering the menu screen.  This eliminates any sort of boot choices.

Note that cartridge offset address is relative to the start of the FX-BMP's chip memory.  So, a value of 0x1000 is chip-memory
address offset of 0x1000, and will be at 0xE8002000 in system memory.\
All numeric values are expressed as 'word' values.

| Byte Offset | Use | Size (Bytes) |
|-----------|-------|-------------:|
| 0x28 | Key string for boot: "PCFXBoot" | 8 |
| 0x30 | Program start location on card (byte offset) | 4 |
| 0x34 | System memory target address (usually 0x8000) | 4 |
| 0x38 | Transfer size (counted in bytes) | 4 |
| 0x3C | Execution address after transfer (usually 0x8000) | 4 |

