# VPS-setup
### Creating an Arch linux host from scratch (with transip.eu BladeVPS)
This README a memo I use for basic Arch Linux installation on these VPS (SeaBIOS, NoVNC).

When installing Arh Linux onto a VPS the archiso is mounted as loop0 device:
 *root@archiso ~ #*
so let's start from here.

##### Change the keymap to match our keyboard's:
	loadkeys fr_CH

## 1) Partition Table and File System Creation

##### Start fdisk to partition the virtual drive:
	gdisk /dev/vda
	
##### At the fdisk prompt, delete old partitions (if there is any) and create a new one:  
 *Type 'o'*. This will clear out any partitions on the drive.  
 *Type 'p'* to list partitions. There should be no partitions left.

### 1.1 Boot partition
 *Type 'n'*, press *ENTER* for Default (or type *'1'*) to create the first partition on the drive, press *ENTER* to accept the default first sector, then *type '+1M'* for the last sector.
 *Type 'ef02'* and press *ENTER* to set the partition type to BIOS boot partition)

### 1.2 Swap Partition
 *Type 'n'*, press *ENTER* to accept default first sector, then *'+4G'* (eg) for swap minimum recommended size and press *ENTER*.
 *Type '8200'* to set the partition type to *swap* and press *ENTER*.
 
### 1.4 Root partition
 *Type 'n'*, press *ENTER* to accept default first sector, then *ENTER* again for default last sector.
 *Type '8e00'* to set this partition type to *Linux LVM*.  
  
 *Type 'p'* to check how partition table is looking, if everything looks good, write the partition table and exit by typing *'w'*.

### 1.5 [OPTIONAL] Create the FAT file system for boot partition (-n is for label option):
	mkfs.vfat /dev/vda1 -n B00T

### 1.6 Create and mount the swap file system (-L is for label option):
 	mkswap /dev/vda2 -L sw4p
 	swapon /dev/vda2
 
### 1.7 Create and mount root file system (-L is for label option):
 	mkfs.ext4 /dev/vda3 -L r00t
 	mount /dev/vda3 /mnt

## 2) Now we can start Installation

### 2.1 To install the base and development packages:
 	pacstrap -K /mnt linux linux-firmware base base-devel
 
### 2.2 Configure the system  
##### Generate an fstab file (use -U to define mountpoints by UUIDs or -L for LABELs): 
	genfstab -pL /mnt >> /mnt/etc/fstab

### 2.3 Change root into the new system:
	arch-chroot /mnt

### 2.4 Create the /etc/hostname file:
 	echo myhostname >> /etc/hostname
 
### 2.5 Set the time zone and adjust time:

##### Copy zoneinfo to localtime
 	ln -s /usr/share/zoneinfo/Europe/Zurich /etc/localtime
  
##### Adjust time:
	hwclock --systohc
 
### 2.6 Install further build utilities, a network manager (I use netctl) and other useful packages:
	sudo pacman -S neovim git wget netctl ifplugd dhcpcd openssh stow
 
### 2.7 Gemerate locales
##### Uncomment the needed locales in *'/etc/locale.gen'*, then generate them with:
	locale-gen
 
##### Add at least LANG=your_locale in /etc/locale.conf  
	nvim /etc/locale.conf
	
### 2.8 Add console keymap and font preferences in /etc/vconsole.conf:
	echo /etc/vconsole.conf >> KEYMAP=fr_CH
 
### 2.9 Configure kernel options with /etc/mkinitcpio.conf:

##### Add virtual machine support:
	nvim /etc/mkinitcpio.conf
	MODULES=(virtio virtio_blk virtio_pci virtio_net virtio_ring)
 
##### Create a new initial RAM disk with:
	mkinitcpio -P
##### If you get these warnings:

	== WARNING: Possibly missing firmware for module: wd719x
	== WARNING: Possibly missing firmware for module: aic94xx
 
[You can safely ignore them !](https://wiki.archlinux.org/index.php/mkinitcpio### Possibly_missing_firmware_for_module_XXXX)
 
### 2.10 Set the root password:
 	passwd
 
## 3) Install a boot loader (Limine):
  	pacman -S limine
   	cp /usr/share/limine/limine-bios.sys /boot/
    	limine bios-install /dev/vda

### 3.1 Create limine.conf
	
	timeout: 5

	/Arch Linux
	    protocol: linux
	    kernel_path: boot():/boot/vmlinuz-linux
	    kernel_cmdline: root=LABEL=r00t rw
	    module_path: boot():/boot/initramfs-linux.img
 
## 4) Create a user with administrator rights:
 	useradd -m -g users -G storage,power,wheel -s /bin/bash "username"
 	nano /etc/sudoers and uncomment the %wheel line
 	passwd "username" to define a password and su "username" to login as newly created user

## 6) Commplete installation and restart VPS system
##### Now we can assume our Installation and basic setup is finished:
	exit && umount -R /mnt
	reboot

## 7) Configure Network
	cd /etc/netctl
	install -m640 examples/ethernet-dhcp internet
	sudo nano internet
	Inteface=ens3
	netctl enable internet && sudo netctl start internet
 
#### NB: recent archiso had DNS resolution deactivated by default.
#### It is mandatory to install packets from remote arch linux mirrors

#### Here is how to check if it is activated:
	systemctl status systemd-resolved.service

#### If inactive, you can start and enable it as follow:
	systemctl enable systemd-resolved && systemctl start systemd-resolved
	
## 8) Should you want to install a desktop environment and some fancy packages:
	pacman -S xfce4 xfce4-goodies slim xorg wget baobab mlocate binutils firefox p7zip xarchiver
	
## 9) Configure Xfce
##### Create .xinitrc:
	nano ~/.xinitrc and "starxfce4" then save
	Edit slim.conf default username and autologin yes for autologin.
	starxfce4'
