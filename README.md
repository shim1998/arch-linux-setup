# arch-linux-setup
My setup of Arch Linux 2021 with KDE-Plasma as Desktop Environment.
Mostly refered Arch Wiki.

- Download Arch Linux from https://archlinux.org/download/
- Use Belena Archer to create image and flash it into a pendrive 
- Boot the PC with the pendrive loaded with the image

## Setup Internet Connection

- If you are using Wired LAN then skip

- use iwctl to connect wirelessly to Wifi and look for network drivers in your PC
```bash
# iwctl
[iwd]# device list
```
- this will show you a list of network drives on your PC
- Choose wireless one (for me it was wlan0)
- Now we search for wireless networks
```bash
[iwd]# station <device name> scan
[iwd]# station <device name> get-networks
```
- Now we get a list of networks to connect to. 
```bash
[iwd]# station device connect SSID
[iwd]# exit
```
- After all of this ensure that net connection is working.
```bash
# ping google.com
```

## Disk Partitioning and mounting


### Partitioning

- Use `lsblk` or `fdisk l` to view your disk partitions
- To modify your partition disk do `fdisk /dev/disk_to_partition`
- If you are dual booting you don't need to create EFI Partition
- Create a ext4 format drive for root folder
- Create a swap space (optional but preferred)
```
Mount point 	Partition 	Partition type 	Suggested size
/mnt/boot or /mnt/efi1 	/dev/efi_system_partition 	EFI system partition 	At least 260 MiB
[SWAP] 	/dev/swap_partition 	Linux swap 	More than 512 MiB
/mnt 	/dev/root_partition 	Linux x86-64 root (/) 	Remainder of the device 
```

### Formatting
- Format the partitions you want to mount
```bash
# mkfs.ext4 /dev/root_partition
# mkswap /dev/swap_partition
```

### Mounting
- Mount the partitions into mnt
```bash 
# mount /dev/root_partition /mnt
# swapon /dev/swap_partition
```

## Install Arch in /mnt

- Use pacstrap to mount linux onto `/mnt`.

```bash
# pacstrap /mnt base linux linux-firmware
```

- Generate fstab
```bash
# genfstab -U /mnt >> /mnt/etc/fstab
```

## Arch Chroot and setting up of new OS

- Change root into arch-chroot
```bash
# arch-chroot /mnt
```

- Set the time zone and set system clock to it
```bash
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
# hwclock --systohc
```

- Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8`. Then generate locale
```bash
# locale-gen
```
- Create `/etc/locale.conf` file and set LANG variable.
```bash
LANG=en_US.UTF-8
```
- Create a hostname for your PC.
```bash
# cat >/etc/hostname
shimthearch-desktop
```

- Generate matching entries for hosts.
```bash
#cat >/etc/hosts
127.0.0.1	localhost
::1		localhost
127.0.1.1	shimthearch-desktop.localdomain	shimthearch-desktop
```

- Install neccessary packages that are needed
```bash
# pacman -S base-devel networkmanager vim git sddm xorg plasma wayland-session os-prober grub wpa_supplicant wireless_tools network-manager-applet gnome-keyring
```

- Setup internet so that no restart is needed
```bash
# systemctl enable NetworkManager.service
# systemctl start NetworkManager.service
```
- Setup user.
```bash
# pacman -S sudo
# useradd <username>
# passwd <username>
```
- Add user to group
```bash
# sudo usermod -a -G wheel audio video storage network
```

- Edit configuration of sudoers list
```bash
# EDITOR=<nano/vim> visudo
```
- Uncomment the wheel group line. 
```bash
## Same thing without a password
%wheel ALL=(ALL) NOPASSWD: ALL
``` 
## OS is ready now, unmount and restart.

- Exit out of chroot by typing exit
```bash
# exit
```
- Unmount
```bash
# umount /mnt
```
- Reboot
```bash
# reboot
```

