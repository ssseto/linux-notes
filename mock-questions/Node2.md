#### 16. Reset Root Password

- **Difficulty:** ★

- Set the root password of **node2** to `seto`. You need to gain system access in order to perform operations for subsequent questions.

  

#### 17. Configure Your System to Use Default Repositories

- **Difficulty:** ★

- YUM repositories are already available from:

  - `http://content/rhel9.0/x86_64/dvd/BaseOS`
  - `http://content/rhel9.0/x86_64/dvd/AppStream`

- Configure your system to use these locations as the default repositories.

  

#### 18. Resize a Logical Volume

- **Difficulty:** ★★

- Resize the logical volume `vo` and its filesystem to **230 MiB**. Ensure that the filesystem contents remain intact.

  *Note: Partition sizes rarely match the requested size exactly, so a size ranging from 213 MiB to 243 MiB is acceptable.*

  

#### 19. Add Swap Partition

- **Difficulty:** ★★

- Add an additional swap partition of **512 MiB** to your system.

- The swap partition should be automatically mounted at system boot.

- Do not delete or in any way alter any existing swap partitions on the system.

  

#### 20. Create a Logical Volume

- **Difficulty:** ★★★★★

- Create a new logical volume according to the following requirements:

  - The logical volume should be named `qa`, belong to the `qagroup` volume group, and have a size of **60 extents**.

  - The logical volume extent size in the `qagroup` volume group should be **16 MiB**.

  - Format the new logical volume with a `vfat` filesystem. This logical volume should be automatically mounted under `/mnt/qa` at system boot.

    

#### 21. Configure System Tuning

- **Difficulty:** ★

- Select the recommended `tuned` profile for your system and set it as the default configuration.