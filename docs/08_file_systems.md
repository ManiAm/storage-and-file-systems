# File Systems

In the previous sections, we covered how a disk is divided into partitions using [MBR](04_mbr.md) or [GPT](05_gpt.md). However, a freshly created partition is just a reserved range of sectors — it has no structure and cannot store files yet.

To make a partition usable, it must be **formatted**. Formatting is the process of writing a **filesystem** onto a partition. This creates the internal data structures (directories, allocation tables, metadata) that the operating system needs to organize, store, and retrieve files.

In short: **partitioning** divides the disk → **formatting** writes a filesystem → the partition is ready to use.

## Filesystem Comparison

The following table lists major filesystems in chronological order:

| **Filesystem** | **Year** | **Creator**            | **OS**                     | **Max File Size** | **Max Volume Size** |
|----------------|----------|------------------------|----------------------------|--------------------|----------------------|
| FAT12          | 1980     | SCP, Microsoft         | MS-DOS, Windows            | 16 MB              | 16 MB                |
| FAT16          | 1984     | IBM, Microsoft         | MS-DOS, Windows            | 4 GB               | 4 GB                 |
| HFS            | 1985     | Apple                  | Classic Mac OS             | 2 GB               | 2 TB                 |
| HPFS           | 1989     | Microsoft/IBM          | OS/2                       | 2 GB               | 2 TB                 |
| ext            | 1992     | Rémy Card              | Linux                      | 2 GB               | 2 GB                 |
| NTFS           | 1993     | Microsoft              | Windows NT and later       | 16 EB              | 16 EB                |
| ext2           | 1993     | Rémy Card              | Linux                      | 2 TB               | 16 TB                |
| XFS            | 1994     | SGI                    | Linux                      | 8 EB               | 8 EB                 |
| UDF            | 1995     | OSTA                   | Cross-platform (optical)   | 16 EB              | 16 TB                |
| FAT32          | 1996     | Microsoft              | Windows, macOS, Linux      | 4 GB               | 2 TB                 |
| HFS+           | 1998     | Apple                  | macOS                      | 8 EB               | 8 EB                 |
| ext3           | 2001     | Stephen Tweedie        | Linux                      | 2 TB               | 16 TB                |
| ZFS            | 2004     | Sun Microsystems       | Solaris, Linux, BSD        | 16 EB              | 16 EB                |
| exFAT          | 2006     | Microsoft              | Windows, macOS, Linux      | 16 EB              | 128 PB               |
| ext4           | 2006     | Theodore Ts'o et al.   | Linux                      | 16 TB              | 1 EB                 |
| Btrfs          | 2009     | Chris Mason            | Linux                      | 16 EB              | 16 EB                |
| APFS           | 2017     | Apple                  | macOS, iOS                 | 8 EB               | 8 EB                 |

> For a comprehensive comparison, see [Wikipedia — Comparison of file systems](https://en.wikipedia.org/wiki/Comparison_of_file_systems).

## Default Filesystems by Operating System

Each operating system has a default filesystem that it uses when formatting new storage:

| Operating System         | Default Filesystem            |
|--------------------------|-------------------------------|
| Linux (most distros)     | ext4                          |
| ChromeOS                 | ext4                          |
| Android                  | ext4 or F2FS                  |
| Windows (current)        | NTFS                          |
| Windows (older versions) | FAT32                         |
| macOS (current)          | APFS                          |
| macOS (older versions)   | HFS+                          |
| iOS                      | APFS                          |
| FreeBSD                  | UFS or ZFS                    |
| Solaris                  | ZFS                           |

## Hierarchical Structure

Nearly all modern filesystems organize data in a **hierarchical** (tree-like) structure:

- **Files** are the basic units of storage — they contain text, images, programs, or any other data.
- **Directories** (also called **folders** in GUI environments) are containers that hold files and other directories (subdirectories).

The topmost directory in the hierarchy is called the **root directory**:

- In Unix/Linux, the root directory is denoted as `/`.
- In Windows, each drive has its own root, denoted by a drive letter (e.g., `C:\`).

**Unix/Linux example:**

```
/
├── home/
│   └── user1/
│       ├── Documents/
│       ├── Downloads/
│       └── file.txt
└── var/
    ├── log/
    ├── www/
    └── cache/
```

**Windows example:**

```
C:\
├── Program Files\
├── Users\
│   └── John\
│       ├── Documents\
│       ├── Pictures\
│       └── report.docx
└── Windows\
    ├── System32\
    └── Logs\
```

> **Directory vs. Folder:** These terms refer to the same concept. "Directory" is the traditional term used in technical contexts, while "folder" was introduced with graphical user interfaces (GUIs) as a more visual, user-friendly label. In practice, they are interchangeable.

## Paths

The location of a file or directory within the hierarchy is called a **path**. A path describes how to navigate the tree structure to reach a specific file or directory.

Path components are separated by `/` (Unix/Linux) or `\` (Windows).

**Absolute path** — starts from the root directory and provides the full location:

```
Unix:    /home/user/file.txt
Windows: C:\Users\User\file.txt
```

**Relative path** — starts from the current directory and specifies the location relative to it:

```
../Documents/report.txt      (go up one level, then into Documents)
./config.yaml                (current directory)
```

## File Extensions

A **file extension** is a suffix at the end of a filename (e.g., `.txt`, `.jpg`, `.exe`) that identifies the file type. It tells the operating system or application what program to use to open or process the file.

How different operating systems handle extensions:

- **Windows** heavily depends on file extensions to determine file types and associated programs. Without a proper extension, Windows may not recognize how to open the file.

- **Unix/Linux** does not strictly rely on extensions. It can identify files by their content using **magic numbers** (signatures embedded within the file) and commands like `file`. Extensions are still used by convention but are not required.

- **macOS** uses file extensions but is less dependent on them compared to Windows. It also uses metadata to identify file types.

## Journaling

A **journaling filesystem** records changes in a separate structure called a **journal** (or log) before committing them to the main filesystem. If a system crash or power failure occurs mid-write, the journal can be used to recover incomplete changes — either completing them or rolling them back. This greatly reduces the risk of filesystem corruption and speeds up recovery.

There are two levels of journaling:

| Journaling Level     | What is Logged                | Reliability | Performance |
|----------------------|-------------------------------|-------------|-------------|
| **Metadata-only**    | Directory structures, file attributes, allocation tables | Good | Better (less write overhead) |
| **Full data + metadata** | All modified data blocks plus metadata | Higher | Lower (more writes) |

**Examples:**

- **NTFS** uses metadata-only journaling.
- **ext3/ext4** support both modes — metadata-only (default) or full data journaling (configurable).

Journaling is now standard in all modern filesystems. Older filesystems like FAT12/16/32 and ext2 do not have journaling, which is why they are more vulnerable to corruption after unexpected shutdowns.

[← Previous: Partition Layout](07_partition_layout.md) | [Next: Disk Tools →](09_disk_tools.md)
