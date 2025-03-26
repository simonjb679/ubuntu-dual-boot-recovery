# Ubuntu Dual-Boot Recovery & GRUB Repair

This project documents my hands-on experience restoring a broken Ubuntu dual-boot setup alongside Windows. It highlights the real-world process of using system-level Linux tools and bootloader repair techniques to recover and reconfigure a system without losing data.

---

## 🔧 Problem Summary
After successfully installing Ubuntu as a dual-boot with Windows, Ubuntu disappeared from my system upon rebooting into Windows. The system booted directly into Windows, and BIOS only listed Windows Boot Manager. Ubuntu was inaccessible, and the EFI partition appeared broken or missing.

---

## 🧰 Tools and Commands Used
- Ubuntu Live USB
- GRUB Bootloader
- chroot environment
- EFI System Partition (ESP)
- Bash shell (Linux terminal)
- `fsck`, `mount`, `umount`, `chroot`, `os-prober`, `update-grub`, `grub-install`, `nano`, `lsblk`, `parted`, `cat`, `mkdir`
- BIOS/UEFI configuration (ASUS TUF B650E motherboard)

---

## 🗂️ Full Step-by-Step Breakdown

### 1. BIOS Configuration
- Entered BIOS, switched to UEFI boot mode
- Disabled Secure Boot by switching to "Custom" and clearing secure boot keys
- Disabled Fast Boot
- Set USB device as the first boot option

### 2. Booted into Ubuntu Live USB
- Used "Try or Install Ubuntu" option
- Accessed terminal with `Ctrl + Alt + F2` or Terminal app

### 3. Identified Partitions
- Used `lsblk` and `sudo parted -l` to identify:
  - Ubuntu root partition: `/dev/nvme0n1p5`
  - EFI partition: `/dev/nvme0n1p1`

### 4. Mounted Ubuntu System and EFI Partition
```bash
sudo mount /dev/nvme0n1p5 /mnt
sudo mount /dev/nvme0n1p1 /mnt/boot/efi
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo mount --bind /run /mnt/run
```

### 5. Entered chroot Environment
```bash
sudo chroot /mnt
```

### 6. Attempted GRUB Install (Initial Failures)
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu
```
- Received error: `failed to make directory '/boot/efi/EFI/ubuntu'`
- Attempted to create directory:
```bash
mkdir -p /boot/efi/EFI/ubuntu
```
- Got `Input/output error` indicating EFI partition was mounted read-only

### 7. Diagnosed and Repaired EFI Partition
- Exited chroot:
```bash
exit
```
- Unmounted and ran `fsck`:
```bash
sudo umount /mnt/boot/efi
sudo fsck /dev/nvme0n1p1
```
- Selected to:
  - Copy original boot sector to backup
  - Remove dirty bit
  - Correct free cluster summary
  - Write all changes

### 8. Remounted as Read/Write and Continued
```bash
sudo mount -o rw /dev/nvme0n1p1 /mnt/boot/efi
sudo chroot /mnt
mkdir -p /boot/efi/EFI/ubuntu
```
✅ Success: EFI directory created without errors

### 9. Reinstalled GRUB and Updated Config
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu
update-grub
```
✅ GRUB installed successfully, Ubuntu detected

### 10. Enabled Windows Detection
- Initially, GRUB did not detect Windows even after reinstall
- Confirmed that `os-prober` was not running due to default config
- Edited GRUB default config to enable OS detection:
```bash
sudo nano /etc/default/grub
# Added or changed this line:
GRUB_DISABLE_OS_PROBER=false
```
- Saved changes and ran:
```bash
update-grub
```
- Still no detection until `os-prober` was manually run and output confirmed
- Installed os-prober:
- Installed os-prober:
```bash
apt update
apt install --no-install-recommends os-prober
```
- Ran `os-prober`:
```bash
os-prober
```
✅ Detected Windows Boot Manager at `/dev/nvme0n1p1`

- Regenerated GRUB menu:
```bash
update-grub
```
✅ GRUB menu now shows both Ubuntu and Windows

### 11. GRUB Configuration and Backup
- Backed up GRUB config:
```bash
sudo cp /boot/grub/grub.cfg /home/simon/grub.cfg.backup
```
- Adjusted timeout:
```bash
sudo nano /etc/default/grub
# Changed GRUB_TIMEOUT=10 to GRUB_TIMEOUT=15
sudo update-grub
```
- Left GRUB_DEFAULT=0 to manually choose OS

---

## 🧠 Troubleshooting Challenges and Fixes
- EFI partition kept mounting as read-only
- Used `fsck` and removed dirty bit to restore write access
- Couldn't create directory (`Input/output error`) until partition was clean
- Encountered "already mounted" errors when retrying steps—solved by unmounting and restarting clean
- Handled missing OS entries by enabling `GRUB_DISABLE_OS_PROBER=false`

---

## ✅ Final Outcome
- GRUB restored with Ubuntu and Windows options
- System fully operational with clean dual-boot behavior
- Learned EFI structure, GRUB internals, Linux recovery, and system-level repair

---

## 💡 Skills Demonstrated
- Ubuntu & Linux command line
- EFI boot structure
- Bootloader recovery (GRUB)
- chroot environment usage
- Disk mounting and file system repair
- os-prober and boot menu configuration
- Technical persistence and adaptability

---

## 📌 Reflection
> I came into this wanting to try Linux, but ended up learning how to recover and rebuild my own dual-boot system. This project gave me real hands-on experience with Linux internals and made me way more confident in the terminal than I ever expected.

---

## 🔗 Tags
`ubuntu` `linux` `grub` `dual-boot` `uefi` `bash` `system-recovery` `fsck` `bootloader` `linux-terminal`

