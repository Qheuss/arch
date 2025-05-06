# Secure Boot with Windows 11 and Arch Linux Dual-Boot Guide

This guide provides instructions for setting up and maintaining Secure Boot on a system dual-booting Windows 11 and Arch Linux.

## Initial Setup

### 1. Preparation

```bash
# Install sbctl if not already installed
sudo pacman -S sbctl

# Check current Secure Boot status
sudo sbctl status
```

### 2. Enter Setup Mode

1. Reboot into UEFI setup (usually by pressing F2, Delete, or F12 during boot)
2. Find Secure Boot settings (typically under Security or Boot tabs)
3. Set to "Setup Mode" or "Clear Keys" (this varies by motherboard)
4. Save and exit

### 3. Create Keys

```bash
# Create new set of keys
sudo sbctl create-keys
```

### 4. Enroll Keys with Microsoft Certificates

```bash
# Enroll keys with Microsoft certificates (necessary for dual-boot)
sudo sbctl enroll-keys --microsoft
```

### 5. Sign Boot Files

```bash
# Sign Linux kernel and bootloader
sudo sbctl sign -s /boot/vmlinuz-linux
sudo sbctl sign -s /boot/efi/EFI/GRUB/grubx64.efi

# Sign fallback bootloaders
sudo sbctl sign -s /boot/efi/EFI/BOOT/BOOTX64.EFI
sudo sbctl sign -s /boot/efi/EFI/BOOT/grubx64.efi
sudo sbctl sign -s /boot/efi/EFI/BOOT/mmx64.efi

# Mount Windows EFI partition if on separate drive/partition
# Adjust paths to match your system
sudo mkdir -p /mnt/windows_efi
sudo mount /dev/nvme1n1p1 /mnt/windows_efi

# Sign Windows boot files
sudo sbctl sign -s /mnt/windows_efi/EFI/Microsoft/Boot/bootmgfw.efi
sudo sbctl sign -s /mnt/windows_efi/EFI/Microsoft/Boot/bootmgr.efi
```

### 6. Verify Files

```bash
# Check that all files are signed
sudo sbctl verify
```

### 7. Enable Secure Boot

1. Reboot back into UEFI setup
2. Enable Secure Boot
3. Save and exit

### 8. Verify Secure Boot Is Active

```bash
# Check that Secure Boot is enabled
sudo sbctl status
```

You should see `Secure Boot: ✓ Enabled` in the output.

## Maintenance Tasks

### After Kernel Updates

```bash
# The pacman hook should automatically sign new kernels, but verify to be sure
sudo sbctl verify
```

If the verification shows any unsigned files, sign them:

```bash
sudo sbctl sign -s /path/to/unsigned/file
```

### After GRUB Updates

```bash
# The pacman hook should automatically sign new GRUB files, but verify to be sure
sudo sbctl verify
```

### After Major Windows Updates

If Windows has a major update that replaces bootloader files:

```bash
# Mount Windows EFI partition if needed
sudo mkdir -p /mnt/windows_efi
sudo mount /dev/nvme1n1p1 /mnt/windows_efi

# Sign Windows boot files again
sudo sbctl sign -s /mnt/windows_efi/EFI/Microsoft/Boot/bootmgfw.efi
sudo sbctl sign -s /mnt/windows_efi/EFI/Microsoft/Boot/bootmgr.efi

# Verify all files are signed
sudo sbctl verify
```

### Key Rotation (Recommended Yearly)

```bash
# Rotate keys
sudo sbctl rotate-keys

# Sign all files with new keys
sudo sbctl sign-all
```

## Troubleshooting

### If Secure Boot Prevents Booting

1. Disable Secure Boot in UEFI setup
2. Boot into Arch Linux
3. Check for unsigned files: `sudo sbctl verify`
4. Sign any unsigned files: `sudo sbctl sign -s /path/to/file`
5. Re-enable Secure Boot

### If Windows Won't Boot with Secure Boot

1. Check that Windows bootloader files are signed:
   ```bash
   sudo sbctl verify | grep Microsoft
   ```
2. If not signed or missing:
   ```bash
   # Mount Windows EFI partition
   sudo mount /dev/nvme1n1p1 /mnt/windows_efi
   # Sign files
   sudo sbctl sign -s /mnt/windows_efi/EFI/Microsoft/Boot/bootmgfw.efi
   sudo sbctl sign -s /mnt/windows_efi/EFI/Microsoft/Boot/bootmgr.efi
   ```

## System-Specific Notes

- ESP (EFI System Partition) is mounted at: `/boot/efi`
- Windows EFI partition is on: `/dev/nvme1n1p1`

Always adjust paths to match your specific system configuration!
