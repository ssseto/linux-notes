# 16. Reset Root Password

- **Difficulty:** ★

- Set the root password of **node2** to `seto`. You need to gain system access in order to perform operations for subsequent questions.


#### I. Intercept the Boot Loader (GRUB2)

1. Reboot or start **Node2**.
2. As soon as the GRUB2 boot menu screen appears, use the arrow keys to select the default kernel entry (usually the top one).
3. Press **`e`** on your keyboard to edit the selected boot entry configuration.

#### II. Modify the Kernel Initialization Arguments

1. Use the arrow keys to navigate down to the line that begins with `linux` or `linux16`.

2. Move your cursor to the very end of this line.

3. Append a space, followed by the initialization override argument:

   ```bash
   rd.break
   ```

4. Press **`Ctrl + x`** (or `F10`) to boot the system with these temporary debug parameters. The system will drop you into a root storage emergency shell (`switch_root#`).

#### III. Remount the Root Filesystem with Write Permissions

By default, the real system filesystem is mounted under `/sysroot` in a strict read-only (`ro`) state. To modify system credentials, you must remount it with read-write (`rw`) privileges:

```bash
mount -o remount,rw /sysroot
```

#### IV. Enter the Chroot Jail Environment

Switch your operational context from the temporary RAM disk environment into the actual system root directory layout:

```bash
chroot /sysroot
```

*(Your command prompt will change to `sh-5.1#`, indicating you are safely targeted inside the real OS partition tree).*

#### V. Update the root Password

Set the new password to `seto` (or the specific string required by your current test instructions) using a standard non-interactive interface:

```bash
echo "seto" | passwd --stdin root
```

#### VI. Force SELinux Filesystem Relabeling (The Most Critical Step)

Because the password update writes changes directly to `/etc/shadow` outside the monitoring control of the SELinux daemon, you must plant an automated flag to force a full security context relabel during the next boot phase. **Skipping this step will lock you out of the system permanently:**

```bash
touch /.autorelabel
```

#### VII. Exit and Resume Booting

Exit the chroot environment and exit the emergency shell to trigger the system initialization:

```bash
exit
exit
```

The system will automatically perform the SELinux context synchronization (this may take a couple of minutes) and reboot into the standard multi-user login screen. You can then log in as `root` with your new password to continue Node2's remaining tasks.

## Command and Syntax Explanation

- **`rd.break`**: A kernel boot argument that instructs the system systemd target loop to interrupt the boot process right before handing over control from the initial RAM disk (`initramfs`) to the actual root filesystem layer (`/sysroot`).
- **`mount -o remount,rw /sysroot`**: Re-evaluates the active kernel mount points, explicitly flipping the `/sysroot`hardware block properties from protected read-only state to editable write permissions.
- **`chroot /sysroot`**: "Change Root". This command overrides the absolute root directory path (`/`) for the running shell process, trapping it inside `/sysroot` so that any edits to `/etc` target the real OS rather than temporary boot structures.
- **`touch /.autorelabel`**: Creates a hidden marker file at the absolute root of the system. On the subsequent boot loop, the SELinux initialization script detects this specific file, applies proper file contexts to all altered systemic configurations (like `/etc/shadow`), and automatically cleans up the flag file.

## Core Knowledge Points Covered

- **GRUB2 Boot Parameters Manipulation:** Knowing how to access, read, and intercept the GRUB2 bootloader screen at the console layer to alter low-level kernel directives safely.

- **Emergency Runlevel File Manipulation:** Remounting file trees (`mount -o remount,rw`) and understanding chroot environments (`chroot`) to perform administrative rescue workflows.

- **Non-Interactive Credential Manipulation:** Resetting account credentials cleanly via input streams (`passwd --stdin`) under minimal shell footprints.

- **SELinux Security Context Alignment:** Mastering the automated system relabeling trigger (`/.autorelabel`) to prevent standard context access blocks or persistent system boot loops.

  

# 17. Configure Your System to Use Default Repositories

- **Difficulty:** ★

- YUM repositories are already available from:

  - `http://content/rhel9.0/x86_64/dvd/BaseOS`
  - `http://content/rhel9.0/x86_64/dvd/AppStream`

- Configure your system to use these locations as the default repositories.


***The solution to this task is exactly the same as that of the second task, so I will not repeat it here.***



# 18. Resize a Logical Volume

- **Difficulty:** ★★

- Resize the logical volume `vo` and its filesystem to **230 MiB**. Ensure that the filesystem contents remain intact.

  *Note: Partition sizes rarely match the requested size exactly, so a size ranging from 213 MiB to 243 MiB is acceptable.*

#### I. Investigate the Target Volume and Filesystem Type

Before executing any resize operations, you must check the filesystem type and locate the active block device path of the logical volume `vo`:

```bash
df -hT | grep vo
```

*(Take note of whether the output lists `ext4` or `xfs`. In standard RHEL9 environments, `vo` is formatted as `ext4` or `xfs`).*

#### II. Resize the Logical Volume and Filesystem Simultaneously

To safely change the logical volume size to exactly **230 MiB** and automatically scale the underlying filesystem in a single operation without data loss, use the `lvextend`/`lvreduce` engine with the `-r` flag:

- **If the logical volume needs to be shrunk or expanded:**

  ```bash
  lvextend -L 230M -r /dev/vgname/vo
  ```

  *or if reducing:*

  ```bash
  lvreduce -L 230M -r /dev/vgname/vo
  ```

> 💡 **Best Practice Exam Tip:** Running the command with the `-r` flag is the most reliable way to pass this task. The `-r` switch automatically triggers the correct underlying tool (such as `resize2fs` for `ext4` or `xfs_growfs` for `XFS`) to scale the storage footprint dynamically.

#### III. Verify the Final Storage Footprint

Verify that the logical volume block layer has adjusted perfectly and that the mounted filesystem reflects the target capacity:

```bash
# 1. Check the block device configuration parameters
lvdisplay /dev/vgname/vo

# 2. Check the active filesystem capacity
df -hT | grep vo
```

## Command Option and Parameter Explanation

- **`-L 230M`**: Specifies the absolute target size constraint for the logical volume (230 Megabytes). Because it declares an absolute target capacity destination, do not prefix the number with a plus `+` or minus `-` sign (e.g., `-L -230M`would mistakenly try to subtract 230 MiB from the current allocation instead of setting it to 230 MiB total).
- **`-r` (Resize Filesystem)**: This is the most crucial argument for storage modification tasks. It instructs the LVM engine to automatically sync the filesystem layer alongside the block layer changes. This guarantees that `ext4`filesystems shrink or expand safely, and `XFS` filesystems expand cleanly online without causing metadata corruption.
- **Storage Path Formatting:** The block path can be referenced via `/dev/vgname/vo` or `/dev/mapper/vgname-vo`interchangeably depending on how the device mapper initializes the mappings.

## Core Knowledge Points Covered

- **LVM Block Capacity Moderation:** Safely modifying live block storage layers (`lvextend`/`lvreduce`) within live system environments without destroying existing logical volume layouts.

- **Unified Filesystem Synchronization (`-r`):** Utilizing combined parameter execution switches to automatically trigger filesystem modifications based on partition metadata detection.

- **Data Integrity Preservation:** Ensuring administrative changes strictly maintain existing files and system assets completely untouched throughout shrinking or expansion phases.

- **Storage Environment Auditing:** Leveraging runtime reporting diagnostics (`df -hT` and `lvdisplay`) to discover volume parameters, structural target layouts, and filesystem boundaries.

  

# 19. Add Swap Partition

- **Difficulty:** ★★

- Add an additional swap partition of **512 MiB** to your system.

- The swap partition should be automatically mounted at system boot.

- Do not delete or in any way alter any existing swap partitions on the system.


#### I. Create a New Storage Partition

1. Check the available storage disks and existing partitions using `fdisk -l` or `lsblk` to identify the disk with unallocated space (e.g., `/dev/sdb` or `/dev/vdb`):

   ```bash
   lsblk
   ```

2. Open the selected disk in the `fdisk` utility (assuming `/dev/sdb` is the target disk containing free space):

   ```bash
   fdisk /dev/sdb
   ```

3. Enter the following keystroke sequence inside the interactive `fdisk` prompt to carve out a new 512 MiB partition:

   - Type **`n`** and press **Enter** *(Create a new partition)*.
   - Press **Enter** to accept the default partition type *(Primary)*.
   - Press **Enter** to accept the default starting sector.
   - Type **`+512M`** and press **Enter** *(Specify the exact size constraint)*.
   - Type **`t`** and press **Enter** *(Change the partition's system ID)*.
   - If prompted for a partition number, select the number corresponding to the partition you just created.
   - Type **`82`** (or type `swap` if using a GPT disk label) and press **Enter** *(Sets the hex code type to Linux Swap)*.
   - Type **`w`** and press **Enter** *(Write the changes to the partition table and exit)*.

4. Inform the Linux kernel operating system of the partition table changes so it maps the new block device immediately without requiring a reboot:

   ```bash
   udevadm settle
   ```

   *(Alternatively, you can run `partprobe`)*.

#### II. Format the Partition as Swap Space

Initialize the newly created partition block (e.g., `/dev/sdb1`) into standard system swap space:

```bash
mkswap /dev/sdb1
```

*Copy the **UUID** output generated by the `mkswap` command from your terminal screen, as it is required for building a persistent mounting configuration in the next step.*

#### IV. Configure Persistent Mounting at System Boot

Edit the primary storage map table configuration file to ensure the swap area mounts automatically at startup:

```bash
vim /etc/fstab
```

Append the following persistent configuration entry at the end of the file (replace the placeholder UUID with your exact copied string):

```ini
UUID=xxxx-xxxx-xxxx-xxxx  none  swap  defaults  0 0
```

Save and exit the editor (press `Esc`, then type `:wq` and press `Enter`).

#### V. Activate the Swap Area and Verify

1. Activate all swap areas defined inside `/etc/fstab` simultaneously:

   ```bash
   swapon -a
   ```

2. Verify that the new 512 MiB space has been pooled successfully and is actively running alongside any pre-existing swap memory allocations:

   ```bash
   swapon --show
   ```

## Command Parameter and Configurations Explanation

- **`fdisk` / `udevadm settle`**: `fdisk` modifies low-level disk sector structures interactively. Running `udevadm settle`acts as a barrier synchronization command, forcing the shell to wait until the system device manager (`udev`) finishes setting up the new `/dev/sdbX` node paths before formatting occurs.
- **`mkswap`**: Formats the target block space, building special metadata header footprints that allow the Linux kernel memory manager to use the space for virtual memory paging operations.
- **`/etc/fstab` parameters (`none swap defaults 0 0`)**:
  - `none`: Indicates that swap space does not possess a traditional target path mount point inside the unified directory tree (`/`).
  - `swap`: Declares the explicit filesystem driver type used to mount the device layer.
  - `defaults`: Implements the standard boot-time mounting options block.
  - `0 0`: Disables dump utility file scanning and filesystem check sequences (`fsck`) since swap memory contains raw memory pages rather than standard persistent file layouts.

## Core Knowledge Points Covered

- **Disk Partitioning Mechanics:** Provisioning physical raw storage disks safely via partitioning tools (`fdisk`), configuring exact size criteria (`+512M`), and mapping appropriate hex type definitions (`82` for Linux Swap).

- **Swap Subsystem Initialization:** Formatting physical storage boundaries into functional system memory extensions using `mkswap`.

- **High-Availability Storage Table Automation:** Registering block-level UUID identifiers into persistent automation definitions (`/etc/fstab`) to guarantee independent stability following system reboots.

- **System Virtual Memory Diagnostics:** Reviewing virtual memory allocation states and verification procedures (`swapon --show` / `free -m`) to confirm real-time resource tuning updates.

  

# 20. Create a Logical Volume

- **Difficulty:** ★★★★★

- Create a new logical volume according to the following requirements:

  - The logical volume should be named `qa`, belong to the `qagroup` volume group, and have a size of **60 extents**.

  - The logical volume extent size in the `qagroup` volume group should be **16 MiB**.

  - Format the new logical volume with a `vfat` filesystem. This logical volume should be automatically mounted under `/mnt/qa` at system boot.


#### I. Initialize the Physical Volume and Create the Volume Group

1. Locate your available unallocated disk or partition (for example, `/dev/sdb2` or a dedicated free disk like `/dev/sdc`) using `lsblk` or `fdisk -l`.

2. Initialize the block device as an LVM Physical Volume (PV):

   ```bash
   pvcreate /dev/sdb2
   ```

3. Create the Volume Group named **`qagroup`** and explicitly define its Physical Extent (PE) size to **16 MiB** using the `-s`flag:

   ```bash
   vgcreate -s 16M qagroup /dev/sdb2
   ```

#### II. Create the Logical Volume by Extent Count

Create a new logical volume named **`qa`** allocated with exactly **60 extents** from the `qagroup` volume group:

```bash
lvcreate -l 60 -n qa qagroup
```

*(Note: Because each extent is set to 16 MiB, allocating 60 extents dynamically yields a total block size of 60×16 MiB=960 MiB).*

#### III. Format the Logical Volume with the vfat Filesystem

Format the newly created logical volume block using the `vfat` filesystem driver as required:

```bash
mkfs.vfat /dev/qagroup/qa
```

#### IV. Configure Persistent Automatic Mounting

1. Create the designated target mount point directory:

   ```bash
   mkdir -p /mnt/qa
   ```

2. Open the file storage system mapping configuration file:

   ```bash
   vim /etc/fstab
   ```

3. Append the following persistent configuration row at the bottom of the file:

   ```ini
   /dev/qagroup/qa  /mnt/qa  vfat  defaults  0 0
   ```

   Save and exit the text editor (press `Esc`, then type `:wq` and press `Enter`).

#### V. Mount and Verify the Storage Layout

1. Trigger the system mounting daemon to parse and mount all entries inside `/etc/fstab`:

   ```bash
   mount -a
   ```

2. Run storage validation checks to verify that the volume layer configuration is correct:

   ```bash
   # 1. Verify the Volume Group PE size
   vgdisplay qagroup | grep "PE Size"
   
   # 2. Verify the Logical Volume Extent Allocation
   lvdisplay /dev/qagroup/qa | grep "Current LE"
   
   # 3. Verify the active filesystem mapping and mount path
   df -hT | grep qa
   ```

## Command Parameter and Syntax Explanation

- **`vgcreate -s 16M`**: The `-s` (or `--physicalextentsize`) option overrides the standard LVM default metadata mapping block size (which is usually 4 MiB) and enforces a strict custom boundary size (16 MiB in this case) on the target Volume Group.
- **`lvcreate -l 60`**: The lowercase `-l` flag instructs the logical volume wizard to allocate block sizes via discrete logical/physical extents quantities rather than hard storage block sizes (which would be defined using uppercase `-L`, such as `-L 960M`).
- **`mkfs.vfat`**: Initializes a standard FAT32 / VFAT local partition configuration layout across the LVM mapper node.
- **`/etc/fstab` fields (`/dev/qagroup/qa /mnt/qa vfat defaults 0 0`)**: Automatically attaches the virtual volume block path to the target subdirectory on boot using standard options and skip rules.

## Core Knowledge Points Covered

- **Custom Volume Group Topology Sizing:** Overriding systemic storage boundaries by explicitly defining individual PE metadata blocks (`vgcreate -s`).

- **Extent-Driven Provisioning:** Allocating virtual block device mappings via index counters (`lvcreate -l`) rather than raw hardware byte capacities.

- **Non-Linux Native Filesystem Handling:** Deploying alternative storage formatting properties (`mkfs.vfat`) to meet multi-platform partition design criteria.

- **Persistent Partition Availability:** Translating volume paths, target structures, and file systems into boot-safe automation rows (`/etc/fstab`) that remain resilient across hard system reboots.

  

# 21. Configure System Tuning

- **Difficulty:** ★

- Select the recommended `tuned` profile for your system and set it as the default configuration.

#### I. Query the Recommended Tuning Profile

Run the `tuned-adm` utility to query the system hardware environment and determine the recommended optimization profile:

```bash
tuned-adm recommend
```

**Expected Output Example:**

```ini
virtual-guest
```

*(Note: In most RHCSA virtualized exam environments, the recommended profile output will be `virtual-guest` or `throughput-performance`).*

#### II. Apply the Recommended Profile as Default

Apply the exact profile returned by the recommendation query (assuming the recommended profile is `virtual-guest`):

```bash
tuned-adm profile virtual-guest
```

#### III. Verify the Active Profile Configuration

Verify that the chosen tuning profile has been successfully applied and is currently active on the system:

```bash
tuned-adm active
```

**Expected Output:**

```ini
Current active profile: virtual-guest
```

## Command Parameter and Profile Explanation

- **`tuned-adm recommend`**: Scans system attributes—such as virtualization layers, storage types, CPU architectures, and audio/video hardware—to output the name of the most ideal optimization template profile.
- **`tuned-adm profile <profile-name>`**: Switches the runtime profile to the designated configuration template. This change is completely persistent and automatically writes rules to the backend configurations so it re-applies automatically upon system boot.
- **Common Tuned Profiles:**
  - `virtual-guest`: Optimizes storage and network settings for RHEL systems running inside enterprise hypervisors.
  - `throughput-performance`: Disables aggressive power-saving modes to tune execution queues for maximum processing compute throughput.

## Core Knowledge Points Covered

- **Dynamic System Performance Tuning:** Leveraging the `tuned` daemon architecture to enforce hardware performance policies dynamically.
- **System Environment Auditing:** Utilizing built-in recommendation algorithms (`tuned-adm recommend`) to evaluate operational requirements.
- **Configuration Persistence Verification:** Enforcing active system-wide profile changes (`tuned-adm profile`) and auditing status lines (`tuned-adm active`) to guarantee configuration retention across system reboots.