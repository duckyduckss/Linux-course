# Linux Boot Process Study Guide

## Boot Process Stages

Linux systems go through **four main boot stages** from power-on to a running OS:

* **Firmware Stage (BIOS/UEFI):** When the computer is powered on, the motherboard firmware (traditionally BIOS, or UEFI on modern systems) takes control. It performs a Power-On Self-Test (POST) to check basic hardware (CPU, RAM, disks, etc.), enumerates hardware devices, then locates a boot device. The firmware reads the first sector of the boot drive (the Master Boot Record, or **MBR**, 512 bytes) into memory and transfers control to the bootloader code found there. *(Note: UEFI firmware uses a special **EFI System Partition** instead of an MBR, but this stage serves the same purpose as BIOS.)*

* **Bootloader Stage:** The bootloader is a small program responsible for loading the operating system kernel into memory. Because the MBR has limited space, bootloaders often work in **multiple stages**. For example, with GRUB (Grand Unified Bootloader) Legacy:

  * **Stage 1** resides in the MBR and is very minimal – it simply loads the next stage.
  * **Stage 1.5** (if present) is stored in the first few sectors immediately after the MBR (around 30 KB) and contains code (like filesystem drivers) needed to locate and load Stage 2.
  * **Stage 2** is the full bootloader (for GRUB, usually located in `/boot/grub`). It presents the boot menu or prompt, lets the user select an OS or kernel, and loads the selected **kernel** (and an initial RAM disk if needed) into memory. It can also pass **kernel parameters** (e.g. specifying a runlevel or options like `single` for rescue mode) before handing control to the kernel.
    Modern GRUB (GRUB2) is a two-stage bootloader but functionally similar: it can read filesystems, present menus, and load the OS. The bootloader stage’s primary task is to get the OS kernel into memory and start it. At this point, the firmware’s job is done and the kernel takes over booting.

* **Kernel Stage:** Once the bootloader launches the Linux kernel, the kernel initializes. This stage involves **hardware initialization and driver loading**. The kernel sets up low-level hardware details and memory management, detects hardware devices, and loads essential drivers (either built-in or via an initial ramdisk). If the root filesystem resides on a complex device (RAID, LVM, etc.) or requires drivers not built into the kernel, an **initrd/initramfs** (initial RAM disk) is used. The initrd is a temporary root filesystem loaded into memory by the bootloader along with the kernel – the kernel mounts this and uses the drivers inside it to access the real root filesystem. Once hardware is initialized and the real root filesystem is mounted, the kernel executes the system’s first process. The kernel is compressed on disk (as a **zImage** or **bzImage**) and will decompress itself in memory; its final act in this stage is to start the **init process**.

* **Init Stage:** The **init process** (with PID 1) is the first userspace process. Its job is to **finish the boot process** by starting all other necessary services and user-space programs. Traditionally, this was handled by the **SysV init** system (with the `/sbin/init` program). Modern Linux distributions have largely replaced the old SysV init with alternatives like **systemd** (and formerly **Upstart** on some systems). Regardless of the implementation, this stage configures the running environment:

  * If using **SysVinit** (System V init): The init process reads `/etc/inittab` to determine the default **runlevel** (a predefined state of services) and which initialization scripts to run. It then executes scripts (typically shell scripts in `/etc/init.d/` or `/etc/rc.d/init.d/`) to start system services for that runlevel. These scripts launch background daemons (networking, cron, getty for terminals, etc.) and set up login prompts.
  * If using **Upstart** (an event-based init used in some distributions): The init process reads job definitions from `/etc/init/`. Upstart starts and stops services in response to events (including reaching a specified runlevel target). It was used by Ubuntu and RHEL 6 but is now mostly legacy.
  * If using **systemd** (the modern replacement on most Linux distros): The init process is actually the systemd daemon ( `/sbin/init` is typically a symlink to `systemd`). Systemd uses unit files (in `/usr/lib/systemd/` and `/etc/systemd/`) to start services and reach a target state (analogous to runlevels). It will transition the system to the default **target** (multi-user, graphical, etc.) by starting all required services in parallel, based on dependencies.

  In all cases, init (or its equivalent) is responsible for spawning **getty/login processes** on ttys, so users can log in, and for adopting any orphaned processes. At the end of this stage, the system is fully booted and operational.

## GRUB Legacy (GRUB 0.x) Bootloader

**GRUB Legacy** refers to the original GRUB (versions 0.xx), which was the standard Linux bootloader before GRUB2. Key points about GRUB Legacy:

* **Multi-Stage Bootloader:** GRUB Legacy operates in 2+ stages as described:

  * *Stage 1* (tiny loader in MBR) → *Stage 1.5* (in post-MBR gap, with filesystem support) → *Stage 2* (full GRUB interface in `/boot/grub/`).
    This design gets around the MBR size limitation by loading more capable code from disk. Stage 2 provides the user interface (the boot menu or GRUB prompt) and loads the OS kernel and **initrd** into memory.

* **Configuration Files:** GRUB Legacy’s configuration is typically stored in **`/boot/grub/grub.conf`** or **`/boot/grub/menu.lst`** (depending on distro; often one is a symlink to the other). Only the **root user** can modify these. This config file defines GRUB’s menu entries and settings. A typical `grub.conf` includes global settings (default entry, timeout, optional splash image, etc.) and one or more **menu entries** each starting with `title`. For example:

  ```plaintext
  default=0
  timeout=5
  splashimage=(hd0,0)/grub/splash.xpm.gz
  hiddenmenu

  title Linux (Example Kernel)
      root (hd0,0)
      kernel /vmlinuz-5.10-example ro root=/dev/sda1 quiet
      initrd /initramfs-5.10-example.img
  ```

  Each `title` stanza specifies how to boot an OS or kernel: the `root (hdX,Y)` line tells GRUB which disk/partition contains the kernel, `kernel` specifies the kernel file and boot parameters (like `ro root=...` for the root filesystem device, and possibly a runlevel or other options), and `initrd` (if present) points to the initial RAM disk image.

* **Device Naming Conventions:** GRUB Legacy uses its own syntax to refer to disks and partitions, which **differs from Linux device names**:

  * Drives are labeled as **hd0, hd1, hd2, ...** regardless of type (no distinction between SATA/SCSI vs IDE; all are hdN). The numbering corresponds to the BIOS order of drives. For example, the first drive seen by BIOS is `hd0`, second is `hd1`, etc.
  * Partitions are numbered starting from **0**. So the first partition on the first drive is `(hd0,0)`, which might correspond to Linux device `/dev/sda1`. Similarly, `(hd0,1)` is the second partition (`/dev/sda2`), and `(hd1,2)` would be the third partition on the second drive (which could be `/dev/sdb3`).
    **Important:** This means there is an offset: what Linux calls partition 1 is partition 0 in GRUB naming. Administrators must be careful to translate device names when editing `grub.conf`. (For example, a Linux root filesystem at `/dev/sda3` would be `root (hd0,2)` in GRUB Legacy.)

* **Installing/Reinstalling GRUB Legacy:** Normally, the Linux distribution’s installer sets up GRUB Legacy in the MBR. If needed, an admin can reinstall it using the `grub-install` command. For instance: `grub-install '(hd0)'` would install GRUB Legacy’s Stage 1 to the MBR of the first disk (hd0), and copy the necessary stage files to the proper locations. This command detects the OS and `/boot` location to set up GRUB correctly. After installation, ensure the configuration file (`grub.conf`) is in place so that GRUB knows about available kernels/OSes.

* **Usage and Features:** GRUB Legacy reads its config at boot to display the menu. It supports an interactive **GRUB prompt** where you can type commands to boot manually or edit entries (useful for recovery, e.g. adding `single` to kernel line to enter single-user mode). Unlike its predecessor LILO (Linux Loader), GRUB does **not** require re-installation to the MBR when its config changes – it reads the config at boot time. (LILO would embed block locations of the kernel at install time and had to be re-run after every change. This is a major reason GRUB replaced LILO.) GRUB Legacy’s flexibility and ability to understand filesystems made it the standard until GRUB2.

## GRUB 2 Bootloader

**GRUB2** (Grand Unified Bootloader version 2) is the modern bootloader used by most Linux distributions. It was a complete rewrite of GRUB with many enhancements:

* **Improvements Over Legacy GRUB:** GRUB2 supports *dynamically loadable modules* (allowing a smaller core and additional features loaded as needed), *non-ASCII character support* in menus, the ability to boot from more complex disk setups (like **LVM volumes or RAID** partitions), and compatibility with both BIOS and UEFI systems. In short, it’s more portable and powerful, addressing limitations of GRUB Legacy.

* **Configuration Philosophy:** In GRUB2, the main configuration file (usually **`grub.cfg`**) is **auto-generated** rather than hand-edited on a routine basis. Edits to `grub.cfg` tend to be overwritten when the system updates kernels or when the configuration is rebuilt. Instead of editing `grub.cfg` directly, administrators are expected to configure GRUB2 via **template files** and then regenerate the final config. Key locations:

  * **Main config file:**

    * On *Debian/Ubuntu* systems: `/boot/grub/grub.cfg`
    * On *Red Hat/Fedora* systems: `/boot/grub2/grub.cfg`
      (The path differs, but content is similar. Some distros also place a symlink at `/etc/grub.conf` for convenience, pointing to the real file in `/boot`.)
  * **Default settings:** `/etc/default/grub` – a file where general settings are defined (timeout, default menu entry, whether to enable graphical splash, kernel command-line defaults, etc.). For example, this file might contain:

    ```plaintext
    GRUB_TIMEOUT=5  
    GRUB_DEFAULT=saved  
    GRUB_SAVEDEFAULT=true  
    GRUB_CMDLINE_LINUX="quiet splash"
    ```

    These variables influence the generated menu (here, a 5-second timeout, and using the last selected entry as the new default with `saved`).
  * **Script fragments:** `/etc/grub.d/` – a directory containing scripts that generate portions of the `grub.cfg`. Each executable file here (like `10_linux`, `20_linux_xen`, `30_os-prober`, `40_custom`, etc.) outputs configuration snippets. For instance, `10_linux` looks for installed Linux kernels and creates menu entries for them, `30_os-prober` adds entries for other operating systems it finds (like Windows), and `40_custom` is a template where an admin can add custom menu entries.
  * **Environment file:** `grubenv` – a file (usually in `/boot/grub` or `/boot/grub2`) that GRUB2 uses to store variables between reboots (for example, if using `GRUB_SAVEDEFAULT` to remember the last boot choice).

* **Updating GRUB2 Configuration:** When a new kernel is installed or an admin changes settings, the command to **regenerate** the config is used:

  * On Debian/Ubuntu: `update-grub` (which internally calls `grub-mkconfig`).
  * On RedHat/Fedora: `grub2-mkconfig -o /boot/grub2/grub.cfg` (specifying the output file).
    These commands scan the system for installed kernels and OSes (using the scripts in `/etc/grub.d` and settings in `/etc/default/grub`) and produce a new `grub.cfg`. Because of this design, manual edits to `grub.cfg` will be lost on the next update; persistent changes should be made in `/etc/default/grub` or by editing/adding scripts in `/etc/grub.d/`.

* **GRUB2 vs GRUB Legacy Usage:** With GRUB2, the expectation is you rarely edit the boot menu by hand. However, you can still access a GRUB prompt at boot (by pressing **c** or editing an entry with **e**) to change kernel parameters temporarily (for instance, to enter rescue mode or troubleshoot). GRUB2’s interface is similar to GRUB Legacy but with more features. It also introduced the concept of a *“saved” default entry*: by setting `GRUB_DEFAULT=saved` and `GRUB_SAVEDEFAULT=true` in `/etc/default/grub`, the OS you boot will be recorded, and GRUB2 will boot the same entry by default next time (useful for selecting a certain kernel permanently from the menu). This behavior is configured via the `grubenv` file.

* **Installation of GRUB2:** Installing GRUB2 to a drive’s MBR or UEFI partition is done with the `grub-install` or `grub2-install` command. For example, to install to the MBR of `/dev/sda`:

  ```bash
  # grub2-install /dev/sda
  ```

  This installs the GRUB2 bootloader code to the disk and places core images in `/boot`. After installation (or whenever configuration changes), `grub2-mkconfig` must be run to generate/update the config file. On UEFI systems, `grub-install` will also set up the EFI bootloader files in the EFI System Partition.

In summary, GRUB2 provides a robust and automated way to manage boot configuration. Understanding its structure (where to configure and how to update) is crucial for maintaining boot settings on modern Linux systems.

## Runlevels and Systemd Targets

Linux defines various **runlevels** (on SysV and Upstart systems) or **targets** (on systemd systems) to represent specific states of machine operation, particularly which services are running.

**Runlevels (SysV/Upstart):** These are numbered **0 through 6** (the kernel recognizes 0-9, but 7-9 are rarely used by convention). Each runlevel has a standard purpose:

* **0:** System halt (shutdown). **Shuts down** the machine.
* **1:** Single-user mode (rescue mode). A minimal environment for administrative tasks; only the root user is logged in, with basic services. (Also invoked by using `single` or `s` as a kernel parameter.)
* **2:** Multi-user mode **without networking**. (On some distros, runlevel 2 is multi-user with no network services started.)
* **3:** Full multi-user mode (text mode). Networking is active, multi-user, but **no graphical interface**. This is the normal runtime for a server (non-GUI).
* **4:** **Unused / custom**. Not universally defined – historically left for admins to define a custom level if needed. (On many Linux distributions, runlevel 4 is not used or treated like 3.)
* **5:** Graphical multi-user mode. Like runlevel 3 but with a graphical login and X window system started (GUI). This is the typical default for desktop systems.
* **6:** Reboot. Tells the init system to reboot the machine.

**Systemd Targets:** Systemd does not use runlevels internally but provides *target units* that serve similar purposes. For compatibility, systemd will translate runlevel numbers to equivalent targets. The common targets include:

* **poweroff.target** – analogous to runlevel 0 (halt and power off).
* **rescue.target** – single-user mode (similar to runlevel 1, also called *rescue mode*).
* **multi-user.target** – multi-user text mode (combines the idea of runlevels 2–3–4 on many distros).
* **graphical.target** – multi-user with GUI (comparable to runlevel 5).
* **reboot.target** – system reboot (like runlevel 6).

Other targets exist (for example, `emergency.target` for an even more minimal shell, or `network.target` for networking being up), but the above are the main equivalents to traditional runlevels. **Note:** Systemd does report a runlevel for backward compatibility – for instance, if the system is in `graphical.target`, the `runlevel` command might show **5**. In fact, systemd will show **N 5** on boot if the system directly booted to graphical target (previous runlevel N meaning none).

**Setting the Default Runlevel/Target:**

* On SysV init systems, the default runlevel is set in **`/etc/inittab`** by a line like `id:5:initdefault:` (which would set default to runlevel 5, for example). Changing that number changes the default startup state after boot.
* On Upstart-based systems (e.g., older Ubuntu or RHEL 6), default runlevel could also be set in `/etc/inittab` or via an Upstart config (for instance Ubuntu’s `/etc/init/rc-sysinit.conf` might have `env DEFAULT_RUNLEVEL=2` or similar).
* On systemd, the default target is determined by a symlink: `/etc/systemd/system/default.target` which typically points to one of the target unit files (like `graphical.target` or `multi-user.target`) located in `/lib/systemd/system/`. To change the default, you update this symlink to point to the desired target. For example:

  ```bash
  # ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target
  ```

  would set the system to boot into multi-user text mode by default. Systemd also provides the convenient command:

  ```bash
  # systemctl set-default graphical.target
  ```

  to change the default target.

**Viewing the Current Runlevel/Target:**

* The command `runlevel` will print the previous and current runlevel (e.g., `N 3` means the system booted directly into runlevel 3 with no previous runlevel). If the system uses systemd, this is derived from the current target (multi-user.target reports as runlevel 3, graphical.target as 5, etc.).
* The command `who -r` also shows the current runlevel and the date/time the system entered it. For example:

  ```
  $ who -r
         run-level 3  2025-05-07 18:30
  ```

  indicates runlevel 3 since 18:30.
* On systemd systems, you can query the current target with `systemctl get-default` (to see the default target) or `systemctl list-units --type=target` to see active targets.

**Changing Runlevels/Targets:**

* **At Boot Time (via Bootloader):** You can override the default runlevel/target by passing an argument to the kernel from the bootloader:

  * With GRUB/SysVinit: select your kernel entry and **edit the kernel line**. Appending a runlevel number (0–6) or the words `single` or `rescue` will tell init to go into that runlevel instead of the default. For example, adding `1` (or `single`) will boot into single-user mode; adding `3` forces multi-user text mode.
  * With GRUB/systemd: append `systemd.unit=<target>` to the kernel command line. For example, `systemd.unit=rescue.target` or `systemd.unit=multi-user.target`. This directs systemd to boot into that target rather than the default. (You could also use the numeric runlevel for compatibility, e.g., `1` will be interpreted as rescue.target, `3` as multi-user.target, etc.)

* **On a Running System:** The system’s runlevel/target can be changed without rebooting:

  * On traditional init or Upstart, the **`init`** or **`telinit`** command is used. Running (as root) `init 5` or `telinit 5` would switch to runlevel 5, for example. `telinit` is essentially a symbolic link to `init` on many systems, with `telinit -t <secs>` allowing a delay.
  * With systemd, you can still use `init <N>` for compatibility (systemd will interpret it), but the native method is using **`systemctl isolate <target>`**. For instance, `systemctl isolate graphical.target` will bring the system to the graphical target (like runlevel 5), starting any services needed for graphical.target and stopping those not needed. Likewise, `systemctl isolate multi-user.target` would drop the system to non-GUI multi-user mode (like runlevel 3).
    *Be cautious:* Changing runlevels/targets will start/stop services which can disrupt users. For example, going from graphical to multi-user will shut down the display manager (and thus any GUI sessions).

In practice, runlevels are mostly a SysV init concept. **Systemd’s targets** serve a similar role but with more flexibility. Systemd does a lot behind the scenes to make runlevel-oriented commands still work for compatibility. For example, running `init 0` on a systemd system triggers `poweroff.target`, and `init 6` triggers `reboot.target`. Understanding these mappings is important for administering both older and newer Linux systems.

## Init Systems: SysVinit vs. systemd (Service Management)

Once the system has booted into the init stage, managing system services (starting/stopping programs in the background, a.k.a. daemons) is handled by the init system. Two major init systems you should know are **SysVinit** and **systemd** (with Upstart as a historical third option):

* **SysVinit (System V Init):** This is the traditional initialization system inherited from UNIX System V. After the kernel starts `/sbin/init`, SysVinit will:

  * Read `/etc/inittab` to determine the **default runlevel** and which scripts to run.
  * For the given runlevel, execute a series of scripts to start services. These scripts reside typically in `/etc/init.d/` (or `/etc/rc.d/init.d/`) and are referenced via symbolic links in runlevel-specific directories (commonly `/etc/rc0.d/, /etc/rc1.d/, ... /etc/rc6.d/`).

    * In each `rcX.d` directory, scripts starting with **`S`** (e.g., `S85httpd`) are run to **Start** services when entering that runlevel. Scripts with **`K`** (e.g., `K15httpd`) are run to **Kill (stop)** services when leaving a runlevel. The numbers following S/K determine the order (lower numbers run earlier). This ensures dependencies are respected (for example, `S10network` might run before `S85httpd`, so that networking is up before the web server starts).
    * These S and K links all point to the actual service scripts in `/etc/init.d/`. The service scripts are typically shell scripts that accept arguments like **start**, **stop**, **restart**, **status**, etc., to control the daemon.
  * SysVinit starts the S scripts of the default runlevel on boot. If the runlevel is changed (via `init` command), it will run the appropriate K scripts of the old level and S scripts of the new level.
  * The first process keeps running as the parent of all others. It waits for “respawn” entries (like getty processes for terminals) and generally babysits orphaned processes.

* **Managing Services in SysVinit:**
  To manually control services under SysVinit:

  * You can call the init script directly. For example: `/etc/init.d/httpd start` will start the Apache HTTP daemon, and `/etc/init.d/httpd stop` will stop it. Running the script with no arguments often shows usage and supported options (for instance, `status`, `reload`, etc., if available).
  * Most distributions provide a convenient **`service`** command. For example: `service httpd restart` will find the `httpd` script in init.d and execute it with the `restart` argument. This saves typing the full path.
  * To configure which services start on which runlevels, you can manipulate the symlinks in the `rcX.d` directories. Tools exist to simplify this:

    * **`chkconfig`** (on Red Hat-based distros) can list and adjust services. `chkconfig --list` shows which services are on/off for each runlevel. You can enable or disable a service for startup with `chkconfig <service> on` (which typically turns it on for runlevels 2,3,4,5) or `off`. For example, `chkconfig httpd on` would create the appropriate S and K links to start httpd in the appropriate runlevels.
    * **`update-rc.d`** (on Debian-based distros) serves a similar purpose, configuring the init script symlinks.
    * These tools read header information in the init scripts (like lines specifying “Default-Start:” runlevels and sequence numbers) to automate the linking process.
  * **Example:** If you want a service to not start in runlevel 5, you could remove or rename its `S` link in `/etc/rc5.d`. But using `chkconfig` or `update-rc.d` is safer. For instance, `chkconfig httpd off` would remove the S85httpd link from runlevels where it was enabled.

* **Systemd:** This is the modern init system adopted by most major Linux distributions (such as RHEL/CentOS 7+, Fedora, Ubuntu 15.04+, Debian 8+, etc.). Systemd is a **system and service manager** that starts everything in parallel (as much as possible) and uses dependency graphs to determine ordering. Key points:

  * The systemd process (PID 1) reads unit files rather than shell scripts. **Unit files** describe services, mounts, devices, sockets, targets, etc. For services (unit files ending in `.service`), the file contains metadata and instructions to start the daemon (the executable path, dependencies, when to start, etc.). These files live in directories like `/usr/lib/systemd/system/` (for default units provided by packages) and `/etc/systemd/system/` (for system-specific overrides or user-created units).
  * Systemd targets (with `.target` files) group units together (as discussed in the runlevels section above).
  * When systemd boots, it activates the default target, starting all required units simultaneously (subject to ordering requirements).
  * Systemd offers features like on-demand starting of daemons (via socket activation), monitoring of processes, parallel startup, and more robust dependency handling than SysVinit.

* **Managing Services in systemd:**
  The primary tool to interact with systemd is the **`systemctl`** command. This single command replaces many of the manual tasks from SysVinit:

  * **Starting/Stopping Services:** Use `systemctl start <service>.service` and `systemctl stop <service>.service`. For instance, `systemctl start sshd.service` will start the SSH daemon. (The “.service” suffix is optional in most cases – systemctl will assume a service if you omit it.)
  * **Restarting/Reloading Services:** `systemctl restart <service>` restarts the service, and `systemctl reload <service>` tells the service to reload its configuration if that’s supported (many daemons have a separate reload action).
  * **Checking Status:** `systemctl status <service>` shows whether the service is running, its PID, recent log output, and other information. This is very handy for troubleshooting as it often prints the last few log lines from the service (which come from the journal).
  * **Enable/Disable Autostart:** To control whether a service starts on boot (i.e., is enabled in a certain target), use `systemctl enable <service>` or `systemctl disable <service>`. Enabling a service creates the necessary symlinks under `/etc/systemd/system/` that tell systemd to include that service in the appropriate target’s startup. For example, enabling a service that should run in multi-user mode might create a symlink in `/etc/systemd/system/multi-user.target.wants/`. Disabling removes those links.
    *Example:* `systemctl enable httpd.service` ensures the Apache server will start on boot (in the relevant target). `systemctl disable httpd.service` would prevent it from starting automatically.
  * **Listing Services:** `systemctl list-units --type=service --all` will list all services and their state (running, exited, dead, etc.). You can also do `systemctl --all` to list all units of all types.
  * **Runlevel (Target) Changes:** As mentioned, `systemctl isolate <target>` will switch to a new target (similar to changing runlevel). There are also specific convenience commands: `systemctl rescue` (to drop to single-user rescue.target) or `systemctl emergency` for emergency shell.
  * **Power Management:** Systemd integrates shutdown/reboot, so commands like `systemctl reboot`, `systemctl poweroff`, `systemctl halt`, `systemctl suspend`, and `systemctl hibernate` will instruct the system to take those actions. (These are essentially equivalent to the traditional `reboot`, `poweroff`, etc., commands.)

* **Compatibility:** On a systemd-based system, the traditional commands (`service`, `chkconfig`, and even `init` for runlevels) are often still available (as wrappers or links) to ease the transition. For example, typing `service nginx start` might internally call `systemctl start nginx.service`. Likewise, `init 3` will be translated by systemd into isolating multi-user.target. However, learning `systemctl` is essential for modern systems administration as it exposes more functionality.

* **Upstart:** (Though largely obsolete now, for completeness.) Upstart was an event-driven init system used by Ubuntu (9.10 through 14.04) and RHEL 6. It replaced `/sbin/init` with a process that reacted to events (like startup, shutdown, service stopped, filesystem mounted, etc.) to start jobs defined in `/etc/init/*.conf`. Each job file described how to start/stop a service and under what conditions. Upstart still used the concept of runlevels to an extent (for example, a job could be tied to starting at runlevel 2,3,4,5 and stopping at 0,1,6). Upstart has been replaced by systemd on virtually all major distros, but you might encounter it on older enterprise Linux systems. Both Upstart and systemd keep `/sbin/init` as their binary name for compatibility. If you need to know which init system is in use on a machine, checking for the presence of `/etc/systemd/` vs `/etc/init/` (Upstart) or looking at the process 1 name (`ps -p 1`) can tell you.

In summary, **SysVinit** uses init scripts and runlevel symlinks to manage services, whereas **systemd** uses unit files and the `systemctl` command. Systemd greatly simplifies service management with a uniform command set and integrates tightly with other parts of the system (including logging via journald, described next). If you’re preparing for an exam, be comfortable with both systems: know how to start/stop services and set them to start at boot in each environment.

## System Shutdown and Reboot Commands

Shutting down or rebooting a Linux system can be done with several commands. These commands ultimately communicate with the init system (SysV or systemd) to change the runlevel/target to halt or reboot states. Key commands include:

* **`shutdown`:** The most versatile command to bring the system down. Syntax is `shutdown [options] <time> [message]`. Common usages:

  * `shutdown -h now` – Halt the system **now**. The `-h` means halt (or power off, see below). This brings the system down immediately (runlevel 0).
  * `shutdown -r +5 "Rebooting in 5 minutes"` – Reboot the system in 5 minutes, with a broadcast message to all logged-in users. The `-r` flag means reboot (i.e., go to runlevel 6), and `+5` specifies a 5-minute delay. The message in quotes will be sent to users.
  * `shutdown 13:00` – Schedules a shutdown at 13:00 (1:00 PM). Without `-h` or `-r`, a default action is to bring the system to single-user mode (runlevel 1) for historical reasons. (If no flag is given, many systems interpret shutdown as halting by default, but in some cases it goes to runlevel 1 – so it’s best practice to always specify `-h` or `-r`.)
  * You can cancel a scheduled shutdown with `shutdown -c`.
    When `shutdown` is invoked, it notifies logged-in users (wall message) and prevents new logins. At the specified time, it kills processes and brings down the system cleanly. On systemd systems, `shutdown` is an alias to systemctl, but usage is the same.

* **`halt` and `poweroff`:** These commands stop the system. On a modern ACPI-compliant system, both will actually power off the machine (cut power). Historically:

  * `halt` would halt the CPU but not necessarily power off the machine.
  * `poweroff` would halt and send ACPI command to turn off power.
    Today, there’s little difference; both typically invoke runlevel 0 (poweroff.target) and turn off the machine. They do not take a time argument (they happen immediately). Essentially, `halt`/`poweroff` are like `shutdown -h now` in effect.

* **`reboot`:** This command instructs the system to reboot immediately. It’s equivalent to `shutdown -r now` (or going to runlevel 6). It will terminate all processes and restart the system.

* **`init 0`, `init 6`:** Using the `init` command to change runlevel is another way: `init 0` is halt, `init 6` is reboot. Similarly, `telinit 0` or `telinit 6` does the same. On a systemd system, these will be translated to the appropriate target.

* **Systemd equivalents:** As mentioned, systemd provides its own commands which are essentially the same calls:

  * `systemctl poweroff` (or `systemctl halt`) – shutdown and power off.
  * `systemctl reboot` – reboot.
  * `systemctl rescue` – go to rescue (single-user) mode.
  * These can be used interchangeably with the traditional commands on systemd systems.

**Broadcast Notifications (wall):** When using `shutdown` with a message, it sends a notice to all users. If you want to send a warning **without actually shutting down**, you can use the `wall` command. For example:

```bash
$ wall "The system will undergo maintenance at midnight."
```

This will broadcast the given message to all logged-in terminals. It’s a simple way to warn users of impending reboots or other important info. (By default, `shutdown` does a wall broadcast of its message. Using `wall` manually is for other arbitrary announcements.)

**Important:** Only the superuser (root) or a user with appropriate privileges can shut down or reboot the system. These commands will sync disks and properly terminate processes – never simply power off a system by cutting power, as that can cause data loss or corruption. Always use the above commands to cleanly shut down.

## System Logging and Boot Diagnostics (dmesg & journalctl)

Understanding what happens during boot (and after) often requires looking at system logs. Linux provides tools to view kernel and system messages, which are crucial for troubleshooting boot issues or hardware events.

* **dmesg:** The `dmesg` command prints the **kernel ring buffer** messages. These are messages generated by the Linux kernel (and early boot process) about hardware detection, drivers loading, and general kernel status. Right after boot, invoking `dmesg` will essentially show all the kernel messages from the boot sequence. This is extremely useful if something went wrong during boot (e.g., a driver failed to load or a filesystem had issues) because you can read the errors or warnings the kernel logged.
  **Usage:** Simply run `dmesg` (you may want to pipe to `less` or `grep` for specific text). Commonly, admins check `dmesg` after adding new hardware or peripherals, since the kernel will log what it finds – for example, plugging in a USB drive will generate kernel messages visible via `dmesg` (including the new device name like `/dev/sdb` etc.).
  Additionally, many systems save the boot dmesg output to **`/var/log/dmesg`** on disk. Note that this file is overwritten on each boot with the latest boot’s messages. On traditional SysV systems, kernel messages also appear in system log files (like `/var/log/messages` or `/var/log/kern.log`, depending on configuration).

* **System Logs (Syslog vs Journald):** Historically, Linux has used text-based log files (managed by syslog daemons such as `syslogd`, `rsyslogd`, etc.) to record system messages. For example, `/var/log/messages` or `/var/log/syslog` would contain a mix of kernel and user-space daemon logs.
  Modern **systemd** systems use a logging service called **journald**, which captures logs (from the kernel and user-space) in a **binary journal** format (stored typically in `/run/log/journal/` for ephemeral logs, or `/var/log/journal/` for persistent logs). This binary format allows efficient filtering and querying but is not plain text.

* **journalctl:** This is the tool to query and view logs from **systemd’s journal** (journald). It replaces examining text files with powerful queries. For example:

  * `journalctl -b` – Show all log entries from the current **boot**. The `-b` (boot) option restricts output to messages since the system last booted. This essentially gives you all the kernel and early user-space logs for this boot, similar to what `dmesg` shows (and more, including service startup messages). It’s very handy for reviewing the entire boot sequence or diagnosing boot problems.
  * `journalctl -k` – Show only kernel messages (similar to dmesg output, but can include older boots if used with `-b` or `--list-boots` to select which boot).
  * You can also do `journalctl -f` (follow, like `tail -f` for logs) and many other filtering options (by process, service, priority, etc.).
    Because journalctl reads the binary logs, you don’t edit the journal files directly. If needed (for example, to send to support), you can export logs with `journalctl > somefile.txt` (which will produce a text dump).

* **Persistent vs Volatile Logs:** On some distributions, journald’s logs are kept only in memory (volatile) by default, meaning you lose them after a reboot (except what’s written to traditional text logs by compatibility). If `/var/log/journal/` exists, journald will store logs persistently there. In any case, `journalctl` abstracts this – you can view the logs whether they are persistent or not, as long as the system is running (or you have saved journals). For exam purposes, remember that journald logs are in binary form and require journalctl to view.

* **Traditional Logs:** Even with systemd, many distributions still enable `rsyslog` or another logger to write out some logs to text files (for example, for compatibility with log parsing tools). Common log files:

  * `/var/log/messages` – general system messages (on Red Hat-based).
  * `/var/log/syslog` – general messages (on Debian-based).
  * `/var/log/kern.log` – kernel-specific messages (on some systems).
  * `/var/log/auth.log` – authentication logs, etc.
    Systemd’s journald can forward to these or they can coexist.

* **Using Logs for Boot Troubleshooting:** If a system isn’t booting properly, you can often use `dmesg` in single-user mode or recovery mode to check kernel output, or mount the filesystem and inspect `/var/log/` for clues. On a running system, `journalctl -b` is excellent for reviewing everything that happened during the last boot (it will show kernel messages, followed by service startup logs). If you only want to see boot loader and kernel messages, `dmesg` or `journalctl -k -b` will focus on the kernel’s part of the boot.

In summary, **dmesg** and **journalctl** are your go-to tools for reading system messages. `dmesg` works anywhere (even without systemd) for kernel ring buffer output. `journalctl` is for systemd-based distros, giving you access to both kernel and user-space logs in a unified way. Understanding how to read these logs will help diagnose boot issues, hardware problems, and service failures during system startup.

