# 1. Configure Network Settings

- **Difficulty:** ★
- Configure **node1** with the following network configuration:

  - **Hostname:** `node1.domain250.example.com`
  - **IP Address:** `172.25.250.100`
  - **Subnet Mask:** `255.255.255.0`
  - **Gateway:** `172.25.250.254`
  - **DNS Server:** `172.25.250.254`


#### I. Start nmtui

Open the Node1 virtual machine directly. Enter: `nmtui`
Upon entering the main interface, you will see three options:

- `Edit a connection`
- `Activate a connection`
- `Set system hostname`

#### II. Configure Hostname

1. Select `Set system hostname` on the main interface and press **Enter**.
2. Input the hostname required by the question: `node1.domain250.example.com`
3. Press **Tab** to navigate to `OK`, then press **Enter** to confirm.
4. Press **Tab** to navigate to `Back`, then press **Enter** to return to the main interface.
   
#### III. Configure IP / Subnet Mask / Gateway / DNS

1. Select `Edit a connection` on the main interface and press **Enter**.
2. Locate your network interface card (e.g., `Wired connection 1`), select it, and press **Enter**.
3. Once in the editing interface, modify the `IPv4 configuration` first:
       - Press **Tab** to navigate to `<Automatic>` next to `IPv4 configuration`, and press **Enter**.
       - Select `Manual` mode and press **Enter**.
4. Configure the IP address and Subnet Mask:
       - Press **Tab** to navigate to the `Addresses` field, then press `<Add>`.
       - Input `172.25.250.100/24` *(Note: `/24` represents the subnet mask `255.255.255.0` specified in the question)*.
5. Configure the Gateway:
       - Navigate to the `Gateway` field and input `172.25.250.254`.
6. Configure the DNS Server:
       - Navigate to the `DNS servers` field, press `<Add>`, and input `172.25.250.254`.
7. Press **Tab** to navigate to `OK`, then press **Enter** to save the configuration.
8. Press **Tab** to select `Back` and return to the main interface.
   
#### IV. Activate the Network Connection (Apply Configurations)

1. Select `Activate a connection` on the main interface and press **Enter**.

2. Locate the network interface card you just configured, select it, and press **Enter** *(It will toggle to display `Activate`or show it is active)*.

3. Once the status changes to `Active`, the activation is successful. Press `Back` to return.

4. Select `Quit` to exit `nmtui`.

5. Restart the network service: `systemctl restart NetworkManager`

## Core Knowledge Points Covered

- **Network Configuration via TUI (Text User Interface):** Testing your proficiency in using the `nmtui` utility to configure system network settings without relying on a graphical user interface (GUI). This is crucial for managing headless servers.
- **Static IP Assignment & Subnet Calculation:** Understanding how to transition from dynamic (DHCP) to manual (Static) IP allocation, and correctly converting a traditional subnet mask (e.g., `255.255.255.0`) into CIDR notation (`/24`).
- **Persistent Hostname Management:** Configuring a Fully Qualified Domain Name (FQDN) that correctly updates system runtime and network profiles so that it remains intact across reboots.
- **Gateway and DNS Resolution:** Correctly routing outbound traffic by specifying the network gateway and establishing name resolution by pointing to the designated DNS server.
- **Network Profile Activation & Daemon Control:** Ensuring you understand that editing network configurations does not automatically apply them; you must know how to activate the profile using `nmtui` or trigger a reload via `systemctl restart NetworkManager`.

   

# 2. Configure Your System to Use Default Repositories

- **Difficulty:** ★

- Configure the default YUM repositories for **node1** using these locations:

  - `http://content/rhel9.0/x86_64/dvd/BaseOS`

  - `http://content/rhel9.0/x86_64/dvd/AppStream`
  
    

First, navigate to the repository configuration directory:

```bash
cd /etc/yum.repos.d/
```

Create and edit a new repository file using `vim` (the filename can be any name ending with `.repo`, here we use `rh.repo`):

```bash
vim rh.repo
```

Add the following configuration blocks inside the file:

```ini
[BaseOS]
name=BaseOS
baseurl=http://content/rhel9.0/x86_64/dvd/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://content/rhel9.0/x86_64/dvd/AppStream
enabled=1
gpgcheck=0
```

Save and exit the editor (press `Esc`, then type `:wq` and press `Enter`).

Finally, clear the old package cache and generate a new metadata cache to verify that the repositories are configured successfully:

```bash
yum clean all
yum makecache
```

*(Note: You can also use `dnf clean all` and `dnf makecache`, as `yum` is a symbolic link to `dnf` in RHEL9).*

## Configuration Block Parameter Explanation

- **`[Repository_ID]`**: A unique identifier for the repository enclosed in square brackets (e.g., `[BaseOS]`).
- **`name`**: A human-readable descriptive name for the repository.
- **`baseurl`**: The URL pointing to the directory where the repository's data and metadata (`repodata`) are located. It can use protocols like `http://`, `https://`, `ftp://`, or `file://`.
- **`enabled=1`**: Tells the package manager to actively use this repository when searching for or installing packages (`1`means enabled, `0` means disabled).
- **`gpgcheck=0`**: Disables GPG signature checking for packages from this repository (`0` means disabled). In an exam environment, this bypasses the requirement to import a public GPG key, allowing quick software installations.

## Core Knowledge Points Covered

- **DNF/YUM Repository Management:** Understanding the architecture of software package management in RHEL9 and knowing exactly where the network/local repository configuration files must reside (`/etc/yum.repos.d/`).

- **Repository Configuration Syntax:** Writing flawless `.repo` configuration files manually without syntax errors, which includes setting key-value pairs (`name`, `baseurl`, `enabled`, `gpgcheck`).

- **Differentiating BaseOS and AppStream:** Handling the multi-repo structure of RHEL9, where core OS packages reside in `BaseOS` and modules/applications reside in `AppStream`.

- **Cache Management:** Flushing old repository metadata and validating new network sources using `clean all` and `makecache` or `repolist` commands to ensure system package metadata is fully up to date.

  

# 3. Debug SELinux

- **Difficulty:** ★★★

- The Web server failed to run on a non-standard port **82**. Please troubleshoot and resolve the issue encountered by the web server as needed, so that it meets the following conditions:

  - The Web server on the system can serve all existing HTML files in `/var/www/html` (Note: Do not delete or otherwise modify the existing file contents).

  - The Web server provides this content on port **82**.

  - The Web server starts automatically at system boot.

  - Ensure the SELinux mechanism is running in **Enforcing** mode.

#### Ⅰ. Modify Apache Configuration to Listen on Port 82

Edit Apache’s main configuration file:

```bash
vim /etc/httpd/conf/httpd.conf
```

Locate the line `Listen 80` and change it to:

```bash
Listen 82
```

Save and exit the editor (`Esc`, then type `:wq` and press `Enter`).

*(Note: If the file already states `Listen 82`, no changes are needed).*

#### Ⅱ. Resolve SELinux Port Restrictions (Critical Step)

By default, SELinux does not allow `httpd` to bind to non-standard ports (ports other than 80/443). Therefore, you must add a policy label for port 82:

```bash
# Permanently add the SELinux port rule
semanage port -a -t http_port_t -p tcp 82

# Verify if the port has been added successfully
semanage port -l | grep http_port_t
```

If you see `82` in the command output, it indicates success.

#### Ⅲ. Start Apache and Enable Auto-start at Boot

```bash
# Start the service
systemctl start httpd

# Configure the service to start automatically at system boot
systemctl enable httpd
```

#### Ⅳ. Allow Port Through Firewall (If firewalld is running)

```bash
# Permanently allow port 82/tcp
firewall-cmd --permanent --add-port=82/tcp

# Reload firewall rules to apply changes
firewall-cmd --reload
```

#### Ⅴ. Verify All Conditions

##### Checking SELinux Mode

```bash
getenforce
```

The output must be `Enforcing`. If it is not, modify `/etc/selinux/config`:

```bash
vim /etc/selinux/config
```

Change `SELINUX=disabled` or `SELINUX=permissive` to `SELINUX=enforcing`, then reboot the system to apply changes.

##### Verifying Security Contexts

```bash
ls -lZ /var/www/html/
```

If the security contexts of the files are not all set to `httpd_sys_content_t`, you must modify them:

```bash
# Set the correct SELinux context for an individual file
chcon -t httpd_sys_content_t /var/www/html/file1

# Double-check the context
ls -lZ /var/www/html/
```

Ensure all files display identical, appropriate contexts before finishing.

## Core Knowledge Points Covered

- **SELinux Port Label Management:** Utilizing `semanage port` to map a non-standard web service port (`82/tcp`) to the network port context type (`http_port_t`), resolving access violations in Enforcing mode.

- **SELinux Enforcement Policies:** Understanding how to check system modes via `getenforce` and persistently configuring the system to run in `Enforcing` mode via `/etc/selinux/config`.

- **File Security Contexts:** Checking and manipulating security labels on the filesystem using `ls -lZ` and `chcon` to guarantee that the Apache daemon possesses proper access permissions to web content files.

- **Service and Network Daemon Control:** Modifying application-layer parameters in configuration files (`httpd.conf`), managing system systemd daemons with `systemctl`, and unblocking network sockets permanently across reboots through `firewall-cmd`.

  

# 4. Create User Accounts

- **Difficulty:** ★

- Create user and group accounts according to the following requirements:

  - A group named `sysmgrs`.

  - A user `natasha`, belonging to `sysmgrs` as a secondary group.

  - A user `harry`, also belonging to `sysmgrs` as a secondary group.

  - A user `sarah`, who has no access to an interactive shell on the system and is not a member of `sysmgrs`.

  - The password for `natasha`, `harry`, and `sarah` should all be `seto`.

#### Ⅰ. Create the `sysmgrs` Group

```bash
groupadd sysmgrs
```

#### Ⅱ. Create Users `natasha` and `harry`, and Add Them to `sysmgrs` as a Supplementary Group

```bash
# Create users and assign their supplementary (secondary) group
useradd -G sysmgrs natasha
useradd -G sysmgrs harry
```

#### Ⅲ. Create User `sarah`, Deny Interactive Shell Access, and Do Not Join `sysmgrs`

```bash
# -s /sbin/nologin specifies that the user cannot log into an interactive shell
useradd -s /sbin/nologin sarah
```

#### Ⅳ. Set the Password to `seto` for All Three Users

*(Note: Using pipe with `passwd --stdin` is the recommended method for exams to perform batch password configurations without interactive prompts).*

```bash
echo "seto" | passwd --stdin natasha
echo "seto" | passwd --stdin harry
echo "seto" | passwd --stdin sarah
```

## Command Parameter Explanation

- **`groupadd`**: Creates a new group definition in the system by adding appropriate entries to `/etc/group`.
- **`useradd -G`**: Defines supplementary (secondary) groups for the user account. The user will still belong to their primary group (usually a group with the same name as the user) but will also inherit permissions from the groups listed after `-G`.
- **`useradd -s /sbin/nologin`**: Sets the user's default login shell to a non-executable path. If this user attempts to log in via SSH or a console local terminal interface, the system will reject the interactive shell session immediately.
- **`passwd --stdin`**: Instructs the `passwd` utility to read the new password token directly from standard input (passed via the pipe `|` command) rather than prompting for interactive terminal responses.

## Core Knowledge Points Covered

- **Local Group Administration:** Knowing how to create system workgroups (`groupadd`) to manage permissions collectively for multi-user shared operational environments.

- **User Lifecycle Management:** Utilizing specialized flags within `useradd` to dictate account characteristics (such as assigning secondary group memberships via `-G`).

- **Shell Security Restrictions:** Mitigating unauthorized shell level exposures for service or target non-login accounts by declaring non-interactive fallback shells like `/sbin/nologin`.

- **Non-interactive Automation:** Implementing standardized automated script pipelines (`echo "password" | passwd --stdin username`) to efficiently configure batch security credentials under a time-restricted environment.

  

# 5. Configure a Cron Job

- **Difficulty:** ★

- Configure a cron job that executes `/usr/bin/echo hello` daily at **14:23** as user `harry`.

#### Ⅰ. Directly Edit Harry's Crontab as the `root` User

```bash
crontab -u harry -e
```

#### Ⅱ. Input the Scheduled Task

Once inside the editor interface, append the following line:

```ini
23 14 * * * /usr/bin/echo hello
```

#### Ⅲ. Save, Exit, and Verify

Save and exit the editor (`Esc`, then type `:wq` and press `Enter`).

Verify that the configuration was successfully written by listing the cron jobs for the user:

```bash
crontab -u harry -l
```

If you see the `23 14 * * * /usr/bin/echo hello` entry in the output, the configuration is successful.

> ⚠️ Pay Close Attention to Spelling!

## Cron Field and Parameter Explanation

The standard format of a cron entry consists of five time-and-date fields followed by the absolute path of the command:

**Minute Hour Day_of_Month Month Day_of_Week Command**

- **`23`**: The exact minute the task triggers (23rd minute).
- **`14`**: The hour in 24-hour format (14 represents 2:00 PM).
- **`*` (Day of Month)**: Executes every day of the month.
- **`*` (Month)**: Executes every month.
- **`*` (Day of Week)**: Executes every day of the week (Sunday through Saturday).
- **`-u harry`**: Specifies the target user context under which the cron job belongs and will execute.
- **`-e`**: Opens the target user's crontab file using the system's default text editor (usually `vim`).
- **`-l`**: Lists the current scheduled jobs for the specified user profile.

## Core Knowledge Points Covered

- **Impersonated Cron Administration:** Utilizing the `-u` option to securely manipulate crontab profiles for separate local users while acting from the administrative root account.

- **Cron Time Field Mapping:** Understanding chronological expressions in Vixie cron format to accurately pin job schedules down to specific hours and minutes (`14:23` → `23 14 * * *`).

- **Absolute Path Execution:** Recognizing the critical importance of declaring the full absolute path (`/usr/bin/echo`) inside automated environment scripts, since cron daemons execute commands within restricted minimal environmental shells (`PATH`).

- **Crontab Validation Techniques:** Leveraging `crontab -l` to reliably dump active memory schedules and audit systemic runtime tables without having to wait for the actual target execution time to pass.

  

# 6.1 Create a Collaborative Directory (Bonus Question 1)

*(One of the three bonus questions will appear randomly)*

- **Difficulty:** ★

- Create a collaborative directory `/home/managers` with the following characteristics:

  - The group ownership of `/home/managers` is `sysmgrs`.

  - The directory should be readable, writable, and accessible by members of `sysmgrs`, but no other users should have these permissions. (Of course, the `root` user has access to all files and directories on the system).

  - Files created in `/home/managers` should automatically inherit the group ownership of the `sysmgrs` group.

#### I. Create the Directory and Set Group Ownership

```bash
# 1. Create the target directory
mkdir /home/managers

# 2. Set the group ownership to sysmgrs
chgrp sysmgrs /home/managers
```

#### II. Set Permissions (Ensure Read/Write Access for Group, Deny Access for Others)

The task requires that members of the `sysmgrs` group can read, write, and access the directory, while all other users have no permissions.

```bash
# Set permissions: rwxrwx--- (Owner: rwx, Group: rwx, Others: ---)
chmod 770 /home/managers
```

#### III. Set the SGID Bit (Automatic Group Ownership Inheritance for New Files)

This is the core point tested in this question. The SGID bit ensures that any new files created inside this directory automatically inherit the group ownership of the parent directory (`sysmgrs`) rather than the primary group of the user who created them.

```bash
# Apply the SGID bit to the directory
chmod g+s /home/managers
```

#### IV. Verify All Configurations

```bash
# Inspect the permissions and attributes of the directory
ls -ld /home/managers
```

You should see an output similar to this, indicating the configuration is correct:

```ini
drwxrws---. 2 root sysmgrs 4096 May 20 10:00 /home/managers
```

*(Note: The lowercase `s` in the group execution field `rws` confirms that both the group execution permission and the SGID bit are active).*

## Command and Permission Field Explanation

- **`mkdir` / `chgrp`**: `mkdir` provisions the physical directory path, while `chgrp` reassigns its group affiliation to the designated group target without modifying owner properties.
- **`chmod 770`**: Represents standard octal notation where the owner (`root`) gets `7` (rwx), the group (`sysmgrs`) gets `7`(rwx), and others get `0` (−−−).
- **`chmod g+s`**: Injects the special Set Group ID (SGID) permission flag into the group permission block. When applied to a directory, it switches filesystem behavior from inheriting the creator's default group to inheriting the parent folder's group identity for all nested creations.

## Core Knowledge Points Covered

- **Collaborative Directory Deployment:** Setting up dedicated shared workspaces (`mkdir`, `chgrp`) intended for team collaboration while securing group-level control gates.

- **Octal and Absolute Permissions Control:** Implementing absolute permission restrictions via `chmod 770` to block unauthorized outside access (`others=0`) while granting full operational rights to team members.

- **Special Filesystem Permissions (SGID):** Mastering the utilization of the SGID bit (`chmod g+s`) on directory nodes to enforce persistent group ownership mechanics automatically across multiple active user accounts.

- **Filesystem Attribute Auditing:** Using advanced `ls` visualization arguments (`-ld`) to inspect special permission bits (`s`) and validate multi-tenant directory states before final system deployment.

  

# 6.2 Set Default Password Policy (Bonus Question 2)

*(One of the three bonus questions will appear randomly. Correct answer for this gets no points)*

- **Difficulty:** ★

- Set a password policy for newly created users such that the password expires by default after **25 days** when a user is created.

#### I. Modify the Default Login Definitions Configuration File

To set a default password aging policy for newly created users, you must edit the `/etc/login.defs` configuration file.

```bash
vim /etc/login.defs
```

#### II. Update the Password Expiration Parameter

Locate the parameter controlling the maximum number of days a password may be used. Change its value to `25` as required by the question:

```ini
PASS_MAX_DAYS   25
```

Save and exit the editor (press `Esc`, then type `:wq` and press `Enter`).

#### III. Verify the Configuration Change

You can use `grep` to quickly verify that the parameter has been correctly updated inside the file:

```bash
grep PASS_MAX_DAYS /etc/login.defs
```

## Configuration Parameter Explanation

- **`/etc/login.defs`**: The primary configuration file that defines site-specific configuration flags for the shadow password suite, controlling default system settings such as password aging, UID/GID ranges, and mailbox locations for new accounts.
- **`PASS_MAX_DAYS`**: Specifies the maximum number of days a password may be valid. After this period elapsed, the system forces the user to change their password during their next login attempt.
- **Important Exam Note:** Changes made to `/etc/login.defs` **only affect accounts created after the modification**. Existing system users or accounts created prior to editing this file will retain their original password expiration settings.

## Core Knowledge Points Covered

- **System-wide Account Default Definitions:** Understanding how to manipulate the `/etc/login.defs` configuration structure to establish uniform baseline policies across the operating system lifecycle.

- **Password Aging and Lifecycle Policy Enforcement:** Implementing standardized organizational security compliance parameters (such as `PASS_MAX_DAYS`) to enforce periodic credential rotation.

- **Differentiating Active Configurations vs. Global Presets:** Recognizing the behavioral boundary of template parameters in `/etc/login.defs`—knowing that it sets ambient properties for future users (`useradd`) but does not dynamically update active shadow entry files (`/etc/shadow`) for pre-existing accounts.

  

# 6.3 Create a System Monitoring Script (Bonus Question 3)

*(One of the three bonus questions will appear randomly. Correct answer for this gets no points)*

- **Difficulty:** ★★
- Create a script named `systeminfo`.
- The script should be placed under `/usr/local/bin` and is used to obtain information about current system processes.

  - The output must be displayed in the following order: Process Owner, Process PID, Virtual Memory consumed by the process, Resident (Actual) Memory, and CPU Percentage.

  - Sort the output by CPU percentage, with the process consuming the most CPU displayed at the very end.

#### I. Create and Edit the Script File

Create a new file named `systeminfo` inside the `/usr/local/bin` directory using `vim`:

```bash
vim /usr/local/bin/systeminfo
```

#### II. Write the Script Content

Input the following Bash code into the file:

```bash
#!/bin/bash
ps -eo user,pid,vsize,rss,%cpu --sort=%cpu
```

Save and exit the editor (press `Esc`, then type `:wq` and press `Enter`).

#### III. Grant Execution Permissions

To make the script executable by the system, you must add the execution permission (`+x`):

```bash
chmod +x /usr/local/bin/systeminfo
```

#### IV. Verify the Script Output

Run the script directly to verify that the format, columns, and sorting order meet the criteria:

```bash
systeminfo
```

## Command and Parameter Explanation

- **`#!/bin/bash`**: The "Shebang." It instructs the operating system kernel to execute the script using the Bash shell interpreter.
- **`ps`**: The process status utility used to capture a snapshot of current active system processes.
- **`-e`**: Instructs `ps` to select **all** processes running across the entire system.
- **`-o`**: Specifies a **user-defined format**. This allows you to explicitly declare which columns to display and in what exact order:
  - `user`: Process Owner.
  - `pid`: Process ID.
  - `vsize`: Virtual Memory consumed by the process.
  - `rss`: Resident Set Size (Actual physical memory consumed).
  - `%cpu`: CPU utilization percentage.
- **`--sort=%cpu`**: Directs `ps` to sort the processes in ascending order based on their CPU usage. This ensures that the process consuming the most CPU is automatically displayed at the very end of the output.

## Core Knowledge Points Covered

- **Custom Script Deployment:** Knowing how to author local utility scripts and placing them in standard execution paths (`/usr/local/bin`) so they can be invoked system-wide without specifying relative directory paths.

- **Advanced Process Auditing via `ps` Format Flags:** Mastering the `-o` parameter to customize structural output reports rather than relying on default `ps aux` dumps.

- **Output Sorting Logic:** Enforcing deterministic data layouts by combining internal application switches (`--sort`) to prioritize specific technical metrics (e.g., CPU load).

- **Executable Security attributes:** Modifying filesystem permissions via `chmod +x` to transition a raw plain text file into a functional, runnable binary tool script.

  

# 6.4 Create a File Search Script (Bonus Question 4)

*(One of the three bonus questions will appear randomly. Correct answer for this gets no points)*

- **Difficulty:** ★★

- Create a script named `myresearch`.

- The script should be placed under `/usr/local/bin` and is used to:

  - Find all files under `user` (or `/usr`) that are smaller than **10M** and have **SGID (Set Group ID)** permissions, and place these files under `/root/myfiles`.
  
    

#### I. Create the Target Directory

Before generating or running the script, ensure that the destination directory where the matched files will be copied exists:

```bash
mkdir /root/myfiles
```

#### II. Create and Edit the Script File

Create a new file named `myresearch` inside the `/usr/local/bin` directory using `vim`:

```bash
vim /usr/local/bin/myresearch
```

#### III. Write the Script Content

Input the following Bash script code into the file:

```bash
#!/bin/bash
find /usr -size -10M -perm /2000 -exec cp -a {} /root/myfiles/ \;
```

Save and exit the editor (press `Esc`, then type `:wq` and press `Enter`).

#### IV. Grant Execution Permissions

Make the script runnable by assigning the appropriate execution flag:

```bash
chmod +x /usr/local/bin/myresearch
```

#### V. Execute and Verify

Run the script to execute the search and copy operation:

```bash
myresearch
```

Verify that the files have been copied correctly by inspecting the destination folder:

```bash
ls -l /root/myfiles
```

## Script Command and Flag Explanation

- **`find /usr`**: Searches recursively starting from the `/usr` directory tree (as specified in the image).
- **`-size -10M`**: Filters for files strictly **less than 10 Megabytes**. *(Note: The minus sign `-` denotes "less than", while a plus sign `+` would denote "greater than").*
- **`-perm /2000`**: Searches for files that have the **SGID bit** set.
  - In octal mode, `2000` isolates the SGID attribute bit.
  - The forward slash `/` prefix acts as an "ANY" matching condition, ensuring that as long as the SGID bit is set, the file is caught regardless of its other standard read/write/execute permissions.
- **`-exec ... \;`**: For every file that matches all criteria, `find` spawns the trailing command inside the block.
  - `{}`: A dynamic placeholder token that represents the current matched file path found.
  - `cp -a`: Copies the files while **preserving all original file attributes**, permissions, and timestamps. This is critical in administrative exams to avoid altering metadata during transfers.
  - `\;`: Terminates the `-exec` statement parameter block.

## Core Knowledge Points Covered

- **Advanced Search Filters with `find`:** Leveraging complex conditional parameters simultaneously, such as constraining boundaries via relative size ranges (`-size -10M`).

- **Special Permission Filtering (SGID):** Identifying executable binaries carrying special privilege flags (`-perm /2000`) using symbolic or octal permission masks.

- **Inline Command Execution (`-exec`):** Passing query pipelines efficiently into sub-commands directly from the search output context without breaking actions into distinct scripting loops.

- **Preserving File Metadata during Operations:** Utilizing safe replication tools (`cp -a`) to prevent systemic permission corruption or owner shifts during automation sequences.

  

# 7. Configure NTP

- **Difficulty:** ★

- Configure your system to be an NTP client of `materials.example.com`. (Note: `materials.example.com` is a DNS alias for `classroom.example.com`).

#### I. Install the Chrony Daemon Package

In RHEL9, the default NTP implementation is managed by the `chrony` service. First, ensure the package is installed:

```bash
yum install -y chrony
```

#### II. Configure the NTP Server Peers

Edit the primary Chrony configuration file to specify the remote time server:

```bash
vim /etc/chrony.conf
```

Locate the server pool lines (usually starting with `pool` or `server`). Comment out or delete any pre-existing default servers, and add the server specified in the question:

```ini
server materials.example.com iburst
```

Save and exit the editor (press `Esc`, then type `:wq` and press `Enter`).

#### III. Restart and Enable the Chrony Service

Apply the configuration changes by restarting the daemon and setting it to start automatically at system boot:

```bash
systemctl restart chronyd
systemctl enable chronyd
```

#### IV. Verify Time Synchronization Status

To ensure your system is communicating with the server and syncing successfully, check the Chrony tracking and sources status:

```bash
# Check overall synchronization tracking metrics
chronyc tracking

# List current time sources and synchronization status
chronyc sources -v
```

*(Note: If you see an asterisk `\*` next to the server name in the `chronyc sources` output, it means the system has successfully synchronized time with that server).*

## Configuration Parameter Explanation

- **`/etc/chrony.conf`**: The central configuration file for the `chronyd` daemon, where servers, access control lists, and drift file paths are declared.
- **`server materials.example.com`**: Tells the daemon to synchronize with this specific individual host rather than an unpredictable pool of servers.
- **`iburst`**: Short for "initial burst". When the chronyd service restarts, it sends a burst of four to eight packets to the server in quick succession instead of waiting for the default polling interval. This allows the client to calculate the time offset and synchronize the system clock within a few seconds of boot.

## Core Knowledge Points Covered

- **NTP Service Implementation via Chrony:** Knowing that RHEL9 uses `chronyd` exclusively for time tracking and synchronization rather than the legacy `ntpd` daemon.

- **Network Peer Configuration:** Identifying and writing valid `server` directives inside configuration baselines while leveraging performance flags like `iburst` for accelerated environment synchronization.

- **Service Lifecycle Automation:** Managing runtime daemons persistently via `systemctl` so network dependencies align correctly following unassisted cold reboots.

- **NTP Diagnostics and Auditing:** Utilizing advanced client commands (`chronyc tracking` and `chronyc sources`) to review time offsets, stratum levels, and active peer synchronization tags.

  

# 8. Configure autofs

- **Difficulty:** ★★

- Configure `autofs` to automatically mount the home directories of remote users as described below:

  - `materials.example.com` exports `/rhome` via NFS to your system. This file system contains pre-configured home directories for the user `remoteuser1`.

  - The home directory for `remoteuser1` is `materials.example.com:/rhome/remoteuser1`.

  - The home directory for `remoteuser1` should be automatically mounted locally under `/rhome` as `/rhome/remoteuser1`.

  - The home directory must be writable by its user.
  
    

#### I. Install the autofs Package

First, ensure that the `autofs` utility package is installed on your system:

```bash
yum install -y autofs
```

#### II. Configure the Master Map File

Edit the primary autofs master configuration file to define the base mount point and point to its corresponding map file:

```bash
vim /etc/auto.master
```

Append the following line at the end of the file:

```ini
/rhome  /etc/auto.nfs
```

Save and exit the editor (press `Esc`, then type `:wq` and press `Enter`).

#### III. Create and Configure the Specific Map File

Create the new map file `/etc/auto.nfs` that you just declared in the master file:

```bash
vim /etc/auto.nfs
```

Add the specific mount entry for the remote home directory inside this file:

```ini
remoteuser1  -rw,sync  materials.example.com:/rhome/remoteuser1
```

Save and exit the editor (press `Esc`, then type `:wq` and press `Enter`).

#### IV. Start and Enable the autofs Service

Apply the configuration changes by restarting the service daemon and setting it to start automatically at system boot:

```bash
systemctl restart autofs
systemctl enable autofs
```

#### V. Verify the On-Demand Mounting

By default, `df -h` will not show the mount because `autofs` only mounts filesystems **on-demand** when the directory is accessed. Switch to the target user or access the directory path to trigger the automatic mount:

```bash
# Switch to the target remote user to verify trigger
su - remoteuser1

# Verify the active mount entry
df -h
```

If the command completes successfully and `df -h` displays the NFS share mounted under `/rhome/remoteuser1`, the configuration is successful.

## Configuration Line Parameter Explanation

- **`/rhome /etc/auto.nfs` (in `auto.master`)**:
  - `/rhome`: The base directory path where the dynamically managed subdirectories will be created and mounted.
  - `/etc/auto.nfs`: The path to the secondary map file containing the explicit mount options and backend server locations.
- **`remoteuser1 -rw,sync materials.example.com:/rhome/remoteuser1` (in `auto.nfs`)**:
  - `remoteuser1`: The key (the name of the dynamic subdirectory created under `/rhome`).
  - `-rw,sync`: Mount options. `-rw` grants read and write permissions as required by the question, and `sync` forces synchronous I/O operations.
  - `materials.example.com:/rhome/remoteuser1`: The absolute source remote NFS export location.

## Core Knowledge Points Covered

- **On-Demand Filesystem Mounting via `autofs`:** Understanding the difference between persistent static mounts via `/etc/fstab` and dynamic, resource-efficient, on-demand filesystem mounting managed by `autofs`.
- **Master and Direct/Indirect Map Hierarchies:** Writing complementary entries in both the global master configuration (`auto.master`) and standalone indirect map files (`auto.nfs`).
- **NFS Client-side Mount Configurations:** Passing precise mount attributes (such as specifying read-write privileges via `-rw`) to secure operational capabilities for network-provided user directories.
- **Validating Dynamic Trigger Mounts:** Mastering verification procedures for reactive file daemons—knowing that you must interact with the target directory path (e.g., via `su -` or `cd`) to force execution and trigger the kernel-level mount.


​    


# 9. Configure a User Account

- **Difficulty:** ★

- Configure a user `seto` with a User ID (UID) of `0815`. The password for this user should be `seto`.

  

#### I. Create the User Account with Specified UID

To create a user with a specific User ID (UID) and ensure it matches the exam criteria, use the `-u` option followed by the UID number:

```bash
useradd -u 0815 seto
```

#### II. Set the User Password Non-Interactively

Set the password to `seto` for the newly created user using the standard pipeline method:

```bash
echo "seto" | passwd --stdin seto
```

#### III. Verify the User Configuration

To check if the user was created with the correct UID and parameters, query the user information using the `id` command:

```bash
id seto
```

Expected Output:

```ini
uid=0815(seto) gid=0815(seto) groups=0815(seto)
```

*(Note: If the `uid=0815` matches perfectly in the command output, the configuration is correct).*

## Command Parameter Explanation

- **`useradd -u`**: The `-u` (or `--uid`) option allows an administrator to manually specify the numerical user ID for the new account. If this option is omitted, the system will automatically assign the next available UID from the standard range defined in `/etc/login.defs`.
- **`id`**: A core utility that prints real and effective user and group IDs for a specified account name, making it the fastest tool to audit account provisioning results during the exam.

## Core Knowledge Points Covered

- **Custom Account Attributes Allocation:** Managing the standard user lifecycle by explicitly defining strict constraints like structural system identifiers (UIDs) rather than relying on standard auto-increment baselines.

- **Non-Interactive Credential Provisioning:** Utilizing standard input pipelines (`echo "password" | passwd --stdin`) to programmatically apply baseline security credentials efficiently under time limits.

- **Identity Verification Mechanics:** Leveraging diagnosis tools (`id`) to inspect localized shadow configurations and validate user metadata conformity before grading.

  


# 10. Find Files

- **Difficulty:** ★

- Find all files owned by user `jacques` and copy them to the `/root/findfiles` directory.

  

#### I. Create the Destination Directory

First, ensure that the target directory where the files will be copied exists:

```bash
mkdir /root/findfiles
```

#### II. Search and Copy the Files

Use the `find` command to locate all files across the filesystem owned by the user `jacques` and copy them to the target directory:

```bash
find / -user jacques -type f -exec cp -a {} /root/findfiles/ \;
```

*(Note: Adding `-type f` restricts the search specifically to **regular files**, avoiding errors or warnings from trying to copy directories or special system files into a single flat folder).*

#### III. Verify the Results

List the contents of the target folder to verify that the files have been copied successfully:

```bash
ls -l /root/findfiles
```

## Command and Parameter Explanation

- **`find /`**: Starts a recursive search from the root system directory (`/`), scanning the entire accessible filesystem.
- **`-user jacques`**: Filters the search results to match only files whose owner attribute corresponds to the username `jacques`.
- **`-type f`**: Restricts the search output to regular files. In RHCSA exams, when copying found entities to a common directory, this prevents the `cp` command from failing when encountering directories.
- **`-exec cp -a {} /root/findfiles/ \;`**: Executes the copy operation for each individual match:
  - `{}`: Spawns as a dynamic placeholder for each matched file path.
  - `-a` (archive): Preserves original file attributes, ownerships, permissions, and timestamps during the duplication process, which is highly recommended for administrative integrity.
  - `\;`: Acts as the mandatory terminator for the `-exec` processing block.

## Core Knowledge Points Covered

- **System-wide Metadata Searching:** Utilizing `find` effectively across the root namespace (`/`) while leveraging ownership metadata conditional keys (`-user`).

- **Object Type Filtering:** Using `-type f` to filter out non-file structures (like directories or block devices), minimizing command failures during bulk transfers.

- **Inline Administrative Operations Execution:** Mastering the execution pipeline interface (`-exec`) to directly bind utility operations (`cp`) to search outputs without scripting external loops.

- **Strict Preservation of Attributes:** Applying the archiving flags (`cp -a`) to maintain security context, modification times, and access permissions across file replications.

  

# 11. Find Strings

- **Difficulty:** ★

- Find all lines containing the string "`ng`" in the file `/usr/share/xml/iso-codes/iso_639_3.xml`. Place a copy of all these lines in their original order into the file `/root/list`.

- `/root/list` must not contain empty lines, and all lines must be exact copies of the original lines in `/usr/share/xml/iso-codes/iso_639_3.xml`.

  

#### I. Filter and Redirect Content Using grep

Use the `grep` command to search for lines containing the exact string `ng` within the source XML file, and redirect the output to create the target list file:

```bash
grep "ng" /usr/share/xml/iso-codes/iso_639_3.xml > /root/list
```

#### II. Verify the Content and Ensure No Empty Lines exist

To meet the criteria that the file must not contain any empty lines and that the text lines preserve their original structural sequence, inspect the output using `cat` or `vim`:

```bash
cat /root/list
```

*(Note: Standard `grep` output inherently matches line-by-line in the exact sequence found within the source file, and it does not append random empty lines, fully satisfying the exam requirements).*

## Command and Syntax Explanation

- **`grep "ng"`**: The Global Regular Expression Print (`grep`) utility scans the target file line-by-line and extracts any lines where the contiguous characters `ng` are present.
- **`>` (Standard Output Redirection)**: The single right-angle bracket operator intercepts the standard text output stream generated by `grep` and writes it directly into the specified file path (`/root/list`). If the file already exists, it will overwrite the previous content completely; if it does not exist, the shell provisions a new plain text file automatically.
- **Preserving Original Order**: Because `grep` processes files sequentially from byte 0 to the end of the file descriptor, the resulting text stream matches the original order naturally.

## Core Knowledge Points Covered

- **Text Processing and Pattern Matching via `grep`:** Leveraging basic text filtering tools to parse configurations or dataset fields matching specific literal criteria.

- **I/O Redirection Mechanics:** Mastering shell redirection utilities (`>`) to capture runtime application output strings and write them cleanly into administrative target nodes.

- **Sequential Stream Integrity:** Understanding how standard filters handle multi-line file processing sequentially, which ensures data preservation criteria (such as maintaining original entry sorting orders) are met.

- **Whitespace Auditing:** Inspecting post-operation artifacts (`cat`) to confirm string extraction accuracy and ensure no malformed white lines disrupt downstream evaluation scripts.

  

# 12.1 Create an Archive

- **Difficulty: ★ **

- Create a tar archive named `/root/backup.tar.bz2`, which should contain the contents of `/usr/local`. The tar archive must be compressed using `bzip2` format.

  

# 12.2 Create an Archive (Alternative Task Type)

- **Difficulty:** ★

- Create a tar archive named `/root/backup.tar.gz`, which should contain the contents of `/usr/local`. The tar archive must be compressed using `gzip` format.

  

#### Task 12.1: Create a `bzip2` Compressed Tar Archive

To create a tar archive compressed using the `bzip2` format, use the `-j` option:

```bash
tar -jcvf /root/backup.tar.bz2 /usr/local
```

#### Task 12.2: Create a `gzip` Compressed Tar Archive

To create a tar archive compressed using the `gzip` format, use the `-z` option:

```bash
tar -zcvf /root/backup.tar.gz /usr/local
```

#### When a tool is not pre-installed and the system returns `Command not found` 

If you encounter missing commands such as `bzip2` on your practice environment or the official Node 2 exam machine, you must install the corresponding packages via the local YUM/DNF repository configured in Task 2.

```bash
# Install gzip if the system reports it missing
yum install -y gzip

# Install bzip2 if the system reports it missing (the tar -j flag depends on the bzip2 package)
yum install -y bzip2
```


#### Verification

You can inspect and list the contents of the newly created archive without extracting it to ensure the files were packaged correctly:

```bash
# Verify bzip2 archive content
tar -tf /root/backup.tar.bz2

# Verify gzip archive content
tar -tf /root/backup.tar.gz
```

## Command Option Explanation

The `tar` utility maps compression parameters to specific operational flags:

- **`-c` (Create)**: Instructs the utility to initialize and create a brand-new archive file.
- **`-v` (Verbose)**: Progressively lists the files being processed in the terminal during the operation. *(Optional for exams, but helpful for visual confirmation).*
- **`-f` (File)**: Specifies the target filename of the archive. **Note:** This option must always be placed immediately before the archive file path.
- **`-j` (bzip2)**: Intercepts the archive stream and filters it through the `bzip2` compression algorithm, yielding high compression ratios (typically resulting in a `.tar.bz2` extension).
- **`-z` (gzip)**: Intercepts the archive stream and filters it through the faster `gzip` compression algorithm (typically resulting in a `.tar.gz` extension).
- **`-t` (List)**: Lists the structural table of contents of an archive file without extracting its payloads onto the active filesystem.

## Core Knowledge Points Covered

- **Filesystem Backups and Archiving via `tar`:** Mastering foundational system packaging mechanics to group directories recursively into portable single-file distribution formats.

- **Algorithmic Compression Filtering:** Differentiating between standard Unix compression formats (`gzip` via `-z` vs. `bzip2` via `-j`) to meet strict storage design specifications.

- **Strict Order of Parameter Execution:** Correctly positioning the `-f` flag directly adjacent to the output destination block to prevent the shell from parsing filenames as command options.

- **Non-destructive Archive Verification:** Utilizing safe payload queries (`tar -tf`) to validate packaged content streams and ensure data integrity prior to exam submission.

  

# 13. Create a Container Image

*(Questions 13 and 14 can be treated and done together as a single task)*

- **Difficulty:** ★★

- As user `wallah`, download `http://classroom/Containerfile`.

- Do not modify the content of this file. Build an image named `pdf`.

  

# 14. Configure a Container as a Service

- **Difficulty:** ★★★★★

- Configure a systemd container service for user `wallah`.

- The container name should be `ascii2pdf`.

- Use the newly created image `pdf`.

- The service should be named `container-ascii2pdf` and must start automatically at system reboot without manual intervention.

- Configure the service to automatically mount `/opt/file` to `/dir1` inside the container at startup, and mount `/opt/progress` to `/dir2` inside the container.

  

# 15. Add Passwordless sudo Configuration

- **Difficulty:** ★

- Allow members of the `sysmgrs` group to execute `sudo` without a password.

#### I. Create or Open a Dedicated sudoers Configuration File

Instead of directly editing the main `/etc/sudoers` file, the safest and best practice in RHEL9 is to drop a separate configuration file into the `/etc/sudoers.d/` directory.

Use the `visudo` command with the `-f` flag to create and safely edit a custom file (e.g., named `sysmgrs`):

```bash
visudo -f /etc/sudoers.d/sysmgrs
```

#### II. Add the Passwordless Privilege Rule

Once inside the editor interface, append the following configuration line:

代码段

```
%sysmgrs ALL=(ALL) NOPASSWD: ALL
```

Save and exit the editor (if using standard `vi/vim` bindings inside visudo: press `Esc`, then type `:wq` and press `Enter`).

#### III. Verify the Configuration

Switch to one of the users belonging to the `sysmgrs` group (for example, `harry`, whom we configured in Task 4) and verify that a privileged command can be run using `sudo` without prompting for a password:

```bash
# Switch to the user context
su - harry

# Test execution of an administrative command
sudo head -n 1 /etc/shadow
```

*(Note: If the command prints the first line of the shadow file immediately without asking for `harry`'s password, the configuration is working correctly).*

## Configuration Syntax Explanation

- **`%sysmgrs`**: The percent sign `%` prefix specifies that this rule applies to a **group** rather than a standalone user account. Any user who has `sysmgrs` as a primary or secondary group will inherit this privilege.
- **`ALL=` (First)**: Specifies the **hosts** from which this rule is valid. `ALL` means this privilege applies on any machine or terminal connection hosting this system.
- **`(ALL)`**: Specifies the **target user context** under which commands can be executed. `(ALL)` allows the group members to impersonate any user on the system (including `root`).
- **`NOPASSWD:`**: The crucial tag required by the exam question. It instructs the security engine to bypass the standard password verification stage when validating execution requests.
- **`ALL` (Last)**: Specifies the **allowed command paths**. `ALL` grants permission to execute any binary executable or administrative tool on the operating system.

## Core Knowledge Points Covered

- **Decentralized Sudo Policy Management:** Utilizing the drop-in `/etc/sudoers.d/` directory layout to manage explicit administrative access controls cleanly without altering default baseline vendor templates.
- **Sudoers Syntax Rules:** Formulating syntax expressions, including specifying group boundaries (`%`) and binding execution targets.
- **Elevated Privilege Security Overrides:** Implementing the `NOPASSWD:` execution flag within permission definitions to streamline administrative workflows or headless automation tasks safely.
- **Syntax Validation using `visudo`:** Leveraging `visudo` to perform automated sanity checks on syntax blocks during file closing to protect the system against terminal lockout errors.