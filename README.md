# Nyx GNU/Linux
A spin off Linux distro based on LFS

## How to
This is the guide to build Nyx from scratch. It's following the directives of the Svn LFS version of linuxfromscratch.org handbook. It' simple, clear, comprehensive, fully written in bash. It's meant for those who are already familiar 
with Linux and source based distros. As written in the ALFS project, it's strongly advised to have built manually yourself a couple
of LFS installations. 

* Clone these 3 needed repositories.

		git clone https://github.com/emmett1/scratchpkg # package manager which will be used with Nyx
		git clone https://github.com/emmett1/ports      # ports (package's build scripts)
		git clone https://github.com/emmett1/nyx        # this repo, toolchain scripts and etc

*Note: The toolchain must be built as an unprivileged user just like stated in the LFS book. Make sure that this user has also the sudo rights*

#### Prepare and mount partition(s)

* Create a partition using cfdisk or etc. (change `X` to your partition number).

		$ sudo cfdisk
		$ sudo mkfs.ext4 -L Nyx /dev/sdaX    # -L for labeling the partition.
		
* Mount partition.

		$ sudo mkdir /mnt/lfs
		$ sudo mount /dev/sdaX /mnt/lfs
		
 		

* Prepare the temporary toolchain directory (change `emmett` to your user name! ).

		$ sudo mkdir -pv /mnt/lfs/tools
		$ sudo chown -Rv emmett:emmett /mnt/lfs
		$ sudo ln -sv /mnt/lfs/tools /

#### Build the toolchain

* Enter the toolchainscripts directory.

		$ pushd nyx/toolchainscripts
		
* Fetch needed sources by running the script. 

		$ ./fetch-sources
		
   - it will download the latest development set of the SVN/LFS book		
* Start building the toolchain by running the script.

		$ ./strap
		$ popd
   - Nano (text-editor) and wget (fetching tool) are installed during the toolchain and will be a valuable aid during 
    the next step in case the building of the proper final Nyx.
    
* Change toolchain's permission to root.

		$ sudo chown -Rv 0:0 /mnt/lfs

#### Prepare needed files for building the final Nyx system

* Create temporary directory in Nyx system to store needed files later after entering chroot.

		$ sudo mkdir /mnt/lfs/nyx
		$ sudo cp -Rv scratchpkg ports nyx/{sources,listinstall} /mnt/lfs/nyx

#### Building the final Nyx system

* Change to root user (the rest of these commands must be run as root).

		# sudo su

* Make the necessary directories inside the nyx installation and then chroot in it (executed by a script).

		# ./chroot-nyx
		
*Note: Now you are in the chroot environment, any command you run is executed the final, definitve Nyx system*

* Install the package manager the "dirty way" in order to install the base system. Later on, scratchpkg will be installed again and will track all the installed packages. 

		# pushd nyx/scratchpkg
		# ./INSTALL.sh
		# popd

* Configure scratchpkg.conf (configuration file) in accordance to your needs.

		# nano /etc/scratchpkg.conf

* Copy sources and ports into right places (those are the files you prepared earlier).

		# cp -Rv nyx/sources/* /var/cache/scratchpkg/sources
		# cp -Rv ports/{core,extra,git,lxde,wip,xfce4,xorg} /usr/ports

* Installing base system. (a baseinstall script is provided by scratchpkg)
	
		# baseinstall

* Install an init to system. Currently rc-init (bsd-init style) and lfs-bootscript (init provided by LFS) is available. Run `scratch -s rcinit` to search available daemon for rc-init, `scratch -s lfsbootscripts` to search available daemon for lfs-bootscripts.

		# scratch -i -p rc-init

* Exit chroot and remove temporary toolchain. (removing the temporary toolchain is considered mandatory because next chroot script will depend on it. You may naturally still tar it into a file and copy it somewhere else).

		# exit
		# rm -fr /mnt/lfs/tools
		
#### Re-chroot into Nyx system to configure

* Chroot in the final Nyx system by running the 'chroot' script.

		./chroot /mnt/lfs

* Make system bootable, install kernel. You can get the kernel at kernel.org or use the one that fetched earlier by `fetch-sources` script. (its recommended to placed kernel sources in /usr/src directory)

		# tar -xvf linux-<version>.tar.xz -C /usr/src
		# cd /usr/src/linux-<version>
		# make menuconfig
		# make -j6
		# make modules_install
		# cp arch/x86/boot/bzImage /boot/vmlinuz-<kernel version>
		# cp System.map /boot

* Configure grub.

		# grub-install /dev/sda
		# grub-mkconfig -o /boot/grub/grub.cfg
		
  Or you can use this example grub.cfg:

		set timeout=5

		menuentry "Nyx GNU/Linux" {
			set root=(hd0,2)
			linux /boot/vmlinuz-4.15 root=/dev/sda2 ro quiet
			}

* Configure system (change `emmett` to your user name and `Asia/Kuala_Lumpur` to your country).

		# vim /etc/fstab
		# passwd
		# vim /etc/hostname
		# vim /etc/hosts
		# vim /etc/rc.conf
		# ln -svf /usr/share/zoneinfo/Asia/Kuala_Lumpur
		# useradd -m -G users,wheel,video,audio -s /bin/bash emmett
		# passwd emmett
		
*Note: At this stage you may reboot and  test your Nyx or you can continue and install needed packages like dhcpcd for networking, Xorg for gui and etc.*

*Note: In the temporary nyx directory copied earlier, there is  a directory 'listinstall' which contains a  package list to install. This one is already sorted by dependency and is really handy. You can install it by using `listinstall` script which is provided by scratchpkg. Example: 'listinstall /nyx/listinstall/xorg'*
