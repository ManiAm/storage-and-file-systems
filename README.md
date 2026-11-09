# Disk Storage and File Systems

This project covers the full stack of disk storage — from physical hardware through logical partitioning to filesystem data structures and directory organization. Each layer builds directly on the previous, forming a bottom-up progression.

## Storage Types

| # | Topic | Description |
| -- | --------------------------------------------------------------------- | ----------------------------------------------------------- |
| 01 | [Storage Types](docs/01_storage.md) | DAS, SAN, NAS, Object Storage, Distributed Filesystems |

## Partitioning

| # | Topic | Description |
| -- | --------------------------------------------------------------------- | ----------------------------------------------------------- |
| 02 | [HDD Structure](docs/02_hdd_structure.md) | Platters, spindles, tracks, cylinders, sectors |
| 03 | [Partitioning](docs/03_partitioning.md) | Partitioning concepts, CHS vs LBA addressing |
| 04 | [MBR](docs/04_mbr.md) | MBR structure, partition table, EBR, device naming |
| 05 | [GPT](docs/05_gpt.md) | GPT structure, partition entries, type GUIDs, device naming |
| 06 | [LVM](docs/06_lvm.md) | Logical Volume Manager — PV, VG, LV, resizing, commands |
| 07 | [Partition Layouts](docs/07_partition_layout.md) | Common MBR and GPT layouts (Windows, Linux, dual boot) |

## File Systems

| # | Topic | Description |
| -- | --------------------------------------------------------------------- | ----------------------------------------------------------- |
| 08 | [File Systems](docs/08_file_systems.md) | File system overview, hierarchy, paths, extensions, journaling |
| 09 | [Disk Tools](docs/09_disk_tools.md) | mkfs, Rufus, ImHex, 010 Editor |
| 10 | [Metadata](docs/10_metadata.md) | Timestamps, ownership, permissions, umask, special mode bits |

## File System Internals

| # | Topic | Description |
| -- | --------------------------------------------------------------------- | ----------------------------------------------------------- |
| 11 | [FAT Filesystem](docs/11_fat_filesystem.md) | FAT12/16/32 internals, clusters, VFAT, long filenames |
| 12 | [NTFS Filesystem](docs/12_ntfs_filesystem.md) | NTFS internals, Master File Table (MFT), attributes |
| 13 | [ext Filesystem](docs/13_ext_filesystem.md) | ext4 internals, block groups, inodes, superblock, extents, VFS |

## File Management and Mounting

| # | Topic | Description |
| -- | --------------------------------------------------------------------- | ----------------------------------------------------------- |
| 14 | [Filesystem Hierarchy](docs/14_filesystem_hierarchy.md) | FHS, Linux directory structure (physical vs virtual) |
| 15 | [Mounting](docs/15_mounting.md) | Mounting concepts, boot sequence, fstab |
