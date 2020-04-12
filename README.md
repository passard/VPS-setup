# VPS-setup
### Creating an Arch linux host from scratch (with transip.eu BladeVPS)
You will find in this README a memo I use for basic Arch Linux installation on these VPS.

All you get when installing Arh Linux onto a VPS is a basic promp:
*root@archiso ~ #

##### Lets change the keymap to match our keyboard's:
	loadkeys de_CH

## 1) Partition Table and File System Creation

##### Start fdisk to partition the virtual drive:
	fdisk /dev/vda
	
##### At the fdisk prompt, delete old partitions (if there is any) and create a new one:  
 *Type 'o'*. This will clear out any partitions on the drive.  
 *Type 'p'* to list partitions. There should be no partitions left.

### 1.1 Boot partition
 *Type 'n'*, then *'p'* for primary, *'1'* (Default) for the first partition on the drive, press *ENTER* to accept the default first sector, then *type '+100M'* for the last sector.  
 *Type 't'*, then *'c'* to set the first partition to type W95 FAT32 (LBA).  
 *Type 'a'*, then *'1'* to toggle bootable flag on Boot partition. 

### 1.2 Extended partition
 *Type 'n'*, then *'e'* for extended, *'2'* (Default) for the first partition on the drive, press *ENTER* to accept the default first sector, then *ENTER* for the last sector (Extended partition will take all the available space left).

### 1.3 Root Partition
 *Type 'n'* : All space for primary partition is in use, so fdisk will automatically add a logical partition.  
 Adding logical partition 5  
 Press *ENTER* to accept default first sector, then *'+49G'* (eg) for root partition size.  
 *Type 't'*, partition number should be 5, press *ENTER*, then *'8e'* to set the first logical partition to type Linux LVM.
 
### 1.4 Swap partition
 *Type 'n'* : All space for primary partition is in use, so fdisk will automatically add a logical partition.  
 Adding logical partition 6  
 Press *ENTER* to accept default first sector, then ENTER again for default last sector.  
 *Type 't'*, partition number should be 6, press *ENTER*, then *'82'* to set the second logical partition to type Linux Swap.  
  
 *Type 'p'* to check how partition table is looking, if everything looks good, write the partition table and exit by typing *'w'*.

### 1.5 Create and mount the FAT file system (-n is for label option):
	mkfs.vfat /dev/vda1 -n b00t
 	mkdir /mnt/boot
 	mount /dev/vda1 /mnt/boot

### 1.6 Create and mount the ext4 file system (-L is for label option):
 	mkfs.ext4 /dev/vda5 -L vr00t
 	mount /dev/vda5 /mnt
 
### 1.7 Create and mount Swap file system (-L is for label option):
 	mkswap /dev/vda6 -L sw4p
 	swapon /dev/vda6

#### NB: recent archiso had DNS resolution deavtivated by default.
#### It is mandatory to install packets from remote arch linux mirrors

#### Here is how to check if it is activate:
	systemctl status systemd-resolved.service

#### If inactive, you can start and enable it as follow:
	systemctl enable systemd-resolved && systemctl start systemd-resolved

## 2) Now we can start Installation

### 2.1 Install the base and development packages
 	pacstrap /mnt linux linux-firmware base base-devel
 
### 2.2 Configure the system  
##### Generate an fstab file (use -U or -L to define by UUID or labels):  
	genfstab -pL /mnt  /mnt/etc/fstab

### 2.3 Change root into the new system:
	arch-chroot /mnt

### 2.4 Create the /etc/hostname file.
 	echo myhostname >> /etc/hostname
 
### 2.5 Set the time zone:
 	ln -s /usr/share/zoneinfo/Europe/Zurich /etc/localtime

### 2.6 Uncomment the needed locales in /etc/locale.gen, then generate them with:
 	locale-gen
##### Add at least LANG=your_locale in /etc/locale.conf  
	nano /etc/locale.conf
	LANG=en_GB.UTF-8
	LC_COLLATE=C
	LC_TIME=en_GB.UTF-8
	
### 2.7 Add console keymap and font preferences in /etc/vconsole.conf:
	cd 
	
### 2.8 Configure kernel options with /etc/mkinitcpio.conf:  
##### Add virtual machine support:  
	MODULES="virtio virtio_blk virtio_pci virtio_net virtio_ring"
##### Create a new initial RAM disk with:  
	mkinitcpio -p linux
##### If you get these warnings:  
	== WARNING: Possibly missing firmware for module: wd719x
	== WARNING: Possibly missing firmware for module: aic94xx
 
[You can safely ignore them !](https://wiki.archlinux.org/index.php/mkinitcpio### Possibly_missing_firmware_for_module_XXXX)
 
### 2.9 Set the root password:
 	passwd
 
## 3) Install a boot loader (GRUB):
 	pacman -S grub
 	grub-install --target=i386-pc /dev/vda
 	grub-mkconfig -o /boot/grub/grub.cfg
 
## 4) Create a user with administrator rights:
 	useradd -m -g users -G storage,power,wheel -s /bin/bash "username"
 	nano /etc/sudoers and uncomment the %wheel line
 	passwd "username" to define a password and su "username" to login as newly created user

## 5) Install basic build utilities
	sudo pacman -S multilib-devel fakeroot git jshon wget make pkg-config autoconf automake patch

## 6) Install a network manager, we will pick netctl, with some useful options:
	pacman -S netctl ifplugd dhcpcd openssh

## 7) Configure Network
	cd /etc/netctl
	install -m640 examples/ethernet-dhcp internet
	sudo nano internet
	Inteface=ens3
	netctl enable internet && sudo netctl start internet
	
## 8) Should you want to install a desktop environment and some fancy packages:
	pacman -S xfce4 xfce4-goodies slim xorg wget baobab mlocate binutils synapse firefox p7zip xarchiver
	
## 9) Configure Xfce  
##### Create .xinitrc:
	nano ~/.xinitrc and "starxfce4" then save
	Edit slim.conf default username and autologin yes for autologin.
	starxfce4'
	
## 9) Commplete installation and restart VPS system
##### Now we can assume our Installation and basic setup is finished:
	exit && umount -R /mnt
	reboot
