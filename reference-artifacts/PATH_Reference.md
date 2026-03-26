# The PATH Variable

**What It Is, How It Works, and How to Configure It on macOS (Bash Shell)**

V6 Career Acceleration Program | Reference Document | March 9, 2026

---

## 1. What PATH Is

PATH is an **environment variable** that tells your shell where to look for executable programs when you type a command. Since you are running **bash** on macOS Tahoe, when you type `git`, `code`, or `dotnet`, bash does not search your entire filesystem. It searches *only* the directories listed in PATH, in order, left to right.

If the executable is not in any PATH directory, bash returns: `command not found`.

## 2. How the Lookup Works

PATH is a single string of directory paths separated by colons:

```
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

When you type a command, bash executes this algorithm:

| Step | Action |
|------|--------|
| 1 | Split the PATH string on every colon. Result: an ordered list of directories. |
| 2 | Starting at the first directory, look for an executable file matching the command name. |
| 3 | If found, execute it and stop searching. If not found, move to the next directory. |
| 4 | If no directory contains the executable, return "command not found". |

> **Key implication:** Order matters. If two directories both contain a `git` executable, the one in the directory listed *first* in PATH wins. This is how Homebrew, nvm, and other tools override system defaults -- they prepend their directories to PATH.

## 3. Where PATH Gets Configured on macOS (Bash)

Since you are using **bash** (not the macOS Tahoe default of zsh), bash reads its own set of configuration files at startup. PATH can be set in any of them:

| File | When It Loads | Use Case |
|------|---------------|----------|
| `/etc/paths` | System boot (all users, all shells) | System-wide base PATH. Do not edit. |
| `/etc/paths.d/*` | System boot (all users, all shells) | System tools add entries here. |
| `~/.bash_profile` | Login shells only | **Primary config file for macOS bash users. Put PATH edits here.** |
| `~/.bashrc` | Non-login interactive shells | Aliases, functions, prompt config. Source from ~/.bash_profile. |
| `~/.profile` | Login shells (fallback) | Only read if ~/.bash_profile does not exist. |

> **Important bash behavior on macOS:** macOS Terminal opens *login* shells by default. This means `~/.bash_profile` runs every time you open a new Terminal window. Unlike Linux (where terminal emulators typically open non-login shells that read `~/.bashrc`), macOS reads `~/.bash_profile`.

Best practice: put your PATH exports in `~/.bash_profile`, and add this line to source bashrc from it:

```bash
# Inside ~/.bash_profile -- load bashrc if it exists
if [ -f ~/.bashrc ]; then source ~/.bashrc; fi
```

## 4. The Syntax

**Appending to PATH (add to end):**

```bash
export PATH="$PATH:/new/directory/here"
```

**Prepending to PATH (add to beginning -- takes priority):**

```bash
export PATH="/new/directory/here:$PATH"
```

`export` makes the variable available to child processes (programs launched from your shell). Without `export`, the variable exists only in the current shell session. `$PATH` references the current value, so you are extending the existing list, not replacing it.

> **Danger:** Writing `export PATH="/new/directory"` without including `$PATH` wipes your entire PATH. Your shell will lose access to basic commands like `ls`, `git`, and `grep`. Always include `$PATH` in the assignment.

## 5. Adding VSCode to PATH

**Option A -- Built-in command (recommended):**

Open VSCode. Press `Cmd+Shift+P` to open the Command Palette. Type `shell command` and select **"Shell Command: Install 'code' command in PATH"**. This creates a symlink at `/usr/local/bin/code`, which is already in your PATH.

**Option B -- Manual (add to ~/.bash_profile):**

```bash
export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
```

After either option, open a **new** Terminal window and verify:

```bash
code --version
```

## 6. Diagnostic Commands

These are the commands you will use to inspect and troubleshoot PATH in bash:

| Command | What It Does |
|---------|--------------|
| `echo $PATH` | Prints the full PATH string (colon-separated). |
| `echo $PATH \| tr ':' '\n'` | Prints each PATH directory on its own line. Much easier to read. |
| `which git` | Shows the full path to the executable bash will use. |
| `type -a git` | Shows ALL matching executables in PATH (bash equivalent of zsh "where"). |
| `cat /etc/paths` | Shows the system-level base PATH entries. |
| `cat ~/.bash_profile` | Shows your user-level shell configuration. |
| `echo $SHELL` | Confirms which shell you are running (should show /bin/bash). |

## 7. The Mental Model

Think of PATH as a phone book for bash. When you type a command, bash looks up the name in the phone book. The phone book has pages (directories) in a specific order. Bash starts at page 1 and stops as soon as it finds a match. If the program is not listed anywhere, you get "command not found."

Adding a directory to PATH is adding a page to the phone book. **Prepending** puts the page at the front (checked first). **Appending** puts it at the back (checked last). That is the entire concept.

---

*V6 Career Acceleration Program | Jeremy S. Ideus | Day 1 Reference*
