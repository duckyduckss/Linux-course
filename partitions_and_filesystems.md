# Partitioning and Filesystem Management Refresher

## Partitioning Basics

* **Partitioning** divides a physical disk into isolated sections. This helps organize data and optimize system use.
* Most desktop OSes hide partitioning, but on Linux it's crucial to plan partitions **before installation** (post-install changes are possible but risky and may require downtime).
* **Why partition?**

  * Isolate different data: e.g. keep user files separate from system files.
  * Prevent one area from filling up others: e.g. put logs or temporary files on their own partition so they can't fill up the root filesystem.
  * Enable special features per filesystem: e.g. apply disk **quotas** on a `/home` partition (limit each user's space) without affecting the rest of the system.
  * Use different mount options per partition for security or performance (e.g. `noexec` on `/tmp`).
* **Common separate partitions:**

  * `/` (root filesystem) – core OS. Keep fairly small to ease recovery.
  * `/boot` – (sometimes separate) holds kernel and boot files (especially in BIOS or encrypted disk setups).
  * `/home` – user data. Separate allows independent backup, format, or quota enforcement for users.
  * `/var` – variable data (logs, spool, print jobs). If logs grow, they won’t fill `/`.
  * `/tmp` – temporary files. Large temp files won’t fill the OS partition.
* **Three key steps** to set up storage on Linux:

  1. **Partition the disk** – carve out one or more partitions on the drive.
  2. **Format each partition with a filesystem** – create a filesystem (e.g. ext4) on the partition (except special partitions like swap).
  3. **Mount the filesystem** – attach each formatted partition’s filesystem into the unified Linux directory tree at a mount point.

*(Linux has no drive letters; disks and partitions appear as device files under `/dev` like `/dev/sda1` for the first partition of drive “sda”. These get mounted to directories.)*

## Partition Tables: MBR vs GPT

* A **partition table** records the layout of partitions on a disk. Two main schemes are used:

  * **MBR (Master Boot Record):** Legacy partition table dating from DOS era.

    * Supports **4 primary partitions** max. To have more, one primary can be an **extended partition** that contains multiple **logical partitions**.
    * Supports disks up to \~2 TiB (32-bit sector addressing limit).
    * Stored at the beginning of the disk (the MBR). Also contains boot loader code in many systems.
    * Partition type identifiers are one-byte “hex codes” (e.g. `0x83` for Linux filesystem, `0x82` for Linux swap, `0x05` or `0x0f` for extended partitions).
  * **GPT (GUID Partition Table):** Modern replacement for MBR, introduced around 2000.

    * Uses 64-bit addressing – supports very large disks (up to \~9 ZB, essentially no practical size limit).
    * Allows up to **128 partitions** (all are “primary”; no extended/logical needed).
    * Stores multiple copies of partition metadata (primary header at disk start, backup header at end) for redundancy. Also includes a *protective MBR* so old tools see the disk as occupied (to avoid overwriting GPT by MBR-only tools).
    * Each partition has a **GUID** identifier and a type GUID (no simple hex codes like MBR).
    * Required by UEFI systems for booting from large disks and generally more robust.
* **Other partitioning schemes:** Linux can also recognize BSD disklabels, Apple Partition Map (APM), etc., but MBR and GPT are most common on PCs.

## Partition Types (Primary, Extended, Logical)

* **Primary partition:** A real partition listed in the partition table. MBR allows up to four primary partitions on one disk.
* **Extended partition:** A special container partition (in MBR) that is used to extend beyond the four-partition limit. Only **one extended** partition can exist, and it occupies one of the up to four slots. The extended partition does not hold data directly.
* **Logical partitions:** Partitions **inside** the extended partition. Logical partitions are numbered starting at 5 (e.g. `/dev/sda5` is the first logical partition if an extended exists). There is no fixed limit on the number of logical partitions aside from disk space (MBR historically can support up to \~59-60 logicals in one extended).
* **Important:** You **never format or mount the extended partition itself** – it’s just a container. Filesystems reside on primary or logical partitions only. Accidentally creating a filesystem on an extended partition will destroy all its logical partitions.
* **GPT note:** GPT does not use extended/logical scheme; every partition is equivalent. So with GPT you can have many partitions without the primary/logical distinction.

## Partitioning Tools

Various utilities are available to create and manage partitions:

* **`fdisk`:** Classic interactive CLI tool for MBR partitioning.

  * Usage: `fdisk /dev/sdX` to edit the partition table of disk `sdX`. (Requires root.)
  * In **interactive mode**: press `m` to see the help menu of commands. Common commands:

    * `n` – create a new partition (prompts for partition number, start, end).
    * `d` – delete a partition.
    * `t` – change a partition’s type code (e.g. set `82` for swap, `83` for Linux). Type `L` to list all hex codes.
    * `p` – print the partition table (current state, unsaved).
    * `w` – write changes to disk (and exit). (`q` to quit without saving changes.)
  * By default, `fdisk` operates on MBR. Modern `fdisk` can also handle GPT, but for GPT it’s more common to use specialized tools or `gdisk`.
  * Non-interactive usage: `fdisk -l` lists partitions on all disks (or `fdisk -l /dev/sdX` for a specific disk). This is useful to quickly view the partition layout.
  * *Output:* `fdisk -l` shows each partition with start/end, blocks, Id (type), and system (type label). For example, Id `83` is “Linux”, `82` is “Linux swap / Solaris”, `05` is “Extended”.
  * **Tip:** Always verify the partition table after changes (use `p` or `fdisk -l`) before formatting. If you make a mistake, you can quit without saving and try again (fdisk is forgiving until you write changes).
* **`gdisk`:** GPT fdisk – works like `fdisk` but for GPT partition tables.

  * Command-line interface is nearly identical to `fdisk` (it even has an `m` menu). Many command letters are the same.
  * Use `gdisk /dev/sdX` for a GPT disk. It will warn or convert if the disk was MBR.
  * Example: `gdisk /dev/sdb` then `o` to create a new empty GPT, `n` to add partitions, etc.
  * Useful commands: `p` to print GPT entries, `w` to save. There’s also an `l` to list known GPT type GUIDs (for setting partition type like Linux filesystem, Linux swap, etc.).
  * gdisk is sometimes called **GPT fdisk**. If working with >2TB drives or needing many partitions, use gdisk or parted rather than legacy fdisk.
* **`parted`:** GNU Parted – a more powerful partitioning tool supporting both MBR and GPT (and others) and able to **resize** partitions without losing data (when possible).

  * **Command-line mode:** You can provide operations in one command. Example:

    * `parted /dev/sdb print` – list the partitions on /dev/sdb (shows number, start/end in GB, type, filesystem if known, etc.).
    * `parted /dev/sdb mklabel gpt` – create a new GPT partition table on /dev/sdb (erases any existing table). Use `msdos` instead of `gpt` for an MBR (DOS) table.
    * `parted /dev/sdb mkpart primary ext4 0% 50%` – create a new primary partition spanning from 0% to 50% of the disk, and hint that it will be ext4 (this doesn’t format it, just sets type GUID and gives a name).
    * `parted /dev/sdb resizepart 1 6GB` – example to resize partition 1 to 6GB (if supported and free space is available).
  * **Interactive mode:** Simply run `parted /dev/sdb` and you get a `(parted)` prompt. You can then type commands like `help`, `print`, `mklabel`, `mkpart`, etc. This is similar to fdisk’s interactive usage but with different syntax.
  * **Non-destructive vs destructive:** Unlike fdisk/gdisk (which are “destructive” partition editors meaning you usually delete and recreate partitions to resize them), parted can non-destructively move and resize certain partitions. This is useful for adjusting partitions on the fly.
  * **Filesystem creation:** Parted can create filesystems too (with the `mkfs` command within parted), but a common practice is to use parted **just for partitioning** and then use dedicated `mkfs` tools to format. In fact, after using parted’s `mkpart`, it’s often recommended to run the appropriate `mkfs` command yourself.
  * **GUI:** Parted has a graphical front-end called **GParted** (GNOME Partition Editor). GParted offers a visual interface for partitioning tasks, useful in desktop environments (not exam-critical, but good to know it exists).
* **Other tools:** There are other partition managers (e.g., `cfdisk` for a curses UI, or installation tools). But `fdisk`, `gdisk`, and `parted` cover most needs on Linux systems.

## Filesystem Creation and Types

After partitioning, new partitions (except swap) need a **filesystem**:

* **`mkfs` (make filesystem):** Generic command to format a partition with a given filesystem.

  * Usage: `mkfs -t <fstype> <partition>` (or equivalently use the filesystem-specific mkfs tool directly, like `mkfs.ext4`, `mkfs.vfat`, etc.).
  * Examples:

    * `mkfs -t ext4 /dev/sdb1` – create an ext4 filesystem on `/dev/sdb1`. This actually invokes the `mke2fs` tool in the background (since ext4 is part of the ext2/3/4 family).
    * `mkfs -t vfat /dev/sdb1` – format `/dev/sdb1` as FAT32 (vfat). This calls `mkfs.vfat` internally (also known as `mkdosfs`).
  * `mkfs` is basically a wrapper. It will call the appropriate **specific formatting program** based on `-t`. For ext2/3/4, it calls **mke2fs**; for vfat, **mkdosfs**; for XFS, **mkfs.xfs**, etc.
  * You can also run those specific tools directly (e.g. `mkfs.ext4 /dev/sdb1`). In practice, either approach is fine, but the specific tools may have additional options not exposed through `mkfs` generic interface.
  * **Safety:** mkfs will refuse to format a partition that is currently mounted. This prevents accidental destruction of a mounted filesystem. You must unmount a partition before reformatting it.
* **Common Linux filesystem types:** Each filesystem type has pros/cons and typical use cases:

  * **ext2** – “Second Extended Filesystem.” An older Linux filesystem (pre-journaling).

    * *Pros:* Simple, low overhead. Works well on small disks or flash drives. (No journal means fewer writes, which can be good for some flash media.)
    * *Cons:* **No journaling**, so recovery after a crash is slow and data loss is more likely (must run `fsck`). Generally replaced by ext3/ext4.
  * **ext3** – “Third Extended Filesystem.” Essentially ext2 + journaling.

    * *Pros:* Journaling for reliability (logs changes, so after a crash the filesystem can recover quickly). You can **upgrade ext2 to ext3 in place** without reformatting. Widely used for many years, well-tested.
    * *Cons:* Slightly slower than ext2 (due to journal writes). Has limits on file and filesystem size (cannot support very large volumes as ext4 can).
  * **ext4** – “Fourth Extended Filesystem.” The modern default in many Linux distros.

    * *Pros:* Supports **very large volumes and files** (multi-terabyte). Journaling (can even be turned off if needed). Backwards compatible with ext3/ext2 (can mount those, and ext3 can be upgraded to ext4). More efficient allocation and delayed allocation for performance.
    * *Cons:* Not drastically different from ext3 in usage. Does not allow shrinking the filesystem (like ext3, you can grow but not shrink it easily once created). Also uses a fixed number of inodes at creation (no dynamic inode creation), so running out of inodes is possible if created with too few (though defaults are usually fine).
  * **XFS** – High-performance 64-bit filesystem originally from SGI IRIX. Default in some distributions (e.g., RHEL 7).

    * *Pros:* Excellent at handling **large files** and multithreaded I/O. Extremely scalable (was designed for big data).
    * *Cons:* **Cannot be shrunk** (you can grow an XFS filesystem, but not reduce its size). Slightly different toolset (uses `xfs_*` commands for maintenance).
  * **vfat (FAT32)** – File Allocation Table, as used by DOS/Windows (also called FAT16/FAT32 depending on size). In Linux, `mkfs.vfat` creates a FAT filesystem.

    * *Pros:* **Highly compatible** – almost all OS (Windows, macOS, Linux) can read/write FAT, so it’s ideal for USB flash drives or shared data disks.
    * *Cons:* No support for large files (>4GB file size limit for FAT32) or large volumes (FAT32 volume limit is 2TB in some OS). No journaling or advanced features. Also, no file permissions or ownership on FAT – not suitable for installing a Linux system.
    * *Note:* Microsoft’s newer NTFS is another common filesystem (especially for Windows), which Linux can read/write via the `ntfs-3g` driver, but Linux normally doesn’t *create* NTFS from scratch in standard tools. For cross-platform, FAT32 or exFAT is used.
  * **ISO 9660 (isofs)** – Read-only filesystem standard for optical discs (CD-ROM). Created with the `mkisofs` tool rather than `mkfs`. Often .iso files are ISO 9660 images.

    * *Pros:* Universally readable (any OS can read ISO 9660 from a CD or ISO file).
    * *Cons:* Read-only (by design). Has levels and extensions for long filenames which can complicate things. Not used for hard drives, only media.
  * **UDF (Universal Disk Format)** – Successor to ISO 9660 for DVDs and Blu-rays (and some packet-writing on CD-RW).

    * *Pros:* Designed for **writable optical media** (DVDs). Can be read/write if used on rewritable discs.
    * *Cons:* Write support on Linux is somewhat limited to certain UDF versions. Mainly used for DVDs; not typical for hard disks.
  * *(There are many other filesystems: Btrfs, ReiserFS, ZFS, etc., but the above are the ones covered in these notes.)*
* **Journaling Filesystems:** ext3, ext4, XFS (and others like JFS, Btrfs) use a **journal** to log metadata updates.

  * **What is a journal?** It’s a log on the disk where changes are recorded *before* they are actually applied to the main filesystem. If the system crashes mid-operation, the journal can be replayed to bring the filesystem to a consistent state quickly.
  * This greatly reduces the time to recover compared to non-journaled FS (which would require a full `fsck` scan). The trade-off is a bit of extra write overhead (writing to the journal).
  * Journaling typically logs metadata (file system structure changes). Some filesystems can be set to journal data as well (ext3 data=journal mode, though rarely used due to performance cost).
  * **Ext4 journaling:** Ext4 can operate with or without a journal (you can disable it if you want less wear on SSD, but then you lose the quick recovery benefit). It also has *journal checksums* to improve reliability.
  * **XFS journaling:** Always journals metadata. Very reliable for large volumes.
  * Overall, journaling provides a balance: near-instant recovery from crashes with minimal performance impact. For exam purposes, know that ext3/ext4 have journals (ext2 doesn’t) and why that matters (fast recovery vs potential data loss).

## Swap Space (Virtual Memory)

Linux uses **swap space** to extend physical memory (RAM) by swapping out idle pages to disk. Swap can be a **partition** or a **file**:

* **Swap Partition:** A dedicated partition with no filesystem, just raw space for memory pages.

  * When creating one with MBR `fdisk`, set the partition type to `82` (Linux swap). With GPT, just label it as swap (GUID type or by name).
  * **Steps to create a swap partition:**

    1. Create the partition (e.g., using fdisk or parted). If using fdisk, after `n` (new partition) use `t` to set type `82`. In parted, you can specify `--type` as linux-swap or simply set the filesystem type to swap later.
    2. Format it as swap with `mkswap <device>` (e.g., `mkswap /dev/sda3`). This writes swap area metadata (like a UUID). Example output: *“Setting up swapspace version 1, size = ... UUID=...”*.
    3. Enable it: `swapon <device>`. This activates the swap partition for immediate use.
    4. (Optional) Verify active swap with `swapon -s` (or `cat /proc/swaps`). It will list swap locations and usage.

    * *Remember:* Using `swapon` directly is temporary. On next boot, that swap won’t be used unless added to `/etc/fstab`. (We’ll cover making it permanent below.)
* **Swap File:** A regular file on an existing filesystem that is set up to be used as swap space.

  * Use a swap file if you don’t have a free partition. It can be created on the fly.
  * **Steps to create a swap file:**

    1. Create an empty file of the desired size. For example, to create a 100 MB swap file:

       ```bash
       dd if=/dev/zero of=/var/extraswap bs=1M count=100  
       ```

       This uses `dd` to write 100 MB of zeros to `/var/extraswap`. (Choose a location with enough free space; `df -h` can help determine where.)
    2. Set it up as swap: `mkswap /var/extraswap` – this marks the file with swap signature and UUID.
    3. Activate it: `swapon /var/extraswap`. Now the kernel treats this file as additional swap.
    4. Verify with `swapon -s` as above.
  * Swap files are convenient, but slightly slower than swap partitions (because the filesystem adds overhead). In most cases the difference isn’t huge, but for heavy swap usage, a partition is optimal.
* **Making swap persistent (fstab entry):**
  To ensure swap is available on boot, add an entry to `/etc/fstab`. For a swap **partition**, use the device or its UUID/label. For a swap **file**, use the path to the file.

  * **Swap partition fstab example:**

    ```
    UUID=6f450a83-9d2e-409f-8bce-826696a49e54  swap  swap  defaults  0 0  
    ```

    Here the first field uses the partition’s UUID (preferred over /dev name), the mount point field is set to `swap` (no actual directory, this keyword tells the system this is swap space), filesystem type is `swap`, options are `defaults` (suitable for swap), and dump/fsck fields are `0` (swap is not dumped or fsck’d).
  * **Swap file fstab example:**

    ```
    /var/extraswap  swap  swap  defaults  0 0  
    ```

    (The only difference is the first field is the file path instead of a device/UUID.)
  * After adding, you can test activation without reboot: run `swapon -a` which reads fstab and enables all swap entries. Then check `swapon -s` to confirm. If there’s an error, it will print; always fix fstab mistakes **before rebooting** (an invalid fstab can cause boot problems).
  * **Swap priority:** You can tune the kernel’s usage of multiple swap areas. In fstab, specify `pri=<number>` in the options. Higher priority (larger number) means that swap space will be used first. By default, multiple swap spaces have equal priority and will be used in a round-robin fashion. If you have both a fast swap partition and a slower swap file, you might set the partition with `pri=2` and the file with `pri=1` (lower). This way the system prefers the faster swap.
* **General guidance:** If possible, use a swap partition for best performance. Use a swap file for flexibility or if adding swap after install. Ensure swap is sized appropriately (LPIC advice: typically memory size or more for hibernation, but it depends on your needs).

## Mounting and Unmounting Filesystems

Once partitions are formatted with filesystems (or designated as swap), you integrate them into the system by **mounting**:

* **Mount points:** A mount point is an existing directory where the filesystem will be attached. It can be an empty directory (recommended) or a directory containing files (though any files there will be hidden while something is mounted on it). Common mount points are created under `/mnt` or `/media` for temporary mounts, or specific directories like `/data`, `/backup` for permanent mounts. Always create the directory first (`mkdir /mountpoint`) if it doesn’t exist.
* **`mount` command:** Used to attach (mount) a filesystem. Basic usage:

  ```bash
  mount -t <fstype> [-o options] <device> <mount_point>
  ```

  * If `-t` is omitted, mount will attempt to autodetect the filesystem type (or consult `/etc/fstab` if an entry exists for that device).
  * If `-o options` is omitted, it uses default options (or those in fstab if referenced).
  * Examples:

    * `mount /dev/sdb1 /mnt` – mount `/dev/sdb1` at `/mnt`. If the device has a known filesystem, the kernel will detect it. This uses default options (read-write, etc.).
    * `mount /dev/sdb2 /opt -o ro` – mount `/dev/sdb2` at `/opt` as read-only (`ro`). This assumes the filesystem type can be auto-detected or is in fstab.
    * `mount -t iso9660 /dev/cdrom /mnt` – mount a CD-ROM (device could be `/dev/cdrom` or `/dev/sr0`) with the ISO 9660 filesystem type explicitly specified. Needed for optical drives or when auto-detect might not guess correctly.
  * After mounting, the content of the device’s filesystem becomes accessible under the mount point path. For example, if `/dev/sda5` is mounted on `/home`, then the files for users reside on that partition.
  * **Mounting via fstab:** You can simply do `mount <mount_point>` (or `mount <device>`) for entries listed in `/etc/fstab`. The system will look up the appropriate device or directory and options from fstab. For instance, `mount /home` will find the `/home` entry in fstab and mount whatever device is listed there. This is handy for mounting by name or re-mounting something that was unmounted.
* **Viewing mounted filesystems:**

  * Running `mount` with no arguments displays all currently mounted filesystems. The format is like:

    ```
    /dev/sda2 on / type ext4 (rw,relatime)
    tmpfs on /run type tmpfs (rw,nosuid,nodev)
    ...  
    ```

    It shows the source (device or pseudo-fs), the mount point, filesystem type, and mount options in parentheses.
  * You will see **physical device mounts** (like `/dev/sdaX on /xyz`) as well as **virtual/pseudo filesystems** (`proc`, `sysfs`, `devpts`, `tmpfs`, etc.). Pseudo-filesystems provide kernel interfaces (e.g. `proc` for process info, `sysfs` for sys info, `tmpfs` for in-memory files). These are listed even though they aren’t real disk partitions. They often have `none` or special names as the source.
  * Another way to view disk usage of mounted filesystems is the `df -h` command (see **Monitoring** section below).
* **Unmounting (`umount`):** To detach a filesystem from the directory tree, use `umount <mount_point|device>`. For example, `umount /mnt` or `umount /dev/sdb1` will unmount the filesystem that was mounted there.

  * *Note:* The command is “umount” (no “n” after the “u”).
  * **When to unmount:** Before removing a removable drive (USB, DVD), or when you need to run fsck on a partition, or before resizing a partition, you must unmount it. Also during shutdown/reboot, scripts will unmount filesystems.
  * **Umount failures:** `umount` may refuse if the filesystem is **busy**. “Busy” means some process is using the filesystem: a file is open, or someone’s working directory is on that filesystem, etc. Common culprits include having a terminal `cd`ed into that directory, or running programs from it.

    * If you get a “device is busy” error, identify what’s holding it up. Tools:

      * `lsof | grep <mount_point>` – list open files on that mount. Example: `lsof | grep /mnt` might show a process with an open file or a shell with its current directory there.
      * `fuser -m <mount_point>` – lists PIDs using the mount (the `-m` treats the argument as a filesystem).
    * Once you find the processes, you can terminate or close them (e.g. `kill` the process or simply `cd` out of the directory in your shell). Then retry `umount`.
  * **Forcing unmount:** (Not in standard notes, but FYI) `umount -f` can force in some cases (e.g., network filesystem hang), and `umount -l` (lazy) detaches the mount point when busy, but these are advanced options – use with caution. Generally, best practice is to close processes properly rather than force.
* **Loop mounts (mounting images):** The `loop` device option allows mounting a file that contains an entire filesystem.

  * Use case: You downloaded an ISO image or created an image of a filesystem. You can mount it directly without burning to disk or writing to a partition.
  * Example:

    ```bash
    mount -o loop software.iso /mnt
    ```

    This will “loopback mount” the ISO file on /mnt. The `-o loop` option creates a virtual loop device associating with the file and mounts it. After this, you can read files from the ISO as if it were a physical CD mounted at /mnt.
  * You can also create your own filesystem image files and mount them similarly (`mount -o loop fs.img /mnt`). This is often used for testing or for chroot environments.
  * When done, `umount /mnt` to detach the image. The loop device will free up.
  * *Tip:* “Almost everything in Linux is a file” – even disk images can be treated like block devices via loop mount. This saves time and resources (like in our example of copying .rpm files out of an ISO without needing to burn a DVD).
* **Changing mount options on the fly (remount):**

  * You can alter the mount flags of an **already-mounted** filesystem using:

    ```bash
    mount -o remount,<new_options> <mount_point>
    ```

    For example, to temporarily turn off access time updates (atime) on `/home` if it’s already mounted:

    ```bash
    mount -o remount,noatime /home
    ```

    This will **re-mount** the `/home` filesystem with the `noatime` option in effect (and any other options it already had, unless overridden). Remounting is useful to, say, change a filesystem to read-only (`mount -o remount,ro /mnt`) or to enable options without unmounting (which might not be possible if the fs is busy, like root filesystem).
  * Remount uses the existing device and doesn’t interrupt active processes (unless the option itself restricts something like turning off writes). It just updates the in-kernel mount flags.
  * **Persistence:** If you want the change to persist, update `/etc/fstab` accordingly; a remount is temporary until next mount. Typically, one would edit fstab and then do a `mount -o remount <mount_point>` to test the new options immediately. If there’s an error in options, the remount will fail (giving you a chance to fix fstab before a reboot). If it succeeds quietly, the options are fine.
  * **Example:** If you edit fstab to add `nosuid` on `/data`, you can apply it immediately with `mount -o remount /data`. If something is wrong (say a typo), `mount` will complain rather than leaving you to find out at next boot.
  * Keep fstab in sync with any remount changes to avoid confusion on next reboot.

## Persistent Mounts with /etc/fstab

The **fstab** file (`/etc/fstab`) is the configuration file that tells the system which filesystems to mount at boot (and swap to enable). It also can be used by the `mount` command when mounting by mount point or device name. Key points:

* **Format:** Each line in `/etc/fstab` corresponds to one filesystem (or swap, or special pseudo-fs) to be mounted. It has **six fields**, typically:

  1. **Filesystem** – What to mount. Usually identified by device file (like `/dev/sda1`), **UUID**, or **LABEL**. Using `UUID=<uuid>` or `LABEL=<label>` is preferred for physical drives, because device names can change (for example, if you add another disk, what was `/dev/sda1` might become `/dev/sdb1`, but the UUID remains the same). You can find UUIDs and labels with the `blkid` command.
  2. **Mount point** – Where to mount it (the directory path). For swap, this is typically `swap` (no real mount point). For certain pseudo filesystems, it can be none, but usually those aren’t in fstab except `/proc` on some systems.
  3. **Type** – Filesystem type. e.g. `ext4`, `xfs`, `vfat`, `swap`, `proc`, etc.
  4. **Mount options** – Comma-separated list of options (or `defaults` for the standard set). This controls things like read/write, permissions, etc. See below for common options.
  5. **Dump** – a numeric field (0 or 1) used by the old `dump` backup utility. `1` means this filesystem should be dumped, `0` means ignore. It’s largely historical; most modern systems set this to `0` for everything or `1` for just critical filesystems. (By convention, `1` is for normal disk filesystems that need backup, and `0` for swap, pseudo, or network filesystems.)
  6. **Pass (fsck order)** – Another numeric field controlling **fsck** (file system check) order at boot.

     * `0` means do not check at boot (use for swap, network filesystems, or pseudo-filesystems like proc).
     * `1` means high priority – typically only the root filesystem gets `1` (so it’s checked first).
     * `2` means check after the ones marked `1`. All other local disk filesystems get `2`. Multiple `2` filesystems can be checked in parallel if on different drives.
     * The system’s boot scripts will run `fsck -A` (all) which consults these numbers to decide check order. So root `/` is 1 (first, alone), and e.g. `/home`, `/var` might be 2 (next, possibly in parallel if separate disks).
* **Example fstab entries:**

  ```
  # <fs>                              <mountpoint>  <type>  <opts>          <dump> <pass>  
  UUID=3db6ba40-67d2-403d-9c0a-9a901697cd8d   /      ext4    defaults        1 1  
  UUID=09d641d5-bc5a-4065-8d80-8ae797dfa7f3   /boot  ext4    defaults        1 2  
  LABEL=mydata                             /data   ext4    defaults        1 2  
  /dev/sdb2                                swap    swap    defaults        0 0  
  ```

  In this example:

  * The root filesystem (`/`) is ext4 on a device identified by UUID, with defaults, and will be dumped (`1`) and fsck first (`1`).
  * `/boot` on another UUID (ext4) defaults, dumped and fsck second (`2`).
  * A data partition labeled "mydata" mounts at `/data` ext4 defaults, fsck after root (`2`).
  * A swap partition on /dev/sdb2 is set to activate as swap, no dump, no fsck.
* **Mount options (in fstab or via `-o` on mount):** If you specify `defaults`, it is a shorthand for a set of common options: `rw, suid, dev, exec, auto, nouser, async, relatime`. Key options include:

  * `rw` / `ro` – Mount read-write (default is `rw`) or read-only (`ro`).
  * `suid` / `nosuid` – Allow or disallow SUID/SGID bits. `nosuid` will prevent setuid programs on that filesystem from elevating privileges (security measure, often used on untrusted media or user partitions).
  * `dev` / `nodev` – Allow or disallow device files. `nodev` means block/char device files on this filesystem won’t function (they can’t be used to interact with hardware). Good for security on non-root filesystems (e.g., mounting a USB drive nodev so a rogue device file on it can’t do anything).
  * `exec` / `noexec` – Allow or disallow running binaries from this filesystem. `noexec` is often set on `/tmp` or removable drives to make it harder to run malicious binaries from there. (Note it’s not foolproof, but it’s a layer of defense.)
  * `auto` / `noauto` – Controls whether the filesystem is automatically mounted at boot (or with `mount -a`). `auto` is the default; `noauto` means do not mount at boot (you’ll mount it manually). Use `noauto` for things like a backup drive that isn’t always connected, or an ISO you only mount on demand.
  * `user` / `nouser` – `user` allows normal users to mount the filesystem (and also implies noexec, nosuid for safety when that user mounts it). `nouser` (default) means only root can mount it. This is often set to `user` for CD-ROMs, floppies, or USBs in fstab so that a normal user can do `mount /mnt/cdrom`.
  * `async` / `sync` – Async (default) means writes are buffered (better performance). `sync` means write-through (each write waits to physically be written). `sync` is rarely used except on some flash drives or network FS where you want to avoid data loss on removal, but it’s much slower.
  * `relatime` / `noatime` / `atime` – These control file access time updates. Traditional `atime` updates the access timestamp every time a file is read. This causes lots of writes. `noatime` disables atime updates (improves performance). `relatime` (relative atime) is a compromise: it updates atime only if the file’s previous atime is older than its modify or change time, or if it hasn’t been updated in a certain period. Modern Linux defaults to relatime, which dramatically reduces disk writes but still keeps atime roughly correct for programs that need it.
  * There are many more options (e.g., `data=journal` for ext3, `uid=1000` for vfat, etc.), often specific to filesystem types. Check the man page for mount or the specific filesystem (e.g. `man mount.ext4`) for details if needed.
  * **Using multiple options:** Just list them separated by commas with no spaces. Example: `defaults,noexec,nodev` would apply all default options except it overrides exec->noexec and leaves others as default. If you use `defaults`, you don’t need to list all the default ones explicitly; it’s understood. If you only want to change a couple of defaults, you can either list all except the ones you change, or use `defaults` and then additional options (note: in fstab, `defaults` is not magical, it’s just replaced by the set of the eight options above, then overridden by any conflicting options listed after).
  * In `mount` output, if you use defaults, it may only show a subset (often just `rw,relatime` etc.) because some options are assumed. Don’t be confused; rely on fstab to know what’s in effect.
* **Editing fstab carefully:** Always make a backup of `/etc/fstab` before editing (`cp /etc/fstab /etc/fstab.backup`). A typo in this file can prevent booting (you might end up in single-user mode or emergency shell if a filesystem fails to mount).

  * After editing, test without reboot: as mentioned, `mount -a` will attempt to mount everything in fstab that isn’t mounted (use with caution), or simply try `mount <mountpoint>` for the new entry. For swap, `swapon -a` will attempt to enable all swap from fstab. If there are errors, fix them and test again. Only reboot when you’re confident fstab is correct.
  * If you mess up and the system doesn’t boot normally, you may need to use a recovery disk or grub’s recovery mode to fix the fstab (or restore the backup). So double-check entries (common errors: wrong UUID, nonexistent mount point directory, forgetting the 0 0 at end for swap, etc.).

## LVM Basics (Logical Volume Management)

**LVM** provides a layer of abstraction over physical disks and partitions, allowing flexible volume management. Key advantages of LVM over traditional static partitions:

* **Flexible resizing:** You can **extend logical volumes on the fly** (even while mounted, if the filesystem supports online grow). This addresses the classic scenario: “Oops, /var is full, need it bigger” – with LVM, you can allocate additional space without downtime, instead of having to rebuild partitions. For example, if `/var` is an LVM volume and it fills up with logs, you can add more extents to it (provided there is free space in the volume group or you add a new disk). No reinstallation needed.
* **Combine multiple disks:** LVM volumes (analogous to “partitions” in LVM context) can span **across multiple physical disks**. If you have two smaller drives, you can make one large logical volume that uses both. E.g., need a 60GB volume but have a 40GB and a 20GB disk – LVM can combine them into one 60GB logical volume. This is helpful in “big data” or expanding storage by adding disks.
* **Snapshot capability:** LVM supports snapshots – you can freeze a point-in-time view of a volume. For instance, to get a consistent backup of `/home` without downtime: take an LVM snapshot of the `/home` volume, and back up the snapshot. Users can keep working; changes after the snapshot won’t affect the snapshot, so the backup is consistent to the moment of snapshot. This avoids the need to unmount or stop usage during backup.

**LVM Components:**

* **Physical Volume (PV):** This is essentially a disk or partition initialized for use by LVM. You take a partition (for LVM it typically has type `8e` in MBR) or even a whole disk (e.g., /dev/sdb) and run `pvcreate` on it to make it an LVM PV. This writes metadata to identify it as belonging to LVM. (You can also use RAID devices or encrypted devices as PVs; LVM is flexible.)
* **Volume Group (VG):** Think of this as a pool of storage made by combining one or more PVs. You create a VG with `vgcreate`, naming the group and listing the PVs to include. For example, `vgcreate vg01 /dev/sda2 /dev/sdb1` would make a volume group “vg01” from two PVs. The VG is like one big virtual disk containing all the space from those PVs. Inside a VG, space is allocated in units called **extents** (usually default 4 MB size).
* **Logical Volume (LV):** The equivalent of a “partition” inside the VG. You carve out logical volumes from the free extents in a VG using `lvcreate`. An LV can be any size up to the total space of the VG (minus what’s already allocated to other LVs). LVs get names and appear as devices (usually under `/dev/<VGName>/<LVName>` or `/dev/mapper/<VGName>-<LVName>`). You can format an LV with a filesystem (ext4, xfs, etc.) and mount it, just like a physical partition. To the OS and user, it behaves like any block device/partition.
* **Summary of LVM workflow:**

  1. **Prepare physical devices:** Connect/install the drives you want to use. If using partitions, create partitions on them (with fdisk/parted) and set type to LVM (`8e` for MBR). If using whole disk, you can skip partitioning and use the raw disk as PV.
  2. **pvcreate:** Initialize the disk or partition for LVM. Example: `pvcreate /dev/sda2` (marks that partition as an LVM PV). Do this for each physical disk/partition you want in LVM.
  3. **vgcreate:** Create a volume group from one or more PVs. Example: `vgcreate data_vg /dev/sda2 /dev/sdb1`. Now `data_vg` contains the storage of those two PVs combined. You can add more PVs later with `vgextend` if needed.
  4. **lvcreate:** Allocate logical volumes from the VG’s free space. Example: `lvcreate -n myvol -L 50G data_vg` creates a 50GiB LV named “myvol” in volume group `data_vg`. If the VG has that much space available, the LV is created and a device `/dev/data_vg/myvol` will appear. (You can also specify `-L 100%FREE` to use all remaining space, or use `-l` with extents count or percentage of VG.)
  5. **Create filesystem:** Now format the new LV with `mkfs` (e.g., `mkfs.ext4 /dev/data_vg/myvol`).
  6. **Mount LV:** Create a mount point and add an entry to fstab just like with a normal partition (you can use the device path or better, use the LV’s UUID from `blkid`). Mount it and use it.
  7. **Later**, if you need to resize, you can **extend an LV** with `lvextend -L +10G /dev/data_vg/myvol` (for example, to add 10GiB more). If the VG has free space or you add a new PV to it, you can extend. After `lvextend`, grow the filesystem (`resize2fs` for ext4, or `xfs_growfs` for XFS, etc.) to actually use the added space. In many cases this can be done online. Shrinking is a bit harder and only some filesystems support it (ext4 offline, XFS not at all), so plan sizes accordingly.
* **LVM in exams:** Focus on the concepts (what problem LVM solves) and basic commands (pvcreate, vgcreate, lvcreate). Also understand terminology (PV, VG, LV). E.g., LPIC might ask “What is the sequence of steps to create an LVM logical volume?” or “Name an advantage of LVM.” The scenarios above (increasing /var, spanning disks, snapshot for backup) illustrate those advantages.

## Systemd Mount Units

Modern Linux systems using **systemd** can manage mounts through *mount units* instead of (or in addition to) fstab entries. This is a more advanced topic, but briefly:

* A **.mount unit** is a systemd configuration file that describes a mount. It contains similar information as an fstab line (What device, Where to mount, Type, Options). These units live in `/etc/systemd/system` (or are generated at runtime for fstab). For example, a mount unit for `/data` might be named `data.mount` and have a `[Mount]` section with `What=/dev/sdb1`, `Where=/data`, `Type=ext4`, `Options=defaults`.
* Systemd automatically generates mount units at boot for each entry in `/etc/fstab`. So if you prefer, you can still use fstab and systemd will create corresponding mount units behind the scenes.
* Why use explicit mount unit files? They allow more fine-grained control and integration with systemd’s dependency system. For instance, you can set an option to wait longer for a network mount, or have mounts ordered before/after certain services. You can also create **automount units** (.automount) for on-demand mounting.
* **Naming:** A mount unit’s filename is based on the mount point path. Example: mount point `/mnt/backup` would translate to unit `mnt-backup.mount` (the slash is replaced with a hyphen). Inside, the `[Mount]` section defines What, Where, Type, Options. There’s also usually an `[Install]` section with `WantedBy=multi-user.target` to have it active at boot multi-user runlevel.
* For most purposes, using fstab is sufficient and simpler. But be aware that on systemd systems, those fstab lines are managed by systemd in the form of these mount units. Unless you have a specific need, you won’t manually write .mount files for basic disk mounting tasks.

## Monitoring Disk Usage and Filesystems

As a sysadmin, you need to keep an eye on disk space and usage. Linux provides tools like **df** and **du** for this:

* **`df` (disk free):** Reports how much space is used and available on mounted filesystems.

  * Common usage: `df -h` – “human-readable” output (sizes in KB/MB/GB rather than blocks).
  * Example output:

    ```
    Filesystem      Size  Used Avail Use% Mounted on  
    /dev/sda2        6.3G  3.2G  2.9G  53% /  
    /dev/sda1        485M   52M  408M  12% /boot  
    tmpfs            351M   84K  351M   1% /dev/shm  
    ```

    This shows each filesystem, its type (if using `-T` flag, as `df -hT` above shows Type column), size, used, available, percentage, and mount point.
  * `df -h` by default lists all **local and pseudo filesystems that take up space**. Notice `tmpfs` (which is memory-based) is listed with its size and usage. Pure kernel interfaces like `proc` might not show because they have no meaningful size. Use `df -a` to include all, but usually you care about real storage and tmpfs.
  * `df -T` adds a column for filesystem type, which is helpful to distinguish ext4 vs xfs vs tmpfs etc.
  * Use `df` to quickly see if any partition is near 100% use. This is often the first check if something is misbehaving (e.g., “can’t write temp file” might be /tmp is full).
* **`du` (disk usage):** Summarizes disk usage of files/directories. While df works at filesystem level, `du` drills down into directories.

  * Running `du <dir>` will output the size of each file and subdirectory under `<dir>` (recursively). By default it gives size in blocks. Use `-h` to get human-readable sizes.
  * `du -s <dir>` gives a **summary** for that directory (instead of listing every file). Useful to see how much a top-level folder consumes as a whole.
  * Often used flags:

    * `-h` for human-readable.
    * `-s` for summary (don’t output each file, just totals for arguments).
    * `--max-depth=N` to limit recursion depth. For example, `du --max-depth=1 /home/user` will show the total for each item directly under `/home/user` and not descend further. This is great for pinpointing which subdirectory is large. (By default, without max-depth, du will eventually output a grand total, but also intermediate totals for every subdirectory it recurses into.)
    * `--exclude=pattern` to skip certain files or directories that match the pattern. Useful if some directories always dominate but you want to see others, or to avoid permission errors by excluding directories you can’t read.
  * **Finding large files/directories:** A common trick is to sort the output. For instance:

    ```bash
    du -h /var/log | sort -h | tail -n 10  
    ```

    This would list the 10 largest entries under /var/log (after sorting by size). Or similarly `du -k /home | sort -n | tail -n 10` to see the largest directories/files in /home (in KB, sorted numerically).
    Another approach: `du -sh /home/*` to get a quick summary of each user’s home directory usage.
  * **Example:**

    ```
    $ du -sh /bin /usr/bin  
    6.6M    /bin  
    38M     /usr/bin  
    ```

    This shows /bin uses 6.6MB, /usr/bin 38MB. Or,

    ```
    $ du -h --max-depth=1 /usr  
    28M    /usr/lib  
    4.0K   /usr/src  
    112M   /usr/bin  
    ...  
    1.2G   /usr  
    ```

    This lists top-level subdirs of /usr and their sizes, then the total for /usr (1.2G here).
  * **Permissions:** Be careful running `du` at root of the filesystem as a normal user – you will get “Permission denied” on many directories (e.g., /root, /var/log, etc.). If you run `du` as root, you can see everything, but some files like system logs might still produce errors if they are open exclusively. The `--exclude` option can skip known problematic or irrelevant paths. In the notes example, they excluded `/usr/lib/audit` because a normal user couldn’t access it. In general, run `du` as root for full picture or tailor the scope to directories you have access to.
  * `du` vs `df`: If `df` shows a filesystem is nearly full, use `du` to find what’s taking up space within that filesystem. For example, if `/home` is full, run `du -sh /home/*` to see which user’s directory is huge. Or if `/` is full, check big directories under it (`/var`, `/usr`, `/home`, etc.).
* **Summary:** Use `df` for high-level free/used space per filesystem, and `du` for investigating directory/file sizes within a filesystem. These tools help prevent and resolve “disk full” issues, which is a common task for sysadmins and often tested conceptually (e.g., how to find what is filling up a disk).

This concludes the partitioning and filesystem management refresher. Remember to practice using these commands and understanding their output, as exam questions may ask about command usage or interpreting results (e.g., identifying a filesystem type from `fstab`, or the effect of a mount option, etc.). Good luck with your exam preparation!
