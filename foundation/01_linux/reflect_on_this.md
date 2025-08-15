# Reflect on This — 01_linux

## 1) Getting Help
Linux offers **three complementary help surfaces** because they target different layers:

- **`man <topic>`** — system manual pages installed on your machine. Best for **full reference docs**: options, exit codes, examples. Works for most programs (e.g., `man grep`).
- **`<cmd> --help`** — **quick, program-owned usage** message. Fast, always ships with the binary, great for **option discovery and syntax reminders**.
- **`help <builtin>`** — help for **shell builtins** (things implemented *inside* your shell, not separate executables). Examples: `cd`, `alias`, `pushd`, `history`.

**When is `help` the only correct choice?**  
When the “command” is a **shell builtin** (e.g., `cd`, `jobs`, `bg`, `fg`, `alias`, `[[ ... ]]`).  
`man cd` often won’t describe the Bash builtin behavior, and `cd --help` doesn’t work (it’s not an external program).  
Use: `help cd`.

**Example commands to run for screenshots:**
```bash
help cd                 # +++
man grep                # +++
find --help             # +++
```

## 2) Searching Files

**Key difference:**

* `ls | grep ".txt"`

  * Only sees the **current directory** listing (not recursive).
  * Parses **formatted output** meant for humans (can break with spaces, colors, columns).
  * The pattern `".txt"` in grep is a **regex** (dot means “any char”), and it can match unintended names.
  * General rule: **don’t parse `ls`**.

* `find . -name "*.txt"`

  * **Recursive**, works across directories reliably.
  * Matches **filenames** directly (no formatting issues).
  * **Far more powerful**: add filters like `-type f`, `-mtime -7`, `-size +1M`, `-exec`.

**When is one more powerful?**

* Need recursion, filters, exact filename matching, or actions → **`find` wins**.
* Quick eyeball of files in the current dir only → `ls | grep` can be “good enough,” but `find` is still safer.

**Example commands to run for screenshots:**

```bash
ls -1 | grep -E '\.txt$'              # +++ (non-recursive, current dir only)
find . -type f -name '*.txt'          # +++ (recursive, robust)
find . -type f -name '*.txt' -mtime -7 -size +1M  # +++ (power filters)
```


## 3) Piping to `grep` (with `history`)

The same concept as `ps aux | grep chrome`, but with `history`:

* **Simple text search in your history:**

```bash
history | grep -i 'docker build'      # +++
```

* **Filter by time (last 7 days) if your shell shows timestamps**
  In Bash, enable timestamps temporarily and then filter by date:

```bash
since=$(date -d '7 days ago' +%F)     # +++
HISTTIMEFORMAT='%F %T ' history \
  | awk -v s="$since" '$2 >= s' \
  | grep -i 'docker build'            # +++
```

> 💡 Tip: Add
>
> ```bash
> export HISTTIMEFORMAT='%F %T '
> ```
>
> to your shell profile so timestamps are always saved.

---

## 4) Archiving (`.tar.gz` vs `.zip`) and Safety

**Why choose `.tar.gz` on Linux?**

* **Preserves Unix metadata** better (permissions, ownership, symlinks, hard links, device nodes).
* Typically yields **smaller archives** with `gzip` or `zstd`.
* Ubiquitous on Linux; streams nicely (e.g., `tar -czf - | ssh ...`).

**Security risk before extracting an archive from an unknown source:**

* **Path traversal / overwrite** (“Zip Slip”): files named like `../../.bashrc` or absolute paths may overwrite critical files.
* **Mitigations:**

  1. **List first**, then extract:

     ```bash
     tar -tzf project.tar.gz | head     # +++
     ```
  2. **Extract into a new dir** and **don’t keep ownership**:

     ```bash
     mkdir -p safe_extract && \
       tar -xzf project.tar.gz -C safe_extract --no-same-owner  # +++
     ```
  3. Avoid running any scripts you didn’t inspect inside the archive.

**Example commands to create & compare:**

```bash
tar -czf project.tar.gz project_dir    # +++
zip -r project.zip project_dir         # +++
```

