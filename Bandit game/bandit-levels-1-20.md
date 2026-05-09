# OverTheWire Bandit — Levels 0 through 20

These are my notes from working through the first twenty Bandit levels on OverTheWire. I tried to keep the explanations honest about what tripped me up, what surprised me, and what I'd do differently next time. Connecting in is always the same drill:

```bash
ssh banditN@bandit.labs.overthewire.org -p 2220
```

Where `N` is the level number. Each level hands you the password for the next one once you find the right file or pull off the right trick.

---

## Bandit Level 0

**Challenge:** Just log in. The password is literally `bandit0`.

**Solution:**

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

**Explanation:**
- `ssh` opens a Secure Shell connection to a remote host.
- `-p 2220` tells it to use port 2220 instead of the default port 22.

**Password:** `bandit0`

**What I learned:** Mostly just a sanity check that SSH works. Worth noting that you'll be typing this command twenty more times, so if you're on a real machine it's worth setting up an SSH config entry to save your fingers.

---

## Bandit Level 0 → Level 1

**Challenge:** The password for the next level is in a file called `readme` in the home directory.

**Solution:**

```bash
ls
cat readme
```

**Explanation:**
- `ls` lists what's in the current directory.
- `cat readme` dumps the file contents to the terminal.

**Password:** `NH2SXQwcBdpmTEzi3bvBHMM9H66vVXjL`

**What I learned:** Easy warmup, but it does set the rhythm — always start with `ls` to see what you're working with before reaching for anything fancier.

---

## Bandit Level 1 → Level 2

**Challenge:** The password is in a file called `-` in the home directory.

**Solution:**

```bash
cat ./-
```

**Explanation:**
- A filename starting with `-` confuses most commands, since they think you're passing a flag.
- Prefixing the path with `./` makes it explicit that you're talking about a file in the current directory, not an option.

**Password:** `rRGizSaX8Mk1RTb1CNQoXTcYZWU6lgzi`

**What I learned:** This was my first "huh, the shell is weirder than I thought" moment. `cat -` actually means "read from stdin", which is a fun footgun. You can also use `cat < -` to sidestep it.

---

## Bandit Level 2 → Level 3

**Challenge:** The password is in a file called `spaces in this filename`.

**Solution:**

```bash
cat "spaces in this filename"
```

**Explanation:**
- Wrapping the filename in quotes tells the shell to treat the spaces as part of the name instead of as separators between arguments.
- Backslash-escaping each space (`cat spaces\ in\ this\ filename`) works just as well.

**Password:** `aBZ0W5EmUfAf7kHTQeOwd8bauFJ2lAiG`

**What I learned:** Tab completion is your friend here — typing `cat sp` and hitting Tab will auto-escape the spaces for you. Saves a lot of typos.

---

## Bandit Level 3 → Level 4

**Challenge:** The password is in a hidden file in the `inhere` directory.

**Solution:**

```bash
cd inhere
ls -la
cat ...Hiding-From-You
```

**Explanation:**
- `ls -la` lists everything including hidden files (the `-a` flag) in long format (`-l`).
- Hidden files in Linux start with a dot, which `ls` skips unless you ask for them.

**Password:** `2EW7BBsr6aMMoJ2HjW067dm8EgX26xNe`

**What I learned:** The filename was three dots followed by a name, which is sneaky — `..` means "parent directory", so something starting with `...` looks confusingly similar at a glance.

---

## Bandit Level 4 → Level 5

**Challenge:** The password is in the only human-readable file in the `inhere` directory.

**Solution:**

```bash
cd inhere
file ./*
cat ./-file07
```

**Explanation:**
- `file ./*` runs the `file` command on every entry, which inspects the contents and tells you what type of file each one is.
- The one labeled `ASCII text` is the human-readable one.
- Note the `./` again — these files all start with `-`, same trick as Level 1.

**Password:** `lrIWWI6bB37kxfiCQZqUdOIYfr6eEeqR`

**What I learned:** `file` is one of those tools I forget exists until I need it. It's much more reliable than guessing from the extension (which usually doesn't even exist on these challenges).

---

## Bandit Level 5 → Level 6

**Challenge:** Find a file in `inhere` that is human-readable, exactly 1033 bytes, and not executable.

**Solution:**

```bash
find . -type f -size 1033c ! -executable
cat ./maybehere07/.file2
```

**Explanation:**
- `find .` searches the current directory tree.
- `-type f` restricts results to regular files.
- `-size 1033c` matches files of exactly 1033 bytes (the `c` suffix means bytes; without it `find` interprets the number as 512-byte blocks).
- `! -executable` negates the executable test, so you only get non-executable files.

**Password:** `P4L4vucdmLnm8I7Vl7jG1ApGSfjYKqJU`

**What I learned:** `find` is honestly the workhorse of this whole game. Once you internalize the `-type`, `-size`, `-user`, `-group`, and `-perm` flags, half of the levels become trivial.

---

## Bandit Level 6 → Level 7

**Challenge:** The password is somewhere on the server, in a file owned by user `bandit7`, owned by group `bandit6`, and 33 bytes in size.

**Solution:**

```bash
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

**Explanation:**
- Searching from `/` walks the entire filesystem.
- `-user` and `-group` filter by ownership.
- `2>/dev/null` throws away the noisy "Permission denied" errors so you can actually see your results.

**Password:** `z7WtoNQU2XfjmMtWA8u5rN4vzqu4v99S`

**What I learned:** Without `2>/dev/null` the output is unreadable. Also, redirecting `stderr` (`2>`) versus `stdout` (`1>` or just `>`) is one of those distinctions that's worth nailing down early.

---

## Bandit Level 7 → Level 8

**Challenge:** The password is in `data.txt` next to the word `millionth`.

**Solution:**

```bash
grep millionth data.txt
```

**Explanation:**
- `grep` searches for a pattern in a file and prints any lines that match.

**Password:** `TESKZC0XvTetK0S9xNwm25STk5iWrBvP`

**What I learned:** First grep level, and it's a softball, but the pattern of "huge file + one keyword" comes up constantly in real sysadmin work.

---

## Bandit Level 8 → Level 9

**Challenge:** The password is the only line in `data.txt` that occurs exactly once.

**Solution:**

```bash
sort data.txt | uniq -u
```

**Explanation:**
- `sort` orders the lines alphabetically — important because `uniq` only spots duplicates that are next to each other.
- `uniq -u` prints only the lines that appear exactly once.

**Password:** `EN632PlfYiZbn3PhVK3XOGSlNInNE00t`

**What I learned:** This is the one that finally drove home why `sort | uniq` is such a common pairing. `uniq` on its own would have missed it entirely if matching duplicates weren't already adjacent.

---

## Bandit Level 9 → Level 10

**Challenge:** The password is in `data.txt` in one of the few human-readable strings, preceded by several `=` characters.

**Solution:**

```bash
strings data.txt | grep "^="
```

**Explanation:**
- `strings` pulls printable ASCII sequences out of binary files.
- `grep "^="` keeps only lines that start with `=` (the caret anchors to the beginning of the line).

**Password:** `G7w8LIi6J3kTb8A7j9LgrywtEUlyyp6s`

**What I learned:** `strings` is great for sneaking text out of any binary blob — executables, memory dumps, you name it. The output had a bunch of garbage `=` decorations, so I had to skim before grabbing the right line.

---

## Bandit Level 10 → Level 11

**Challenge:** The password is in `data.txt`, which contains base64-encoded data.

**Solution:**

```bash
base64 -d data.txt
```

**Explanation:**
- `base64 -d` decodes base64 input. The `-d` flag is short for `--decode`.

**Password:** `6zPeziLdR2RKNdNYFNb6nVCKzphlXHBM`

**What I learned:** Worth memorizing the look of base64 — letters, digits, `+`, `/`, padded with `=`. You'll see it everywhere in JWTs, email attachments, and config files.

---

## Bandit Level 11 → Level 12

**Challenge:** The password is in `data.txt`, but every letter has been ROT13-shifted.

**Solution:**

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**Explanation:**
- `tr` translates one set of characters into another.
- The mapping shifts each letter forward 13 places, which is exactly what ROT13 does. Since the alphabet has 26 letters, applying ROT13 twice gets you back to the original — encoding and decoding are the same operation.

**Password:** `JVNBBFSmZwKKOP0XbFXOoW8chDz5yVRv`

**What I learned:** ROT13 is more of a curiosity than a real cipher, but it shows up in puzzles, spoiler tags, and old Usenet posts. Also the first time I really used `tr` for something nontrivial.

---

## Bandit Level 12 → Level 13

**Challenge:** The password is in `data.txt`, a hexdump of a file that has been compressed and recompressed multiple times.

**Solution:**

```bash
mkdir /tmp/sam12 && cp data.txt /tmp/sam12 && cd /tmp/sam12
xxd -r data.txt > data.bin
file data.bin
# Loop: rename based on type, decompress, repeat. The chain went:
#   gzip → bzip2 → gzip → tar → tar → bzip2 → gzip → ascii (final)
mv data.bin data.gz && gunzip data.gz
mv data data.bz2 && bunzip2 data.bz2
mv data data.gz && gunzip data.gz
mv data data.tar && tar -xf data.tar
mv data5.bin data.tar && tar -xf data.tar
mv data6.bin data.bz2 && bunzip2 data.bz2
mv data6 data.tar && tar -xf data.tar
mv data8.bin data.gz && gunzip data.gz
cat data8
```

**Explanation:**
- `xxd -r` reverses a hexdump back into the original binary.
- `file` identifies what flavor of compression you're dealing with at each step.
- The trick is to rename the file with the right extension before you decompress, since `gunzip` and friends are picky about what they accept.

**Password:** `wbWdlBxEir4CaE8LaPhauuOo6pwRmrDw`

**What I learned:** This is the longest level so far, and easily the most annoying. Working in `/tmp` is essential since your home directory is read-only. I also got into a rhythm of just running `file data` after every step instead of trying to remember which compression came next.

---

## Bandit Level 13 → Level 14

**Challenge:** The password is stored in `/etc/bandit_pass/bandit14`, which is only readable by `bandit14`. You're given an SSH private key to log in as `bandit14`.

**Solution:**

```bash
ssh -i sshkey.private bandit14@localhost -p 2220
cat /etc/bandit_pass/bandit14
```

**Explanation:**
- `ssh -i` specifies an identity file (a private key) instead of using a password.
- Logging into `localhost` makes a new SSH connection back to the same machine, but as a different user.

**Password:** `fGrHPx402xGC7U7rXKDaxiWFTOiF0ENq`

**What I learned:** Public-key auth in three lines. Worth noting SSH may complain about file permissions on the key — `chmod 600 sshkey.private` fixes that.

---

## Bandit Level 14 → Level 15

**Challenge:** Submit the current level's password to port 30000 on localhost to receive the next one.

**Solution:**

```bash
echo "fGrHPx402xGC7U7rXKDaxiWFTOiF0ENq" | nc localhost 30000
```

**Explanation:**
- `nc` (netcat) opens a raw TCP connection to the given host and port.
- Piping the password into it sends it as the connection's input, then prints whatever the server replies with.

**Password:** `jN2kgmIXJ6fShzhT2avhotn4Zcka6tnt`

**What I learned:** Netcat is the duct tape of networking. You can use it for ad-hoc chat, file transfer, port scanning, throwaway servers — all from one tiny binary.

---

## Bandit Level 15 → Level 16

**Challenge:** Same as the previous level, but now over SSL/TLS on port 30001.

**Solution:**

```bash
echo "jN2kgmIXJ6fShzhT2avhotn4Zcka6tnt" | openssl s_client -connect localhost:30001 -ign_eof
```

**Explanation:**
- `openssl s_client` is basically netcat for TLS — it negotiates an encrypted session and then lets you send/receive data over it.
- `-ign_eof` keeps the connection open after stdin closes, otherwise the server's response can get cut off before it arrives.

**Password:** `JQttfApK4SeyHwDlI9SXGR50qclOAil1`

**What I learned:** Forgetting `-ign_eof` had me convinced the server was broken for about ten minutes. Watch for the "DONE" line in the openssl handshake too — sometimes it gets mixed in with the actual response.

---

## Bandit Level 16 → Level 17

**Challenge:** The next password is on a server somewhere between ports 31000–32000 on localhost. Only one of those ports speaks SSL, and only that one will give you the password if you submit the current one.

**Solution:**

```bash
nmap -p 31000-32000 localhost
# Found open ports: 31046, 31518, 31691, 31790, 31960
# Check each one for SSL:
echo "JQttfApK4SeyHwDlI9SXGR50qclOAil1" | openssl s_client -connect localhost:31790 -ign_eof
```

**Explanation:**
- `nmap` scans the given port range and reports which ones are open.
- For each open port I tried `openssl s_client` until one returned an SSH private key instead of "Wrong! Please enter the correct current password."

**Password:** Returned as an SSH private key, used to connect to bandit17.

**What I learned:** This was the first level that genuinely felt like recon. `nmap -sV` would have told me which ports were SSL straight away, but it's good practice to do it the manual way once.

---

## Bandit Level 17 → Level 18

**Challenge:** You log in as bandit17 using the key from the previous level. There are two files, `passwords.old` and `passwords.new`. The new password is the only line in `passwords.new` that differs from `passwords.old`.

**Solution:**

```bash
ssh -i bandit17.key bandit17@bandit.labs.overthewire.org -p 2220
diff passwords.old passwords.new
```

**Explanation:**
- `diff` compares two files line by line and prints what's different.
- The lines prefixed with `>` come from the second file (the new one), so that's where the new password lives.

**Password:** `hga5tuuCLF6fFzUpnagiMN8ssu9LFrdg`

**What I learned:** `diff -u` gives you the unified-diff format, which is what tools like `git` use under the hood. Worth getting comfortable with.

---

## Bandit Level 18 → Level 19

**Challenge:** Someone modified `.bashrc` to immediately log you out when you SSH in as bandit18. The password lives in `readme` in the home directory.

**Solution:**

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```

**Explanation:**
- If you give SSH a command after the host, it runs that command non-interactively and never starts a login shell.
- That sidesteps the booby-trapped `.bashrc` entirely.

**Password:** `awhqfNnAbc1naukrpqDYcF95h7HoMTrC`

**What I learned:** This is a very real technique for recovering broken accounts on actual servers. You can also pass `-T` to skip pty allocation, or `bash --noprofile --norc` once you're in.

---

## Bandit Level 19 → Level 20

**Challenge:** A setuid binary in the home directory lets you run a single command as another user. Use it to read the next password.

**Solution:**

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

**Explanation:**
- A setuid binary runs with the privileges of its owner instead of whoever started it. You can spot them with `ls -l` — there's an `s` where the user's `x` bit normally goes.
- `bandit20-do` is owned by `bandit20`, so anything you run through it executes as `bandit20`.

**Password:** `VxCazJaVykI6W36BkBU0mJTCM8rR95XT`

**What I learned:** Setuid is one of the classic privilege-escalation paths in Linux security. If you ever audit a system, `find / -perm -4000 2>/dev/null` to list all setuid binaries is one of the first things you should run.

---

## Bandit Level 20 → Level 21

**Challenge:** A setuid binary called `suconnect` will connect back to you on a port you choose. If you send it the current level's password, it'll reply with the next one.

**Solution:**

```bash
# Terminal 1 — start a listener that sends the current password and prints the response
nc -lvp 4242 < /tmp/sam20/pass.txt

# Terminal 2 — point suconnect at that listener
./suconnect 4242
```

**Explanation:**
- `nc -lvp 4242` listens on port 4242 in verbose mode. The redirected file gets sent as soon as something connects.
- `suconnect` opens a TCP connection to localhost on the port you give it, sends the level 20 password, reads back what's on the other end, and if it matches, prints the level 21 password.

**Password:** `NvEJF7oVjkddltPSrdKEFOllh9V1IBcq`

**What I learned:** First level that needed two terminals and a bit of choreography. The order matters — start the listener first, then run `suconnect`. Otherwise `suconnect` connects to nothing and exits.

---

## Closing notes

A few patterns that came up over and over:

- `find` plus a couple of filters solves a startling number of problems.
- `file` is the right answer any time you're staring at a mystery blob.
- `/tmp` is the only writable place when you're a low-privilege user — get used to making yourself a working directory there.
- Always redirect `stderr` to `/dev/null` when walking the whole filesystem; otherwise the noise hides the signal.
- Read the level description carefully. Most "I'm stuck" moments turned out to be me skimming past a constraint that mattered.

I'd recommend keeping a personal cheat sheet of these commands as you go. By Level 20 the muscle memory really starts to pay off.
