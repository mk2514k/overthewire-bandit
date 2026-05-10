# OverTheWire-Bandit (Write-ups_

**Platform:** [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)  
**Purpose:** Bandit is a beginner-friendly wargame designed to teach the Linux command line through hands-on terminal puzzles. Each level presents a real problem — no multiple choice, no theory — just you, a terminal, and a challenge to solve.

These write-ups document my methodology, thought process, and key learnings at each level. Passwords are intentionally omitted.

---

## Level 0: Connecting via SSH

**Objective:** Connect to the Bandit server using SSH.

**What I did:**  
The first level isn't really a puzzle — it's an introduction to SSH (Secure Shell), the protocol used to connect to remote machines securely. Using the credentials provided on the OTW website, I opened a terminal and connected directly to the Bandit server.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

**What I learned:**  
SSH is fundamental to everything in Linux security. The `-p` flag specifies a non-standard port (2220 here instead of the default 22). Every level from here uses SSH to log in with the password found in the previous level.

---

## Level 0 → 1: Reading a File in the Home Directory

**Objective:** Find the password stored in a file in the home directory.

**What I did:**  
After logging in, I listed the contents of the home directory and found a file called `readme`. I used `cat` to read its contents directly.

```bash
cat readme
```

**What I learned:**  
`cat` outputs the contents of a file to the terminal. The home directory is where you land by default when you log in — `ls` lists what's there. Simple combination, but it's the foundation of navigating any Linux system.

---

## Level 1 → 2: Reading a Dashed Filename

**Objective:** The password is stored in a file called `-`.

**What I did:**  
Running `cat -` doesn't work — the shell interprets `-` as a special argument meaning "read from standard input" rather than a filename. To get around this, I specified the file path explicitly using `./` to tell the shell it's a file in the current directory, not a flag.

```bash
cat ./-
```

**What I learned:**  
The shell treats `-` as a special symbol. Prefixing with `./` forces it to be read as a literal file path. This comes up regularly with unusual filenames in security work — you can't always trust that a filename will behave the way you expect.

---

## Level 2 → 3: Filename with Spaces

**Objective:** The password is stored in a file called `spaces in this filename`.

**What I did:**  
If you type `cat spaces in this filename`, the shell reads each word as a separate argument and tries to open four different files. To treat the whole thing as one filename, I wrapped it in double quotes.

```bash
cat "spaces in this filename"
```

**What I learned:**  
Spaces in filenames break commands unless you handle them correctly. Quoting is one solution; another is escaping each space with a backslash (`cat spaces\ in\ this\ filename`). Both work — quoting is cleaner to read.

---

## Level 3 → 4: Finding a Hidden File

**Objective:** The password is stored in a hidden file inside a directory called `inhere`.

**What I did:**  
I was given the directory name `inhere`. I used `find` to locate the file path, then `cat` to read it.

```bash
find inhere
cat inhere/.hidden
```

**What I learned:**  
In Linux, files beginning with a dot (`.`) are hidden — `ls` won't show them by default. `ls -a` reveals them, and `find` will also surface them. Hidden files are commonly used for configuration and are worth knowing how to locate — a useful habit in both sysadmin and security contexts.

---

## Level 4 → 5: Finding the Human-Readable File

**Objective:** Find the only human-readable file inside the `inhere` directory (which contains multiple files).

**What I did:**  
Rather than opening each file one by one, I used the `file` command with a wildcard to inspect all files at once, then filtered for the one containing readable text.

```bash
file inhere/-file0*
grep "text"
```

Breaking it down:
- `inhere/-file*` — the asterisk expands to every file inside the folder
- `file` — inspects each file and reports its type (ASCII text, data, etc.)
- `grep "text"` — filters the output to show only the human-readable result

**What I learned:**  
The `file` command is invaluable when you don't know what you're dealing with. Binary files show as "data" — ASCII or UTF-8 files show as "text". Combining it with a wildcard and grep means you can scan an entire directory in one command rather than checking files individually.

---

## Level 5 → 6: Finding a File by Properties

**Objective:** Find a file inside `inhere` that is human-readable, exactly 1033 bytes, and not executable.

**What I did:**  
This level introduced using `find` with multiple filtering flags simultaneously. Rather than checking each file manually, I used `find` to narrow the results by file type, size, and permissions in one command.

```bash
find inhere -type f -size 1033c
```

Breaking it down:
- `-type f` — only look at regular files, not directories
- `-size 1033c` — exactly 1033 bytes (`c` specifies bytes, not blocks)

This returned a single file path. I then read it with `cat`.

**What I learned:**  
`find` is far more powerful than just locating files by name. The ability to filter by size, type, permissions, owner, and modification time makes it a core tool in both Linux administration and security work. The `c` suffix for bytes is easy to forget — the default unit is 512-byte blocks, which would give a completely wrong result.

---

## Level 6 → 7: Finding a File Anywhere on the Server

**Objective:** The password is stored somewhere on the server, owned by user `bandit7`, group `bandit6`, and exactly 33 bytes in size.

**What I did:**  
This time the file could be anywhere on the system, not just in the home directory. I searched from the root (`/`) with user, group, and size filters — and added `2>/dev/null` to suppress the flood of "Permission denied" errors that come from trying to read directories I don't have access to.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

Breaking it down:
- `/` — search from the root of the entire filesystem
- `-user bandit7` — file must be owned by this user
- `-group bandit6` — file must belong to this group
- `-size 33c` — exactly 33 bytes
- `2>/dev/null` — redirects error messages (stderr) away so only valid results are shown

This returned a single file path. Used `cat` with that path to get the password.

**What I learned:**  
`2>/dev/null` is one of those things you use constantly once you know it. `2>` redirects standard error (error messages) and `/dev/null` is a special file that discards anything written to it. Without it, the output of a root-level search is almost unreadable. This was also my first time combining multiple `find` criteria together — the flags stack cleanly and the result was precise.

---

## Level 7 → 8: Searching Inside a File with grep

**Objective:** The password is stored next to the word "millionth" inside a large file called `data.txt`.

**What I did:**  
The file is too large to read manually. After reviewing the recommended commands for the level, I identified `grep` as the right tool — it searches for lines matching a pattern inside a file.

```bash
grep millionth data.txt
```

This returned a single line containing the word "millionth" followed by the password.

**What I learned:**  
`grep` is one of the most used commands in Linux. It scans a file line by line and returns any line containing your search term. In a file with thousands of lines, it reduces the result to exactly what you need instantly. The pattern here was simple (a plain word), but `grep` also supports regular expressions for more complex searches.

---

## Level 8 → 9: Finding the Unique Line

**Objective:** The password is the only line in `data.txt` that appears exactly once.

**What I did:**  
My first instinct was to use `uniq` directly on the file, but that failed — `uniq` only removes *adjacent* duplicate lines, not duplicates spread throughout the file. I needed to sort the file first so all duplicates were grouped together, then pipe into `uniq -u` to show only the lines that appear once.

```bash
sort data.txt | uniq -u
```

Breaking it down:
- `sort` — arranges all lines alphabetically, grouping identical lines together
- `|` — the pipe sends the output of `sort` directly into the next command
- `uniq -u` — outputs only lines that are not repeated

This returned a single line — the password.

I also read this after completing the level, which helped solidify how piping works:  
[Ryan's Tutorials — Piping](https://ryanstutorials.net/linuxtutorial/piping.php)

**What I learned:**  
Piping (`|`) is one of the most powerful features of the Linux command line — it chains commands together so the output of one becomes the input of the next. `uniq` alone isn't enough here; understanding *why* you need `sort` first is the real lesson. Also learned `man uniq` to read the manual for any command — `-u` is the flag for unique-only lines.

---

## Level 9 → 10: Extracting Human-Readable Strings

**Objective:** The password is stored in `data.txt` among non-readable binary data, preceded by several `=` characters.

**What I did:**  
My first attempt was `sort data.txt | grep '='` which surfaced lines containing `=` — but the file is mostly binary, so the output was a mix of readable and unreadable content. The key insight was that the password lines are preceded by `=` signs, so filtering for `=` narrows it down significantly.

```bash
sort data.txt | grep '='
```

The output showed several lines — the human-readable ones containing `========== the`, `========== password`, `========== is`, and `========== (the actual password)` stood out clearly from the noise.

**What I learned:**  
When a file contains binary data mixed with text, `grep` can still find readable patterns within it. The `=` characters acted as a marker — the problem description told me what to look for, and grep found it. A cleaner approach for future use is the `strings` command (extracts all human-readable strings from a binary file), but `grep` with a known pattern works well when you have a clear anchor to search for.

---

*Write-ups continue as levels are completed.*
