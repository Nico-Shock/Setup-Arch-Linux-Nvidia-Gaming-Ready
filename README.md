## Set Up Keyboard Layout

```
localectl list-keymaps
```
List all available keyboard layouts.

```
loadkeys de
```
Load the selected keyboard layout (default is US).

## Check if Booted to UEFI

```
efivar -l
```
If lots of unknown text appears on your screen that you don’t understand, it means you booted into UEFI mode.

## Check Internet Connection

```
ping archlinux.org
```
This will ping a website and show an output in bytes to check if you’re connected to the internet.

## Get Disk Information

```
lsblk
```
This will show all information about your installed disks. You can use it anytime after changing your disks to display the updated information.

## Partition the Disk

```
cfdisk /dev/nvme0n1
```
This will use the `cfdisk` tool for easier partitioning of your disk.

- Select **gpt**.
- Create a new partition with 1024M size and set the type to **EFI System**.
- Create another partition with the size of half your RAM or up to 8GB (16GB is optional but not necessary) and set the type to **Linux Swap**.
- Create a third partition of 20–40GB and set the type to **Linux root**.
- Use the remaining available space to create a final partition, leaving the type as default.

## Format the Partitions

```
mkfs.fat -F32 /dev/nvme0n1p1
```
Format the EFI partition to FAT32.

```
mkswap /dev/nvme0n1p2
```
Format your swap partition.

```
swapon /dev/nvme0n1p2
```
Enable your swap partition.

```
mkfs.ext4 /dev/nvme0n1p3
```
Format your root partition (20–40GB) to the Linux ext4 file system.

```
mkfs.ext4 /dev/nvme0n1p4
```
Format your home partition to the Linux ext4 file system.

## Mount the Partitions to Install Linux Base System

```
mount /dev/nvme0n1p3 /mnt
```
Mount your root partition to `/mnt`.

```
mkdir /mnt/boot
mkdir /mnt/home
```
Create directories for your bootloader and home partition.

```
mount /dev/nvme0n1p1 /mnt/boot
```
Mount the EFI partition to `/boot`.

```
mount /dev/nvme0n1p4 /mnt/home
```
Mount your home directory.

## Setup Pacman

```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```
Backup the Pacman configuration.

```
pacman -Sy pacman-contrib
```
Install Pacman contrib for configuration.

```
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup >> /etc/pacman.d/mirrorlist
```
Rank the mirror list to optimize download speeds.

```
nano /etc/pacman.conf
```
Enable parallel downloads by removing the `#` from the respective line. (If you are unsure about the config, leave it at 5.)

## Install Linux Base System

```
pacstrap -K /mnt base base-devel linux linux-firmware
```
Install the base Linux system to your root directory.

```
genfstab -U -p /mnt >> /mnt/etc/fstab
```
Generate an `fstab` file.

## Access the Root System

```
arch-chroot /mnt /bin/bash
```
Switch to the shell of your installed system.

## Install Requirements

```
pacman -S networkmanager nano bash-completion
```
Install `NetworkManager`, and `nano`.

```
systemctl enable NetworkManager
```
Enable the `NetworkManager` service.

## Configure Arch System

```
nano /etc/locale.gen
```
Find your language (e.g., `de_DE`), uncomment it, and save changes.

```
locale-gen
```
Apply the language settings to your system.

```
echo LANG=de_DE.UTF-8 >> /etc/locale.conf
```
Add your language configuration.

```
export LANG=de_DE.UTF-8
```

```
ls -sf /usr/share/zoneinfo
```
Navigate to your timezone directory (e.g., `Europe/Berlin`) and link it:

```
ln -sf /usr/share/zoneinfo/Europe/Berlin >> /etc/localtime
```

```
echo iusearchbtw >> /etc/hostname
```
Replace `iusearchbtw` with your preferred hostname.

```
hwclock --systohc --utc
```

```
systemctl enable fstrim.timer
```

```
nano /etc/pacman.conf
```
- Uncomment `[multilib]` (but not `multilib-testing`) and save the changes.
- Enable parallel downloads by removing the `#` from the respective line. (If you are unsure about the config, leave it at 5.)
Make Download faster (second time)

```
pacman -Sy
```

```
passwd
```
Set a password for the root user.

```
useradd -m -G wheel -s /bin/bash <username>
```
Replace `<username>` with your desired username.

```
passwd <username>
```
Set a password for your new user.

```
nano /etc/sudoers
```
Uncomment `%wheel ALL=(ALL) ALL`. (or just search for `# %`)

Add `Defaults rootpw` to require the root password for sudo.

## Install Bootloader

```
exit
systemctl daemon-reload
arch-chroot /mnt /bin/bash
mount -t efivarfs efivarfs /sys/firmware/efi/efivars/
```

```
bootctl install
```
Installs the bootloader.

```
nano /boot/loader/entries/arch.conf
```
Name the config file whatever you want.  
Paste this into the config file:  

```
title Arch
linux /vmlinuz-linux
initrd /initramfs-linux.img
```

```
echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p3) rw" >> /boot/loader/entries/arch.conf
```

## Install NVIDIA Drivers

```
pacman -S nvidia-dkms libglvnd nvidia-utils opencl-nvidia lib32-libglvnd lib32-nvidia-utils lib32-opencl-nvidia nvidia-settings
```

```
sudo pacman -S linux-headers
```

```
nano /etc/mkinitcpio.conf
```
Add the following to `MODULES=()`: `nvidia nvidia_modeset nvidia_uvm nvidia_drm`

```
mkdir /etc/pacman.d/hooks
nano /etc/pacman.d/hooks/nvidia.hook
```
Add the following:
```
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia

[Action]
Depends=mkinitcpio
When=PostTransaction
Exec=/usr/bin/mkinitcpio -P
```

## Reboot the System

```
exit
umount -R /mnt
reboot
```
## Change Keyboard Layout

   ```
   localectl list-keymaps
   ```
 
   ```
   sudo localectl set-keymap de
   ```
   ```
   sudo reboot
   ```

## Make Download Faster (third time)

```
nano /etc/pacman.conf
```
Enable parallel downloads by removing the `#` from the respective line. (If you are unsure about the config, leave it at 5.)

## Install Xorg

```
sudo pacman -S xorg-server xorg-apps xorg-xinit xorg-twm xorg-xclock xterm
```

## Install a Desktop Environment

```
sudo pacman -S gnome gdm
```
Alternatively, for KDE Plasma:
```
sudo pacman -S plasma sddm
```

```
sudo systemctl enable gdm
```
Or enable `sddm` for KDE Plasma.

# Post Setup

## Install Required and Recommended Stuff

```
sudo pacman -Syu
```

```
sudo pacman -S flatpak dolphin mpv git fastfetch wget gedit fzf thermald zram-generator rust cmake pkg-config make qt6-base qt6-tools polkit-qt6 python
```
Only install Dolphin if you using KDE Plasma.

## Install Yay and Paru

```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
sudo rm -r yay
```

```
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
cd ..
sudo rm -r paru
```

## Install CachyOS Repos

```
curl https://mirror.cachyos.org/cachyos-repo.tar.xz -o cachyos-repo.tar.xz
tar xvf cachyos-repo.tar.xz && cd cachyos-repo
sudo ./cachyos-repo.sh
sudo pacman -S linux-cachyos linux-cachyos-headers
sudo pacman -S cachyos-gaming-meta
cd ..
sudo rm -r cachyos-repo.tar.xz cachyos-repo
```
This will make you Arch Linux more Stable.

## Install CachyOS Kernel

```
paru -S cachyos-kernel-manager
```

### Launch the CachyOS Kernel Manager

Select 'Configure' and under 'Options,' select the 'RC - Release Candidate,' then click on 'Build Kernel.'

## Theming for Gnome

Exensions for Gnome:

- Exension List
- Applincator and KStsatusNotifertlem Support
- Blur my Shell
- Just Perfection
- Dash to Dock
- User Themes
- Arc Menu
- Places Status Indicator
- System Monitor
- Media Label and Controls
- Coverflow
- Impatience
- Gnome 4x UI Improvements
- Caffeein
- moveclock
- Weather O´Clock
- compact top bar
- Tiling Shell
- Magic Lamp Effect
- (Open Bar)

```
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git
cd WhiteSur-gtk-theme
./install.sh -l
cd ..
rm -rf WhiteSur-gtk-theme
```

## Theming for KDE

```
git clone https://github.com/joshrandall8478/kde-rice
```

```
cd kde-rice/home 
cp .arch-application-launcher.png ~
```

OR JUST GO HERE: `https://github.com/joshrandall8478/kde-rice?tab=readme-ov-file`
