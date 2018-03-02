# Nyx GNU/Linux
A spin off Linux distro based on LFS.

## How to
This is guide to build Nyx from scratch.


* Clone this 3 repository needed.

		git clone https://github.com/emmett1/scratchpkg # package manager will used with Nyx
		git clone https://github.com/emmett1/ports      # ports (package's build scripts)
		git clone https://github.com/emmett1/nyx        # this repo, toolchain scripts and etc

*Note: build toolchain must build using unprivilege users*

#### Prepare and mount partitions

* Create partion using cfdisk or etc. (change `X` to your partition number)

		$ sudo cfdisk
		$ sudo mkfs.ext4 -L Nyx /dev/sdaX
		
* Mount partition.

		$ sudo mkdir /mnt/lfs
		$ sudo mount /dev/sdaX /mnt/lfs

* Prepare temporary toolchain directory (change `emmett` to your user).

		$ sudo mkdir -pv /mnt/lfs/tools
		$ sudo chown -Rv emmett:emmett /mnt/lfs
		$ sudo ln -sv /mnt/lfs/tools /

#### Build the toolchain

* Enter toolchainscripts directory.

		$ pushd nyx/toolchainscripts
		
* Fetch needed sources by running the script.

		$ ./fetch-sources
		
* Start build toolchain ny running the script.

		$ ./strap
		$ popd

* Change toolchains permision to root.

		$ sudo chown -Rv 0:0 /mnt/lfs

#### Prepare needed files for build final Nyx system

* Create temporary directory in Nyx system to store needed files later after entering chroot.

		$ sudo mkdir /mnt/lfs/nyx
		$ sudo cp -Rv scratchpkg ports nyx/{sources,listinstall} /mnt/lfs/nyx

#### Build final Nyx system

* Change to root user (the rest of this command need to run as root).

		# sudo su

* Make needed directories inside target nyx install then chroot (script).

		# ./chroot-nyx
		
*Note: Now you ere in chroot environmet, any command you run is happen in final Nyx system*

* Install package manager dirty way to install base system. Later scratchpkg get installed again and will tracked.

		# pushd nyx/scratchpkg
		# ./INSTALL.sh
		# popd

* Configure scratchpkg configuration suit to your need.

		# nano /etc/scratchpkg.conf

* Copy sources and ports into right places (needed files prepared earlier).

		# cp -Rv nyx/sources/* /var/cache/scratchpkg/sources
		# cp -Rv ports/{core,extra,git,lxde,wip,xfce4,xorg} /usr/ports

* Installing base system.
	
		# baseinstall

* Install an init to system (rc-init or lfs-bootscripts). Currently have rc-init (bsd-init style) and lfs-bootscript (init provide by LFS). Run `scratch -s rcinit` to search availble daemon for rc-init, `scratch -s lfsbootscripts` to search available daemon for lfs-bootscripts.

		# scratch -i -p rc-init

* Exit chroot and remove temporary toolchain. (remove temporary toolchain consider mandatory because next chroot script will depends on it).

		# exit
		# rm -fr /mnt/lfs/tools
		
#### Re-chroot into Nyx system to configure

* Make system bootable, install kernel.

		# tar -xvf linux-<version>.tar.xz -C /usr/src
		# cd /usr/src/linux-<version>
		# make menuconfig
		# make -j6
		# make modules_install
		# cp arch/x86/boot/bzImage /boot/vmlinuz-<kernel version>
		# cp System.map /boot

* Configure grub

		# grub-install /dev/sda
		# grub-mkconfig -o /boot/grub/grub.cfg
		
Or you can use this example grub.cfg:

		set timeout=5

		menuentry "Nyx GNU/Linux" {
			set root=(hd0,2)
			linux /boot/vmlinuz-4.15 root=/dev/sda2 ro quiet
			}

* Configure system (change `emmett` to your user)

		# vim /etc/fstab
		# passwd
		# vim /etc/hostname
		# vim /etc/hosts
		# vim /etc/rc.conf
		# useradd -m -G users,wheel,video,audio -s /bin/bash emmett
		# passwd emmett
		
*Note: At this stage you can reboot to test you Nyx or you can install needed package like dhcpcd for networking, Xorg for gui and etc.*

*Note: In the temporary nyx directory copied earlier theres a directory 'listinstal' which have list package to install dependecy order sorted. You can install it by using `listinstall` script come with scratchpkg. Example: 'listinstall /nyx/xorg'*
