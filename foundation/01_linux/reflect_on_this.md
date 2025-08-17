# 01 – Linux Permissions: Reflection + Secure Shared Project

## Reflect on This

### 1) File vs. Directory Execute (x)
- **On a file**: `x` means “allow this binary/script to run.”
- **On a directory**: `x` means **traverse**—you may enter the directory and access entries by name.  
  You can’t `cd` into a directory or open files **inside** it without directory `x`, even if those files are world-readable. This prevents someone from walking into locations they shouldn’t, while still allowing read access where traversal is permitted.

---

### 2) The 777 Risk (web server scenario)
Imagine a PHP web app where `uploads/` contains `avatar.php` with **`777`**:
- An attacker uploads or edits `avatar.php` to contain malicious PHP (web shell).
- Because it’s `rwxrwxrwx`, the server and any user can write/execute it.
- The attacker then browses to `https://site/uploads/avatar.php`, executes arbitrary commands (e.g., reading DB creds, writing new files, defacing the site, pivoting to the server OS).  
**One over-permissive file** becomes a remote code execution foothold.

---

### 3) Symbolic vs. Octal `chmod`
Given `-rwx-w-r--` and you only want to **add group execute**:
- With **symbolic**: `chmod g+x a_file.txt` precisely adds one bit and **doesn’t touch** anything else.
- With **octal**, you must first decode the current mode, then re-encode it correctly; it’s easy to flip unintended bits.  
Symbolic changes are **safer and less error-prone** for targeted adjustments.

---

### 4) `sudo` + typoed `rm -rf`
You intended: `rm -rf ./temp_files/`  
You typed: `sudo rm -rf / temp_files/` (notice the space after `/`)
- That command attempts to recursively delete **the filesystem root `/`** (catastrophic), then `temp_files/`.
- With `sudo`, you bypass normal protections, so you can nuke critical system files, break the OS, and lose data.  
This is why you use `sudo` sparingly and double-check destructive commands.

---

### 5) Ownership for Collaboration
`sudo chown -R :developers project/` sets the **group** of the project to `developers`. Then you grant `g+rwX` on directories/files:
- **Principle of least privilege**: limit access to just the team.
- **Scalability**: add/remove teammates by editing group membership, not by changing file perms.
- **Auditable & clean**: keeps “others” at `---`, reducing accidental exposure.

---

## Project: Secure Shared Project Directory

**Goal**: Owner has full control, a specific group can contribute, **others have no access**.

> ⚠️ Note on correctness: Group members **must have `x` on directories** to enter and create files. A blanket `760` (group no `x`) on directories will **block** teammates from `cd` and from creating files. Below is a corrected, secure setup that satisfies the goal **and** the test steps.

### Step 1: Create a Teammate User

```bash
# Create teammate
sudo useradd -m dev_user
sudo passwd dev_user
```
