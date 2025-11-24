# Digital Forensics Lab: The Hidden Partition & Data Destruction Case

This guide provides a step-by-step walkthrough for instructors and students to create the "Suspicious USB" evidence used in this digital forensics training scenario. The objective is to simulate a suspect's attempts to hide data and destroy evidence on a USB drive.

---

## Prerequisites

- **Operating System:** A Kali Linux environment (virtual machine or live boot).
- **Hardware:** One USB drive of at least 8GB capacity. This will become the evidence media.

**Disclaimer:** The following commands will permanently destroy all data on the target USB drive. Ensure you have selected the correct device (`/dev/sdb` in this example) before proceeding. The device name may vary on your system.

---

## Phase 1: Media Sterilization (Forensic Wiping)

**Goal:** To ensure the USB drive is forensically clean. This process overwrites every sector on the device with random data, destroying any pre-existing file systems and partition tables. This guarantees that any data found later was planted as part of the scenario.

**Command:**

```bash
sudo dcfldd if=/dev/urandom of=/dev/sdb status=progress
```

> **Instructor Note:** We use `dcfldd`, a forensics-focused version of the `dd` utility, for this task. The input file `/dev/urandom` is a special file in Linux that serves as a pseudo-random number generator. Using it as the input source fills the drive with cryptographically strong random data, creating high entropy and making it impossible to recover any previous data.

## Phase 2: Partitioning Strategy (Anti-Forensics)

**Goal:** To create a partition scheme that mimics a suspect's attempt to hide data. We will create a visible "decoy" partition and a hidden partition that is not normally visible to operating systems like Windows.

**Tool:** `fdisk`

**Command:**

```bash
sudo fdisk /dev/sdb
```

Once inside the `fdisk` utility, execute the following sequence of commands:

1.  **Create the Decoy Partition:**
    - `n` (for a new partition)
    - `p` (for primary)
    - `1` (for partition number 1)
    - `[Enter]` (for the default first sector)
    - `+4G` (to set the size to 4 Gigabytes)

2.  **Set the Decoy Partition Type:**
    - `t` (to change a partition's system id)
    - `c` (to select type `W95 FAT32 (LBA)`)

3.  **Create the Hidden Partition:**
    - `n` (for a new partition)
    - `p` (for primary)
    - `2` (for partition number 2)
    - `[Enter]` (for the default first sector)
    - `[Enter]` (to use the remaining available space)

4.  **Set the Hidden Partition Type:**
    - `t` (to change a partition's system id)
    - `2` (to select partition 2)
    - `17` (to select type `Hidden HPFS/NTFS`)

5.  **Write Changes and Exit:**
    - `w` (to write the new partition table to the disk and exit)

> **Instructor Note:** The key anti-forensics technique here is setting the second partition's type to `0x17`. Standard operating systems, particularly Windows, will not assign a drive letter to or attempt to mount partitions with this type ID. This effectively hides the partition and its contents from a non-technical user or a cursory examination.

## Phase 3: File System Creation

**Goal:** To format the newly created partitions with the appropriate file systems. The decoy partition will use FAT32, a common file system for USB drives, while the hidden partition will use NTFS.

**1. Format the Decoy Partition (FAT32):**

```bash
sudo mkfs.fat -F 32 -n "USB_DISK" /dev/sdb1
```

**2. Format the Hidden Partition (NTFS):**

```bash
sudo mkfs.ntfs -f -L "SYSTEM_DRV" /dev/sdb2
```

> **Instructor Note:** We are labeling the partitions (`USB_DISK` and `SYSTEM_DRV`) to make them identifiable. The `-f` flag in the `mkfs.ntfs` command performs a "fast" format, which is sufficient for this scenario. Using different file systems (FAT32 vs. NTFS) can also be a point of interest during the analysis phase, as NTFS supports features like alternate data streams that FAT32 does not.

## Phase 4: Planting Evidence (The Hidden Partition)

**Goal:** To populate the hidden NTFS partition with incriminating files that the suspect wishes to conceal.

**1. Mount the Hidden Partition:**

First, we need to create a directory to mount the partition and then mount it.

```bash
sudo mkdir /mnt/hidden
sudo mount /dev/sdb2 /mnt/hidden
```

**2. Create Evidentiary Files:**

Now, we will create a set of files designed to simulate illegal activities.

```bash
# Create a text file with fake financial logs
sudo bash -c 'echo "Transaction ID: 1A4B, Amount: $50,000, Recipient: 4X8Z" > /mnt/hidden/financial_logs.txt'

# Create a fake confidential document
sudo bash -c 'echo "TOP SECRET BLUEPRINT FOR PROJECT OMEGA" > /mnt/hidden/secret_plans.pdf'

# Simulate an encrypted crypto wallet file with random data
sudo dd if=/dev/urandom of=/mnt/hidden/wallet_backup.dat bs=1M count=10

# Simulate a piece of malware by copying a harmless system binary and renaming it
sudo cp /bin/ls /mnt/hidden/Ransomware_Decryptor.exe

# Create a compressed archive of system logs to simulate data theft
sudo tar -czf /mnt/hidden/stolen_server_logs.tar.gz /var/log/syslog
```

**3. Unmount the Partition:**

Finally, unmount the partition to ensure the data is written and the partition is hidden again.

```bash
sudo umount /mnt/hidden
```

> **Instructor Note:** The variety of files planted here provides multiple avenues for investigation. The `wallet_backup.dat` file, filled with random data, has high entropy, which is characteristic of encrypted containers or wallets. The renamed `ls` binary will have a mismatched file signature (an ELF executable with a `.exe` extension), a classic red flag in malware analysis. These details make the scenario more realistic and challenging.

## Phase 5: Camouflage & Destruction (The Public Partition)

**Goal:** To make the public partition look like it's in normal use and to simulate the deliberate destruction of a specific, sensitive file.

**1. Mount the Public Partition:**

```bash
sudo mkdir /mnt/normal
sudo mount /dev/sdb1 /mnt/normal
```

**2. Create Decoy Files:**

We will create some large, empty files and a folder to act as camouflage, making the drive appear innocuous.

```bash
# Create large placeholder files
sudo fallocate -l 1G /mnt/normal/Avengers_Endgame.mp4
sudo fallocate -l 500M /mnt/normal/Windows_Activator.iso

# Create a typical photos folder
sudo mkdir /mnt/normal/DCIM
```

**3. The Shredding Event:**

This is the critical step where the suspect actively destroys evidence.

```bash
# First, create a sensitive file to be destroyed
sudo bash -c 'echo "EmployeeID,Name,Salary\n1,John Doe,$95000\n2,Jane Smith,$105000" > /mnt/normal/Employees_DB_Dump.csv'

# Now, execute the shred command to destroy it
sudo shred -u -z -v -n 2 /mnt/normal/Employees_DB_Dump.csv
```

**4. Unmount the Partition:**

```bash
sudo umount /mnt/normal
```

> **Instructor Note:** The `shred` command is a powerful anti-forensics tool. Let's break down the flags used:
> - `-n 2`: Overwrites the file's location on the disk with random data 2 times.
> - `-z`: Performs a final overwrite with zeros to hide the fact that shredding took place.
> - `-v`: Shows verbose progress, which is useful for the lab but something a real suspect might omit.
> - `-u`: Deletes (truncates and removes) the file after overwriting. 
> This process makes traditional file recovery impossible and is a clear indicator of intent to destroy evidence. Forensic analysts will find the remnants of the shredding process in unallocated space, which is a finding in itself.

---

**Lab Setup Complete:** The suspicious USB drive is now fully prepared. It can be provided to students as the primary piece of evidence for a forensic acquisition and analysis exercise.

