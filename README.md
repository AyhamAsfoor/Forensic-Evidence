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


# Digital Forensic Examination Report

---

| | |
| :--- | :--- |
| **Agency Name** | Digital Forensics Unit |
| **Case Number** | CASE-C512 |
| **Date of Report** | 2025-11-24 |
| **Examiner** | Ayham Asfoor |

---

## 1. Executive Summary

This report details the forensic examination of a USB drive (Evidence ID: EV-E01) seized in connection with an investigation into illegal financial operations. The examination was conducted to identify and recover potential evidence while assessing whether anti-forensic techniques were employed to obstruct the investigation.

The analysis successfully uncovered a hidden partition on the USB drive, which was not visible to standard operating systems. Within this hidden partition, several files of evidentiary value were recovered, including illegal transaction records, confidential blueprints, a cryptocurrency wallet file, and a malicious executable. Furthermore, the examination of the public-facing partition revealed compelling evidence of data destruction. A file was found to have been subjected to a shredding process, where its contents were overwritten with random data before deletion, a clear and deliberate attempt to destroy evidence.

In summary, the subject utilized sophisticated anti-forensic measures, including data concealment within a hidden partition and the willful destruction of files. The evidence recovered strongly supports the allegation of involvement in illegal financial activities and demonstrates a conscious effort to hide and destroy incriminating data.


## 2. Evidence Identification

- **Evidence ID:** EV-E01
- **Description:** A black USB drive of approximately 8GB capacity, seized from the suspect. The drive was received in a sealed anti-static bag.
- **Chain of Custody:** The evidence was logged under case number CASE-C512 and handled by Investigator Ayham Asfoor before being submitted to the Digital Forensics Unit for examination.

## 3. Forensic Acquisition

The forensic acquisition was conducted in a controlled laboratory environment to ensure the integrity and admissibility of the evidence. The primary objective was to create a bit-for-bit identical copy (forensic image) of the original evidence (EV-E01) without altering the source media in any way.

### 3.1. Methodology and Tools

- **Forensic Workstation:** A dedicated forensic workstation running Kali Linux (6.12.38+kali-amd64) was used for the acquisition.
- **Acquisition Software:** Guymager 0.8.13-2+b1 was utilized to perform the imaging process.
- **Write Blocker:** A software-based write-blocker (`blockdev --setro`) was engaged to prevent any write operations to the source evidence, thereby preserving its original state.
- **Forensic Media:** A sterilized hard drive was used as the destination for the forensic image.

### 3.2. Imaging and Hashing

A complete physical image of the suspect USB drive (EV-E01) was acquired in the Expert Witness Format (EWF-E01). During the acquisition, cryptographic hashes of the source media were calculated. After the image was created, it was automatically verified by recalculating its hashes and comparing them against the source hashes to ensure a true and accurate copy.

The results, as recorded by Guymager, are as follows:

| Hash Algorithm | Source Hash (Calculated from EV-E01) | Verified Image Hash (Calculated from USB_CASE.Exx) | Status |
| :--- | :--- | :--- | :--- |
| **MD5** | `58325c847efcc27937a168f2ed880ed8` | `58325c847efcc27937a168f2ed880ed8` | ✅ Verified |
| **SHA-1** | `b363a6e3694a2d6567bb245a28fec3acd1dbc261` | `b363a6e3694a2d6567bb245a28fec3acd1dbc261` | ✅ Verified |
| **SHA-256** | `ff759152402677e7583e0787fcd5f09b40cf3fe83590ab156d4f28f84af369bc` | `ff759152402677e7583e0787fcd5f09b40cf3fe83590ab156d4f28f84af369bc` | ✅ Verified |

- **Acquisition Start Time:** 2025-11-24 21:27:32
- **Acquisition End Time:** 2025-11-24 21:35:58
- **Result:** The forensic image was successfully created and verified, confirming that the acquired image is an exact duplicate of the source evidence.

## 4. Examination & Analysis

The verified forensic image (`USB_CASE.Exx`) was mounted in a read-only state and analyzed using the Autopsy Digital Forensics Platform (version 4.20.0). The examination focused on identifying the partition structure, recovering deleted files, and searching for hidden data.

### 4.1. Partition and File System Analysis

The analysis immediately revealed an unusual partition scheme designed to conceal data. The 8GB drive was partitioned as follows:

- **Partition 1 (`/vol2`):** A 4GB primary partition formatted with the FAT32 file system. This partition was visible to the operating system and contained a series of decoy files, including `Windows_Activator.iso`, `Avengers_Endgame.mp4`, and personal photographs. The purpose of this partition was to give the drive an appearance of normal, everyday use.

- **Partition 2 (`/vol3`):** A 3.5GB primary partition whose type was set to `0x17` (Hidden NTFS). This partition is not automatically mounted or displayed by Windows or other standard operating systems, effectively hiding its existence from a casual user. It was formatted with the NTFS file system.


### 4.2. Evidence of Anti-Forensics: Data Wiping

Within the unallocated space of the public FAT32 partition (`/vol2`), remnants of a deleted file named `00000000` were discovered. A hexadecimal view of the file's content showed that it did not contain coherent data but rather random, meaningless characters. This pattern is a definitive indicator of a data sanitization or "shredding" tool. Such tools are used to overwrite a file's content multiple times before deletion to make its recovery impossible. This action constitutes a deliberate and active attempt to destroy evidence.


### 4.3. Recovered Evidence from Hidden Partition

A full analysis of the hidden NTFS partition (`/vol3`) led to the recovery of several files of significant evidentiary value. These files were stored in the root directory and were fully intact:

- `financial_logs.txt`: A text file containing detailed records of what appear to be illegal financial transactions, including dates, amounts, and recipient identifiers.
- `secret_plans.pdf`: A PDF document containing confidential blueprints and schematics.
- `wallet_backup.dat`: A data file consistent with a cryptocurrency wallet backup, suggesting the use of digital currencies for transactions.
- `Ransomware_Decryptor.exe`: An executable file identified by antivirus scanners as a malicious tool related to ransomware operations.


## 5. Conclusion & Expert Opinion

Based on the totality of the forensic evidence, it is my expert opinion that the suspect took deliberate and calculated steps to conceal and destroy digital evidence. The use of a hidden partition to store incriminating files demonstrates a clear intent to hide this data from discovery. The files recovered from this partition, including financial logs and malicious software, are highly indicative of involvement in illegal activities.

Furthermore, the evidence of file shredding on the public partition is irrefutable proof of spoliation—the intentional destruction of evidence. The suspect attempted to permanently erase specific data, believing it would be unrecoverable. This act, combined with the use of a hidden partition, shows a sophisticated understanding of anti-forensic techniques and a guilty mindset.

The suspect’s actions were not accidental but were part of a conscious and methodical effort to obstruct a forensic examination. The combination of data hiding and data destruction points to a strong likelihood that the suspect was attempting to conceal their involvement in the illegal financial operations under investigation.

## 6. Appendix

### 6.1. Technical Specifications

- **Source Evidence (EV-E01):**
  - **Device:** USB Flash Drive
  - **Reported Size:** 8.1 GB (8,053,063,680 bytes)
  - **Acquired Partitions:** 2

- **Forensic Image (USB_CASE.Exx):**
  - **Format:** Expert Witness Format (EWF)
  - **Compression:** None
  - **Segment Size:** 2 GB (default)

- **Software Used:**
  - **Acquisition:** Guymager 0.8.13-2+b1
  - **Analysis:** Autopsy 4.20.0
  - **Operating System:** Kali Linux 6.12.38+kali-amd64
