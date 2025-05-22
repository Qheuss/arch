# Manual Arch Linux installation with AMD GPU and timezone:

## Preparation

- Temporarily disable Secure Boot

  - Enter your ASRock BIOS settings (F2 or Delete key at startup)
  - Navigate to the Security section
  - Set Secure Boot to "Disabled" or "Other OS"
  - Save and exit

- Boot from Arch Linux USB
  - Insert your Arch Linux USB
  - Boot from it (usually through a boot menu, often F12)

#### 1. Verify Boot Mode and Internet Connection

```bash
# Verify you're in UEFI mode
ls /sys/firmware/efi/efivars

# Verify internet connection
ping -c 3 archlinux.org
```

#### 2. Update the System Clock

```bash
timedatectl set-ntp true
```

#### 3. List all available drives to identify them:

```bash
# List all drives and partitions
lsblk
fdisk -l
```

##### Based on my setup:

- 1TB SSD for system and programs (e.g., /dev/nvme2n1)
- 2TB SSD for games (e.g., /dev/nvme0n1)
- 500GB SSD for backups (e.g., /dev/nvme3n1)
- 1TB SSD for Windows 11 (already installed, e.g., /dev/nvme1n1)

###

#### 4. Partition Your System SSD (1TB)

```bash
fdisk /dev/nvme2n1  # Replace with your actual 1TB system drive
```

- g (create new GPT partition table)
- n (new partition, #1, default start, +512M for size) - EFI partition
- t (change type, select 1, type "1" or "ef00" for "EFI System")
- n (new partition, #2, default start, +16G for size) - Swap partition
- t (change type, select partition 2, then type 19 for "Linux swap" - type code 8200)
- n (new partition, #3, default start, default end) - Root partition
- w (write changes and exit)

#### 5. Partition Your Games SSD (2TB)

```bash
fdisk /dev/nvme0n1  # Replace with your actual 2TB games drive
```

- g (create new GPT partition table)
- n (new partition, use defaults for all prompts to create one large partition)
- w (write changes and exit)

#### 6. Partition Your Backup SSD (500GB)

```bash
fdisk /dev/nvme3n1  # Replace with your actual 500GB backup drive
```

- g (create new GPT partition table)
- n (new partition, use defaults for all prompts to create one large partition)
- w (write changes and exit)

#### 7. Format the Partitions

```bash
# Format EFI partition
mkfs.fat -F32 /dev/nvme2n1p1

# Format swap
mkswap /dev/nvme2n1p2
swapon /dev/nvme2n1p2

# Format root partition
mkfs.ext4 /dev/nvme2n1p3

# Format games partition
mkfs.ext4 /dev/nvme0n1p1

# Format backup partition
mkfs.ext4 /dev/nvme3n1p1
```

#### 8. Mount the Partitions

```bash
# Mount root partition
mount /dev/nvme2n1p3 /mnt

# Create and mount EFI directory
mkdir -p /mnt/boot/efi
mount /dev/nvme2n1p1 /mnt/boot/efi

# Create and mount games directory
mkdir -p /mnt/mnt/games
mount /dev/nvme0n1p1 /mnt/mnt/games

# Create and mount backup directory
mkdir -p /mnt/mnt/backup
mount /dev/nvme3n1p1 /mnt/mnt/backup
```

#### 9. Install Base System

```bash
pacstrap /mnt base linux linux-firmware base-devel nano
```

#### 10. Generate Fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab

# Verify the fstab looks correct
cat /mnt/etc/fstab
```

#### 11. Chroot Into the System

```bash
arch-chroot /mnt
```

#### 12. Set Time Zone and Localization

```bash
# Set Brussels timezone
ln -sf /usr/share/zoneinfo/Europe/Brussels /etc/localtime
hwclock --systohc

# Set locale
nano /etc/locale.gen
# Uncomment en_US.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

#### 13. Network Configuration

```bash
# Set hostname to arch-qh
echo "arch-qh" > /etc/hostname

# Configure hosts file
nano /etc/hosts
```

Add these lines:

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch-qh.localdomain   arch-qh
```

#### 14. Set Root Password and Create User

```bash
# Set root password
passwd

# Create your user (replace 'yourusername' with your desired username)
useradd -m -G wheel,audio,video,storage -s /bin/bash yourusername
passwd yourusername

# Install sudo and configure wheel group
pacman -S sudo
EDITOR=nano visudo
```

##### Uncomment %wheel ALL=(ALL) ALL

#### 15. Install and Configure GRUB

```bash
# Install necessary packages
pacman -S grub efibootmgr os-prober sbctl ntfs-3g

# Enable OS detection for Windows
nano /etc/default/grub
```

## Enable OS prober to detect Windows

```bash
nano /etc/default/grub
```

##### Add or uncomment: GRUB_DISABLE_OS_PROBER=false

```bash
# Install GRUB to the EFI partition
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --modules="tpm"

# Generate the GRUB configuration
grub-mkconfig -o /boot/grub/grub.cfg
```

#### 16. Install Networking Tools

```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

#### 17. Install AMD Graphics Drivers

```bash
pacman -S xf86-video-amdgpu mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau
```

#### 18. Install Desktop Environments

```bash
# Install X11
pacman -S xorg-server xorg-xinit xorg-apps

# Install i3 for gaming
pacman -S i3-wm i3status i3lock dmenu picom

# Install Hyprland for everyday use
pacman -S polkit xdg-desktop-portal-wlr
pacman -S hyprland waybar wofi
```

#### 19. Install Sound System

```bash
# Install pipewire
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber

# Enable pipewire
systemctl --user enable --now pipewire.service
systemctl --user enable --now pipewire-pulse.service
systemctl --user enable --now wireplumber.service

# Install sound manager
pacman -S pavucontrol
```

#### 20. Install Common Applications

```bash
# Install programs
pacman -S firefox kitty thunar thunar-volman gvfs file-roller neofetch htop rofi

# Install fonts
pacman -S ttf-dejavu ttf-liberation noto-fonts noto-fonts-emoji ttf-fira-code
```

#### 21. Install Gaming Software

```bash
pacman -S steam lutris wine-staging gamemode lib32-gamemode
```

#### 22. Install additional tools

```bash
pacman -S git wget curl zip unzip p7zip
```

#### 23. Enable Multilib Repository

```bash
# Edit pacman.conf
nano /etc/pacman.conf
```

##### Uncomment these lines:

```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```

##### Then update package databases

```bash
pacman -Sy
```

#### 24. Backup System

```bash
# Install backup tools
pacman -S timeshift rsync
```

#### 25. Exit and Reboot

```bash
# Exit chroot
exit

# Unmount all partitions
umount -R /mnt

# Reboot
reboot
```

#### 26. Post-Installation Steps

Verify that GRUB detected Windows:

```bash
sudo os-prober
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

To launch into i3:

```bash
# Only if not already done
echo "exec i3" > ~/.xinitrc
```

```bash
# Then when you want to start it just type:
startx
```

To launch into Hyprland:

```bash
hyprland
```

Configure i3:

```bash
mkdir -p ~/.config/i3
cp /etc/i3/config ~/.config/i3/
nano ~/.config/i3/config
```

Configure Hyprland:

```bash
mkdir -p ~/.config/hypr
touch ~/.config/hypr/hyprland.conf
```

Set up Timeshift for backups:

```bash
sudo timeshift --create --comments "Fresh Install"
```

All steps have been verified for the archlinux-2025.05.01-x86_64.iso release.
