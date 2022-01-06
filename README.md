# archsettings
Notes about my preferred arch setup

### Arch VMWare Player 15 install notes
#### Overview
In general, just follow the wiki:
```
wiki.archlinux.org/title/Installation_guide
```
However, I took extra notes to supplement it where extra detail was helpful.

#### Boot in UEFI mode
It seemed important to setup for UEFI boot mode. However, ```# ls /sys/firmware/efi.efivars``` did not exist. The fix is to add the following to the VM's vmx file:
```
firmware = "efi"
```
Then I could choose to boot in UEFI mode, and the directory was populated.

#### Create the /dev/sda disk partitions
```
$ fdisk /dev/sda
Command: g	<-- GPT parition table for UEFI
Command: n	<-- Add a new partition
Partition number: 1
First sector: 	<-- accept default
Last sector: +550M 	<-- boot partition size (at least 300MB per archwiki)
Command: n
Partition number: 2
First sector:	<-- accept default (will be end of last one)
Last sector: +2G	<-- swap partition size (must be > 512MiB per archwiki)
Command: n
Partition number: 3
First sector:
Last sector:	<-- accept default (will take up rest of disk, for our rootfs!)
Command: t	<-- Change partition type
Partition number: 1
Partition type: 1	<-- EFI (per archwiki)
Command: t
Partition number: 2
Partition type: 19	<-- Linux swap (per archwiki)
Command: t
Partition number: 3
Partition type: q	<-- quit, unchanged, was already linux filesysem!
Command: w	<-- Write table to disk and exit
```

#### Format the partitions
We created three partitions of block disk /dev/sda, as /dev/sda1, /dev/sda2, /dev/sda3.

##### EFI Partition
mkfs.fat -F 32 /dev/sda1

##### Swap partition
mkswap /dev/sda2

##### Root file system partition
mkfs.ext4 /dev/sda3

#### Mount the paritions
Mount root partition to /mnt

create any necessary directories.

For example, create EFI mount point:
```
$ mkdir /mnt/boot
```

Then mount EFI partition to it:
```
$ mount /dev/sda1 /mnt/boot
```

While here, make var and home directories:
```
$ mkdir /mnt/var
$ mkdir /mnt/home
```

#### Installation/pacstrap
Go ahead and run following command before pacstrap, in order to update packages and package database:
```
$ pacman -Syy
```

Regarding pacstrap, follow the minimum, but also add vim here, since we'll need text editor for several remaining steps:
```
$ pacstrap /mnt base linux linux-firmware vim
```

#### Localization
Be sure to set the environment variable after creating locale.conf file:
```
$ echo LANG=en_US.UTF-8 > /etc/locale.conf
$ export LANG=en_US.UTF-8
```

#### Hosts
##### /etc/hostname
cmgrassarch

##### /etc/hosts
```
127.0.0.1    localhost
::1          localhost
127.0.1.1    cmgrassarch.localdomain    cmgrassarch
```

#### Add non-root user
Go ahead and then change the root user password. Afterwards, create the non-root user (this step was not in the wiki)
```
$ useradd -m cmgrass
$ passwd cmgrass
$ usermod -aG wheel,audio,video cmgrass		<-- wheel gives sudo access, so we don't need to log into root
```

#### Install sudo with pacman, add sudo to wheel group
```
$ pacman -S sudo
$ visudo
```
```
<uncomment line: %wheel ALL=(ALL) ALL>
```

#### Bootloader
wiki tells us to pick and install a bootloader. I went with GRUB.
Step into the bootloader/GRUB wiki for specific details.
```
$ pacman -S grub
```

Should add some supporting tools:
```
$ pacman -S efibootmgr dosfstools os-prober mtools
```

Make efi directory in boot directory:
```
$ mkdir /boot/EFI
```

Then, tell GRUB (the bootloader) about it:
```
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

Then, generate a default grub config file:
```
$ grub-mkconfig -o /boot/grub/grub.cfg
```

#### Install networkmanager
```
pacman -S networkmanager
```

Enable it with systemd:
```
$ systemctl enable NetworkManager
```

#### Install git
```
pacman -S git
```

#### Exit chroot, unmount, reboot
```
$ exit
$ umount /mnt/boot
$ umount /mnt
$ reboot
```

### Installing KDE Plasma desktop environment
Up to this point, we should have the minimal Arch linux running.

Now we can add the desktop environment.

There is good information in the arch wiki. Following were my notes.

```
pacman -S xorg          <- Accept all default packages at the prompt
```
```
pacman -S plasma kde-applications xdg-user-dirs gnu-free-fonts       <-- Accept all default packages at the prompt 
```
```
systemctl enable sddm
```
```
pacman -S firefox libreoffice     <-- Accept all default packages
```
```
pacman -S xf86-video-vmware xf86-input-vmmouse
```
```
pacman -S open-vm-tools
```
```
systemctl enable vmtoolsd
```
```
reboot
```

### Other tools to install
Need tmux:
```
pacman -S tmux
```

Need development base
```
pacman -S base-devel
```

Need dos2unix
```
pacman -S dos2unix
```

Need inetutils (for GNU ftp client):
```
pacman -S inetutils
```

### KDE Plasma System Settings
Window Management->Window Rules->Add Rule
```
Description: generalWindowOpacity
Window class: Unimportant
Match: No
Window Types: All except: Desktop
Active opacity: 92
Inactive opacity: 92
```

Input Devices->Keyboard
```
Delay: 180ms
Rate: 60 repeat/s
```

Appearance
```
Breeze dark
```

Blanking screen
```
Power management->Energy saving->Screen Energy Saving: Disable
```

### Other VMWare settings
I followed the wiki to get copy/paste to/from host working (archwiki: VMware/Install Arch Linux as a guest, Drag and drop, copy/paste)

Following is the summary:
```
pacman -Syy   <--- Update packages and lists
pacman -S open-vm-tools
pacman -S gtkmm3
```

Then, per the wiki, I verified the vmware desktop xdg setting file exists:
```
etc/xdg/autostart/vmware-user.desktop
```

Afterwards, I noticed shared folders stopped working, so I had to do the following, which fixed it:

Run Command:
```
/usr/bin/vmhgfs-fuse --enabled
```

Update /etc/fstab
```
# VMWare shared folders
.host:/ /mnt/hgfs fuse.vmhgfs-fuse allow_other 0 0
```

### Fix mozilla font
Maybe the correct serif was not available? Anyway, someone on the arch forum suggested these, and the worked:
```
pacman -S ttf-dejavu ttf-liberation
```
