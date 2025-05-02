# Linux Refresher Guide

## Introduction

Linux is a family of open-source, Unix-like operating systems that run on everything from servers and desktops to smartphones and embedded devices. Unlike Windows (with its proprietary software and Active Directory integration) or macOS (tied to Apple’s hardware and software ecosystem), Linux is highly customizable and community-driven. Modern Linux distributions provide a wide range of software, and many common applications are web-based or cross-platform, making it easier than ever to switch to Linux.

**Linux vs Other Operating Systems:** One key difference is that Linux can operate with or without a graphical interface – you can work entirely via a command-line interface (CLI) if needed, which is common for servers. Windows mainly uses a GUI (with PowerShell for command-line tasks), and macOS always runs a GUI (though it provides a Unix terminal). Linux also adheres to a different directory structure and permission model than Windows. The Linux philosophy embraces small, single-purpose programs that can be combined in scripts to perform complex tasks.

**System Lifecycle:** In system administration, it’s useful to think in terms of a system’s lifecycle. A typical Linux system’s lifecycle includes phases like **Design** (planning the system or service), **Develop** (building or configuring the system), **Deploy** (releasing it into production use), **Manage** (day-to-day operation, updates, and maintenance), and eventually **Retire** (decommissioning or replacing the system). This guide will cover essential Linux commands and concepts that are useful across these phases, especially for managing and troubleshooting Linux systems via the command line.

---

## Package Management

Linux distributions use package managers to install, update, and remove software. There are two major families of package management systems:

* **RPM-based** (Red Hat Package Manager) – used by Red Hat, CentOS, Fedora, etc. Typically uses tools like `yum` or `dnf` (and `rpm` for low-level operations).
* **Debian-based** – used by Debian, Ubuntu, etc. Uses the APT suite (`apt-get`, `apt`, `apt-cache`) and `dpkg` for low-level package operations.

Package managers allow you to install software from centralized **repositories** (collections of software packages). They handle dependencies (ensuring required libraries or packages are installed). Below we cover common package management commands for each family.

### RPM-Based Package Management (Yum/RPM)

On RPM-based systems, **Yum** (Yellowdog Updater, Modified) was the traditional high-level package manager. (Newer distributions have **DNF** which is largely compatible with Yum commands.) Yum uses repository configuration files typically in `/etc/yum.repos.d/` (each file defines a repo source). **RPM** is a lower-level tool that installs individual `.rpm` package files and queries the package database.

Common `yum` commands and their purposes:

| **Yum Command**          | **Description**                                                                     |
| ------------------------ | ----------------------------------------------------------------------------------- |
| `yum update`             | Refresh repository indexes and offer to update all packages with updates available. |
| `yum search <name>`      | Search for packages by name or keyword (e.g., `yum search httpd`).                  |
| `yum install <package>`  | Install a package (and its dependencies).                                           |
| `yum remove <package>`   | Remove a package (and optionally dependencies no longer needed).                    |
| `yum check-update <pkg>` | Check if a specific package has an update available.                                |
| `yum upgrade`            | Upgrade all installed packages to latest versions.                                  |
| `yum deplist <package>`  | List a package’s dependencies.                                                      |
| `yum clean packages`     | Clean up cached packages (useful if packages were downloaded but not installed).    |
| `yum list installed`     | List all packages currently installed on the system.                                |

Common `rpm` commands (RPM operates on `.rpm` package files or installed package info):

| **RPM Command**        | **Description**                                                                                     |
| ---------------------- | --------------------------------------------------------------------------------------------------- |
| `rpm -ivh package.rpm` | Install an RPM package file (`-i` = install; `-v` = verbose; `-h` = show progress with hash marks). |
| `rpm -q <pkg>`         | Query if a package is installed (by name).                                                          |
| `rpm -qi <pkg>`        | Show detailed information about an installed package.                                               |
| `rpm -qR <pkg>`        | List the dependencies (requirements) of an installed package.                                       |
| `rpm -e <pkg>`         | Remove (erase) an installed package.                                                                |

> *Practical tip:* On modern Red Hat/CentOS systems, the `dnf` command replaces `yum` but usage is nearly identical (e.g., `dnf install <package>`). Also, graphical package managers or update tools exist, but for exam purposes the command-line tools are key.

### Debian-Based Package Management (APT/dpkg)

Debian-based systems use the **APT** (Advanced Package Tool) suite to manage packages from online repositories listed in `/etc/apt/sources.list` (and files under `/etc/apt/sources.list.d/`). The low-level tool `dpkg` handles installing individual `.deb` package files and maintaining information about installed packages.

Common APT commands:

| **APT Command**                | **Description**                                                                                                                            |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `apt-get update`               | Update the package index (lists of available packages) from repositories. **Run this before searching or upgrading.**                      |
| `apt-cache search <name>`      | Search for a package in the locally cached index by name or description.                                                                   |
| `apt-get install <pkg>`        | Install a package by name (from the repositories or local .deb if specified). Also installs dependencies.                                  |
| `apt-get remove <pkg>`         | Uninstall a package (keep configuration files).                                                                                            |
| `apt-get remove --purge <pkg>` | Uninstall a package *and* remove associated config files.                                                                                  |
| `apt-get autoremove`           | Remove packages that were installed as dependencies but are no longer needed.                                                              |
| `apt-get upgrade`              | Upgrade all installed packages to newer versions (does not install or remove packages, only upgrades existing ones).                       |
| `apt-get dist-upgrade`         | Upgrade packages, *allowing* addition or removal of packages to complete the upgrade (useful for kernel or distribution version upgrades). |

> **Note:** Newer versions of APT provide the `apt` command (without “-get”) which combines functionalities and has a friendlier output. For example, `apt search <pkg>` or `apt install <pkg>` can be used similarly. The exam may still refer to the classic `apt-get` commands.

Common `dpkg` commands (for low-level package file management):

| **dpkg Command**         | **Description**                                                                                                                                                      |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dpkg -i <file.deb>`     | Install a Debian package file (`.deb`). (May fail if dependencies are missing – in that case, install those or use `apt-get -f install` to fix broken dependencies.) |
| `dpkg -r <package_name>` | Remove an installed package (same as apt-get remove, does not delete config files).                                                                                  |
| `dpkg -P <package_name>` | Purge an installed package (remove including configuration files).                                                                                                   |
| `dpkg --get-selections`  | List all packages installed on the system (and installation status).                                                                                                 |

Always consult the man pages (`man yum`, `man apt-get`, `man dpkg`) for more options. Package management is crucial for installing software and keeping the system updated.

---

## Command-Line Basics

Interacting with Linux often involves using the shell via a command-line interface (CLI). A **shell** is a program that interprets your commands and communicates with the Linux kernel to execute them. The most common shell on Linux is **Bash** (Bourne Again Shell), but others exist (Zsh, sh, fish, etc.).

### Using the Shell and Terminal

When you log into a Linux system (either directly or via terminal/SSH), you are placed into a shell session. In a **terminal emulator** or a console, you’ll see a prompt (usually ending in `$` for regular users or `#` for the root user) where you can type commands.

* **Interactive vs Non-interactive shells:** An **interactive shell** is what you get when you open a terminal or SSH session – it reads commands from your input. A **non-interactive shell** runs scripts or background tasks without direct user input.
* **Switching consoles:** On a Linux machine with a text interface or when using Ctrl+Alt+F-key on a GUI, you can often access multiple virtual consoles. For example, pressing `Alt + F2` might switch to a second login prompt (tty2), `Alt + F1` back to the first, etc. On many systems `Alt + F1` through `Alt + F6` correspond to text consoles. (If you’re in a GUI, you may need `Ctrl + Alt + F3`, etc., to switch.)
* **Current shell:** To see which shell you are using, you can echo the `$SHELL` environment variable: `echo $SHELL`.
* **Default shell:** Each user has a default login shell set in **/etc/passwd** (we’ll discuss this file later). Common shells include `/bin/bash`, `/bin/sh`, `/bin/zsh`, etc. You can start a different shell by simply typing its name (e.g., type `tcsh` to start the Tcsh shell, if installed, in your current session).
* **Command prompt customization:** Shell prompts can be customized and often show your username, host, and current directory. For example, a default bash prompt might look like `user@hostname:~/current-dir$`.

### Command History and Autocomplete

The shell keeps a **history** of commands you’ve entered, which is saved to a file (for Bash, this is typically `~/.bash_history`). This allows you to recall and reuse commands:

* Press **Up/Down arrow keys** to scroll through recent commands.
* Press **Tab** to autocomplete a partially typed command or filename (pressing Tab twice shows possible completions if there are multiple). This feature significantly speeds up typing and helps avoid typos.
* Use **Ctrl+R** to initiate a reverse search through your command history – start typing a previous command fragment, and it will auto-fill a matching command.

There are also quick shortcuts for history expansion in Bash:

* `history` – list your command history with line numbers.
* `!n` – run command number *n* from the history (e.g., `!5` runs the 5th command in history list).
* `!!` – run the last command again (same as pressing Up then Enter).
* `!<text>` – run the last command that starts with `<text>` (e.g., `!ssh` runs the last command that began with “ssh”).
* `!?<text>?` – run the last command that *contains* `<text>` anywhere in it.
* `^old^new^` – a quick way to re-run the previous command but with a small substitution: this finds the first occurrence of `old` in the previous command and replaces it with `new`. For example, after mistakenly typing `cat /etc/hots`, you could run `^hots^hosts^` to correct the typo and execute `cat /etc/hosts`.

**Example:** Suppose your history looks like this:

```bash
 19  ls -la /var/log
 20  cd /etc
 21  cat /etc/hosts
```

* Typing `!19` would execute `ls -la /var/log`.
* Typing `!-2` (meaning the second to last command) would execute `cd /etc` (since that’s 2 commands back from the current one).
* Typing `!cat` would re-run `cat /etc/hosts` (the last command starting with “cat”).

These shortcuts can save time in the exam (and real life), but use them carefully to avoid running the wrong command.

### Shell Configuration Files

When a shell starts, especially for login sessions, it reads certain configuration files to set up the environment (like setting environment variables, aliases, shell preferences). It’s important to know which files are executed and when:

* **Login shell** – When you log in to a system (via console or SSH), Bash reads system-wide and user configuration for login shells. The typical order:

  1. `/etc/profile` – system-wide initialization script (applies to all users).
  2. Then **one** of the following user files (Bash will execute the first one it finds and skip the others): `~/.bash_profile`, `~/.bash_login`, or `~/.profile`. These files can set user-specific environment variables, PATH, prompt style, etc.
  3. After those, for Bash, the login shell may also invoke `~/.bashrc` (often your `~/.bash_profile` contains a command to source `~/.bashrc`).
  4. When a login shell exits (you log out), it runs `~/.bash_logout` if it exists (useful for cleanup tasks).
* **Non-login interactive shell** – e.g., opening a new terminal emulator in a graphical session or running a secondary shell by typing `bash` at a prompt. In this case, Bash does **not** read the above files. Instead, it reads:

  * `~/.bashrc` – user-specific Bash configuration for interactive shells (aliases, custom functions, etc.). This file itself often calls `/etc/bashrc` (or `/etc/bash.bashrc` on some distros) to include system-wide settings.

In summary, **`~/.bash_profile`** (or `~/.profile`) is for login shells, and **`~/.bashrc`** is for interactive shells. Many users configure one to call the other to consolidate settings. Other shells (sh, zsh, etc.) have their own config files, but the concept is similar.

### Environment Variables

**Environment variables** are dynamic values that affect the processes or shells in Linux. They are basically key-value pairs stored in the environment that child processes can inherit. Examples include `PATH`, `HOME`, `USER`, etc.

* To see your current environment variables, use the `env` or `printenv` command. (`set` will show shell variables as well, including functions, so it’s more verbose.)
* Some commonly used environment variables:

  * **HOME** – your home directory (e.g., `/home/alice`).
  * **PATH** – a colon-separated list of directories that the shell searches for executables. For example, `/usr/bin:/bin:/usr/sbin:/home/alice/.local/bin` etc. If a command is not in your PATH, you must specify its full path to run it (or `cd` into the directory and prefix with `./`).
  * **USER** or **USERNAME** – your username.
  * **LOGNAME** – also your username (often same as USER).
  * **SHELL** – the path to your default shell (e.g., `/bin/bash`).
  * **PWD** – Present working directory (the current directory you are in).
  * **OLDPWD** – your previous working directory (useful to recall with `cd -`).
  * **OSTYPE** – type of operating system (e.g., `linux-gnu`).
  * **HISTSIZE** and **HISTFILESIZE** – how many commands to keep in history in memory and on disk, respectively.
  * **EDITOR** – preferred text editor (e.g., `vi` or `nano`), which some programs use.
* **Viewing a single variable:** You can print a variable’s value by prefixing it with `$`, for example: `echo $PATH`.

**Setting variables:** In a shell session, you can define or modify a variable. For instance:

```bash
MYVAR="hello"    # set a shell variable MYVAR
echo $MYVAR      # outputs "hello"
export MYVAR     # export to environment so that child processes will see it
```

Important notes when setting variables:

* Do *not* put spaces around the `=` sign. Use `VAR=value`, not `VAR = value`.
* By default, variables are local to the current shell. Use `export VAR` to make it an environment variable (inheritable by any programs or subshells you launch from that session).
* Environment variables set in shell configuration files (like `~/.bash_profile` or `~/.bashrc`) will be set each time you start a new session.

**User-defined variables** (custom variables) follow the same rules. Conventionally, environment variable names are uppercase, but this is not strictly required. Variable names cannot start with a number and usually only contain letters, numbers, or underscores. For example:

```bash
CITY="London"
echo "I live in $CITY."   # uses the variable
```

This would output “I live in London.”

### Command Syntax and Options

A command typed into the shell generally has this structure:

```
command [options] [arguments]
```

* **Command** – This is the program or built-in utility you want to run (like `ls`, `grep`, `cp`, etc.). It’s actually an executable file located somewhere in your filesystem (for example, `/bin/ls`). The shell searches for the command in the directories listed in the `$PATH` environment variable. If the command is not in your PATH, you must specify the full path or run it differently. (For example, to run a script in the current directory, use `./scriptname` because `.` is not in PATH by default for security reasons.)
* **Options (Flags)** – These modify the behavior of the command. Options often start with a dash (`-`) or two dashes (`--`). Single-letter options (like `-a`) can sometimes be bundled (like `-la` which is equivalent to `-l -a`). GNU-style long options have two dashes (e.g., `--all`). For example, `ls -l -a` lists files in long format and includes hidden files; this can be combined as `ls -la`.
* **Arguments** – These are typically file names, directories, or other data the command acts on. For example, in `cp file1.txt /backup/`, the source file `file1.txt` and the destination directory `/backup/` are arguments.

**Example:** `ls -lh /var/log`

* `ls` is the command (list directory contents).
* `-lh` are options (`-l` = long listing format, `-h` = human-readable sizes).
* `/var/log` is an argument (the directory to list; if omitted, `ls` defaults to current directory).

**Formatting and spacing:**

* Commands and options are case-sensitive and typically lowercase (there is a `WHO` command distinct from `who`, for instance).
* It doesn’t matter how many spaces or tabs you put between arguments, as long as there’s at least one whitespace.
* If a command line is very long, you can use the `\` (backslash) at the end of a line to indicate the command continues on the next line. This is useful for readability when typing long commands or in scripts. For example:

  ```bash
  ls -l /var/log \ 
      /etc /home/user/somedir \
      /nonexistent 2> errors.txt
  ```

  (Here the backslashes allow the `ls` command with multiple arguments to span multiple lines. The shell will treat it as one continuous line.)

### Globbing (Wildcards for Filename Expansion)

**Globbing** refers to the shell’s wildcard matching that expands patterns to file or directory names. This is a form of shortcut that the shell processes *before* the command is executed (it’s not regex, but simpler wildcard patterns). Common wildcard characters:

* `*` – Matches **zero or more** characters in a filename. For example, `*.txt` matches all files ending in “.txt” (such as `notes.txt` or `report.txt`), and `file*` would match `file`, `file1`, `filename`, etc.
* `?` – Matches exactly **one** character. For example, `?.txt` would match any one-letter filename with “.txt” (like `a.txt` or `Z.txt`).
* `[abc]` – Matches **one** character that is any of the characters inside the brackets. For example, `file[12].txt` matches `file1.txt` or `file2.txt` but not `file3.txt`.
* `[a-z]` – You can specify a range of characters. This pattern matches one character in the specified range. For example, `report[0-9].txt` matches `report0.txt` through `report9.txt`.
* `[^abc]` – The `^` at the start of brackets means NOT any of these characters. For example, `project[^A]` would match `projectB` or `project1` but not `projectA`.

**Examples:**

* `ls *.jpg` – list all files in the current directory ending with `.jpg`.
* `rm file?.txt` – remove files like `file1.txt`, `fileA.txt` (any single character in place of `?`).
* `cp 202[0-2]-report.txt /backup/` – copy files named `2020-report.txt`, `2021-report.txt`, or `2022-report.txt` to /backup.
* `ls [Bb]udget*.xlsx` – list any Excel files starting with “Budget” or “budget”.

Globbing is extremely useful for working with multiple files. Remember that the shell, not the command, expands the wildcard. If a glob pattern doesn’t match any file, some shells will pass the pattern literally to the command (or print an error, depending on settings). Also, quote the pattern or escape wildcards if you don’t want them to expand (more on quoting next).

### Quoting and Escaping

When you work with command-line arguments that contain spaces or special characters, you need to **quote** or **escape** them so the shell treats them as a single unit or literal string.

* **Double quotes (`"`)** – Enclosing text in double quotes tells the shell to treat most characters literally, *but* it still allows variable expansion and certain special sequences. For example, `echo "Home: $HOME"` will interpolate the `$HOME` variable inside the quotes.
* **Single quotes (`'`)** – Enclosing text in single quotes treats everything literally, including variables or special characters. Nothing is expanded or interpreted. For example, `echo 'Home: $HOME'` will literally print `$HOME` instead of the home path.
* **Backslash escape (`\`)** – A backslash in front of a character “escapes” it, meaning the shell will take that character literally. This is often used to escape special characters in a mostly unquoted string. For instance, `echo You owe \$5` will output “You owe \$5” because the dollar sign was escaped. Backslashes can also be used before a space in a filename, e.g. `My\ File.txt` refers to a file named "My File.txt".

Summary of quoting behavior:

| **Quotation**     | **Effect**                                                                                                           | **Example**                                        |
| ----------------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `"double quotes"` | Preserves spaces and most special chars literally, **but** allows `$VAR` expansion and `` `command` `` substitution. | `echo "User is $USER"` – variable will expand.     |
| `'single quotes'` | Preserves everything literally (no expansion of variables or commands).                                              | `echo 'User is $USER'` – prints `$USER` as text.   |
| `\ (backslash)`   | Escapes the single character that follows. Useful for spaces, \$, `*`, etc., without quoting entire string.          | `echo It costs \$100\*` – prints `It costs $100*`. |

Use quotes whenever an argument contains spaces or shell metacharacters (like `*`, `?`, `~`, `$`, `!`, `(`, `)`, `|`, `&`, `<`, `>` etc.) that you want to be taken literally. If you forget to quote a filename with spaces, for example, the shell will interpret it as separate arguments.

---

## Getting Help in Linux

Linux systems include comprehensive documentation accessible from the command line. Three key resources are **man pages**, **info pages**, and other local documentation (like README files). Understanding how to use these will help you quickly find syntax and options during an exam or in practice.

### Manual Pages (man)

**Man pages** (manual pages) are the built-in reference documentation for Unix commands and APIs. They are meant as quick references for people who know what a command does, but need details on usage or options. To read a man page, use the `man` command followed by the topic (usually a command name). For example, `man ls` shows the manual for the `ls` command.

Key points about man pages:

* Man pages are divided into **sections** (1 through 9 for different categories: 1 = user commands, 5 = file formats, 8 = admin commands, etc.). If a topic exists in multiple sections, you can specify the section number. For example, `man 5 passwd` shows the man page for the file format of `/etc/passwd`, whereas `man 1 passwd` (or just `man passwd`) shows the user command `passwd`.
* Common sections you’ll use: 1 (general commands), 5 (file formats and configuration files), 8 (system administration commands). The section number is sometimes shown in parentheses after a command name, e.g., **passwd(5)**.
* Use `man -k <keyword>` or the `apropos <keyword>` command to search for man pages by keyword. (This searches the man page descriptions for the keyword.)
* Use `whatis <command>` to get a one-line summary of a command (from the man page description).
* Navigation inside `man`: It uses a pager (often `less`). Press **Space** or **Page Down** to go forward, **b** or **Page Up** to go back. Press **/** to search within the page (type the search term and Enter), and **n** to go to the next match. Press **q** to quit the man page.
* Man pages follow a common structure:

  * **NAME** – The name of the command or function, and a brief description.
  * **SYNOPSIS** – Shows how to use the command, including options and arguments. Optional parts are often in brackets `[]`. A vertical bar `|` indicates “or”, and an ellipsis `...` means “repeat as needed”. For example, a synopsis might show `[OPTIONS] FILE...` indicating the command can take options and one or more files.
  * **DESCRIPTION** – A textual description of what the command does, possibly with more detail.
  * **OPTIONS** – Explanation of each option/flag.
  * Other sections like **EXAMPLES**, **FILES** (files used by the command), **SEE ALSO** (related commands or docs), **BUGS**, **AUTHOR**, etc., may appear depending on the command.

Man pages can sometimes be terse or overly detailed. They are great for syntax and option reference, but not always the best learning tutorial if you’re entirely new to a command.

### Info Pages and Documentation

Some programs, particularly those from the GNU project, have **info pages** in addition to or instead of man pages. Info pages are viewed with the `info` command and are typically more verbose, often organized like a book with chapters and sections, including hyperlinks (links are usually indicated by `*` in the text).

* To read an info page, use `info <command>` (e.g., `info coreutils` for the Coreutils manual, which covers many basic commands).
* Inside an info page, navigation is different from man: use the arrow keys or Page Up/Page Down, and press **Enter** to follow a hyperlink. Press **U** to go up a level, **N** for next, **P** for previous node, and **Q** to quit.

In practice, man pages are more commonly used, but it’s good to know info exists. Many GNU tools will direct you to info for more complete documentation.

**Additional local documentation:**

* Many packages install documentation in `/usr/share/doc/<package>/`. This may include README files, examples, or more detailed guides. For example, if you installed `nginx`, you might find docs in `/usr/share/doc/nginx/`.
* Some software includes documentation in other formats (HTML, PDF, etc.) in those directories.
* Configuration files for system services are usually in `/etc`. Often these files have comments that explain settings. Always check config files (e.g., `/etc/ssh/sshd_config`) for inline documentation.
* Use package manager queries to find documentation: For instance, on an RPM-based system, `rpm -qd <package>` lists documentation files installed by that package.

**Reading various file formats:** If you encounter a documentation file and need to read it, here are some tips:

| **File Extension**                                  | **How to Read**                                                                                                        |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `.txt` (plain text)                                 | Open with any text viewer (`cat`, `less`, or a text editor like `nano` or `vim`).                                      |
| `.gz` (gzip compressed)                             | Decompress with `gunzip file.gz` (or use `zcat`/`zless` to view without explicitly decompressing).                     |
| `.bz2` (bzip2 compressed)                           | Decompress with `bunzip2 file.bz2` (or `bzcat`/`bzless`).                                                              |
| `.html` (web page)                                  | View with a text-based browser (`lynx`, `w3m`) or a GUI browser if available. Or `less` will show the raw HTML source. |
| `.pdf` (PDF document)                               | Needs a PDF reader (`evince`, `okular` in GUI, or command-line tools like `pdftotext` to convert to text).             |
| Man page file (e.g., `/usr/share/man/man1/ls.1.gz`) | Use `man` to view it properly, or decompress and read with `less`.                                                     |
| Info file (often in `/usr/share/info/`)             | Use `info` command, or `pinfo` for a nicer interface (if installed).                                                   |

Usually, using `cat` or `less` on a text file is your first approach. For compressed text, piping through a decompressor works: e.g., `zcat file.gz | less`.

---

## Linux File System Structure

Linux organizes files in a **hierarchical directory structure**. All files and directories stem from a single root directory, denoted by `/`. This is often called the **Filesystem Hierarchy Standard (FHS)**, which most Linux distributions follow. Under the root `/`, there are standard directories each serving a specific purpose:

| **Directory** | **Purpose**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **/** (root)  | The top of the directory tree. Contains all other files and directories. (Note: “root directory” is different from “/root” which is root user’s home.)                                                                                                                                                                                                                                                                                                                                                           |
| **/bin**      | Essential binaries (programs) needed for basic system operation, accessible by all users (e.g., `ls`, `cp`, `bash`).                                                                                                                                                                                                                                                                                                                                                                                             |
| **/sbin**     | System binaries – like /bin but for system administration commands (usually those that require root), e.g. `fsck`, `iptables`.                                                                                                                                                                                                                                                                                                                                                                                   |
| **/boot**     | Files needed to boot the system, like the Linux kernel (`vmlinuz`), initramfs image, bootloader files (GRUB), etc.                                                                                                                                                                                                                                                                                                                                                                                               |
| **/dev**      | Device files – special files that represent hardware devices or virtual devices. For example, `/dev/sda` might be a hard disk, `/dev/tty1` a console. This allows hardware to be accessed like files.                                                                                                                                                                                                                                                                                                            |
| **/etc**      | System configuration files (mostly text). Contains startup scripts, network config, user account info, etc. (For example, `/etc/passwd`, `/etc/ssh/sshd_config`.) No binary programs here.                                                                                                                                                                                                                                                                                                                       |
| **/home**     | Home directories for regular users. e.g., `/home/alice` for user "alice". Users store personal files and settings here.                                                                                                                                                                                                                                                                                                                                                                                          |
| **/root**     | Home directory for the **root** user (administrator). (This is separate from `/` even though named “root”). Usually `/root` is only accessible to root.                                                                                                                                                                                                                                                                                                                                                          |
| **/lib**      | Shared libraries needed by programs in /bin and /sbin. (On 64-bit systems you may see `/lib64` for 64-bit libraries.) These are similar to DLLs on Windows.                                                                                                                                                                                                                                                                                                                                                      |
| **/usr**      | “User” or Unix system resources. This is a large hierarchy for installed software and read-only data. Many subdirectories: `/usr/bin` (additional user programs), `/usr/sbin` (additional system programs), `/usr/lib` (libraries for /usr/bin), `/usr/share` (architecture-independent data, like docs, icons, etc.), `/usr/local` (for user-compiled or locally installed programs to avoid overwriting distro files). Generally, `/usr` contains the bulk of applications and files not critical for booting. |
| **/var**      | Variable data files – things that change often: logs, spool files (for mail, printing), caches, transient files. For example, log files are in `/var/log`, spool for print jobs in `/var/spool`, and so forth. Web servers often use `/var/www` for site files.                                                                                                                                                                                                                                                  |
| **/tmp**      | Temporary files. Applications store temp data here. Typically cleared on reboot (on many distros, /tmp is emptied at boot). Any user can typically write to /tmp, but there are special permissions (sticky bit) to ensure users can’t delete each other’s files (more on that later).                                                                                                                                                                                                                           |
| **/var/tmp**  | Temporary files that should **persist** between reboots. Similar to /tmp, but not wiped at reboot. Programs use this for longer-term temporary storage.                                                                                                                                                                                                                                                                                                                                                          |
| **/mnt**      | Mount point for temporarily mounting filesystems (e.g., mounting a USB drive or an additional disk manually). It’s basically an empty directory used as a placeholder to attach other filesystems to the main tree.                                                                                                                                                                                                                                                                                              |
| **/media**    | Mount point for removable media. Many Linux desktops automatically mount USB drives or DVDs under `/media/<username>/<device_name>`. It serves a similar purpose to /mnt, but often used by automounters.                                                                                                                                                                                                                                                                                                        |
| **/opt**      | Optional software. Often used for manually installed software or large third-party applications. E.g., a package that is not part of the distro might live in `/opt/<package>/`. (Historically for add-on packages.)                                                                                                                                                                                                                                                                                             |
| **/proc**     | A virtual filesystem that reflects kernel and process status (processes, hardware info). The `/proc` directory is populated by the kernel at runtime. For example, `/proc/cpuinfo` contains info about the CPU, `/proc/<PID>/` has info about process with that ID. These are not real files on disk – they exist in memory and provide an interface to kernel data structures.                                                                                                                                  |
| **/sys**      | Another virtual filesystem (sysfs) that exposes kernel device and system information. Organized by devices and other categories. Modern Linux uses /sys and /proc together for introspection and configuration of hardware.                                                                                                                                                                                                                                                                                      |
| **/srv**      | “Service” data – contains data for services provided by the system. For example, web server files might be in `/srv/www` or ftp files in `/srv/ftp`. (Not all distros use this heavily, but it’s part of FHS.)                                                                                                                                                                                                                                                                                                   |
| **/run**      | Runtime variable data (added in newer Linux systems). This is like /var/run (older locations) – used for volatile runtime data like process ID files, sockets, etc. Data in /run is typically cleared at boot. For example, `/run/nginx.pid` might hold the PID of the nginx service.                                                                                                                                                                                                                            |

Understanding these directories is critical. For example, if a question asks where logs are kept, you should recall **/var/log**. If a configuration file for a system service is needed, it’s likely in **/etc**. User data should go in **/home**. Knowing the purpose of each directory helps in navigating the system and locating files.

*Practical tip:* You can use commands like `tree / -L 1` (if `tree` is installed) to show the top-level directories and get a sense of the structure. Also, `df -hT` will show mounted filesystems and their mount points, giving clues to what storage devices correspond to which parts of the directory tree.

---

## Working with Files and Directories

Being proficient in Linux means knowing how to navigate and manipulate the file system from the command line. Here are fundamental commands to manage directories and files:

* **pwd** – Print Working Directory. This simply outputs the full path of the directory you are currently in. It’s useful to know where you are in the filesystem.

  ```bash
  $ pwd
  /home/alice/projects
  ```

  (This shows we are in the `/home/alice/projects` directory.)

* **cd** – Change Directory. Used to navigate the filesystem.

  * `cd <directory>` to move into a directory. For example, `cd /etc` will take you to the `/etc` directory.
  * `cd ..` moves up one level (to the parent directory).
  * `cd -` switches to the previous directory you were in (handy toggle).
  * If used without arguments, `cd` returns you to your home directory (same as `cd ~`).

  *Tip:* A path beginning with `/` is an **absolute path** (from root). A path not beginning with `/` is relative to the current directory. `~` refers to your home directory.

* **ls** – List directory contents. This is one of the most frequently used commands.

  * `ls` by itself lists the names of files and subdirectories in the current directory (excluding those that start with `.` which are hidden).
  * Common options:

    * `-a` – list **all** entries, including dotfiles (hidden files that begin with `.`).
    * `-l` – use the **long listing** format, which shows details like permissions, owner, size, and modification date.
    * `-h` – with `-l`, print sizes in human-readable format (KB, MB, etc., instead of raw bytes).
    * `-R` – list directories **recursively** (i.e., list subdirectories’ contents as well).
    * `-t` – sort output by modification **time** (newest first).
    * `-S` – sort output by file **size** (largest first).
    * `-r` – **reverse** the sort order (e.g., `ls -lr` for reverse alphabetical, or combine as `ls -ltr` to list oldest first).

  For example, `ls -alh /var/log` would show a detailed listing of `/var/log` including hidden entries, with sizes in KB/MB.

* **mkdir** – Make Directory. Creates a new directory.

  * Usage: `mkdir <dirname>` creates a subdirectory in the current directory.
  * Use `-p` (parents) to create nested directories in one go. For example: `mkdir -p projects/2023/january` would create the directory `projects` (if it doesn't exist), inside it create `2023`, and inside that create `january`. This is useful to avoid errors when intermediate directories don’t exist.

* **rmdir** – Remove Directory. Deletes an *empty* directory.

  * Usage: `rmdir <dirname>`. This will only work if the directory is empty. If not, you need to remove its contents first or use `rm -r`.
  * Typically, we often use `rm -r` for removing directories (see below), but rmdir is a safe way to remove empty dirs.

* **cp** – Copy files or directories.

  * Basic usage: `cp source target`. For example, `cp file1.txt file1.bak` (copy file1.txt to file1.bak).
  * To copy multiple files into a directory: `cp file1 file2 file3 target_directory/`.
  * Common options:

    * `-r` or `-R` – **recursive**; needed to copy directories (it copies all files and subdirectories within).
    * `-p` – **preserve** attributes (timestamps, ownership, permissions).
    * `-u` – **update**; only copy if the source is newer than the destination or if destination is missing. This avoids unnecessary overwriting.
    * `-i` – **interactive**; prompt before overwriting any existing file (asks for confirmation).
  * Example: `cp -Rpv project/ project_backup/` would verbosely copy the `project` directory to `project_backup`, preserving attributes.

* **mv** – Move (or rename) files and directories.

  * Usage for renaming: `mv oldname.txt newname.txt` (the file now is called newname.txt).
  * Usage for moving: `mv file1.txt /path/to/destination/` (now file1.txt resides in the target directory).
  * If the target is an existing file, `mv` will overwrite it (you can use `-i` to prompt before overwriting).
  * `mv` can also move directories (it’s essentially instantaneous if moving within the same filesystem, because it just updates pointers).
  * You can also move multiple files to a directory: `mv file1 file2 file3 directory/`.

* **rm** – Remove (delete) files or directories.

  * Usage: `rm file.txt` removes the file. It **permanently deletes** the file (there is no recycle bin/trash when using rm directly, so be careful!).
  * `rm -i file.txt` will ask “remove regular file 'file.txt'?” for confirmation.
  * To remove **directories and their contents**, use `rm -r directory/`. This recursively deletes the directory and everything under it. (Be extremely cautious with recursive remove.)
  * `rm -f` forces removal without prompting or complaining about non-existent files.
  * Combined common usage: `rm -rf <dir>` – force remove a directory and all contents without prompts (commonly used but dangerous if you point to the wrong path, so double-check before pressing Enter).
  * **Safety tip:** You can alias `rm` to `rm -i` in your shell config to always prompt, which can prevent accidents.

* **touch** – Often used to create an empty file or update a file’s timestamp.

  * `touch newfile.txt` will create an empty file named newfile.txt if it doesn’t exist, or if it exists, update its last modified time to now.
  * You can use `-d` to set a specific date: e.g., `touch -d "2023-01-15 10:00" file.txt` sets the file’s modification time to Jan 15 2023, 10:00.
  * `touch` is handy in scripting to create marker files or to trigger other processes that watch timestamps.

**Viewing file contents quickly:**

* `cat filename` – prints the entire content of the file to the screen (standard output). Great for short files, but for long files you’ll only see the last part on screen (since it scrolls past).
* `less filename` – opens the file in a pager so you can scroll (with Up/Down or Page keys) and search. Press `q` to quit out of `less`. (Use `-N` to show line numbers in `less`, and you can search with `/pattern`.)
* `head -n 20 filename` – show the first 20 lines of a file (useful to preview a file or check its header).
* `tail -n 50 filename` – show the last 50 lines of a file. `tail -f logfile` will **follow** the file, continuously outputting new lines as they are added (good for monitoring logs in real-time).

We will cover more about viewing and searching file content in a later section. But the above commands let you navigate and manipulate files/directories. It’s important to practice these until you can use them quickly, as they are the bread and butter of Linux CLI usage.

---

## Archives and Compression

Linux provides powerful tools to combine multiple files into archives and to compress data to save space. It’s important to distinguish between **archiving** (combining files together) and **compression** (reducing size). Some tools do one, some do both.

### Tar Archives (Combining Files)

**`tar`** (tape archive) is the traditional Unix tool to create file archives (originally for tape backups). A tar archive (often with extension `.tar`) is a single file that contains many files and preserves directory structure, but it’s not compressed by itself. However, tar has options to call compression utilities while creating or extracting archives, which is commonly done.

Common `tar` options (you can combine them after a single dash, e.g., `-czf`):

* **-c** – Create an archive.
* **-x** – Extract an archive.
* **-t** – List the contents of an archive (table of contents).
* **-f** – File name of the archive (this option is usually immediately followed by the archive name, e.g., `-f archive.tar`).
* **-v** – Verbose output (lists files being added or extracted, useful to see progress).
* **-z** – Compress or decompress using gzip (when creating or extracting).
* **-j** – Compress or decompress using bzip2.
* **-J** – (capital J) Compress using XZ (for .xz files, not as commonly used in basic scenarios).

**Creating a tar archive:**

* To archive a directory (without compression):
  `tar -cf archive.tar /path/to/directory`
  This puts `/path/to/directory` (and everything under it) into `archive.tar`.
* To archive multiple specific files:
  `tar -cf configs.tar /etc/hosts /etc/passwd /etc/group`
  This creates `configs.tar` containing those three files.
* It’s good practice to name archives with `.tar` extension for clarity (though not required).

**Extracting a tar archive:**

* `tar -xf archive.tar` will extract the archive into the current directory (it will recreate any directory structure stored in the tar).
* Use `-C /target/dir` to extract into a specific directory: `tar -xf archive.tar -C /tmp/extract_here`.
* You can list contents before extracting: `tar -tf archive.tar` (t = list, f = file). This shows the files in the archive so you know what will happen.

**Archiving with compression:** You often see single-file archives like `.tar.gz` or `.tgz` (tarball compressed with gzip), or `.tar.bz2` (tarball with bzip2 compression).

* **Creating a gzip-compressed tar:**
  `tar -czf archive.tar.gz /path/to/dir`
  Here, `-c` (create), `-z` (gzip), `-f archive.tar.gz` (output file). This will produce a compressed archive. The order of `z` and `f` doesn’t matter as long as both are there; many people remember it as `czf` for create zipped file.
* **Creating a bzip2-compressed tar:**
  `tar -cjf archive.tar.bz2 /path/to/dir`
  using `-j` for bzip2 compression.
* **Extracting a compressed tar:**
  `tar -xzf archive.tar.gz` for gzip,
  `tar -xjf archive.tar.bz2` for bzip2.
  (tar will auto-detect compression by the file, so you could just do `tar -xf archive.tar.gz` in many cases and it figures it out, but using `-z`/`-j` explicitly is a sure thing.)

**Examples:**

```bash
# Create a tar archive of /var/log (uncompressed)
tar -cf logs.tar /var/log

# Create a gzip-compressed tar archive of the current directory
tar -czf myproject.tar.gz .

# List contents of a tar.gz file
tar -tzf myproject.tar.gz

# Extract a tar.bz2 archive
tar -xjf data_backup.tar.bz2
```

Remember that tar by default does not compress unless you add the flags for it. The exam might expect you to know how to create or extract archives, and the common extensions associated (`.tar` for tar only, `.tar.gz` or `.tgz` for tar+gzip, `.tar.bz2` for tar+bzip2).

### Zip Files (Archive + Compression)

**`zip`** is a widely used format (especially in Windows) that *both* archives and compresses in one step. The `zip` command on Linux creates zip files, and `unzip` extracts them.

* Creating a zip: `zip archive.zip file1 file2 dir3`. This will compress file1, file2, and directory dir3 into archive.zip. By default, it recurses into directories.

  * Use `-r` explicitly to ensure recursion: `zip -r archive.zip directory/` (recursively zip the contents of "directory").
  * By default, zip doesn’t preserve Linux file ownership or permissions as well as tar does. Zip is more for sharing files (e.g., with Windows users) or simple compression tasks.
* Extracting a zip: `unzip archive.zip` (will extract into the current directory, creating files/dirs stored in the zip).

**Examples:**

```bash
zip backup.zip *.txt          # Compress all .txt files into backup.zip
zip -r source_code.zip src/   # Recursively archive and compress the "src" directory
unzip backup.zip             # Extract the zip file
```

The zip format is convenient and easy, but in Linux environments, tarballs are more common for packaging lots of files, because of better metadata preservation.

### Gzip and Bzip2 (Compression Utilities)

These are compression tools that operate on single files (they do not create multi-file archives by themselves). They are often used in conjunction with tar.

* **gzip** – Very common compressor, fast and decent compression ratio.

  * To compress a file: `gzip filename` produces `filename.gz` and by default **deletes** the original file (to keep it, use `gzip -c filename > filename.gz` or use the `-k` (keep) option if available).
  * To decompress: `gunzip filename.gz` (or `gzip -d filename.gz` does the same).
  * You can also compress data from standard input and output to standard output, which is useful in pipelines (e.g., `cat bigfile | gzip > bigfile.gz`).
  * A useful flag: `-c` writes compressed output to stdout (so you can redirect it), and `-k` tells gzip to keep the input file.
  * **Example:** `gzip -k data.tar` will produce `data.tar.gz` and leave `data.tar` in place. Without `-k`, `data.tar` would be removed after compression.

* **bzip2** – A compressor that usually achieves better compression than gzip but is slower.

  * Usage is similar to gzip: `bzip2 filename` compresses (and typically removes original, creating `filename.bz2`), and `bunzip2 filename.bz2` decompresses.
  * Like gzip, you can use `-k` to keep the original, or use `-c` to output to stdout.

* **xz** – Another compressor (not mentioned in the original notes, but for completeness: xz can compress better than bzip2 for some data, but slower. Creates .xz files. Usage `xz filename`, `unxz filename.xz`).

**Practical compression tips:**

* To compress a directory, you typically use tar and then gzip/bzip2, e.g., `tar -czf dir.tar.gz dir/`. A standalone `gzip` won’t compress multiple files into one archive – it only compresses each file given (and if given a directory, it compresses each file inside it, resulting in multiple .gz files).
* `zip` on the other hand can take a directory and internally archive it.

**Summary:**

* Use `tar` to archive multiple files (with optional `-z` or `-j` for compression).
* Use `gzip`/`bzip2` for compressing single files (or in streams).
* Use `zip` if you need a single-step archive+compress (especially when exchanging with Windows systems).

Make sure you can recognize by filename which tool to use. If you see a `.tar.gz` or `.tgz`, you know it’s tar+gzip (use tar with `-z`). If you see a `.zip`, use `unzip`. If it’s `.gz` alone, it’s just gzip (maybe for a single file or a compressed stream), so `gunzip` or use tar if it’s an archive inside (like `.tar.gz`). `.bz2` similarly.

---

## Viewing and Searching Text Files

Linux provides many tools to view, filter, and search text files directly from the command line. This is crucial for examining configuration files, logs, and output of other commands. Below are essential commands and their usage:

### Viewing File Contents

| **Command**           | **Purpose & Usage**                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `cat filename`        | **Concatenate** – typically used to print the entire content of a file to the screen. For small files this is fine. For larger files, combine with paging or filtering (or use `less`). `cat file1 file2 > combined` can concatenate files into a new file.                                                                                                                                                                                                                    |
| `less filename`       | Open the file in a paging viewer. Allows scrolling with arrow keys/Page keys. Press `Q` to quit. Inside `less`, use **`/search`** to find text (press `n` for next match), and **`?search`** to search backwards. `less` is ideal for viewing large files because it doesn’t load the whole file into memory at once. Options: `-N` to show line numbers; `-M` to show a more verbose prompt with file position; hit **G** (Shift+g) to jump to end, **g** to go to beginning. |
| `head -n 10 filename` | Show the first 10 lines of a file (use `-n` to specify a different number of lines). Without `-n`, default is 10. This is useful for quickly viewing file headers or verifying content.                                                                                                                                                                                                                                                                                        |
| `tail -n 10 filename` | Show the last 10 lines of a file. `tail` is commonly used with log files to see the latest entries. The `-f` option (follow) is extremely useful: `tail -f /var/log/syslog` will continuously output new lines appended to the file (like a live feed). Press Ctrl+C to stop following.                                                                                                                                                                                        |
| `tail -f filename`    | Follow the file’s end. Often one might use `tail -f /var/log/xxx.log` to monitor logs as they come in. Combine with other options like `-n` to start with some lines then follow.                                                                                                                                                                                                                                                                                              |

Other useful viewers:

* `more` – an older pager (less is newer and “more capable than more” – hence the name).
* `nl file` – cat with line numbering (or use `cat -n file`).
* `od -c file` – outputs file in various representations (octal/hex dump, etc.) – not commonly needed unless debugging non-text content.

### Searching within Text (grep)

**grep** is the go-to tool for searching text for lines that match a pattern. (“grep” stands for globally search a regular expression and print matching lines.)

Basic usage: `grep "pattern" filename`. This will print all lines in the file that contain the pattern.

Common grep options:

* `-i` – ignore case (case-insensitive search).
* `-n` – prefix each matching line with its line number in the file.
* `-w` – match whole words only (e.g., searching for "doc" won’t match "document").
* `-v` – invert match (show lines that *do not* match the pattern).
* `-r` – recursive; if given a directory, search all files under it (you can also use `-R`).
* `-A NUM` / `-B NUM` – show NUM lines After or Before each match (for context).
* `-E` – treat pattern as an extended regular expression (enables more regex features, see below).

Examples:

```bash
grep admin /etc/passwd               # show lines containing "admin" in the file
grep -i error /var/log/syslog        # case-insensitive search for "error"
grep -n "server 123" config.txt      # show line numbers for matches of "server 123"
grep -v "^#" /etc/ssh/sshd_config    # show lines that do NOT start with "#" (i.e., exclude comments)
grep -R "TODO" ~/projects            # recursively find "TODO" in files under ~/projects
grep -A2 -B2 "interface" /etc/network/interfaces  # show 2 lines before and after each match of "interface"
```

**Note:** If searching a large directory recursively, you might want to combine grep with `find` or use `grep --include` to limit by file type (for example, only \*.log files). But for exam basics, understanding `grep -R` is usually enough.

### Text Processing Tools (sort, cut, wc, etc.)

* **sort** – sorts lines of text alphabetically or numerically.

  * By default, sort alphabetically (lexically). Use `-n` for numeric sort, `-r` for reverse order.
  * Example: `sort names.txt` (alphabetical A→Z), `sort -r names.txt` (Z→A), `sort -n numbers.txt` (sort lines as numbers).
  * You can sort by columns with options like `-k` (key) and `-t` (field separator), but basics: know -n and -r.

* **cut** – cuts out columns or fields from lines of text.

  * Useful for extracting specific portions of each line, by character or by delimiter-separated fields.
  * `-c` option to cut by character positions, or `-d <delim> -f <fields>` to cut by delimiter-separated fields.
  * Example: Given a file where fields are colon-separated (like /etc/passwd), `cut -d: -f1 /etc/passwd` would list just the first field of each line (usernames). If fields are space-separated, `cut -d" " -f 2-4 file.txt` would extract fields 2 through 4.
  * Another example: `echo "2023-07-15" \| cut -d- -f2` would output `07` (the second field between the hyphens).

* **wc** – word count (and more). It counts lines, words, and characters in files.

  * `wc filename` gives three numbers: lines, words, and bytes (characters) in the file, plus the filename.
  * Options: `-l` (lines only), `-w` (words only), `-c` (bytes). So `wc -l /etc/passwd` tells how many lines (accounts) are in /etc/passwd.
  * You can also pipe output into wc, e.g., `ls /etc | wc -l` counts how many entries are in /etc.

* **tr** – translate or delete characters. For instance, `tr 'A-Z' 'a-z'` will convert input from uppercase to lowercase. Or `tr -d '\r'` to delete carriage returns (useful when dealing with Windows text files on Linux).

* **awk** and **sed** – powerful text processing languages/tools (awk for field processing, sed for stream editing). These are more advanced; know that they exist. For an essentials exam, you might just need to know basic use:

  * `awk '{ print $1, $3 }' file` prints first and third columns of each line (by default splits on whitespace).
  * `sed -n '5,10p' file` prints lines 5 through 10 of a file. Or `sed 's/old/new/g' file` replaces “old” with “new” on each line (printing to stdout).
    If the exam is truly basic, you might not need to get deep into awk/sed, but it could be touched on.

### Combining Commands with Pipes

The power of the command line comes from **pipes**. A **pipe** (`|`) takes the output of one command and feeds it as input to another command. This allows you to build multi-step processing in one line.

For example:

```bash
grep -i error /var/log/syslog | wc -l
```

This searches `syslog` for "error" (case-insensitive), and then pipes the matching lines to `wc -l` which will count them. The result is the number of lines containing "error".

Another example:

```bash
ps -ef | grep apache | grep -v grep
```

This lists all processes, then filters to those containing "apache", then filters out the `grep` process itself (a common pattern to find if a service is running).

You can chain many commands:

```bash
ls -1 /etc | sort | head -20
```

This lists one file per line in /etc, sorts them, and shows the first 20.

**Using `xargs`:** Sometimes you want to pipe output as arguments to another command (not as standard input). `xargs` is a command that reads from stdin and executes another command with the input as arguments. E.g., `grep -Rl "TODO" . | xargs rm` would find files containing "TODO" and then remove them (be careful with such usage!). Or safer: `grep -Rl "pattern" . | xargs -d '\n' grep "otherpattern"` – this is advanced, just note that xargs exists for making pipelines where the final command expects arguments, not input stream.

For the basics, the main idea is: **pipe** (`|`) allows combining commands like building blocks.

### Using Regular Expressions (Basics)

**Regular expressions (regex)** are patterns that describe text. Many commands (grep, sed, awk, etc.) use regex for matching. While a full regex lesson is lengthy, here are a few basic constructs (for grep, which uses a slightly limited BRE by default):

* `.` (dot) – matches **any single character** except newline. Example: `gr.p` matches "grip", "grap", "grp" (with any char in place of dot).
* `*` – applied as a suffix, means “repeat the preceding element **0 or more** times”. Example: `zo*` matches "z", "zo", "zoo", "zoooo", etc. In `grep`, `*` usually requires something to repeat (except some versions allow starting like `*abc` which is not meaningful by itself).
* `^` – matches the **beginning** of a line (when at start of pattern). Example: `^ERROR` matches lines that start with "ERROR".
* `$` – matches the **end** of a line (when at end of pattern). Example: `;$` matches any line that ends with a semicolon.
* `\[ \]` – character classes. We covered those in globbing, regex uses similar syntax:

  * `[0-9]` matches any one digit.
  * `[A-Za-z]` matches any one uppercase or lowercase letter.
  * `[^...]` is a negated class (match any character *not* listed).
* `\<` and `\>` – word boundaries in some regex implementations (in GNU grep, `\b` might not work in basic mode; you can use `\<word\>` to match "word" as a whole word).
* `\+`, `\?`, `\|` – these have special meaning in **extended** regex (like ERE, enabled with `grep -E` or using `egrep`). `+` means 1 or more, `?` means 0 or 1, and `|` means OR between patterns. In basic grep, these may need escaping or are not available.

**Examples:**

* `grep "^root:" /etc/passwd` – finds lines starting with "root:" (likely the root user’s entry in passwd).
* `grep -E "cat|dog" pets.txt` – finds lines containing "cat" or "dog" (using `-E` for extended regex to allow the `|` alternation).
* `grep -E "[0-9]{3}-[0-9]{4}" file` – (with extended regex) find patterns like "123-4567" (3 digits, hyphen, 4 digits).
* `grep -E "[A-Z]{2,}" file` – find lines with at least two consecutive uppercase letters.

**Tip:** If unsure, you can often use grep with `-E` to simplify regex usage in an exam setting, since it supports `+?` etc. Or use `egrep` which is equivalent to `grep -E`. If you only need simple “contains substring” or “starts with”, you might not need heavy regex.

### I/O Redirection

We touched on pipes which redirect output to another program. Linux also allows redirecting output to files, and input from files, using certain symbols:

* `>` redirect stdout (standard output) to a file, replacing the file’s contents. For example:
  `ls -l /etc > etc_list.txt`
  This takes the output of `ls -l /etc` and writes it to **etc\_list.txt** (if the file exists, it is overwritten; if not, it’s created).
* `>>` redirect stdout and **append** to a file (preserve existing content).
  Example: `echo "Backup done at $(date)" >> backup.log` will append that line to backup.log, keeping previous logs.
* `2>` redirect stderr (standard error) to a file.
  Example: `grep -R "needle" / 2> errors.txt` – tries to search “needle” from root, and since that will produce a lot of “Permission denied” errors on stderr, those are collected into errors.txt.
* `2>>` append stderr to a file (rather than overwrite).
* You can redirect both stdout and stderr:

  * One way: `command > output_and_errors.txt 2>&1` – this first sends stdout to the file, then `2>&1` says “now send stderr (2) to wherever stdout (1) is going” (which is that file). The order matters; `2>&1 > file` would not work as intended because you need to set stdout target before duplicating stderr to it.
  * Shortcut in Bash: `&>` redirects both stdout and stderr to the same target. For example, `command &> allout.txt` (this is not POSIX but a Bash convenience).
* `stdin` (standard input) redirection:

  * `<` can feed a file as input to a command. For example: `mail -s "Subject" user@example.com < email_body.txt` – this runs the `mail` command and takes the content of email\_body.txt as the message body from stdin.
  * Many commands also accept filename arguments so `<` is less used than output redirection, but it’s good to know.
* **Here documents** (`<<`) and **here strings** (`<<<`) allow feeding literal text into commands, but that’s more advanced scripting (e.g., `cat <<EOF ... EOF` to feed multiple lines). Possibly beyond essentials scope.

**Examples of redirection:**

```bash
# Save output, and errors separately
find / -name "myfile.txt" > output.txt 2> errors.txt

# Append output to an existing log file
df -h >> system_status.log

# Discard errors by redirecting to the special null device
make 2> /dev/null

# Combine stdout and stderr to one file
python myscript.py > script.log 2>&1
```

`/dev/null` is a special file that throws away anything written to it (bit bucket). Redirecting to `/dev/null` is a way to ignore output or errors.

Understanding redirection is crucial for command-line efficiency and writing scripts. It allows capturing output for later review or combining outputs from commands.

---

## Shell Scripting Basics

A shell script is a text file containing a sequence of shell commands that can be executed as a program. Scripting allows you to automate tasks. Here we cover the basics of writing and running shell scripts, as well as a brief look at text editors commonly used to create those scripts.

### Editing Text Files (nano and vi/vim)

Two common text editors available on most Linux systems are **nano** and **vi** (or its improved version, **vim**). Knowing at least basic operation of one editor is important for editing configuration files or writing scripts.

**nano:** A simple, user-friendly command-line editor. It shows key shortcuts at the bottom of the screen.

* Launch it by `nano filename` (if file doesn’t exist, it will start a new one).
* Basic commands (Ctrl + key):

  * `Ctrl+O` (Write Out) to save the file (it will prompt for confirmation or filename).
  * `Ctrl+X` to exit (it will prompt to save if you have unsaved changes).
  * `Ctrl+K` to cut the current line (you can press it multiple times to cut multiple lines; think of it as cutting to a clipboard).
  * `Ctrl+U` to uncut (paste) the last cut text at the cursor position.
  * `Ctrl+W` to search for text (enter the search term, press Enter, and it finds the next occurrence).
  * `Ctrl+\\` (Ctrl + backslash) to search and replace.
  * `Ctrl+G` to bring up the help menu (which lists all commands).
  * Navigation: use arrow keys, PageUp/PageDown. Home/End keys to jump to start or end of line.
  * nano is modeless (you just type text to insert it).

**vi/vim:** A more powerful editor but with a steep learning curve because it has modes (command mode and insert mode).

* Launch by `vi filename` or `vim filename`.
* When vi/vim starts, you are in **command mode**. Keys you press are interpreted as editor commands (not inserted as text).
* To enter **insert mode** (to actually insert text), press `i` (insert at cursor), `a` (append after cursor), or `o` (open a new line below and insert). In insert mode, you can type text normally.
* Press `Esc` to return to command mode.
* Common command-mode commands:

  * `:w` – write (save) the file.
  * `:q` – quit. (Use `:q!` to quit without saving changes.)
  * `:wq` or `ZZ` – save and quit.
  * Movement in command mode: `h` (left), `j` (down), `k` (up), `l` (right) – one character movements. Arrow keys also usually work.
  * `dd` – delete (cut) the current line. `5dd` deletes 5 lines.
  * `yy` – yank (copy) the current line. `5yy` yanks 5 lines.
  * `p` – paste the yanked or deleted text.
  * `/text` – search forward for “text”. Use `n` to jump to next match.
  * `u` – undo last change. `Ctrl+R` – redo (in vim).
  * `x` – delete the character under the cursor.
  * `:set nu` – (optional) show line numbers, can be helpful.
* **vimtutor:** If you have vim installed, running `vimtutor` launches a tutorial that teaches vi basics in a step-by-step fashion. This is highly recommended if you plan to use vi.

For exam purposes, if you’re not comfortable with vi, you can stick to nano for editing tasks. But vi is ubiquitous (it’s almost always on any Unix/Linux system, whereas nano might not be), so it's wise to know at least how to quit vi (`:q!`).

### Writing and Running Shell Scripts

**Creating a script:** A shell script is just a text file with commands. By convention, they often have a `.sh` extension, but it’s not required. The first line of a script usually specifies the interpreter, which is known as the **shebang** line. For a Bash script, use:

```bash
#!/bin/bash
```

This must be the very first line (no leading spaces) to tell the system to execute this script with `/bin/bash`. If you omit it, the script might run with whatever your default shell is, which is often bash anyway – but it’s good practice to include it.

**Example script:** Create a file `hello.sh` with the following content:

```bash
#!/bin/bash
# This is a comment - the shell will ignore anything after a # (except the shebang line above).

echo "Hello, $USER! Today is $(date)."
```

This script greets the user and shows the date.

**Making it executable:** Save the file, then run `chmod +x hello.sh` to give the execute permission. Now you can run it with `./hello.sh` (assuming you’re in the directory where the file is). If the directory is not in your PATH, you need `./`; otherwise put the script in a directory that is in PATH or call it via full path.

**Script arguments:** Scripts can accept arguments just like commands. Inside the script, special variables are used:

* `$0` is the script’s name.
* `$1` is the first argument, `$2` the second, and so on.
* `$#` is the number of arguments passed.
* `$@` or `$*` represent all arguments.
* `$?` is the exit status of the last command (0 means success, non-zero means error).
* In scripting, you often check `$?` or use constructs like `&&` and `||` to handle errors.

**Conditional execution:** The symbols `&&` and `||` can be used on the command line or in scripts to execute commands based on success or failure of the previous command:

* `command1 && command2` – run command2 **only if** command1 succeeded (exit status 0).
* `command1 || command2` – run command2 **only if** command1 failed (non-zero exit status).
* These can be chained. For example:
  `rm somefile && echo "Deleted successfully" || echo "Failed to delete"`
  This tries to remove "somefile". If that succeeds, it echoes success. If it fails, it echoes failure. (Be cautious: if `rm` succeeds, the `||` part won't run at all, because the chain stops after a successful `&&`; if `rm` fails, the `&&` part is skipped and the `||` part runs.)

**Conditional statements in scripts:** Bash supports if-else constructs and loops.

* Basic **if**:

  ```bash
  if [ condition ]; then
      # commands if condition is true
  fi
  ```

  Note: There must be a space after `[` and before `]` in the condition. Often we use `test` or `[` which is a shell built-in for conditions.

* **if-else**:

  ```bash
  if [ condition ]; then
      # true branch
  else
      # false branch
  fi
  ```

  You can also add `elif [ cond2 ]; then` for else-if.

* **Examples of conditions**:

  * `[ -f filename ]` checks if a file exists and is a regular file.
  * `[ -d dirname ]` checks if a directory exists.
  * `[ "$VAR" = "value" ]` (or `==` in \[\[ ]]) checks string equality.
  * `[ $NUM -gt 5 ]` numeric comparison (here greater than 5). Use `-lt`, `-ge`, `-eq`, `-ne`, etc., for other comparisons.
  * `[ -z "$VAR" ]` true if VAR is empty (zero length).
  * Combine with `&&` or `||` inside \[ ] as `-a` (and) and `-o` (or) for test, or better use `[[ ]]` extended test in Bash which allows `&&` and `||`.

* **Loop** (for example, a simple for loop):

  ```bash
  for i in 1 2 3 4 5; do
      echo "Number $i"
  done
  ```

  This will loop through the list `1 2 3 4 5` and execute the echo each time.
  Another common form:

  ```bash
  for file in *.txt; do 
      echo "Processing $file"
  done
  ```

  This loops through all files ending in .txt in the directory.

* **While loop:**

  ```bash
  while [ condition ]; do
      # commands
      # typically update something that affects condition
  done
  ```

  Example:

  ```bash
  COUNT=5
  while [ $COUNT -gt 0 ]; do
      echo "$COUNT..."
      COUNT=$((COUNT - 1))
  done
  echo "Liftoff!"
  ```

* **Shebang nuance:** You can write scripts for other shells or languages by changing the interpreter in the shebang (e.g., `#!/bin/python` for a Python script, if exam touches that concept).

Let’s illustrate a simple practical script example that uses some of these concepts:

**Example bash script:**

```bash
#!/bin/bash
# Script: find_and_list.sh
# Usage: ./find_and_list.sh <directory> <output_file>
# Description: List contents of a directory and save to a file, with error handling.

LOCATION=$1       # first argument: directory to list
OUTPUTFILE=$2     # second argument: file to save output

if [ -z "$LOCATION" ]; then
    echo "Error: No directory specified."
    echo "Usage: $0 <directory> <output_file>"
    exit 1   # exit with status 1 indicating error
fi

if [ -z "$OUTPUTFILE" ]; then
    echo "Error: No output file specified."
    echo "Usage: $0 <directory> <output_file>"
    exit 1
fi

if [ ! -d "$LOCATION" ]; then
    echo "Error: Directory '$LOCATION' not found."
    exit 1
fi

ls -lh "$LOCATION" > "$OUTPUTFILE"
if [ $? -eq 0 ]; then
    echo "Directory listing of $LOCATION saved to $OUTPUTFILE."
    # Display the contents of the output file as confirmation
    echo "Preview:"
    head -n 5 "$OUTPUTFILE"
else
    echo "Failed to list directory $LOCATION." >&2
    exit 1
fi
```

This script takes two arguments, checks for errors (missing args or invalid directory), then lists the directory contents into the output file. `$?` is used to check if `ls` succeeded. It demonstrates `if`, argument handling, and basic file test `[ -d ]`.

To run the script above in an exam environment, you would create the file, ensure it’s executable, and run it with appropriate arguments. Always test your script with different scenarios if possible.

Even if you don’t memorize all syntax, remember:

* Include `#!/bin/bash` in scripts.
* Use `chmod +x` to make script executable.
* Use `$1`, `$2`, ... for arguments; `"$@"` for all args.
* Use `[` or `[[` for conditions and be careful with spaces.
* Close blocks with `fi` for if, `done` for loops.
* Comments start with `#` (ignored by shell).
* You can exit with a status (0 for success, non-zero for error) using `exit N`.

Shell scripting is a big topic, but these basics will cover simple tasks such as writing an install script, a backup script, etc., which might be relevant.

---

## Understanding Computer Hardware and System Resources

Linux provides various commands to view hardware information and system resource usage. Although hardware management is often automatic, as an admin you should know how to query this information.

### Checking CPU, Memory, and Hardware Info

* **CPU Info:** The file `/proc/cpuinfo` contains details about the processors (CPU) in the system (vendor, model, speed, cores, etc.). Instead of opening it directly, a quick way is:

  ```bash
  $ grep "model name" /proc/cpuinfo
  ```

  to see the CPU model. But there's also a command:

  * `lscpu` – lists CPU architecture info in a pretty format.

* **Memory (RAM) Info:**

  * `free -h` – displays the amount of free and used memory (RAM) and swap in the system (the `-h` flag gives human-readable output in MB/GB). This is a quick way to see memory usage. Key lines:

    * **Mem:** shows total, used, and free physical memory.
    * **Swap:** shows total, used, free swap space.
    * The “-/+ buffers/cache” line (in older output) or interpretation is essentially how much memory is available for applications (accounting for Linux’s tendency to use spare memory for caching). In newer `free` (with `-h`), they directly list **available** memory which estimates truly free memory for apps.

* **Disk/Storage Info:**

  * `lsblk` – lists block devices (drives and their partitions) in a tree form. It shows the name (e.g., sda, sda1), size, type (disk/part), and mountpoints if any. This is very handy to see all attached disks and how they’re divided into partitions.
  * `df -h` – reports disk space usage for mounted filesystems. It shows device, total size, used, available, and mount point for each. The `-h` makes sizes human-readable.
  * `fdisk -l` – if run as root, lists partition tables of disks (legacy MBR and GPT info). Good for detailed view on how disks are partitioned. (Alternatively `gdisk -l` for GPT).

* **Device Inventory:**

  * `lspci` – lists all PCI devices (like system peripherals: network cards, sound cards, USB controllers, etc).
  * `lsusb` – lists USB devices connected.
  * `lshw` – a comprehensive hardware listing (if installed). It can output all hardware details.
  * `dmidecode` – dumps system hardware info from the BIOS/firmware. This can show details like BIOS version, serial numbers, RAM slot info, etc. Usually needs root to get full info.

* **Motherboard/BIOS:**

  * `dmidecode` as mentioned provides info on these, including system manufacturer, product name, BIOS version, etc.

* **Graphics:**

  * `lspci | grep -i vga` to find the graphics card info (most graphics cards are PCI devices).

* **Overall System:**

  * `uname -a` – prints kernel name, network node (hostname), kernel release & version, machine hardware name, etc. For just kernel version: `uname -r`.
  * `hostnamectl` – on modern systems, this gives hostname and also the Operating System name and kernel, architecture (works on systemd-based distros).

**Device naming conventions:**

* Hard disks in Linux might appear as `/dev/sda`, `/dev/sdb`, etc. (“sd” stands for SCSI/SATA disk, basically any modern block storage). The first drive is sda, second is sdb, and so on. Partitions on those drives are numbered: sda1, sda2, etc.
* Newer NVMe SSDs appear as `/dev/nvme0n1` (and partitions nvme0n1p1, etc.).
* Removable USB drives also often show as /dev/sdX when plugged in.
* Optical drives might show as /dev/sr0 or similar.
* `df -h` will list mount points like `/` or `/home` and the device they’re on (like /dev/sda1).
* `mount` command (with no args) prints all currently mounted filesystems along with device and options.

### Monitoring Running Processes and System Load

Linux is a multi-tasking OS, and processes (running programs) can be monitored and controlled.

* **ps (process status):** This command snapshots the running processes. Common usage:

  * `ps -ef` – list all processes with full details (UID, PID, parent PID, start time, CPU, command, etc). This is often piped to grep to find a specific process by name, e.g. `ps -ef | grep nginx`.
  * `ps -eH` – list all processes in a hierarchical format (showing the parent-child relationship through indentation).
  * `ps -u <user>` – show processes for a specific user.
  * `ps aux` – alternative BSD-style flags, similar info to -ef. (`ps aux | grep something` is a common idiom).

* **top:** An interactive process viewer that updates in real-time (every few seconds). Run `top` and you’ll see a live list of processes sorted by CPU usage by default.

  * In top, you can press:

    * `M` to sort by memory usage,
    * `P` to sort by CPU (which is default),
    * `N` to sort by process ID,
    * `q` to quit.
    * `h` for help.
    * `k` then enter a PID to kill that process (it will prompt for signal,  default is 15 or you can use 9 for force kill).
    * `1` toggles display of each CPU core’s usage.
    * `Shift + E` (in some implementations) might cycle memory units (KB -> MB).
  * `top` is great to see if the system is under load, which processes are consuming CPU or memory, etc.

* **htop:** If available (not always installed by default), it's an improved version of top with nicer UI and navigation (scrolling, etc.). But for exam basics, top is typically referenced.

* **kill:** Used to send a signal to a process, usually to terminate it.

  * Usage: `kill PID` sends the default TERM signal to politely ask process to terminate.
  * `kill -9 PID` sends SIGKILL which the process cannot ignore – it will be terminated forcefully (if possible). Use -9 if a normal kill doesn’t work.
  * `kill -HUP PID` (SIGHUP) often causes a process to reload configuration (daemon programs often catch HUP to reload configs without restarting fully).
  * There are also commands `pkill` (kill by process name pattern) and `killall` (kill all processes by name), but be cautious with those (e.g., `killall apache2`).

* **System load and uptime:**

  * `uptime` – shows how long the system has been running, how many users are logged in, and the load averages (1, 5, 15 minute average of CPU load). Load average roughly indicates how many processes are competing for CPU. (For example, "load average: 0.50, 1.20, 1.00" means in the last 1 minute average 0.5 processes running or waiting on CPU, etc.)
  * The first part of `uptime` output is similar to `who -b` which shows last boot time.

* **Process management example:** If a program freezes, you might:

  1. Use `ps` or `pgrep` to find its PID.
  2. Try `kill PID` (graceful shutdown).
  3. If after a bit it’s still there, `kill -9 PID`.
  4. If multiple processes: `pkill name` or `killall name` can target by name.

### System Logs and dmesg

Logs are crucial for troubleshooting:

* Most system logs reside in **/var/log/**. Common ones:

  * `/var/log/syslog` or `/var/log/messages` – the main system log (depends on distro: Ubuntu/Debian uses `syslog`, RHEL/CentOS uses `messages`). This contains general messages from the kernel and system services.
  * `/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (Red Hat) – authentication and security related logs (e.g., login attempts, sudo usage).
  * `/var/log/kern.log` – kernel-specific messages (if separated).
  * `/var/log/dmesg` – boot-time kernel messages (some distros).
  * Many services have their own logs, often in subdirectories (e.g., `/var/log/apache2/error.log` for Apache, or `/var/log/mysql/` for MySQL logs).

* **dmesg:** This command prints the kernel ring buffer messages – basically messages from the kernel (especially during boot, or when hardware events occur). Running `dmesg` will show a lot of info about hardware initialization, driver messages, etc. It’s useful to run `dmesg` right after plugging in a device to see if the kernel recognized it. Also good for diagnosing hardware issues.

  * `dmesg | less` if you need to scroll.
  * `dmesg | grep -i usb` to filter for USB events, for example.

* **syslog vs klogd:** In classic setups, `syslogd`/`rsyslogd` handles logging for user-space applications (and also receives kernel logs forwarded by klogd). `klogd` is a daemon that reads kernel messages and passes them to syslog. On modern systems, `systemd-journald` may handle logs.

  * If `rsyslog` or a similar system is running, it typically writes logs to files as mentioned above.
  * The **boot log** (if enabled) might be in `/var/log/boot.log`.

* **Viewing logs:** Use `tail -f` to watch a log as it grows:

  ```bash
  sudo tail -f /var/log/syslog
  ```

  (Syslog may require root to read in some distros).

* **Searching logs:** You can use `grep` to find relevant entries, e.g.:

  ```bash
  grep -i "error" /var/log/syslog
  grep "timed out" /var/log/syslog
  ```

* Logs rotation: Many distros use a utility (`logrotate`) to rotate logs (e.g., keep logs for a week or a month, then compress old ones). So you might see files like `/var/log/syslog.1`, `syslog.2.gz` etc.

Knowing log locations and `dmesg` is valuable for debugging system issues (like “why is my network interface not coming up?” – check dmesg or logs to see if there was an error).

---

## Networking Fundamentals

Linux has robust networking capabilities and a host of command-line tools to configure and troubleshoot network connections. Key concepts include checking IP configuration, connectivity tests, DNS lookup, and understanding how to add routes.

### Viewing Network Configuration

* **ifconfig** – (interface config) is a traditional tool to view or configure network interfaces. It’s considered deprecated on modern Linux in favor of the `ip` command, but it’s still widely known.

  * Running `ifconfig` (as root or with no arguments) will list active network interfaces and their IP addresses, netmasks, etc.
  * `ifconfig eth0 up` or `ifconfig eth0 down` would enable/disable an interface.
  * On newer systems, `ifconfig` might not be installed by default, replaced by:

* **ip command** (from `iproute2` suite):

  * `ip addr show` – shows all interfaces and their IP addresses (both IPv4 and IPv6).
  * `ip addr show dev eth0` – shows details for a specific device.
  * `ip link show` – shows interfaces at a lower (link) level (useful to see if an interface is up/down).
  * `ip route show` – displays the routing table (which route is used for which network).
  * `ip neigh` – shows ARP table (neighbor table).
  * The `ip` command can also be used to set addresses: e.g., `ip addr add 192.168.1.10/24 dev eth0` to set an IP, and `ip link set eth0 up` to bring it up. For the exam, knowing how to display info is primary.

* **Network interfaces** typically have names like:

  * `eth0`, `eth1` (classic wired Ethernet interfaces),
  * `wlan0` (wireless LAN),
  * `enp3s0` or `ens33` (newer predictable naming: `en` for Ethernet, `wl` for wireless, with hints of location on bus).
  * `lo` – the loopback interface (always 127.0.0.1 for IPv4).

* **Check interface config:**

  ```bash
  ip addr show
  ```

  Look for an interface with a valid IP (e.g., `inet 192.168.1.100/24`).

* **Enable/disable interface (temporary):**

  ```bash
  sudo ip link set eth0 up
  sudo ip link set eth0 down
  ```

  (Replace eth0 with your interface name; requires root/sudo.)

* **Check connection (ping):** Use `ping` to test connectivity to another host:

  * `ping 8.8.8.8` – sends ICMP echo requests to 8.8.8.8 (Google DNS). If replies come, network connectivity is working. Ctrl+C to stop pinging.
  * `ping -c 4 hostname` – ping a host 4 times then stop (the `-c` count option).
  * If ping by IP works but not by name, it’s a DNS issue.

* **DNS lookup:**

  * `nslookup hostname` – queries the default DNS server to resolve a name to IP or vice versa. (This tool is a bit outdated but straightforward; it might warn that it’s deprecated in favor of `dig`).
  * `dig hostname` – more advanced DNS lookup tool. `dig hostname A` asks for the A record (IPv4). `dig @8.8.8.8 example.com ANY` queries Google’s DNS for all records of example.com.
  * `host hostname` – a simple DNS lookup utility as well.
  * `/etc/resolv.conf` – file that contains the DNS server addresses the system will query (nameserver entries). If you have `nameserver 8.8.8.8` in there, it means Google’s DNS is used. On some systems, this file is managed by NetworkManager or resolvconf, so direct editing might be overwritten.

* **Default route/gateway:** The "default route" is where traffic goes if it’s not destined for the local network. Usually your router’s IP. Use:

  * `ip route show` to see something like `default via 192.168.1.1 dev eth0`.
  * Older: `route -n` to see kernel routing table (shows destination 0.0.0.0/0 meaning default).
  * `netstat -rn` similarly (netstat is older combined tool).

### Common Networking Tools and Commands

| **Tool/Command**       | **Purpose**                                                                                                                                                                                                                             |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ping <host>`          | Check connectivity to a host. Sends ICMP echo requests. If you get replies, network is working. Use `-c <count>` to limit pings.                                                                                                        |
| `traceroute <host>`    | Trace the route packets take to reach a host (shows each hop/router on the way). Helpful to diagnose where connectivity might fail. If not installed, use `mtr` or on Windows `tracert`.                                                |
| `ifconfig` / `ip addr` | Display network interface configurations (IP addresses, etc.). Use `ip addr` (or older ifconfig).                                                                                                                                       |
| `route` / `ip route`   | Show or manipulate the routing table. `route -n` shows numeric addresses for routes. `ip route add ...` can add routes. (For exam, knowing how to read output is key.)                                                                  |
| `netstat`              | (Deprecated in favor of `ss` but still used) Display network connections, routing tables, interface stats, etc. Common uses: `netstat -rn` (routing table), `netstat -tulpn` (list listening ports and which program owns them).        |
| `ss`                   | Socket statistics – replacement for netstat for checking open ports and connections. `ss -tulpn` is analogous to the above netstat example.                                                                                             |
| `curl` or `wget`       | Tools to fetch content from URLs. `curl http://example.com` - fetches the page; `wget URL` - downloads a file. Useful to test if you can reach a web service, etc. (May or may not be in scope for essentials, but often you see them.) |
| `dig`                  | Perform DNS queries (as mentioned, more advanced than nslookup). Example: `dig example.com ANY` or `dig +short example.com A`.                                                                                                          |
| `arp` or `ip neigh`    | View the ARP table (mapping of IP addresses to MAC addresses on the local network).                                                                                                                                                     |

A quick word on **netstat** since it was mentioned in the notes:

* `netstat -a` – show all sockets (listening and established, both TCP and UDP).
* `netstat -tuln` – show listening (`-l`) TCP/UDP ports with numeric addresses (`-n` stops name resolution). This is useful to see what ports the system is listening on.
* `netstat -s` – show network protocol statistics.
* `netstat -rn` – show routing table (numeric).

As noted, `netstat` might not be present on minimal modern systems, replaced by `ss` (which has similar options, e.g., `ss -tuln`).

### Configuring Network Settings (Basics)

Usually, network is managed by your distro’s tools (NetworkManager or /etc network scripts), but it’s useful to know:

* **Static IP configuration:** If not using DHCP, you set a static IP by editing config files:

  * On Debian/Ubuntu, edit `/etc/network/interfaces` or create a config under `/etc/netplan/` (for newer Ubuntu releases using Netplan).
  * On RedHat/CentOS, edit files like `/etc/sysconfig/network-scripts/ifcfg-<interface>`. For example `/etc/sysconfig/network-scripts/ifcfg-eth0`. There you set `BOOTPROTO=static` and specify `IPADDR=`, `NETMASK=`, `GATEWAY=` etc.
  * In either case, after editing, you’d restart the network service or bring the interface down/up.
* **/etc/resolv.conf:** Set your DNS servers here by adding lines: `nameserver 8.8.8.8` (for example). However, if using a network manager service, it might overwrite this file.
* **Default gateway:** On Debian, in `/etc/network/interfaces` you’d add `gateway x.x.x.x`. On RedHat ifcfg scripts, `GATEWAY=x.x.x.x`.
* **Adding a route manually:** `sudo ip route add 10.0.0.0/8 via 192.168.1.254 dev eth0` – this would route traffic for 10.\* network through 192.168.1.254 on eth0. Remove with `ip route del ...`.
* **Bringing network up/down:** `systemctl restart networking` (Debian) or `systemctl restart NetworkManager` / `nmcli` usage, depending on environment.

For exam basics, focus on interpreting output (like "Which command shows the IP address of the system?" – answer: `ifconfig` or `ip addr`. "Where do you set DNS servers?" – `/etc/resolv.conf`. "How to test connectivity?" – `ping`, etc.)

### Example: Diagnosing a network issue

If you cannot reach a website:

1. Ping an IP (like 8.8.8.8). If that works but pinging `google.com` doesn’t, suspect DNS.
2. Check `/etc/resolv.conf` or try `dig google.com` to see if DNS resolves.
3. If ping to 8.8.8.8 fails, check `ip addr` to see if you have an IP. If not, maybe the interface is down or DHCP failed.
4. If IP is there, check `ip route` to see if a default gateway is set. If not, you won’t reach outside your LAN.
5. Check if your interface is up: `ip link show`.
6. Use `ping <gateway>` (your router IP). If that fails, local network issue (cable, wifi, etc).
7. Use `arp -n` to see if you have an entry for the gateway’s MAC.

Understanding each of those steps and commands is key to network troubleshooting on Linux.

---

## User and Group Management

Linux is a multi-user system. Users and groups are fundamental for permissions and security. Here’s how to manage user accounts, and what distinguishes the root user from others.

### Root and Standard Users

* **root**: This is the superuser/administrator account on Linux. It has UID (user ID) 0 and unrestricted access to the system. The root user can read, modify, or delete any file (bypassing file permissions), and can run any command. Because of this power, it’s used carefully – often you run as a normal user and use tools like `sudo` to execute single commands as root when needed.
* **Standard (regular) users**: These are non-root accounts, each with their own home directories (e.g., `/home/alice` for user alice) and limited privileges. They can typically only modify files in their own home and in world-writable places like /tmp (unless given additional privileges).

**User account information:**

* Basic user info is stored in the text file **`/etc/passwd`**. Despite the name, it no longer contains actual passwords (for security reasons). Each line of /etc/passwd represents one user and has fields separated by colons:

  ```
  username:password:UID:GID:GECOS:home_directory:login_shell
  ```

  For example:

  ```
  alice:x:1001:1001:Alice Doe:/home/alice:/bin/bash
  ```

  Here:

  * `username` is alice.
  * `password` field is `x` which means the actual hashed password is stored in /etc/shadow (x is a placeholder).
  * `UID` is 1001 (unique user ID number).
  * `GID` is 1001 (the primary group ID, typically a private group for the user).
  * `GECOS` is a comment field (often the full name).
  * `home_directory` is /home/alice.
  * `login_shell` is /bin/bash (the default shell when she logs in).
* **`/etc/shadow`** contains the hashed passwords and password-related settings. Only root can read this file. Each line corresponds to a user, starting with username, then the password hash (or `!` or `*` if the account is locked/no password). It also includes fields like last password change date, days until password expires, etc. For example:

  ```
  alice:$6$abc123...:18714:0:99999:7:::
  ```

  (That long string is a SHA-512 hashed password with salt.)
* **`/etc/group`** lists group definitions. Format: `groupname:password:GID:member1,member2,...`. Usually password is empty or `x` (shadow stores group passwords if any, seldom used). GID is group ID number. The member list is for supplementary group members.
  e.g.:

  ```
  developers:x:1010:alice,bob,charlie
  ```

  means there’s a group named developers with GID 1010, and alice, bob, charlie are members.

**User information commands:**

* `id username` – shows a user’s UID, primary GID, and supplementary groups. If you run just `id` (no username) it shows info for the current user.

  ```bash
  $ id alice
  uid=1001(alice) gid=1001(alice) groups=1001(alice),1010(developers),1011(hr)
  ```
* `groups username` – lists the groups that a user is a member of.
* `finger username` – if available (not installed by default on some distros for security/privacy), it displays information like user’s real name, home directory, last login time, etc. E.g.,

  ```
  $ finger alice
  Login: alice                            Name: Alice Doe
  Directory: /home/alice                  Shell: /bin/bash
  Last login: Mon Jul  5 09:10 on tty2
  ```
* `last username` – shows last login sessions for the user (from wtmp log).
* `who` – shows currently logged in users. Variations:

  * `who` by itself shows something like:

    ```
    alice    pts/0        2025-05-02 10:15 (192.168.1.50)
    ```

    (username, terminal, login time, and remote host if any).
  * `who -b` – shows last system boot time.
  * `who -r` – shows current runlevel (e.g., "run-level 5" for a system with GUI on runlevel 5).
  * `w` – shows who is logged in and what command they are running. It includes a system uptime line and a list of users with idle times and their current process.
* `last` – shows history of logins (also reboots entries). `last reboot` will show last reboot times.

**sudo vs su:**

* `su -` (substitute user) – switch to another user account, typically `su -` with no username defaults to root (it will prompt for root’s password). The `-` ensures it’s a login shell, meaning you get root’s environment (including PATH, home directory).
* `sudo command` – executes a single command with root privileges (or as another user if specified). This requires that your user is allowed to use sudo (configured in `/etc/sudoers` or in the `wheel` group etc.) and you will normally type *your own* password (not root’s) to confirm. The idea is to avoid logging in as root while still allowing certain users to run admin commands.
* The `/etc/sudoers` file (edited with `visudo`) defines which users can run what as root (or other users). Often, membership in a certain group (like `sudo` or `wheel`) grants full sudo access.

**User types:**

* System users vs regular users: UIDs below a certain threshold (like <1000 on many distros) are often reserved for system accounts (like `daemon`, `www-data`, etc. which run services). Regular user accounts start at 1000 or 500 depending on distro.
* There are also special users like `nobody` (very limited privileges) used for certain unprivileged processes.

### Creating and Managing Users and Groups

* **useradd** – create a new user.

  * Basic usage: `sudo useradd username`. This creates the user, but on some systems it might not create a home directory by default unless you use `-m`.
  * Common options:

    * `-m` – create a home directory for the user (under /home by default).
    * `-c "Full Name"` – comment field (typically the user’s real name).
    * `-s /bin/bash` – specify the login shell (if not default).
    * `-G group1,group2` – specify supplementary groups the user should belong to.
    * `-u UID` – manually assign a UID (if you have a specific one in mind).
    * `-d /custom/home/dir` – set a custom home directory path.
  * Example: `sudo useradd -m -c "John Doe" -s /bin/bash -G developers john`.
  * After useradd, you typically set the password: `sudo passwd john` (it will prompt to enter a new password).
  * The default settings for useradd (like default shell, base home directory, skeleton files, etc.) are usually defined in `/etc/default/useradd` and the skeleton files (like initial .bashrc, etc.) in `/etc/skel/`.

* **adduser** – (on Debian-based systems) a friendlier interactive script that does useradd and asks for info, sets up home, etc. On Red Hat, `useradd` itself does those if invoked properly.

* **userdel** – remove a user.

  * Usage: `sudo userdel username`. By default this will remove the user’s entry but leave their home directory and files intact.
  * To remove the user *and* their home directory, use `userdel -r username` (the `-r` stands for remove files).
  * Caution: Deleting a user does not automatically kill their running processes if any, or remove crontab entries, etc.

* **usermod** – modify an existing user.

  * Use to change user’s info like group membership, username, etc.
  * Examples:

    * `usermod -aG groupname username` – Add user to a supplementary group (the `-a` (append) is crucial when using `-G` to not remove existing groups).
    * `usermod -s /bin/zsh alice` – change alice’s login shell to zsh.
    * `usermod -l newname oldname` – change username (login name).
    * `usermod -d /new/home/path -m username` – move user’s home to a new path (with -m to move contents).

* **groupadd** – create a new group.

  * Usage: `sudo groupadd developers` – creates a group named developers (with a new GID).

* **groupdel** – delete a group (careful: files with that group as owner will then have a non-existent group ID).

* **passwd** – set or change a user’s password.

  * Normal user can run `passwd` to change their own password (they’ll be prompted for current password then new one).
  * Root can do `passwd username` to set a password for that account (or to reset).
  * `passwd -l username` locks the account (disables password login by putting `!` at start of hash in shadow).
  * `passwd -u username` unlocks the account.
  * `chage` is another tool to adjust password expiry settings (force password change intervals, etc).

**Files to know:**

* `/etc/login.defs` – defaults for user creation, password policies, etc.
* `/etc/skel/` – skeleton directory, contains files that get copied to new users’ home directories when created (like default .bash\_profile, .bashrc, etc).
* User login records:

  * `/var/log/lastlog` (view with `lastlog` command to see last login of each user),
  * `/var/log/wtmp` (for `last` command),
  * `/var/log/btmp` (failed login attempts, view with `lastb`).

**Permissions for home directories:**

* Typically, home dirs are created with owner = user, group = user (or some default group), and mode 700 or 755 depending on distro (700 means only user can access, 755 means world-readable but only user can write).
* On some systems, user private group scheme is used: each user has a primary group same as their username.

**Switching users:**

* `su - user` will switch to that user (prompting for that user’s password unless you’re root). If you are root, you can su to any account without password.
* `sudo -i` can simulate an interactive shell as root (similar to su -).
* `sudo su -` will effectively give you a root shell as well (first sudo prompts for your password if needed, then runs su as root which doesn’t prompt).

**Security tip:** It’s bad practice to routinely log in as root. Use `sudo` for specific tasks. Many distros disable direct root login over SSH by default (requiring you to login as user and sudo).

---

## File Permissions and Ownership

Linux file permissions determine who can read, write, or execute a file. Understanding the permission model is essential for managing security and access.

Every file and directory in Linux has an **owner user** and an **owner group**, and a set of permission bits that apply to the owner, the group, and others (everyone else).

### Understanding Permission Notation

When you list files with `ls -l`, the output looks like:

```
-rwxr-xr-x  1 alice  developers  1234 May  2 12:34 script.sh
drwxr-xr-x  2 alice  developers  4096 May  1 09:10 mydir
```

Breaking down the first column (permissions string) `-rwxr-xr-x`:

* It’s 10 characters: the first character indicates type of file (`-` for regular file, `d` for directory, `l` for symlink, etc).
* The remaining 9 characters are actually three groups of three:

  * `rwx` – permissions for the owner (user).
  * `r-x` – permissions for the group.
  * `r-x` – permissions for others (everyone else).

Each position in a triad corresponds to:

* `r` = read permission,
* `w` = write permission,
* `x` = execute permission (for a file, execute means the ability to run it as a program; for a directory, execute means the ability to *enter* it and access files inside, often called search permission for directories).

If a permission is absent, it’s represented by a dash `-` in that position.

So in the example:

* Owner (alice) has `rwx` on script.sh: can read, write, and execute it.
* Group (developers) has `r-x`: can read and execute, but not write.
* Others have `r-x`: can read/execute.

For `mydir` which is a directory with `drwxr-xr-x`:

* Owner can list (`r`), create/delete (`w`), and enter (`x`) the directory.
* Group/others can list and enter, but not create/delete.

**Execute on directories:** A directory needs `x` (and `r`) for a user to access files inside it. Without `x`, you cannot `cd` into it or access files even if you know the name. Without `r` but with `x`, you can enter it if you know it’s there, but you can’t list the contents.

**Ownership columns:** In `ls -l`, the next two columns after permissions are owner and group. In the example, owner is alice and group is developers.

**Numeric (octal) representation:**
Permissions are often represented as a three-digit octal number:

* r = 4, w = 2, x = 1. Sum them for each category.

  * rwx = 7 (4+2+1)
  * r-x = 5 (4+0+1)
  * rw- = 6 (4+2+0)
  * etc.
* So `rwxr-xr-x` corresponds to 7 5 5, often written 755.
* A common default for directories is 755 (owner full, others can read/enter but not write).
* A common default for files is 644 (owner rw, others r).
* 600 means only owner can read/write (private file).
* 700 means only owner can read/write/exec (private dir or script).
* 777 means everyone can do everything (generally avoid 777 except maybe for /tmp style sticky bit directories).

### Changing Permissions (chmod)

**chmod** (change mode) is the command to change permissions. There are two ways to use it: symbolic and numeric.

* **Symbolic mode:** You specify who (u = owner/user, g = group, o = others, a = all) and what changes.

  * Use `+` to add permission, `-` to remove, `=` to set exactly.
  * Examples:

    * `chmod u+x file.sh` – give execute permission to the owner.
    * `chmod go-w file.txt` – remove write permission from group and others for file.txt.
    * `chmod o= file.txt` – remove all permissions from others (no read, no write, no exec for others).
    * `chmod a+r file.txt` – give everyone read permission (so even if it didn’t exist, ensure user, group, others all have at least read).
    * `chmod u=rwx,g=rx,o=r myprog` – explicitly set owner perms to rwx, group to r-x, others to r-- (this would be 754 actually).
  * You can combine multiple in one command: `chmod g+w,o-x file` (adds write for group, removes execute for others).

* **Numeric mode:** You specify an octal number with usually 3 or 4 digits.

  * 3 digits (like 644, 755 as discussed) affect user, group, others.
  * 4th digit (making it 4 digits) can set special modes (setuid, setgid, sticky – see below).
  * Example: `chmod 600 secret.txt` – owner can read/write, nobody else can access.
  * \`chmod 755 /usr/local- **Numeric mode (octal):** You directly specify the three permission bits as a number. For example:
  * `chmod 755 file` sets owner = 7 (rwx), group = 5 (r-x), others = 5 (r-x). This is common for making a script executable by everyone (owner can edit it, others can run it).
  * `chmod 644 file` sets owner = 6 (rw-), group = 4 (r--), others = 4 (r--). Typical for a text file that the user can edit, others can only read.
  * `chmod 600 file` gives owner rw, and no permissions for anyone else (good for private config files like SSH keys).
  * You can use a leading `0` or `\` to indicate octal if needed in certain shells. But generally `chmod 600 filename` is understood.

Both symbolic and numeric modes are useful. Numeric is quicker if you know the exact permissions you want. Symbolic is useful to modify specific bits without altering others (e.g., `chmod o-r filename` removes others’ read permission, keeping other bits intact).

**Note:** Directories often use `x` (execute) bit differently – as mentioned, `x` on a directory allows entering that directory. Usually you give directories `rwx` for owner and `rx` for others as needed.

### Changing Ownership (chown and chgrp)

* **chown** – change the owner (and optionally group) of a file/directory.

  * Only root can change a file’s owner. (Users can’t give away their files to someone else on most systems unless root does it.)
  * Usage: `sudo chown newuser filename`. This sets the owner to `newuser` (and leaves group as it was).
  * To change owner and group together: `sudo chown newuser:newgroup filename`. You can also use `.` instead of `:` (e.g., `chown user.group file`).
  * To change just the group with chown, you can do `chown :newgroup filename` (omit the user before the colon).
  * The `-R` flag will recursively change ownership in a directory tree. e.g., `sudo chown -R alice:alice /home/alice` to ensure Alice owns everything in her home.
* **chgrp** – change group ownership of a file/directory.

  * Usage: `chgrp newgroup filename` (you must either be root or the owner of the file and a member of the target group to do this).
  * `-R` for recursive as well.

**Why care about groups?** Groups are a way to share files between users. For example, if users Alice and Bob are both in group `project`, a file owned by Alice\:project with mode 770 (rwx for owner and group, none for others) would allow both Alice and Bob to read/write it (since Bob is in the project group), but no one else on the system can access it. Properly setting group ownership and permissions is essential for collaboration in multi-user environments.

---

## Special Files and Directories

Linux has some special file system features and types of files that behave differently, as well as a special permission called the *sticky bit*. Two important topics are **links** (symbolic and hard links) and the sticky bit on world-writable directories.

### Symbolic and Hard Links

A **link** is essentially an additional name for a file. There are two kinds:

* **Hard link:** A direct directory entry pointing to the same inode (actual file content on disk) as another file. Hard links are like cloning a file entry – the two names are equal and both refer to the same data. There is no “original” vs “copy” concept; they are identical references. Because of this:

  * Hard links cannot span across different filesystems or drives (must be on the same partition, since they reference an inode).
  * You cannot hard-link directories (to avoid loops in the filesystem).
  * If you delete one hard link, the data is not deleted until all links are deleted (the file’s data stays as long as at least one name exists).
  * Example usage: `ln existing_file.txt hardlink_to_file.txt`
  * After this, `ls -l` will show the link count (the number after permissions and before owner) increased to 2 for both names. Both names will appear with the same inode number (use `ls -i` to see inode numbers).
* **Symbolic link (symlink):** A special file that points to another file by name. It’s like a shortcut or alias.

  * A symlink *can* span filesystems and can point to directories or files (even to a target that doesn’t exist yet, which will be broken until the target exists).
  * If you delete the target file, a symlink that pointed to it becomes a “broken link” (dangling pointer).
  * When you `ls -l`, a symlink is indicated by `l` as the first character and shows something like `link_name -> target_name`.
  * Example usage: `ln -s /path/to/real/file.txt linkname.txt`
  * If you open or execute `linkname.txt`, the system follows the link to the real file.
  * Removing a symlink does not remove the target file; it just removes the shortcut.
  * Symlinks have their own permissions (often `rwxrwxrwx` by default, which are largely ignored except the executable bit is needed to traverse a symlink to a directory in some cases). Generally, access is controlled by the target file’s permissions, not the symlink’s.

Use cases:

* Hard links are occasionally used to keep a file accessible under another name or in multiple places (e.g., having the same configuration file in two locations).
* Symlinks are extremely common for convenience: e.g., `/etc/alternatives` uses symlinks to point to default programs, or linking `/usr/bin/python` to a specific version, etc. They are also used to link libraries, and for making shortcuts to long paths.

**Creating links:**

```bash
ln original.txt duplicate_hardlink.txt       # hard link
ln -s /usr/local/nginx/conf nginx_conf_link  # symbolic link example
```

If you `cd` into a symlinked directory, your prompt might show the real path or the link path depending on how you cd (shell builtins can handle `cd -P` physical vs `cd -L` logical). This detail might be beyond what you need for an exam, but just know symlinks can point to directories.

### Sticky Bit on Directories (Restricted Deletion Flag)

The **sticky bit** is a special permission primarily used on directories. When set on a directory, it means that even if the directory is world-writable, **only the owner of a file (or root) can delete or rename that file within the directory**. This is most famously used for **/tmp**.

* **/tmp** is a world-writable directory (any user can create files there) but it has the sticky bit set (displayed as a `t` in the execute position for “others” in `ls -ld /tmp`). This prevents users from deleting each other’s temp files.
* Another example is `/var/tmp` and other shared directories where you want multiple users to write but not interfere with each other’s files.

Without sticky bit, if a directory is `rwxrwxrwx` (777), any user could delete or rename files there even if they don’t own them (because they have write on the directory itself, which controls file deletion). With sticky, they can only delete if they own the file.

**How to identify and set sticky bit:**

* `ls -ld /tmp` might show: `drwxrwxrwt  10 root root  ... /tmp`. The `t` at the end replaces what would be `x` for "others" and indicates sticky bit is on (for a directory with execute for others).
* To set sticky bit on a directory: `chmod +t dirname` or in octal add the `1` in front. E.g., `chmod 1777 /some/dir` will make it world-writable with sticky (the `1` is the sticky bit, resulting in perms `rwxrwxrwt`).
* To remove it: `chmod -t dirname` or use octal like `chmod 0777 dirname` (removing the leading 1).

**Special bits summary (for completeness):**

* The sticky bit’s historical use on files (for old executables in memory) is obsolete; today it’s only meaningful on directories.
* Two other special permission bits:

  * **Setuid** (`s` in the owner’s exec spot): If set on an **executable file**, when someone runs that program it runs with the file owner’s privileges. For instance, `/usr/bin/passwd` has setuid bit and is owned by root, so when a normal user runs `passwd` to change their password, the program runs as root to edit /etc/shadow. (Security caution: setuid programs must be carefully written to avoid exploitation.)
  * **Setgid** (`s` in the group exec spot): If set on an **executable**, the program runs with the file’s group privileges. If set on a **directory**, any file created inside will inherit that directory’s group (instead of the creator’s primary group), which is useful for collaborative directories so that all files stay group-owned by a particular group. (Many shared project folders have `chmod g+s` set.)
  * On `ls -l`, setuid files show `rws` for owner if owner has exec, or `rwS` (capital S) if setuid is on but owner doesn’t have execute. Similarly for setgid in the group section.
* To set setuid: `chmod u+s file`; to set setgid: `chmod g+s file_or_dir`. In octal, setuid = 4xxx, setgid = 2xxx, sticky = 1xxx (they can be combined, e.g., 4755 would be setuid + 755).
* **Example:** `/usr/bin/passwd` might appear as `-rwsr-xr-x root root ... passwd` (note the `s` in place of owner execute).

For exam focus, remember:

* **/tmp** is 1777 (world-writable + sticky) so that users can create temp files but not tamper with others’ files.
* The `t` in `ls -l` output for a directory indicates sticky bit.
* Use `chmod +t` to set it.
* Use cases: any shared directory like `/tmp`, `/var/tmp`, or perhaps a shared upload directory, should have sticky if multiple users have write access.

---

This guide covered a broad range of Linux essentials: package management, command line usage, file system layout, basic file operations, text processing, shell scripting, system management, networking, user management, permissions, and special filesystem features. It serves as a refresher for common commands and concepts.

For exam preparation, ensure you practice these commands hands-on. Understanding not just the theory but also how to use the commands with their options is key. With these fundamentals, you’ll be well-equipped to handle typical tasks and troubleshoot issues on a Linux system. Good luck with your Linux certification and remember: the more you experiment with these commands in a real shell, the more comfortable you’ll become!
