# Filesystem Hierarchy Standard (FHS)

Unlike Windows, which assigns each storage device a separate drive letter (`C:`, `D:`, `E:`), Linux organizes all files and directories into a **single unified tree** rooted at `/`. Every disk, partition, and device is [mounted](15_mounting.md) into this tree, appearing as an ordinary directory. The **Filesystem Hierarchy Standard** (FHS) defines where things should go within this tree — ensuring consistency and predictability across Linux distributions for users, developers, and system administrators.

## FHS Versions

| Version | Year | Description |
|---------|------|-------------|
| 1.0     | 1994 | Initial release. Defined the basic directory structure and layout. |
| 2.0     | 1997 | Major update. Introduced clearer definitions for `/usr`, `/var`, and `/opt`. |
| 2.1     | 1999 | Minor adjustments. Clarified `/boot` usage and introduced `/mnt` for temporary mounts. |
| 2.2     | 2001 | Further refinements. Enhanced descriptions for `/dev` and `/proc`. |
| 2.3     | 2004 | Introduced `/srv` for site-specific data and `/media` for removable media. Clarified `/tmp` and `/var/tmp`. |
| 3.0     | 2015 | Simplified directory requirements. Removed `/lib64` requirement, allowing `/lib` for both 32-bit and 64-bit. |
| 3.1     | 2020 | Latest release. Updated `/run` and deprecated `/var/run`. |

After version 3.1, the FHS has been minimally updated, as most distributions have adopted the standard and evolved independently.

## Physical vs Virtual Directories

Linux directories fall into two categories:

- **Physical directories** are stored on disk and persist across reboots. They hold the files and data needed for system and user operations.

- **Virtual directories** are created dynamically by the kernel at runtime. They exist only in memory (RAM), consume no disk space, and provide live interfaces to processes, hardware, and kernel internals. These are the same [pseudo-filesystems](13_ext_filesystem.md#pseudo-filesystems) discussed in the previous section.

### Physical Directories

| Directory | Description | Examples |
|-----------|-------------|----------|
| `/`       | Root of the entire filesystem tree. | All other directories branch from here. |
| `/home`   | User home directories. | `/home/alice`, `/home/bob` |
| `/root`   | Home directory for the root user. | `/root/.bashrc` |
| `/boot`   | Kernel and bootloader files. | `vmlinuz`, `initrd`, GRUB config |
| `/bin`    | Essential user binaries. | `ls`, `cp`, `mv`, `cat` |
| `/sbin`   | Essential system administration binaries. | `reboot`, `fsck`, `fdisk` |
| `/lib`    | Shared libraries and kernel modules. | Libraries for `/bin` and `/sbin` |
| `/etc`    | System-wide configuration files. | `/etc/hosts`, `/etc/passwd`, `/etc/fstab` |
| `/media`  | Auto-mounted removable media. | USB drives, CDs, DVDs |
| `/mnt`    | Temporary manual mount point. | Ad-hoc filesystem mounts |
| `/opt`    | Optional third-party software. | `/opt/myapp` |
| `/srv`    | Data served by the system. | `/srv/www`, `/srv/ftp` |
| `/usr`    | User applications and shared resources. | `/usr/bin`, `/usr/lib`, `/usr/share` |
| `/var`    | Variable data that changes frequently. | `/var/log`, `/var/cache` |
| `/tmp`    | Temporary files, cleared on reboot. | Short-lived application data |

### Virtual Directories

| Directory | Description | Examples |
|-----------|-------------|----------|
| `/dev`    | Device files representing hardware and virtual devices. | Disks (`/dev/sda`), terminals (`/dev/tty`), null device (`/dev/null`). |
| `/proc`   | Process and system information provided by the kernel. | CPU info (`/proc/cpuinfo`), memory info (`/proc/meminfo`). |
| `/sys`    | Hardware and device attributes exposed by the kernel. | Network interfaces (`/sys/class/net`), block devices (`/sys/block`). |
| `/run`    | Runtime data for the current boot session. | PID files, sockets, and transient system files (`/run/systemd`). |

## Filesystem Support in the Kernel

Linux filesystem drivers (ext4, FAT, NTFS, etc.) run inside the **kernel**, not as user-space applications. The driver code is part of the kernel source tree. Distributions choose which drivers to include and which filesystem to use by default (most use ext4), but the underlying implementation is provided by the kernel itself.

[← Previous: ext Filesystem](13_ext_filesystem.md) | [Next: Mounting →](15_mounting.md)
