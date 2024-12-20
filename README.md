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
- Create a new partition with 2048M size and set the type to **EFI System**.
- Create another partition with the size of half your RAM or up to 8GB (16GB is optional but not necessary) and set the type to **Linux Swap**.
- Create a third partition of 20–40GB and set the type to **Linux root**.
- Use the remaining available space to create a final partition, leaving the type as default.

*You don't need to create a separate root partition for your system. This is only necessary for additional security, but it's not really required. However, if you plan to install more apps, resizing the partition will be necessary — though this is not needed if you choose not to create a separate root partition.*

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

## Make downloads faster

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
ls -sf /usr/share/zoneinfo/
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
hwclock --systohc --localtime
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
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
```
Name the title whatever you want, so you can replace "Arch Linux" with the name you want in your bootloader menu.

```
echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p3) rw" >> /boot/loader/entries/arch.conf
```

## Change Keyboard Layout

   ```
   localectl list-keymaps
   ```
 
   ```
   sudo localectl set-keymap de
   ```

## Install Xorg

```
pacman -S xorg
```

## Install a Desktop Environment

```
pacman -S gnome gdm
```
Alternatively, for KDE Plasma:
```
pacman -S plasma sddm
```

```
systemctl enable gdm
```
Or enable `sddm` for KDE Plasma.

## Install Recommended and Required Stuff

```
sudo pacman -Syu
```

```
sudo pacman -S flatpak git fastfetch wget gedit fzf thermald zram-generator rust python python-pip dolphin gnome-tweaks
```
Only install `dolphin` if you using KDE Plasma. And only install `gnome-tweaks` on Gnome.

```
yay -S ptyxis gnome-terminal
```

## Reboot the System

```
exit
umount -R /mnt
reboot
```

## Install CachyOS Repos & Nvidia Drivers

```
curl https://mirror.cachyos.org/cachyos-repo.tar.xz -o cachyos-repo.tar.xz
tar xvf cachyos-repo.tar.xz && cd cachyos-repo
sudo ./cachyos-repo.sh
sudo pacman -S linux-cachyos linux-cachyos-headers cachyos-gaming-meta linux-cachyos-nvidia-open nvidia-utils lib32-nvidia-utils nvidia-settings cachyos-kernel-manager cachyos-settings yay
cd ..
sudo rm -r cachyos-repo.tar.xz cachyos-repo
```

## Install Chaotic-AUR-Repos

```
sudo su
```

```
pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com
pacman-key --lsign-key 3056513887B78AEB
pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst'
pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'
```

```
exit
```

```
sudo gedit /etc/pacman.conf
```
Cut all the `cachyos` repos (from line 77 to line 87) to the end of the config. Remove the last 5 lines and replace them with the `cachyos` repos.  
Then add:

```
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```
to the bottom, like the other ones.

```
sudo pacman -Sy
```

## Install yay and paru.

```
sudo pacman -S yay
```

```
yay -S paru
```

## Install CachyOS Kernel

```
sudo pacman -S cachyos-kernel-manager
```

### Launch the CachyOS Kernel Manager

Select 'Configure' and under 'Options,' select the 'RC - Release Candidate,' then click on 'Build Kernel.' (Only if you do not have the latest RC kernel `linux-cachyos-rc`, or if you prefer to use a stable kernel with lower performance, you should use the `linux-cachyos` kernel.)
After that, execute the installation to install the kernel.
Boot again into you USB Drive and Mount the root and boot partition and access it with chroot and change you Bootloader config to:

## Change Bootloaderconfig

```
sudo nano /boot/loader/entries/arch.conf
```

```
linux /vmlinuz-linux-cachyos-rc
initrd /initramfs-linux-cachyos-rc.img
```
Change the lines in the config to match and boot into the new kernel you installed.

## Theming for Gnome

### Extensions for Gnome:

- Extension List
- Application and KStatusNotifierItem Support
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
- Caffeine
- Compact Top Bar
- Tiling Shell
- Magic Lamp Effect
- Open Bar

For OpenBar pre-config download [this](https://github.com/Nico-Shock/sdfgdfg/releases/download/download/Purple) file and import it to OpenBar.

```
yay -S candy-icons-git
```

```
yay -S ttf-dejavu-sans-mono-powerline-git
```

Then in Ptyxis press `Ctrl+` and select a theme of your choice.
I selected the 11th line from the bottom, the slightly darker purple.
Select `DejaVu Sans Mono Bold 12` for the font.


```
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git
cd WhiteSur-gtk-theme
./install.sh -l
cd ..
sudo rm -r WhiteSur-gtk-theme
```

```
sudo pacman -S zsh
```

Search for a Theme (I use [this](https://github.com/ohmyzsh/ohmyzsh/blob/master/themes/xiong-chiamiov-plus.zsh-theme) theme by the way.)
I modified it to this:

```
PROMPT=$'%{\e[0;34m%}%B┌─[%b%{\e[0m%}%{\e[1;32m%}%n%{\e[1;30m%}@%{\e[0m%}%{\e[0;36m%}%m%{\e[0;34m%}%B]%b%{\e[0m%} - %b%{\e[0;34m%}%B[%b%{\e[1;37m%}%~%{\e[0;34m%}%B]%b%{\e[0m%} - %{\e[0;34m%}%B[%b%{\e[0;33m%}'%D{"%a %b %d, %H:%M"}%b$'%{\e[0;34m%}%B]%b%{\e[0m%}
%{\e[0;34m%}%B└─%B[%{\e[1;35m%}$%{\e[0;34m%}%B] <>%{\e[0m%}%b '
PS2=$' \e[0;34m%}%B>%{\e[0m%}%b '
ZSH_THEME="ThemeName"
```
Add `ZSH_THEME="ThemeName"` and put a name in it.

How to modify the theme:

```
sudo nano ~/.zshrc
```
Paste the code for your theme in the file or modify it the way you want.

```
chsh -s $(which zsh)
```
Make zsh your default terminal.

Recommended for VMware:

```
sudo pacman -S open-vm-tools xf86-video-vmware xf86-input-vmmouse
```

```
sudo systemctl enable vmtoolsd
sudo systemctl start vmtoolsd
```

Recommended for KVM:

```
sudo pacman -S spice-vdagent
```

```
sudo systemctl enable spice-vdagentd
sudo systemctl start spice-vdagentd
```

### *Make sure the theming steps are only examples of how I would theme my Linux on Gnome. You can customize it infinitely.*
