# Partitioning

Partitioning is the process of dividing a physical storage device (such as a hard drive or SSD) into separate logical sections called **partitions**. Each partition functions as an independent volume and can have its own filesystem, operating system, or data storage purpose.

A **partitioning scheme** defines the rules for how partitions are created, sized, and organized on the disk. Common partitioning schemes include:

| Partitioning Scheme              | Max Partitions                      | Max Disk Size                | Boot Support                             |
|----------------------------------|-------------------------------------|------------------------------|------------------------------------------|
| **MBR (Master Boot Record)**     | 4                                   | 2 TB                         | BIOS                                     |
| **GPT (GUID Partition Table)**   | 128 (typically)                     | 9.4 ZB (virtually unlimited) | UEFI (supports BIOS with protective MBR) |
| **Dynamic Disk (Windows)**       | Varies (depends on volume types)    | Depends on disk type         | BIOS/UEFI                                |
| **[LVM (Logical Volume Manager)](06_lvm.md)** | Flexible (depends on volume groups) | Flexible                     | BIOS/UEFI                                |

MBR and GPT are explained in detail in the following sections: [MBR →](04_mbr.md) and [GPT →](05_gpt.md). LVM is covered in [LVM →](06_lvm.md).

## CHS and LBA Addressing

As covered in [HDD Structure](02_hdd_structure.md#chs-addressing), early systems used **CHS (Cylinder-Head-Sector)** addressing to locate data by its physical position on the disk, while modern systems use **LBA (Logical Block Addressing)**, which assigns a simple sequential number to each sector.

This section covers the technical details of both methods, their limitations, and why the transition to LBA was necessary.

| Feature              | CHS                                   | LBA                                |
|----------------------|---------------------------------------|------------------------------------|
| Addressing method    | Cylinder, Head, Sector (physical)     | Linear block numbers (logical)     |
| Complexity           | Complex — depends on disk geometry    | Simple — sequential numbering      |
| Size limitation      | 8.4 GB (max with IDE BIOS extensions) | Supports disks larger than 2 TB    |
| Usage                | Older systems and legacy hardware     | All modern hard drives and SSDs    |

### CHS Address Format

A CHS address specifies data location using three values:

- **Cylinder (C):** the radial position on the platter — range: 0–1023
- **Head (H):** the platter surface (read/write head) — range: 0–254
- **Sector (S):** the position within the track — range: 1–63 (sectors start at 1, not 0)

For example, CHS address **(0, 1, 1)** means: cylinder 0 (first cylinder), head 1 (second platter surface), sector 1 (first sector on that track).

### LBA Address Format

LBA assigns each sector a unique sequential number starting from 0:

    Sector 0, 1, 2, ..., N

where N is the total number of sectors on the disk minus one.

For example, a 100 GB SSD with 4 KB sectors contains approximately 26.2 million blocks, each identified by an LBA number from 0 to 26,214,399.

LBA removes all dependency on physical disk geometry. Modern interfaces (SATA, NVMe) and operating systems use LBA exclusively.

### CHS to LBA Conversion

A CHS address can be converted to an LBA address using the formula:

    LBA = (C × Hmax + H) × Smax + (S − 1)

| Symbol       | Description                     |
|--------------|---------------------------------|
| **C**        | Cylinder number                 |
| **H**        | Head number                     |
| **S**        | Sector number (starts at 1)     |
| **H_max**    | Total number of heads per cylinder |
| **S_max**    | Total number of sectors per track  |

### CHS Size Limitations

Due to the fixed bit widths allocated to each CHS field, CHS addressing imposes hard limits on the maximum addressable disk size:

**Original BIOS limit (504 MB):**

    1,024 cylinders × 16 heads × 63 sectors × 512 bytes = 528,482,304 bytes ≈ 504 MB

This applied to early BIOS implementations before Enhanced BIOS Extensions (EDD) were introduced.

**Enhanced IDE BIOS limit (8.4 GB):**

    1,024 cylinders × 255 heads × 63 sectors × 512 bytes = 8,422,686,720 bytes ≈ 8.4 GB

These limits were a major reason for the transition to LBA.

### MBR and Maximum Disk Size

The MBR partition table stores partition start and end addresses using **32-bit LBA values**. This means it can address a maximum of 2^32 sectors:

    2^32 × 512 bytes per sector = 2,199,023,255,552 bytes ≈ 2 TB

This is the fundamental reason why MBR cannot support disks larger than 2 TB, which led to the development of GPT.

### Transition from CHS to LBA

During the boot process, the BIOS initially uses CHS to locate the bootable partition for backward compatibility. Once the bootloader is loaded, it switches to LBA for all further disk operations, since modern drives and operating systems rely on LBA for its simplicity and scalability.

### Addressing in SSDs

SSDs use flash memory (NAND) instead of spinning disks, so they have no physical cylinders, heads, or tracks. However, SSDs still use LBA as their addressing interface — the operating system sees a flat sequence of blocks, just like with an HDD.

Internally, the SSD's **Flash Translation Layer (FTL)** maps LBA addresses to physical memory cells in the NAND flash. This mapping also handles:

- **Wear leveling** — distributing writes evenly across cells to extend the drive's lifespan.
- **Garbage collection** — reclaiming space from deleted data.
- **Bad block management** — remapping failed memory cells transparently.

For compatibility with legacy systems, SSD firmware can emulate CHS geometry (e.g., reporting 1,024 cylinders, 255 heads, 63 sectors), even though these values have no physical meaning.

[← Previous: HDD Structure](02_hdd_structure.md) | [Next: MBR →](04_mbr.md)
