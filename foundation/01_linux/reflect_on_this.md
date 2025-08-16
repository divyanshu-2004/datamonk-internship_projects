# Reflect on This – Linux Permissions

This document contains reflections on the security and practical applications of Linux file and directory permissions.

---

## 1. File vs. Directory Permissions  
The **execute (x)** permission has different meanings depending on whether it is applied to a file or a directory:  

- **For files**: The execute permission allows the file to be run as a program or script. Without it, the file can be read or written (depending on other permissions), but it cannot be executed.  
- **For directories**: The execute permission allows a user to **enter and navigate** inside the directory. Without execute permission, even if you have read permission on the directory, you cannot list or access files inside it.  

👉 This makes execute essential for directories because without it, you can’t move into the directory or access its contents, even if you know the filenames.

---

## 2. The 777 Risk  
Granting `777` permissions means **any user** on the system can read, write, and execute the file or directory.  

- **Scenario on a web server:**  
  Suppose a PHP configuration file (`config.php`) that stores database credentials is given `777` permissions.  
  - An attacker could upload a malicious PHP file to the server.  
  - Since the directory and file are world-writable, the attacker could modify `config.php` to execute arbitrary commands.  
  - This could lead to full compromise of the database and possibly the entire website/server.  

👉 Thus, `777` should be avoided because it removes all security boundaries.

---

## 3. Symbolic vs. Octal `chmod`  
Imagine a file with the following permissions:  
-rwx-w-r--


Now you only want to **add group execute permission** without changing any other permissions.  

- If you try using **octal mode**, you need to recalculate the exact numeric representation for the updated permissions, which can be error-prone and could accidentally overwrite other permissions.  
- If you use **symbolic mode**:  

```bash
chmod g+x a_file.txt
```

This safely adds execute permission for the group only, without touching other bits.

👉 That’s why symbolic mode is safer and better for making small, precise permission changes.

# 4. sudo’s Power

Suppose you meant to run:
```bash
rm -rf ./temp_files/
```

But instead you accidentally typed:
```bash
sudo rm -rf / temp_files/
```

Here’s what happens:

 - sudo elevates the command to superuser privileges, bypassing normal safety checks.

The command is interpreted as:

- rm -rf / → recursively deletes everything from the root directory /, effectively wiping out the entire operating system.

- temp_files/ would then be treated as a separate target, but by that point the system is already destroyed.

👉 With sudo, this mistake becomes catastrophic because the command has the power to delete all system files and data irreversibly.

