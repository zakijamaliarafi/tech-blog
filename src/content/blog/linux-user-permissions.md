---
title: 'Understanding Linux User Permissions'
description: 'A comprehensive guide to file permissions, ownership, and the chmod command.'
pubDate: 'Apr 26 2026'
---

Linux is inherently a multi-user operating system. To ensure security and privacy, it employs a strict permissions system to control who can access and modify files and directories.

## The Three Permission Types

Every file and directory in Linux has three basic permissions:
1.  **Read (`r`):** Allows viewing the contents of a file or listing the contents of a directory.
2.  **Write (`w`):** Allows modifying a file or creating/deleting files within a directory.
3.  **Execute (`x`):** Allows running a file as a program or entering a directory.

## The Three User Categories

These permissions are applied to three distinct categories of users:
1.  **User (Owner):** The person who owns the file.
2.  **Group:** Other users who are members of the file's group.
3.  **Others:** Everyone else on the system.

## Reading the Permissions

When you run `ls -l`, you'll see a string like `-rwxr-xr--` at the beginning of the line. Here is how to break it down:
*   The first character indicates the file type (`-` for a regular file, `d` for a directory).
*   The next three characters (`rwx`) are the Owner permissions (Read, Write, Execute).
*   The middle three characters (`r-x`) are the Group permissions (Read, Execute, but no Write).
*   The last three characters (`r--`) are the Others permissions (Read only).

## Modifying Permissions with `chmod`

You can change permissions using the `chmod` command. There are two primary ways to use it:

### 1. Symbolic Mode
This uses letters to add (`+`), remove (`-`), or set (`=`) permissions.
*   Make a script executable for the owner: `chmod u+x script.sh`
*   Remove write access for others: `chmod o-w document.txt`

### 2. Numeric (Octal) Mode
This uses numbers to represent permissions: Read = 4, Write = 2, Execute = 1. You add these numbers together for each user category.
*   `7` = 4+2+1 (Read, Write, Execute)
*   `6` = 4+2 (Read, Write)
*   `5` = 4+1 (Read, Execute)

Example: To give the owner full permissions (`7`), the group read/execute (`5`), and others read-only (`4`), you would use:
```bash
chmod 754 filename
```

Understanding how to read and modify permissions is crucial for system security and proper application deployment.
