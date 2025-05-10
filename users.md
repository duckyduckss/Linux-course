# Linux User and Group Management – Exam Study Guide

## /etc/passwd and /etc/shadow: Structure and Purpose

**`/etc/passwd`** – This file is a plaintext user account database. It lists all user accounts (including special system accounts like `bin`, `lp`, `nobody`, etc.) along with basic information needed for user login. Each line in `/etc/passwd` represents one user. The file is world-readable (so programs can map user IDs to names), but only the superuser (root) can edit it. Without an entry here, a user cannot log in. For security, the actual password is **not** stored in this file (the password field usually just contains an `x` or `*` as a placeholder). For example, a typical entry looks like:

```
alice:x:1002:1005:Alice White:/home/alice:/bin/bash
```

This shows username `alice`, `x` indicating the password is stored elsewhere, UID `1002`, GID `1005`, GECOS (comment) "Alice White", home directory `/home/alice`, and login shell `/bin/bash`.

**`/etc/shadow`** – This file contains sensitive password information and related security settings for each user. It was introduced to enhance security by moving hashed passwords out of the world-readable `/etc/passwd` file. Only root (or processes with appropriate privileges) can read `/etc/shadow`. Each user in `/etc/passwd` has a corresponding line in `/etc/shadow`. In `/etc/passwd`’s password field you’ll typically see an `x` (or `*`), which means “check `/etc/shadow` for the real password”. Storing hashes in `/etc/shadow` (which is usually `600` permissions, owner root) prevents normal users from even viewing password hashes, improving overall system security.

## Fields in User Account Files

### Fields in `/etc/passwd`

Each line in `/etc/passwd` has **7 colon-separated fields**, in the format:

```
username:password:UID:GID:GECOS:home_directory:login_shell
```

Here’s what each field means:

* **Username (login name)** – The account name used to log in (e.g. `alice`). This is typically a short identifier for the user.
* **Password** – An `x` or `*` in this field indicates that the **encrypted password is stored in** `/etc/shadow` (and is not visible here). Historically, the hashed password was stored here, but modern systems use `x` as a placeholder and rely on `/etc/shadow` for security. (If this field is blank or `*`/`!`, it can indicate no password or a locked account in legacy setups, but on Linux an `x` is standard.)
* **UID (User ID)** – The numerical user ID of the account. For regular user accounts this is typically a non-zero number (e.g. 1000+ on many distros, with 0 reserved for root). System accounts often have UIDs in a lower range (see **UID/GID ranges** below).
* **GID (Primary Group ID)** – The numerical group ID for the user’s primary group. This corresponds to an entry in `/etc/group`. Every user has one primary group (and can belong to additional supplementary groups).
* **GECOS (Comment)** – The comment field (also called the GECOS field) which usually contains the user’s full name or other info (office, phone, etc.). It’s free-form text used for informational purposes. (Commas can separate subfields like name, room, phone in some conventions, but colons `:` are not allowed in this field.)
* **Home Directory** – The absolute path to the user’s home directory (e.g. `/home/alice`). This is where the user’s personal files and directories reside.
* **Login Shell** – The absolute path to the user’s default login shell program (e.g. `/bin/bash`). If this is set to a non-interactive shell (like `/usr/sbin/nologin` or `/bin/false`), the account is effectively disabled for interactive login.

### Fields in `/etc/shadow`

Each line in `/etc/shadow` has **8 fields** (with a 9th field reserved for future use), in the format:

```
username:encrypted_password:lastchg:min:max:warn:inactive:expire
```

These fields are:

* **Username** – The account name, which matches the username in `/etc/passwd`.
* **Encrypted Password** – The hashed password (often in MD5, SHA-512, etc. format). For example, it may look like `$6$<salt>$<hash>` if using SHA-512. Special cases:

  * If this field is blank (empty), **no password is required** to log in (highly insecure).
  * If it contains `*` or `!` (and no other characters), the account is **locked/disabled** (no login possible with a password). For instance, many system service accounts have `*` to indicate they can’t be used to log in.
  * If it starts with `!` followed by a hash, it means the password is locked **after having been set** – the `!` prevents login with the current hash until it’s removed (unlocking).
* **Last Password Change (lastchg)** – The date of the last password change, stored as the number of days since the Unix epoch (Jan 1, 1970). For example, `18010` might correspond to a specific date. This field is updated when the user (or admin) changes the password.
* **Minimum Days (min)** – The minimum number of days that must pass after a password change before the user is allowed to change it again. A value of `0` means no minimum enforced (the user could change again immediately).
* **Maximum Days (max)** – The maximum number of days a password is valid before it *expires* (user is forced to change it). For example, `90` means the password expires after 90 days. A common default is `99999` (roughly 273 years), effectively meaning **no forced expiration** by default.
* **Warning Days (warn)** – The number of days before password expiration that the user is warned that their password is about to expire. For instance, `7` means the user will start getting warnings 7 days prior to expiration upon login.
* **Inactive Days (inactive)** – The number of days after the password has expired that the account remains *active* (allowing login) before it is locked due to inactivity. For example, `30` means the user can still log in up to 30 days after password expiry (they’ll be forced to change password on login). `-1` or empty usually means this feature is disabled (no forced lockout after expiration).
* **Account Expiration Date (expire)** – The date on which the user account *expires* (gets disabled), expressed as days since epoch (or a date in `YYYY-MM-DD` format in some tools). After this date, the account is locked **regardless of password**. This is often empty for accounts that don’t have a set expiry. (Note: when the expiry date is reached, the account is locked but not removed; the user cannot log in past that date without admin intervention.)
* **Reserved** – (Optional) Historically a placeholder for future use or administrative flags. It’s usually blank. Modern systems typically have 9 fields total, with the last field unused.

**Summary:** The `/etc/passwd` file is for basic user info (name, UID/GID, home, shell) and is readable by all, while `/etc/shadow` stores the secure password hash and aging info (readable only by root). Together, these define user accounts on the system.

## Managing User Accounts (useradd, usermod, userdel)

Managing user accounts involves creating new users, modifying their properties, and deleting users when they’re no longer needed. The key commands are **`useradd`**, **`usermod`**, and **`userdel`**.

### Adding Users with `useradd`

The `useradd` command creates a new user account by adding entries to system files like `/etc/passwd`, `/etc/shadow`, and `/etc/group`. Only root (or an administrator with appropriate privileges) can run this command. Basic usage: `useradd [options] <username>`.

* By default (with no options), `useradd` will create the user with default settings defined in system config files (see **/etc/login.defs** and **/etc/default/useradd** below). For example, it will choose the next available UID and GID, create a home directory under `/home/<username>` (if configured to do so), assign a default shell (usually `/bin/bash`), and set the account’s password as locked until a password is assigned.

* When `useradd` succeeds, it doesn’t produce output. You should check the result by inspecting the last line of `/etc/passwd` or using commands like `tail -1 /etc/passwd`. After adding a user, you’ll see a new line for that user. For example, after `useradd test_user1`, you might see an entry like `test_user1:x:1005:1008::/home/test_user1:/bin/bash`.

* Initially, the new account’s password is **locked**. In `/etc/shadow`, the password field for a new user will typically be `!` (exclamation mark), indicating no valid password set (login disabled). You must set a password before the user can log in. Do this with the `passwd <username>` command. For example, `passwd test_user1` will prompt to enter a new password for that account. After setting a password, the `/etc/shadow` entry will contain the hash instead of `!`.

* **Default Settings:** The behavior of `useradd` is governed by defaults in **/etc/login.defs** and **/etc/default/useradd**. For instance, the default base home directory (usually `/home`), default shell, skeleton directory, UID/GID ranges, etc., are defined there. You can view the current defaults with `useradd -D`. For example:

  ```bash
  $ useradd -D  
  GROUP=100 HOME=/home INACTIVE=-1 EXPIRE= SHELL=/bin/bash SKEL=/etc/skel CREATE_MAIL_SPOOL=yes
  ```

  This output shows default group ID `100`, home base `/home`, no account expiry (`EXPIRE` empty), shell `/bin/bash`, skeleton dir `/etc/skel`, etc.. These correspond to entries in `/etc/default/useradd`. You can change a default by using `useradd -D` with an option (e.g., `useradd -D -s /bin/zsh` to set default shell to zsh) or by editing the config file.

* **Common `useradd` Options:** To override defaults when creating a user, you can specify options:

  * `-u <UID>` – Manually specify a User ID for the new account.
  * `-g <group>` – Specify the primary group name or GID for the user. The group must exist already. If not given, on many Linux systems **User Private Group (UPG)** behavior will create a new group with the same name as the user by default (see **User Private Groups** below). Otherwise, it might default to a generic group (like `users` with GID 100).
  * `-G <group1,group2,...>` – Specify a comma-separated list of supplementary (secondary) groups the user should be added to.
  * `-d <dir>` – Specify a custom home directory path for the user. If not provided, the default is usually `/home/<username>`.
  * `-m` – Instruct `useradd` to **create the home directory** if it does not exist, copying the skeleton files into it. On many modern systems, `useradd` does this by default (depending on settings in `/etc/default/useradd`), but `-m` ensures the home is created.
  * `-k <skeldir>` – Use an alternate skeleton directory for populating the new home (implies `-m`). Default is `/etc/skel`.
  * `-s <shell>` – Set the login shell for the user (e.g. `-s /bin/zsh`). Default is typically `/bin/bash` on Linux.
  * `-c "<comment>"` – Set the GECOS/comment field for the user (e.g. full name).
  * `-e <YYYY-MM-DD>` – Set an account expiration date (the account will be disabled after this date).
  * `-f <INACTIVE_DAYS>` – Set the inactive days (grace period after password expiry before the account locks).
  * `-N` – On systems with User Private Groups, this option **disables** the automatic creation of a group with the same name as the user. The user will then be placed in the specified primary group (`-g`) or the default group if `-g` isn’t used.

  **Example:**

  ```bash
  # useradd -m -s /bin/bash -g team -c "Alice White" alice
  ```

  This creates user `alice`, with a home directory `/home/alice` (created and populated with defaults), login shell `/bin/bash`, primary group `team` (which must exist), and comment “Alice White”. If the system uses UPG by default, using `-g team` along with `-N` would prevent making a group named alice. In this case we assumed we want `team` as primary group.

* **Note:** Creating a user is just the first step. Always remember to set the user’s password (`passwd username`) or otherwise configure authentication (like SSH keys) after creating the account, since by default the account will be locked with no password.

### Modifying Users with `usermod`

The `usermod` command is used to change existing user account properties – essentially editing the user’s entry in system files in a safe way. This is preferable to manually editing `/etc/passwd` or `/etc/shadow`. Only root can use `usermod`. Usage: `usermod [options] <username>`.

Common `usermod` options and tasks:

* **Change Home Directory:** `-d <new_home_dir>` will change the home directory path stored in `/etc/passwd`. By default, `usermod -d` **does not** move the user’s existing files to the new location; it only updates the path. You can use `-d <dir> -m` together to move the content from the old home to the new directory automatically (ensure the new directory doesn’t exist or is empty). Otherwise, you must manually create the directory and move files. Example:

  ```bash
  # usermod -d /home/qa/test_user1 test_user1 
  ```

  (Then manually move files or use `-m` to auto-migrate content.)
* **Change Login Shell:** `-s <shell>` will change the user’s login shell. For example, `usermod -s /bin/csh alice` changes Alice’s shell to C shell. Make sure the shell is listed in `/etc/shells`.
* **Change Comment (GECOS):** `-c "<text>"` changes the comment/GECOS field in `/etc/passwd`. E.g., update a user’s full name or other info: `usermod -c "Engineering Team" alice`.
* **Change Username:** `-l <new_login>` allows renaming the user (login name). This will change the username in `/etc/passwd` (and related files). Typically you might also want to rename the home directory manually to match and update ownership of files. (Use with caution; not shown in the provided material, but it’s a known option.)
* **Change Primary Group:** `-g <group>` changes the user’s primary group to the specified existing group. The group must exist beforehand – `usermod` will not create a new group. For example, `usermod -g developers alice` makes “developers” Alice’s primary group. (Her old primary group will remain on the system but she will no longer belong to it unless it was also a secondary membership.)
* **Add Secondary (Supplementary) Groups:** `-G <group1,group2,...>` sets the list of secondary groups the user is a member of. **Important:** By default, using `-G` replaces the user’s entire supplementary group list with the new list. To *add* a group (without removing existing ones), use `-a` (append) together with `-G`. For example:

  ```bash
  # usermod -a -G wheel,developers alice
  ```

  This adds Alice to the “wheel” and “developers” groups in addition to any groups she’s already in. Without the `-a` flag, Alice would be removed from any secondary group not listed in the `-G` list.
* **Lock/Unlock a User Account:** `-L` locks the account password (renders it unusable) and `-U` unlocks it. This works by prepending or removing an `!` in the `/etc/shadow` password field (same effect as the `passwd -l/-u` commands). See **Locking/Unlocking Accounts** below for details.
* **Expire Password/Force Reset:** `-e <date>` can set an account expiration date (after which login is disabled). Or `-e 0` (or `-e 1`) can be used to force the user to change password at next login by expiring their password (similar to `passwd -e`) – in practice, `usermod -e` with a date is usually for account expiration, while `passwd -e` is commonly used to expire the password for next login.
* **Other options:** `-p <encrypted_pw>` can set a password hash directly (not commonly used, `passwd` is safer). `-f <days>` can set the inactive days after password expiry. `-u <UID>` can change the user’s UID (not recommended unless necessary; you must adjust ownership of that user’s files to the new UID manually or with tools).

Always verify changes by inspecting the relevant files (`/etc/passwd`, `/etc/shadow`, `/etc/group`) or using commands like `id <user>` or `getent` after modifying. For example, after changing a shell or GECOS, you can `grep username /etc/passwd` to confirm.

*Note:* On some systems (especially Red Hat-based), `usermod` will refuse certain changes (like UID or home directory) if the user is currently logged in. This is a safety feature to avoid conflicts.

### Deleting Users with `userdel`

The `userdel` command removes a user account from the system. This means it will delete the user’s entry from `/etc/passwd` and `/etc/shadow`, and optionally remove the user’s home directory and mail spool. Usage: `userdel [options] <username>`.

Key points:

* Running `userdel <username>` will remove the user’s login from the system’s account files. Specifically, it **deletes the user’s lines from** `/etc/passwd` and `/etc/shadow`, and it will remove any reference to that user in the `/etc/group` file (i.e., the user is removed from any groups). However, by default it **will not delete** the user’s home directory or any files owned by the user on the system.
* To also remove the user’s home directory and mail spool, use the `-r` option: `userdel -r <username>`. This will recursively delete the user’s home directory and the mail spool file (usually under `/var/mail/username`). **Note:** `-r` will **not** hunt down and delete files owned by the user that reside outside the home directory. Any files elsewhere (e.g. in `/tmp`, /var, project directories) remain and will still be owned by the now-removed UID. It’s good practice to find and clean up or reassign such files. For example, before deleting the user, you might do: `find / -user <username> -exec rm -i {} \;` (or chown them to another user) to avoid orphaned files.
* You cannot remove a user who is currently logged in or executing processes, unless you force it. If the user is logged in, `userdel` will normally return an error. In emergency cases (like immediate termination of an employee), you can use `userdel -f <username>` to force deletion. This option forces the account removal even if the user is still logged in. Be cautious with `-f`: processes owned by that user might continue running under the UID, and the home directory will remain unless `-r` is also used.
* After deleting a user, any files they owned will now be owned by an unknown UID (the numeric ID remains, but it’s not mapped to a username anymore). For example, if user `test_user` with UID 1005 owned some file, after deletion that file might appear owned by “1005” instead of a name. This is why it’s important to handle or remove such files as noted.
* **Primary group not removed:** `userdel` does **not** automatically remove the user’s primary group, even if it was a unique “private” group for that user. If you want to delete the associated group as well (say, in a user-private-group setup), you must run `groupdel <groupname>` separately. (The system will not allow deleting a group that is still listed as a primary group for some user, but once the user is deleted, that check is clear.)
* It’s wise to backup or archive the user’s data before deletion if needed. Some admins prefer to lock the account for a while (to prevent login) before deletion, in case data or access is needed by others, then remove after a grace period.

## User Private Groups (UPG) and Group Administration

Linux systems manage group memberships through the `/etc/group` and `/etc/gshadow` files. In many distributions, a **User Private Group (UPG)** scheme is used by default to simplify permissions management. Understanding UPG and how to manage groups is important.

### User Private Groups (UPG) Concept

Many Linux distros (including Red Hat, Fedora, Debian, Ubuntu) create a unique private group for each new user by default. This means when you add a user `alice`, the system also creates a group `alice` with the same name, and makes that the user’s primary group. The user is the only member of their private group. This behavior can be overridden (see `useradd -N`), but it’s the default on those systems.

* **Purpose:** Historically, Unix systems often put all users into a common group (like a `users` or `staff` group). That led to scenarios where files with group permissions open to that group were effectively accessible by everyone, since all users shared the group. UPG addresses this by giving each user their own group, so any files they create with group permissions are initially only accessible by them (because no one else shares that private group). In other words, having a unique primary group per user means the group permission on files is effectively equivalent to an extra personal permission for that user. This improves default file security when users forget to set restrictive permissions.
* **Drawback:** One unintended side effect is that users might become too accustomed to using broad “other” permissions to share files, which can reintroduce the original problem (if a user opens up permissions to let someone else access a file, they might grant access to all other users if done incorrectly). The proper solution for collaboration is to create dedicated groups for sets of users who need to share files, rather than relying on world-accessible permissions.
* **Group Creation:** With UPG enabled, `useradd` will automatically create the group. The group typically has the same name as the user and a unique GID. For example:

  ```bash
  # useradd test_user2 
  # grep test_user2 /etc/passwd
  test_user2:x:1007:1009::/home/test_user2:/bin/bash
  # tail -1 /etc/group
  test_user2:x:1009:
  ```

  Here, `test_user2` was created with UID 1007, and we see a group `test_user2` with GID 1009 was also created, and the user’s entry shows their primary GID as 1009. This is the UPG behavior in action.
* **Disabling UPG for a user:** If you want to assign a user to an existing group as primary (instead of a new private group), you can use `useradd -N -g <group>`. This will skip creating a new group. For instance, `useradd -N -g staff bob` would put bob’s primary group as `staff` (assuming it exists) and not make a `bob` group.
* **Group administration by users:** In some setups, you might make the user the administrator of their own private group so they can add others to their group if needed. Using `gpasswd -A <user> <group>` you can assign a user as a group administrator. For example, `gpasswd -A alice alice` could allow Alice to manage membership of her group. Then Alice could use `gpasswd -a <otheruser> alice` to add someone to her group. (This is an advanced usage; by default, only root can manage groups.)

Overall, UPG is mostly behind the scenes; just remember that on UPG systems every user gets their own group by default, so don’t be surprised to see matching usernames and group names in the account info.

### Group Management Commands (groupadd, groupmod, groupdel, gpasswd)

Groups are defined in the `/etc/group` file, which lists group accounts, their GIDs, and memberships. Key commands for managing groups:

* **Viewing Groups:** The `/etc/group` file can be viewed similarly to `/etc/passwd`. Each line shows a group name, an `x` (if a password is set in `/etc/gshadow`), the GID, and a comma-separated list of members. e.g., `developers:x:1010:alice,bob`. Use commands like `grep` or `getent group` (discussed later) to query it.

* **Add a Group – `groupadd`:** This command creates a new group on the system. Usage: `groupadd [options] <groupname>`. For example, `groupadd programmers` will create a group named “programmers” with the next available GID. By default, the system picks the lowest unused GID >= `GID_MIN` (defined in login.defs, often 1000 for regular groups). You can specify a GID with `-g <GID>` if you need a specific value. After creation, the new group appears in `/etc/group` (and `/etc/gshadow`). Example: after `groupadd programmers`, `/etc/group` might show `programmers:x:1008:`.

* **Modify a Group – `groupmod`:** This command changes an existing group’s properties. Common uses:

  * `groupmod -n <newname> <oldname>` to rename a group.
  * `groupmod -g <newGID> <group>` to change a group’s GID (numerical ID). Be cautious: you should update any file ownerships that reference the old GID, as they won’t automatically change. Usually it’s best to avoid changing GIDs unless necessary.
    Example: `groupmod -n c-programmers programmers` would rename the group “programmers” to “c-programmers” (useful if you named it wrong).

* **Delete a Group – `groupdel`:** Deletes a group from the system. Usage: `groupdel <groupname>`. This will remove the group’s entry from `/etc/group` (and `/etc/gshadow`). **Important:** You **cannot delete a group that is a user’s primary group** while that user exists. If you try (e.g., `groupdel alice` while user alice still has that as primary group), you’ll get an error “cannot remove the primary group of user ‘alice’”. You’d need to delete or modify the user first. It’s fine to delete groups that have members, but those members will lose that secondary group membership after deletion. Also, check for files on the system that have this group ownership. If a group that owns files is deleted, those files will show the old GID number instead of a name (since the name no longer exists). For example, if `/tmp/sample` is owned by a group `test_group` and you delete `test_group`, an `ls -l` will show the group as the numeric GID rather than a name. It’s usually wise to find and reassign or delete such files **before** removing the group.

* **Group Passwords and Administration – `gpasswd`:** Groups can have an associated password and group administrators, which are stored in `/etc/gshadow` (the shadow counterpart for groups). The `gpasswd` command is used to administer these:

  * `gpasswd <group>` (no options) – Set or change the group’s password (you’ll be prompted for a new password). Group passwords are an older mechanism, rarely used, that allow users not in the group to temporarily join the group by using the `newgrp` command (they will be prompted for this password). If you want to allow that, you can set a password. An exclamation mark `!` in the gshadow password field means the group is locked (no password), which is the default for most groups, effectively disallowing the use of `newgrp` by non-members.
  * `gpasswd -r <group>` – Remove the group’s password (i.e., lock it, so no password can be used to join).
  * `gpasswd -A <user1,user2,...> <group>` – Assign one or more **group administrator(s)** for the group. Group admins can add (`-a`) or remove (`-d`) members without being root.
  * `gpasswd -a <user> <group>` – Add a user to the group. (The user will appear in the group’s member list in `/etc/gshadow` and `/etc/group`.) Similarly, `gpasswd -d <user> <group>` will remove a user from the group.
  * Group administrators (set with `-A`) can also use these `gpasswd -a/-d` commands for their group (without root).

  *Example:*

  ```bash
  # gpasswd -A alice developers    # Make 'alice' an admin of 'developers' group
  # gpasswd -a bob developers     # Add 'bob' to 'developers' group (as root or alice if she's admin)
  ```

  After these, the `/etc/gshadow` entry for developers might look like:
  `developers:!:alice:bob,charlie` – meaning group `developers`, no password (`!`), admin is alice, members are bob and charlie.

* **/etc/gshadow file:** For completeness, know that each line in `/etc/gshadow` has the format:
  `groupname:encrypted_password:admin_list:member_list`. Example entry: `programmers:$1$nfffgggc$pGt#4:useradmin:sarah,mickey,dinesh`. Here `programmers` has a hashed password (meaning a group password is set), `useradmin` is the group admin, and `sarah,mickey,dinesh` are members who will not be asked for a password (they are direct members). Usually most groups have `!` for password (meaning no valid password), no admins, and list members if any.

In practice, group passwords are seldom used; typically an administrator just adds users to groups as needed (with `usermod -aG` or `gpasswd -a`). The `newgrp` command (discussed next) is influenced by these settings.

## Skeleton Directory and User Environment Setup (`/etc/skel`)

When a new user is created and their home directory is made, the system populates it with default files by copying from the **skeleton directory**: `/etc/skel`. This ensures every user starts with some useful default configuration files.

* **`/etc/skel` Purpose:** This directory contains a “skeleton” of files and directories that should be present in every new user’s home. For example, it usually contains files like `.bashrc`, `.profile`, `.bash_logout`, and possibly others. When you create a user with `useradd -m`, everything in `/etc/skel` is copied into the new home directory. This gives the user a baseline environment (like a default shell configuration, etc.) without the admin having to set it up manually each time.
* **Common files in `/etc/skel`:** You will typically find:

  * **`.bashrc`** – Bash startup script for interactive shells, containing alias definitions and other shell settings.
  * **`.profile`** (or `.bash_profile` depending on distro) – Script executed at login, often to set environment variables and run commands that should happen at login.
  * **`.bash_logout`** – Script run at logout, perhaps to clear the screen or other cleanup.
  * Other application-specific dotfiles can be placed here as needed (e.g., `.vimrc`, `.gitconfig`, etc., if you want all new users to have a starter config for Vim or Git).
* **Important Notes:**

  * `/etc/skel` is only used at the moment of account creation. If you add a file to `/etc/skel` later, it **will not** magically appear in existing users’ directories. It only affects new accounts.
  * The files copied into the user’s home directory will be owned by the new user (the ownership is adjusted during copy). After that, the user can modify or delete them freely — they are their files.
  * If you want to make a global change (like add a new dotfile for all users), you’d have to manually distribute it or inform users, because placing it in skel only helps for future accounts.
* **Customizing `/etc/skel`:** As an admin, you can add whatever default configuration you want new users to have. For example, if you want every new user to start with some company-specific bookmarks in Firefox, you could set up a template Firefox profile in skel. The provided material describes a process: create a temporary user, set up Firefox bookmarks, then copy that user’s `~/.mozilla` directory into `/etc/skel`. Then every new user will get those Firefox bookmarks by default, since the `.mozilla` directory is copied over.
* **Multiple Skeleton Directories:** You can actually have different skeleton directories for different types of users. The `useradd` command with the `-k` option allows specifying an alternate skel directory. For instance, you might prepare `/etc/skel_dev` for developer accounts, `/etc/skel_sales` for sales, etc., each with different environment setups. Then use `useradd -m -k /etc/skel_dev devuser1` to create a new user with the developer skel files. This technique helps if you want to customize environments based on user roles at account creation time.

In summary, remember that `/etc/skel` is the template for new user homes. It ensures users have a sane starting environment (with login scripts, etc.). As an admin, keep it updated with any files you want all new users to have.

## Viewing and Modifying Group Membership (groups, getent, newgrp)

Understanding which groups a user belongs to is essential for managing permissions. Several commands help view and change group memberships:

* **`groups` (command):** This command, when run by itself, lists the groups the current user is a member of. You can also specify a username (e.g. `groups alice`) to see that user’s groups. For example:

  ```bash
  $ groups       # run by user sysadmin
  sysadmin adm sudo
  ```

  The output shows the user’s primary group first (here `sysadmin`) and then any secondary groups (`adm`, `sudo` in this case). If you do `groups root`, you might see `root : root` (meaning root’s primary group is root and no other groups). This command is handy for quick checks of membership.
* **`getent` (get entries):** This is a more general query tool for system databases, including groups. The syntax is `getent <database> <key>`. For example, `getent group <groupname>` will retrieve the entry from the group database (which includes `/etc/group` and possibly other sources like LDAP) for that group. One trick is you can use `getent group <username>` *if* the username corresponds to a group name (as in a user private group setup) to find that user’s default group entry. For instance, `getent group sysadmin` might output `sysadmin:x:1001:`, meaning there is a group named sysadmin with GID 1001 and no listed members (aside from the implied user). Generally:

  * `getent passwd alice` would show alice’s `/etc/passwd` entry.
  * `getent group developers` would show the developers group entry with all members listed.
  * `getent group alice` (when a group of that name exists, as with UPG) shows that group line, which essentially tells you alice’s primary group info (GID, etc.).

  The **difference between** `groups` and `getent`: The `groups` command shows the *current* groups of a user (and requires you to be that user or specify one). It will reflect any changes in the current login session (for example, if you used `newgrp`, see below). `getent group <name>` simply looks up the group database for a given name. If you give it a username that is also a group name, it returns the group record (which is effectively the user’s default primary group). Notably, `groups alice` will list all groups alice is in (with her current primary first), whereas `getent group alice` will only show the group named “alice” (which is her primary group record, and no secondary groups). In short, `groups` = user-centric view, `getent group` = group-centric lookup.
* **`newgrp`:** This command allows a logged-in user to change their current primary group during a login session, by starting a new shell with a different primary group. Usage: `newgrp <groupname>`. Suppose user `sysadmin` is in groups `sysadmin`(primary), `adm`, and `sudo`. If they run `newgrp adm`, their primary group in the new shell becomes `adm`. You can verify by running `groups` again: the `adm` group will now appear first in the list (indicating it’s primary for that session). This can be useful if the user wants any files they create in that shell to be group-owned by `adm` instead of their default group. To return to the original group, the user can exit that shell (end the `newgrp` session) and primary group reverts.

  * If the user is **not a member** of the group they are trying to `newgrp` into, the system will prompt for the group’s password (if one is set in `/etc/gshadow`). This is where group passwords (set via `gpasswd`) come into play. In modern practice, group passwords are rarely set, so typically only members of a group or an administrator can change into that group. If a password is set and provided correctly, the user effectively joins the group for that session.
  * Note: Changing group with `newgrp` does not change the user’s membership permanently; it’s just for that shell session. It’s often simpler for an admin to add a user to a group than for users to use group passwords.

**Use case example:** Suppose `alice` is a member of groups `alice` (primary) and `developers` (secondary). By default, files she creates are group-owned by `alice`. If she’s working on a project with the `developers` group, she might run `newgrp developers` to make `developers` her primary group, then create files which will be group-owned by `developers` without needing to manually chgrp them. When done, she exits the shell and is back to normal.

## Locking/Unlocking Accounts and Password Status

Sometimes you need to disable a user’s access without deleting the account, or check whether an account is locked. Linux provides ways to lock/unlock user passwords and to view password status:

* **Locking an account:** Use `passwd -l <username>` to **lock** a user’s password. This places an `!` in front of the encrypted password in `/etc/shadow`, which makes the hash invalid. For example, if `/etc/shadow` for user `sysadmin` had `"sysadmin:$6$abc...:..."`, after locking it becomes `"sysadmin:!$6$abc...:..."`. The `!` means no potential password will match (since the hash no longer starts with the proper prefix). Effectively, the user cannot authenticate with their password. (They could still log in by other means, like SSH key, if set up – locking affects password authentication.)
* **Unlocking an account:** Use `passwd -u <username>` to **unlock** the user’s password. This removes the leading `!` (or `!!`) from the shadow entry, restoring the original hash so the password works again. `usermod -U <user>` does the same in terms of shadow file editing.
* **Account locked vs. no password:** It’s important to distinguish *locked account* vs. *no password*.

  * A locked account has an `!` in front of the hash (or sometimes `!!` if it was never set). This means even if the user knew the old password, it won’t work until unlocked.
  * An account with no password (i.e., empty hash field in `/etc/shadow`) means it **does not require a password** to log in. That is a big security risk (anyone could just press Enter), and generally Linux won’t create such accounts unless explicitly configured (some special-purpose systems might for local console use). You can create that state with `passwd -d <username>` which deletes the password, leaving it empty. After `passwd -d user`, the shadow line might show something like `user::18345:0:99999:7:::` (note the empty password field between first two colons). “No password” is not the same as “locked” — no password means open access (or perhaps login only possible via other auth like keys), whereas locked means no access via password.
* **Checking Password Status:** The `passwd -S <username>` command summarizes a user’s password status (usable, locked, etc.) and aging info in a single line. For example:

  ```bash
  $ passwd -S sysadmin  
  sysadmin P 04/24/2019 0 99999 7 -1
  ```

  The output fields mean:

  * `sysadmin` – username
  * `P` – password status (P = **password set (usable)**, L = **locked**, NP = **no password**).
  * `04/24/2019` – date of last password change.
  * `0` – min days before change (PASS\_MIN\_DAYS).
  * `99999` – max days password is valid (PASS\_MAX\_DAYS, here effectively no expiration)
  * `7` – warning period (days before expiry to warn).
  * `-1` – inactivity period after expiry (–1 means inactive feature disabled).

  In this example, `P` indicates the account is not locked and has a password. If it were `L`, that would indicate the password is locked (you’d see the `!` in shadow). `NP` would mean the account currently has no password set.
* **Forcing a Password Change on Next Login:** If an administrator wants a user to reset their password on the next login (for example, setting up an initial password that must be changed), they can expire the user’s password immediately. Command: `passwd -e <username>`. This sets the user’s password as expired. The user will be prompted to enter a new password at their next login. For instance, `passwd -e sysadmin` will mark sysadmin’s password expired. When sysadmin next logs in, after entering the old password, the system will require a new password to be set before continuing.
* **Inactivating an Account vs Locking:** In addition to manually locking, remember the shadow fields can automatically inactivate an account. If the “expire date” is reached (field 8 in shadow) or if password expired and the inactive days (field 7) have passed, the account becomes disabled (login not allowed). This is another way an account can be locked (by policy rather than an explicit admin action). In an exam context, locking usually refers to the manual `!` lock using `passwd -l` or `usermod -L`.

In short, **locking an account** is a quick way to disable login without deleting the user. Always verify the status with `passwd -S` or by inspecting `/etc/shadow`. Unlock when needed with the complementary command.

## Brief Overview of PAM (Pluggable Authentication Modules)

**PAM** is a framework used by Linux (and Unix-like systems) to handle authentication in a flexible and modular way. Rather than each application (login, SSH, sudo, etc.) having its own authentication methods compiled in, PAM provides a set of libraries and configuration files that these programs can use for authentication, account management, session setup, and password management.

Key points for an exam refresher:

* **Pluggable Authentication Modules** allows system administrators to easily modify how authentication is done, by editing config files, without needing to recompile programs. For example, you can require a user to pass through an LDAP check, or a OTP (one-time password) module, or just use local passwords, all by configuring PAM.
* PAM is organized into four types of functionalities (or management groups):

  * **Authentication**: Verifying user credentials (like checking passwords, or other auth methods).
  * **Account Management**: Controlling account availability (has the account expired? Is the user allowed to log in at this time?).
  * **Session Management**: Tasks to set up/tear down a login session (mounting directories, printing login messages, etc.).
  * **Password Management**: Handling password changes (enforcing strong passwords, updating multiple password stores).
* Configuration is usually in `/etc/pam.d/` (each service has a file) or `/etc/pam.conf`. These configs define stacks of modules for each of the above categories. For instance, the login service might require the pam\_unix module (for local passwords) and then pam\_lastlog (to print last login info), etc.
* **Why it matters for user management:** PAM often enforces password policies (like requiring a minimum password length, or remembering old passwords to prevent reuse) and account policies (like not allowing login after certain conditions). For example, when we mentioned earlier that PAM can remember old passwords – this refers to the `pam_pwhistory` or `pam_unix` modules keeping a history so users can’t immediately reuse an old password after a forced change. PAM is also what can enforce things like “password must be at least 8 characters with one number” via modules.
* For LPIC or similar exams, you typically don’t need to know the exact syntax of PAM config, but be aware of its role. PAM is the reason why commands like `passwd` and login behaviors are consistent across services – because they all consult the same underlying modules.

In summary, **PAM provides a consistent and flexible way to handle authentication** on Linux. It’s a complex topic, but as an admin you should know it exists and that login services consult PAM configs to decide how users authenticate and what restrictions apply. (For more detail, one could consult the [Linux-PAM project site](http://www.linux-pam.org).)

## Important Configuration Files for User Management

Lastly, let’s review some key configuration files related to user and group management and their roles:

* **`/etc/login.defs`:** This is a global configuration file for login and user account defaults. It’s a plain text file read by user management tools (like `useradd`) and login programs. Key settings in this file include:

  * **UID and GID ranges:** Parameters `UID_MIN`, `UID_MAX`, `GID_MIN`, `GID_MAX` define the range of user IDs and group IDs for normal users. For example, many systems have `UID_MIN 1000` and `UID_MAX 60000`. This means by default new user accounts will be assigned UIDs starting at 1000 and up. (Historically, Red Hat started at 500, but modern RHEL also uses 1000.) System user accounts (for services) typically use UIDs below this range (like 1–999). The login.defs file may have `SYS_UID_MIN`/`SYS_UID_MAX` commented or set to delineate that range. Similarly, `GID_MIN 1000` means new groups start at GID 1000.
  * **Password aging defaults:** `PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE` set the default password expiry policy for new accounts. For instance, `PASS_MAX_DAYS 99999` (the usual default) means passwords don’t expire by default. If you set `PASS_MAX_DAYS 90`, then any *new* user account created will have a 90-day expiration set (unless overridden by options). `PASS_MIN_DAYS` might be 0 (no restriction on changing password again). `PASS_WARN_AGE` could be, say, 7 to warn a week before expiry. **Note:** Changing these in login.defs affects only accounts created after the change; it doesn’t retroactively update existing users. Use `chage` for existing accounts.
  * **UMASK and others:** login.defs often sets default `UMASK` for file creation for login sessions, default home directory permissions, mail spool settings, etc. For example, it might contain `UMASK 077` to suggest home directories be private. While not explicitly in our provided material, be aware it influences useradd behavior like default home mode.

  Essentially, `/etc/login.defs` is consulted whenever you create a user (for defaults) and by some login processes to enforce policies. It’s a good place to check or set site policy for new accounts (like “all new accounts expire after 1 year” or “use a specific UID range”).

* **`/etc/default/useradd`:** This file stores default values for the `useradd` command. When you run `useradd -D`, it reads this file to display the defaults, and using `useradd -D -<option>` will update it. Typical fields include:

  * `GROUP` – default primary group ID or name (on systems with UPG, this might be set to a generic group but typically is ignored in favor of creating a private group; on non-UPG, it could be `100` for group “users”).
  * `HOME` – base directory for home directories (usually `/home`).
  * `SHELL` – default login shell (e.g. `/bin/bash`).
  * `SKEL` – location of skeleton directory (default `/etc/skel`).
  * `CREATE_MAIL_SPOOL` – yes/no on creating a mail spool file for the user (in `/var/mail`). The default is typically yes.
  * `INACTIVE` – default inactive days after password expiry (corresponds to `-f` in useradd, often `-1` meaning disabled).
  * `EXPIRE` – default account expiration date (often empty, meaning no expiration by default).

  These defaults can be edited by an admin to change how `useradd` behaves. For example, if you want all new users to have `/bin/zsh` as their shell, you could do `useradd -D -s /bin/zsh` which updates `/etc/default/useradd` so that future `useradd` invocations default to zsh.

* **`/etc/gshadow`:** As discussed, this is the counterpart to `/etc/group` that stores secure group information. Only readable by root, it contains group passwords (if any), group administrators, and members. When you use `groupadd`, `groupmod`, `groupdel`, or `gpasswd`, this file is updated alongside `/etc/group`. Key points:

  * An `!` in the password field means the group has no password (locked), which is the norm.
  * If a password is set (rare), it will be hashed here. Non-members who know the password could use `newgrp` to join the group temporarily.
  * The admin field lists who can administer the group (set via `gpasswd -A`). Admins can add/remove members without needing root.
  * The members field lists group members who are not group admins (admins are implicitly members too).

  In day-to-day admin, you don’t edit `/etc/gshadow` manually; you use commands like `gpasswd` or `usermod` and they will update it. Just be aware it exists and mirrors group info in a secure way, similar to how `/etc/shadow` relates to `/etc/passwd`.

* **Other files to be aware of:**

  * **`/etc/group`:** (Although not explicitly listed in the question, it’s fundamental) It lists all groups and their members (by username). This file is world-readable. When you add a user to a group (via any method), it updates this file’s member list for that group. Format: `group_name:password:GID:member1,member2,...`. Usually the password field is `x` indicating to consult `/etc/gshadow` for any group password.
  * **PAM config files:** e.g. `/etc/pam.d/common-password` or service-specific files – these control password complexity and account rules. Not typically covered in depth at LPIC-1 level, but know they exist if password policies seem to be enforced beyond what’s in login.defs (it’s often PAM modules).
  * **`/etc/shadow` and `/etc/passwd`:** We discussed these at length; they are config files in a sense (they configure accounts). Always ensure their integrity and correct formatting. Use commands (`vipw`, `vigr`) or tools to edit if needed.
