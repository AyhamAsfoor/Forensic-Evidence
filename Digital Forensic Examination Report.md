# Digital Forensic Examination Report

---

| | |
| :--- | :--- |
| **Agency Name** | Digital Forensics Unit |
| **Case Number** | CASE-C512 |
| **Date of Report** | 2025-11-24 |
| **Examiner** | Ayham Asfoor & Moawia Orabi |

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

*[Insert Screenshot 1: Autopsy view showing the two partitions, with the hidden 0x17 partition highlighted.]*

### 4.2. Evidence of Anti-Forensics: Data Wiping

Within the unallocated space of the public FAT32 partition (`/vol2`), remnants of a deleted file named `00000000` were discovered. A hexadecimal view of the file's content showed that it did not contain coherent data but rather random, meaningless characters. This pattern is a definitive indicator of a data sanitization or "shredding" tool. Such tools are used to overwrite a file's content multiple times before deletion to make its recovery impossible. This action constitutes a deliberate and active attempt to destroy evidence.

*[Insert Screenshot 2: Hexadecimal editor view of the shredded file, displaying random garbage data.]*

### 4.3. Recovered Evidence from Hidden Partition

A full analysis of the hidden NTFS partition (`/vol3`) led to the recovery of several files of significant evidentiary value. These files were stored in the root directory and were fully intact:

- `financial_logs.txt`: A text file containing detailed records of what appear to be illegal financial transactions, including dates, amounts, and recipient identifiers.
- `secret_plans.pdf`: A PDF document containing confidential blueprints and schematics.
- `wallet_backup.dat`: A data file consistent with a cryptocurrency wallet backup, suggesting the use of digital currencies for transactions.
- `Ransomware_Decryptor.exe`: An executable file identified by antivirus scanners as a malicious tool related to ransomware operations.

*[Insert Screenshot 3: Autopsy file browser view of the recovered files within the hidden NTFS partition.]*

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

---

**End of Report**
