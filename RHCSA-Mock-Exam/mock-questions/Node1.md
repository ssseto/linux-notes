#### 1. Configure Network Settings

- **Difficulty:** ★

- Configure **node1** with the following network configuration:

  - **Hostname:** `node1.domain250.example.com`

  - **IP Address:** `172.25.250.100`

  - **Subnet Mask:** `255.255.255.0`

  - **Gateway:** `172.25.250.254`

  - **DNS Server:** `172.25.250.254`

    

#### 2. Configure Your System to Use Default Repositories

- **Difficulty:** ★

- Configure the default YUM repositories for **node1** using these locations:

  - `http://content/rhel9.0/x86_64/dvd/BaseOS`

  - `http://content/rhel9.0/x86_64/dvd/AppStream`

    

#### 3. Debug SELinux

- **Difficulty:** ★★★

- The Web server failed to run on a non-standard port **82**. Please troubleshoot and resolve the issue encountered by the web server as needed, so that it meets the following conditions:

  - The Web server on the system can serve all existing HTML files in `/var/www/html` (Note: Do not delete or otherwise modify the existing file contents).

  - The Web server provides this content on port **82**.

  - The Web server starts automatically at system boot.

  - Ensure the SELinux mechanism is running in **Enforcing** mode.

    

#### 4. Create User Accounts

- **Difficulty:** ★

- Create user and group accounts according to the following requirements:

  - A group named `sysmgrs`.

  - A user `natasha`, belonging to `sysmgrs` as a secondary group.

  - A user `harry`, also belonging to `sysmgrs` as a secondary group.

  - A user `sarah`, who has no access to an interactive shell on the system and is not a member of `sysmgrs`.

  - The password for `natasha`, `harry`, and `sarah` should all be `seto`.

    

#### 5. Configure a Cron Job

- **Difficulty:** ★

- Configure a cron job that executes `/usr/bin/echo hello` daily at **14:23** as user `harry`.

  

#### 6.1 Create a Collaborative Directory (Bonus Question 1)

*(One of the three bonus questions will appear randomly)*

- **Difficulty:** ★

- Create a collaborative directory `/home/managers` with the following characteristics:

  - The group ownership of `/home/managers` is `sysmgrs`.

  - The directory should be readable, writable, and accessible by members of `sysmgrs`, but no other users should have these permissions. (Of course, the `root` user has access to all files and directories on the system).

  - Files created in `/home/managers` should automatically inherit the group ownership of the `sysmgrs` group.

    

#### 6.2 Set Default Password Policy (Bonus Question 2)

*(One of the three bonus questions will appear randomly. Correct answer for this gets no points)*

- **Difficulty:** ★

- Set a password policy for newly created users such that the password expires by default after **25 days** when a user is created.

  

#### 6.3 Create a System Monitoring Script (Bonus Question 3)

*(One of the three bonus questions will appear randomly. Correct answer for this gets no points)*

- **Difficulty:** ★★
- Create a script named `systeminfo`.
- The script should be placed under `/usr/local/bin` and is used to obtain information about current system processes.

  - The output must be displayed in the following order: Process Owner, Process PID, Virtual Memory consumed by the process, Resident (Actual) Memory, and CPU Percentage.

  - Sort the output by CPU percentage, with the process consuming the most CPU displayed at the very end.



#### 6.4 Create a File Search Script (Bonus Question 4)

*(One of the three bonus questions will appear randomly. Correct answer for this gets no points)*

- **Difficulty:** ★★

- Create a script named `myresearch`.

- The script should be placed under `/usr/local/bin` and is used to:
  - Find all files under `user` (or `/usr`) that are smaller than **10M** and have **SGID (Set Group ID)** permissions, and place these files under `/root/myfiles`.
  
    

#### 7. Configure NTP

- **Difficulty:** ★

- Configure your system to be an NTP client of `materials.example.com`. (Note: `materials.example.com` is a DNS alias for `classroom.example.com`).

  

#### 8. Configure autofs

- **Difficulty:** ★★

- Configure `autofs` to automatically mount the home directories of remote users as described below:
  - `materials.example.com` exports `/rhome` via NFS to your system. This file system contains pre-configured home directories for the user `remoteuser1`.
  
  - The home directory for `remoteuser1` is `materials.example.com:/rhome/remoteuser1`.
  
  - The home directory for `remoteuser1` should be automatically mounted locally under `/rhome` as `/rhome/remoteuser1`.
  
  - The home directory must be writable by its user.
  
    

#### 9. Configure a User Account

- **Difficulty:** ★

- Configure a user `seto` with a User ID (UID) of `815`. The password for this user should be `seto`.

  

#### 10. Find Files

- **Difficulty:** ★

- Find all files owned by user `jacques` and copy them to the `/root/findfiles` directory.

  

#### 11. Find Strings

- **Difficulty:** ★

- Find all lines containing the string "`ng`" in the file `/usr/share/xml/iso-codes/iso_639_3.xml`. Place a copy of all these lines in their original order into the file `/root/list`.

- `/root/list` must not contain empty lines, and all lines must be exact copies of the original lines in `/usr/share/xml/iso-codes/iso_639_3.xml`.

  

#### 12.1 Create an Archive

- **Difficulty: ★ **

- Create a tar archive named `/root/backup.tar.bz2`, which should contain the contents of `/usr/local`. The tar archive must be compressed using `bzip2`.

  

#### 12.2 Create an Archive (Alternative Task Type)

- **Difficulty:** ★

- Create a tar archive named `/root/backup.tar.gz`, which should contain the contents of `/usr/local`. The tar archive must be compressed using `gzip` format.

  

#### 13. Create a Container Image

*(Questions 13 and 14 can be treated and done together as a single task)*

- **Difficulty:** ★★

- As user `wallah`, download `http://classroom/Containerfile`.

- Do not modify the content of this file. Build an image named `pdf`.

  

#### 14. Configure a Container as a Service

- **Difficulty:** ★★★★★

- Configure a systemd container service for user `wallah`.

- The container name should be `ascii2pdf`.

- Use the newly created image `pdf`.

- The service should be named `container-ascii2pdf` and must start automatically at system reboot without manual intervention.

- Configure the service to automatically mount `/opt/file` to `/dir1` inside the container at startup, and mount `/opt/progress` to `/dir2` inside the container.

  

#### 15. Add Passwordless sudo Configuration

- **Difficulty:** ★

- Allow members of the `sysmgrs` group to execute `sudo` without a password.