# Ubuntu Dual-Boot Recovery & GRUB Repair

This project documents my hands-on experience restoring a broken Ubuntu dual-boot setup alongside Windows. It highlights the real-world process of using system-level Linux tools and bootloader repair techniques to recover and reconfigure a system without losing data.

---

## 🔧 Problem Summary

After successfully installing Ubuntu as a dual-boot with Windows, Ubuntu disappeared from my system upon rebooting into Windows. The system booted directly into Windows, and BIOS only listed Windows Boot Manager. Ubuntu was inaccessible, and the EFI partition appeared broken or missing.

---

## 🧰 Tools and Commands Used

- **Ubuntu Live USB**
- **GRUB Bootloader**
- **chroot** environment
- **EFI System Partition (ESP)**
- `fsck`, `mount`, `os-prober`, `update-grub`, `grub-install`, `nano`
- BIOS/UEFI configuration (ASUS TUF B650E motherboard)

---

## 🗂️ Step-by-Step Tasks Completed

1. **BIOS Configuration**
   - Disabled Secure Boot
   - Enabled UEFI boot
   - Set USB as first boot device

2. **Live USB & Terminal Access**
   - Booted into Ubuntu Live USB
   - Opened terminal using `Ctrl + Alt + F2`

3. **Identified and Mounted Ubuntu Partition**
   ```bash
   sudo mount /dev/nvme0n1p5 /mnt
   sudo mount --bind /dev /mnt/dev
   sudo mount --bind /proc /mnt/proc
   sudo mount --bind /sys /mnt/sys
   sudo chroot /mnt

