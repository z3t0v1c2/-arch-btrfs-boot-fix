# 🛠️ Arch Linux Btrfs Boot Repair Guide

A universal guide to fixing boot failures on Arch Linux after performing a Btrfs snapshot rollback. This method addresses "emergency shell" or "GRUB rescue" errors caused by out-of-sync **initramfs** and **bootloader** configurations.


---

## 🔍 The Problem
After restoring a `@` (root) subvolume from a snapshot, the system often fails to boot because the kernel images and bootloader config do not match the current state of the subvolume.

## 🚀 Recovery Steps

### 1. Identify Partitions
Boot from your **Arch Linux Live USB** and identify your drive names:
```bash
lsblk
Note: Identify your Btrfs partition (e.g., /dev/nvme0n1p2) and your EFI partition (e.g., /dev/nvme0n1p1).

2. Mount and Setup
Mount the top-level Btrfs volume (ID 5) to manage your subvolumes:

Bash
# Mount the parent Btrfs partition
mount -o subvolid=5 /dev/sdXy /mnt

# Replace current @ with your snapshot if not already done
mv /mnt/@ /mnt/@_broken_backup
btrfs subvolume snapshot /mnt/@snapshots/your-snapshot-name /mnt/@
3. Chroot & Repair
Mount the restored subvolume and enter the system environment:

Bash
# Mount the restored root
mount -o subvol=@ /dev/sdXy /mnt

# Mount the EFI partition (May be /boot or /efi depending on your setup)
mount /dev/sdXz /mnt/boot

# Enter the system
arch-chroot /mnt
4. Synchronize & Finalize
Inside the chroot, regenerate the initramfs and update the bootloader:

Bash
# Regenerate initramfs for all kernels
mkinitcpio -P

# Update GRUB configuration
grub-mkconfig -o /boot/grub/grub.cfg

# Exit and reboot
exit
umount -R /mnt
reboot
💡 Important Tips
Microcode: Ensure intel-ucode or amd-ucode is installed before running grub-mkconfig.

Mount Points: Double-check if your EFI partition is mounted at /boot or /efi. Check your /etc/fstab if unsure.

NVMe Drives: If you are using an NVMe SSD (like on an gaming laptop), your partitions will look like /dev/nvme0n1pX.

Disclaimer: This is a general guide. Always verify your partition names before running commands.
