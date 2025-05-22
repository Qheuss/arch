# Manual Arch Linux installation with AMD GPU and timezone:
1. Boot from Arch Linux USB
Boot from your Arch Linux USB drive and verify internet connection:
```bash
ping -c 3 archlinux.org
```
3. Update the System Clock
```bash
timedatectl set-ntp true
```
4. Identify and Prepare Your Drives
List all available drives to identify them:
```bash
lsblk
fdisk -l
```
Based on my setup:

1TB SSD for system and programs (e.g., /dev/sda)
2TB SSD for games (e.g., /dev/sdb)
500GB SSD for backups (e.g., /dev/sdc)
1TB SSD for Windows 11 (already installed, e.g., /dev/sdd)

4. Partition Your System SSD (1TB)
```bash
fdisk /dev/sda
```
Create the following partitions:

512MB EFI partition (type: EFI System)
16GB swap partition (type: Linux swap)
Remainder for root partition (type: Linux filesystem)

5. Partition Your Games SSD (2TB)
```bash
fdisk /dev/sdb
```
Create a single large partition for games.
6. Partition Your Backup SSD (500GB)
```bash
fdisk /dev/sdc
```
Create a single partition for backups.
7. Format the Partitions
```bash
# Format EFI partition
mkfs.fat -F32 /dev/sda1
```

## Format swap
```bash
mkswap /dev/sda2
swapon /dev/sda2
```

## Format root partition
```bash
mkfs.ext4 /dev/sda3
```

## Format games partition
```bash
mkfs.ext4 /dev/sdb1
```

## Format backup partition
```bash
mkfs.ext4 /dev/sdc1
```
8. Mount the Partitions
```bash
# Mount root partition
mount /dev/sda3 /mnt
```

## Create and mount EFI directory
```bash
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

## Create and mount games directory
```bash
mkdir -p /mnt/mnt/games
mount /dev/sdb1 /mnt/mnt/games
```

## Create backup directory (will mount later)
```bash
mkdir -p /mnt/mnt/backup
```
9. Install Base System
```bash
bashpacstrap /mnt base linux linux-firmware base-devel nano
```
11. Generate Fstab
bashgenfstab -U /mnt >> /mnt/etc/fstab
12. Chroot Into the System
basharch-chroot /mnt
13. Set Time Zone and Localization
bash# Set time zone for Brussels
ln -sf /usr/share/zoneinfo/Europe/Brussels /etc/localtime
hwclock --systohc

## Set locale
nano /etc/locale.gen
### Uncomment en_US.UTF-8 UTF-8 (or your preferred locale)
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
13. Network Configuration
bash# Set hostname to arch-qh as requested
echo "arch-qh" > /etc/hostname

## Configure hosts file
nano /etc/hosts
Add these lines:
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch-qh.localdomain   arch-qh
14. Set Root Password and Create User
bash# Set root password
passwd

## Create your user
useradd -m -G wheel,audio,video,storage -s /bin/bash yourusername
passwd yourusername

## Install sudo
pacman -S sudo
EDITOR=nano visudo
### Uncomment %wheel ALL=(ALL) ALL
15. Install and Configure Bootloader
bash# Install GRUB and required packages
pacman -S grub efibootmgr os-prober

## Enable OS prober to detect Windows
nano /etc/default/grub
### Add or uncomment: GRUB_DISABLE_OS_PROBER=false

## Install GRUB
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

## Generate GRUB config
grub-mkconfig -o /boot/grub/grub.cfg
16. Install Networking Tools
bashpacman -S networkmanager
systemctl enable NetworkManager
17. Install AMD Graphics Drivers
Since you have an AMD GPU:
bashpacman -S xf86-video-amdgpu mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon
18. Install X11 and Basic Desktop Environment
bashpacman -S xorg-server xorg-xinit xorg-apps
19. Install i3 for Gaming
bashpacman -S i3-wm i3status i3lock dmenu
20. Install Hyprland for Everyday Use
bashpacman -S hyprland waybar wofi
21. Install Display Manager
bashpacman -S sddm
systemctl enable sddm
Configure SDDM to show both i3 and Hyprland:
bashmkdir -p /usr/share/wayland-sessions/
mkdir -p /usr/share/xsessions/
22. Install Sound System
bashpacman -S pulseaudio pulseaudio-alsa pavucontrol
23. Install Common Applications
bashpacman -S firefox alacritty thunar neofetch htop
24. Install Gaming Software
bashpacman -S steam lutris wine-staging gamemode
25. Configure Backup System
bash# Mount backup partition in fstab
echo "UUID=$(blkid -s UUID -o value /dev/sdc1) /mnt/backup ext4 defaults 0 2" >> /etc/fstab

## Install backup tool
pacman -S timeshift rsync
26. Final Configuration
bash# Install fonts
pacman -S ttf-dejavu ttf-liberation noto-fonts

## Install additional tools
pacman -S git wget curl
27. Exit and Reboot
bashexit
umount -R /mnt
reboot
28. Post-Installation Steps
After rebooting and logging in:

Configure i3:

bashmkdir -p ~/.config/i3
cp /etc/i3/config ~/.config/i3/
nano ~/.config/i3/config

Configure Hyprland:

bashmkdir -p ~/.config/hypr
touch ~/.config/hypr/hyprland.conf

Set up session selection in SDDM:

bashsudo nano /usr/share/wayland-sessions/hyprland.desktop
Add:
[Desktop Entry]
Name=Hyprland
Comment=Hyprland Wayland Compositor
Exec=Hyprland
Type=Application

Configure your games partition:

bashmkdir -p ~/games
sudo mount /dev/sdb1 ~/games

Set up Timeshift for backups:

bashsudo timeshift --create --comments "Fresh Install"
With these adjustments, your installation will use the correct timezone (Brussels), set your computer name to "arch-qh", and install the appropriate drivers for your AMD GPU. This setup provides you with an optimized dual-environment system with i3 for gaming and Hyprland for everyday use, all on dedicated storage drives.
