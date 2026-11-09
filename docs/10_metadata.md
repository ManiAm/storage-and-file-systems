# Metadata

In the previous sections, we covered how disks are partitioned and formatted with [filesystems](08_file_systems.md). Once a filesystem is in place, it needs a way to keep track of information *about* each file — not just the file's content, but things like its name, size, who owns it, and when it was last modified. This information is called **metadata**.

Metadata includes:

- File name
- File size
- File type
- Timestamps (creation, modification, access times)
- Ownership (user and group)
- Permissions (read, write, execute)

Different filesystems store metadata in different structures:

| Filesystem | Metadata Structure |
|------------|-------------------|
| FAT        | Directory entries  |
| NTFS       | Master File Table (MFT) records |
| ext2/3/4   | Inodes (index nodes) |

> These structures are covered in detail in the filesystem-specific docs ([FAT](11_fat_filesystem.md), [NTFS](12_ntfs_filesystem.md), [ext](13_ext_filesystem.md)).

## File Timestamps

Every file has several timestamps that record when it was accessed, modified, or changed. These are stored as part of the file's metadata.

**Access Time** (`atime`)

The last time the file was read or accessed. For example, viewing a file with `cat file.txt` updates its atime.

**Modification Time** (`mtime`)

The last time the file's **content** was modified. For example, editing and saving a file updates its mtime. Build tools like `make` use mtime to decide if a file needs to be rebuilt.

**Change Time** (`ctime`)

The last time the file's **metadata** (permissions, ownership) or content was changed. For example, running `chmod` on a file updates its ctime.

> **Note:** Modifying content updates both `mtime` and `ctime`. Changing only metadata (e.g., permissions) updates `ctime` but not `mtime`.

**Creation Time** (`crtime` or `btime`)

The time when the file was originally created. This is supported by most modern filesystems:

- **NTFS** — always records creation time
- **ext4** — stores `crtime` in the inode (exposed via `statx()` since Linux kernel 4.11)
- **APFS** — always records creation time
- **FAT** — stores creation time in directory entries

> On older Linux systems, `stat` may show `Birth: -` even on ext4. This is a tooling limitation (requires coreutils ≥ 8.31 and kernel ≥ 4.11), not a filesystem limitation.

### `stat` Command

The `stat` command displays detailed metadata about a file, including all timestamps.

**Example walkthrough:**

Create an empty file:

```
$ touch test.txt
$ stat test.txt
```

The timestamps are initially identical:

```
  Access: 2024-12-22 16:46:35.263265515 -0800
  Modify: 2024-12-22 16:46:35.263265515 -0800
  Change: 2024-12-22 16:46:35.263265515 -0800
   Birth: 2024-12-22 16:46:35.263265515 -0800
```

Read the file — `atime` is updated:

```
$ cat test.txt
$ stat test.txt
```

```
  Access: 2024-12-22 16:47:13.404425170 -0800   ← updated
  Modify: 2024-12-22 16:46:35.263265515 -0800
  Change: 2024-12-22 16:46:35.263265515 -0800
   Birth: 2024-12-22 16:46:35.263265515 -0800
```

Change permissions — `ctime` is updated:

```
$ chmod 700 test.txt
$ stat test.txt
```

```
  Access: 2024-12-22 16:47:13.404425170 -0800
  Modify: 2024-12-22 16:46:35.263265515 -0800
  Change: 2024-12-22 16:48:54.723108339 -0800   ← updated
   Birth: 2024-12-22 16:46:35.263265515 -0800
```

Append content — both `mtime` and `ctime` are updated:

```
$ echo "Hello World" >> test.txt
$ stat test.txt
```

```
  Access: 2024-12-22 16:47:13.404425170 -0800
  Modify: 2024-12-22 16:53:10.647880884 -0800   ← updated
  Change: 2024-12-22 16:53:10.647880884 -0800   ← updated
   Birth: 2024-12-22 16:46:35.263265515 -0800
```


## File Ownership

In Linux/Unix, every file has an **owner** (the user who created it) and a **group** (a collection of users). Ownership determines who has control over a file:

- The **owner** has the most control, including the ability to modify permissions.
- The **group** allows multiple users to share access, making collaboration easier.

Use `ls -l` to see ownership:

```
$ ls -l color.jpg
-rwxr-xr-- 1 maniam developers 0 Jan 20 11:16 color.jpg
                ↑        ↑
              owner    group
```

To change ownership, use `chown` (requires root or file ownership):

```
$ chown alice color.jpg               # change owner
$ chown alice:developers color.jpg    # change owner and group
$ chown :staff color.jpg              # change group only
```

## File Permission

Each file and directory has three types of permissions:

| Permission | Symbol | Files | Directories |
|------------|--------|-------|-------------|
| Read       | `r`    | View file content | List directory contents |
| Write      | `w`    | Modify file content | Add, delete, or rename files in directory |
| Execute    | `x`    | Run as program/script | Enter (navigate into) the directory |

These permissions are assigned to three categories:

- **Owner** (user) — the person who owns the file
- **Group** — a set of users granted the same access level
- **Others** — all other users on the system

### Permission Notation

Permissions can be represented **symbolically** or **numerically**.

#### Symbolic Notation

When you run `ls -l`, permissions are displayed as a 10-character string:

```
$ ls -l
-rwxr-xr-- 1 maniam maniam 0 Jan 20 11:16 color.jpg
```

| Position        | Meaning |
|-----------------|---------|
| 1st character   | File type (`-` file, `d` directory, `l` symlink) |
| Characters 2–4  | Owner permissions (`rwx`) |
| Characters 5–7  | Group permissions (`r-x`) |
| Characters 8–10 | Others permissions (`r--`) |

A `-` in any position means that permission is not granted.

#### Numeric (Octal) Notation

Each permission type (read, write, execute) is represented as a binary bit — `1` means granted, `0` means denied. The three bits form an octal digit:

| rwx | Octal | Permission              |
|-----|-------|-------------------------|
| 111 | 7     | read, write and execute |
| 110 | 6     | read and write          |
| 101 | 5     | read and execute        |
| 100 | 4     | read only               |
| 011 | 3     | write and execute       |
| 010 | 2     | write only              |
| 001 | 1     | execute only            |
| 000 | 0     | none                    |

For example, `rwxr-xr--` = **754** (owner: 7, group: 5, others: 4).

> **Tip:** `ls -l` only shows symbolic notation. To see the numeric value, use `stat`:
>
> ```
> $ stat -c '%a %n' color.jpg
> 754 color.jpg
> ```

### `umask`

When a new file or directory is created, Linux applies a **default base permission** and then subtracts permissions according to the `umask` value:

```
final_permissions = base_permissions - umask
```

The default base permissions are:

- **666** (rw-rw-rw-) for files
- **777** (rwxrwxrwx) for directories

To check the current umask:

```
$ umask
0022
```

With a umask of `022`:

|             | Base | – umask | = Result | Symbolic |
|-------------|------|---------|----------|----------|
| Files       | 666  | – 022   | **644**  | rw-r--r-- |
| Directories | 777  | – 022   | **755**  | rwxr-xr-x |

This means new files are readable by everyone but writable only by the owner, and new directories allow everyone to read and enter but only the owner can add or remove files.

> **Root vs. non-root:** The default umask for regular users is typically `0022`. For root, it is often `0027` (more restrictive), ensuring system files are not accessible to unauthorized users.

### Common Permission Examples

| Numeric | Symbolic    | Use Case |
|---------|-------------|----------|
| 777     | `rwxrwxrwx` | Full access for everyone (use cautiously) |
| 755     | `rwxr-xr-x` | Executable files/directories: owner has full access, others can read and execute |
| 644     | `rw-r--r--` | Text/config files: owner can edit, others can only read |
| 700     | `rwx------` | Private: only owner has access |
| 600     | `rw-------` | Private file (no execute): owner can read/write only |
| 750     | `rwxr-x---` | Team access: owner has full, group can read/execute, others blocked |
| 664     | `rw-rw-r--` | Shared editable: owner and group can edit, others read-only |
| 444     | `r--r--r--` | Read-only for everyone |

### Changing Permissions (`chmod`)

Use `chmod` to change file or directory permissions. It supports both symbolic and numeric notation:

```
$ chmod 755 script.sh           # numeric: rwxr-xr-x
$ chmod u+x script.sh           # symbolic: add execute for owner
```

Symbolic notation uses **who** + **operator** + **permission**:

| Who | Meaning |
|-----|---------|
| `u` | Owner (user) |
| `g` | Group |
| `o` | Others |
| `a` | All (owner + group + others) |

| Operator | Meaning |
|----------|---------|
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exact permission |

Examples:

```
$ chmod g+w report.txt          # add write for group
$ chmod o-rx config.yaml        # remove read and execute for others
$ chmod a=r README.md           # set read-only for everyone
$ chmod u=rwx,g=rx,o= app.sh    # owner: rwx, group: rx, others: none
```

To apply permissions recursively to a directory and its contents:

```
$ chmod -R 755 /var/www/html
```

## Special Mode Bits

Beyond standard permissions, Linux has three **special mode bits** that enable additional behaviors:

### Sticky Bit (octal: 1)

When set on a directory, only the **file owner**, **directory owner**, or **root** can delete or rename files — even if other users have write access. This prevents users from deleting each other's files in shared directories.

The most common example is `/tmp`:

```
$ ls -ld /tmp
drwxrwxrwt 24 root root 16384 Jan 20 23:16 /tmp
```

The `t` at the end indicates the sticky bit is set. To set it:

```
$ chmod +t /my_shared_dir
$ chmod 1777 /my_shared_dir    # numeric equivalent
```

**Example:** Even with 777 permissions on `/tmp`, user `nima` cannot delete user `maniam`'s file:

```
$ touch /tmp/myfile
$ chmod 777 /tmp/myfile
$ su nima
$ rm /tmp/myfile
rm: cannot remove '/tmp/myfile': Operation not permitted
```

### Set User ID — SUID (octal: 4)

When set on an executable file, it runs with the **permissions of the file's owner** rather than the user executing it. A common example is `passwd`, which needs root access to modify `/etc/shadow`:

```
$ ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 68208 Jan 20 10:00 /usr/bin/passwd
```

The `s` in the owner's execute position indicates SUID is set.

### Set Group ID — SGID (octal: 2)

- **On files:** the program runs with the permissions of the file's **group**.
- **On directories:** new files created inside inherit the **directory's group** (instead of the creator's primary group), which is useful for team collaboration.

### Special Bits Summary

| Octal | Bits Set |
|-------|----------|
| 0     | None |
| 1     | Sticky Bit |
| 2     | SGID |
| 4     | SUID |
| 7     | All (SUID + SGID + Sticky) |

These are specified as a fourth digit prepended to the normal three-digit permission (e.g., `1777` = sticky + rwxrwxrwx).

[← Previous: Disk Tools](09_disk_tools.md) | [Next: FAT Filesystem →](11_fat_filesystem.md)
