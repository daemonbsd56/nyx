# Nyx GNU/Linux
A spin off Linux distro based on LFS

## How to
This is the guide to build Nyx from scratch. It's following the directives of the Svn LFS version of linuxfromscratch.org handbook. It' simple, clear, comprehensive, fully written in bash. It's meant for those who are already familiar 
with Linux and source based distros. As written in the ALFS project, it's strongly advised to have built manually yourself a couple
of LFS installations. 

* Clone these 3 needed repositories.

		git clone https://github.com/emmett1/scratchpkg   # package manager which will be used with Nyx
		git clone https://github.com/emmett1/ports        # ports (package's build scripts)
		git clone https://github.com/emmett1/nyx          # this repo, toolchain scripts and etc

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
		$ sudo chown -Rv $USER:$USER /mnt/lfs
		$ sudo ln -sv /mnt/lfs/tools /

#### Build the toolchain

* Enter the toolchainscripts directory.

		$ pushd nyx/toolchainscripts
		
* Fetch each needed sources and building the toolchain by running the `bootstrap` script.
    - Run `./bootstrap fetch` to fetch only sources.
    - Append TMP=(path) to change build directory (default=/tmp/build).
    - Append LOG=(path) to change build log directory (default=`pwd`/log).
    - Append SRC=(path) to change fetched sources directory (default=`pwd`/src).
    - `bootstrap` script is resumeable. Re-run `./bootstrap` script to resume where you left it.
    - Nano (text-editor) and wget (fetching tool) are installed during the toolchain and will be a valuable aid during 
      the next step in case the building of the proper final Nyx.
    - By default this script will compile using -j6 MAKEFLAGS, you can change it in `functions` file.

		$ ./bootstrap

		$ popd
		
    
* Change toolchain's permission to root.

		$ sudo chown -Rv 0:0 /mnt/lfs
		
*Note: If you intend to keep temporary toolchain for use in future to build LFS/Nyx system, now is the right time to back it up. Later commands will alter the toolchain currently in place, rendering them useless for future builds.*

*Note: Use this command to backup the toolchain: `cd /mnt/lfs && tar -cvJpf /lfs-toolchain.txz *` (backed up toolchain in your host root (/) directory*

#### Prepare needed files for building the final Nyx system

* Create temporary directory in Nyx system to store needed files later after entering chroot.

		$ sudo mkdir /mnt/lfs/nyx
		$ sudo cp -Rv scratchpkg ports nyx/baseinstall /mnt/lfs/nyx

* Optionally copy sources fetched by toolchain script to Nyx system.

		$ sudo cp -Rv nyx/toolchainscripts/src /mnt/lfs/nyx


#### Building the final Nyx system

* Change to root user (the rest of these commands must be run as root).

		$ sudo su

* Make the necessary directories inside the nyx installation and then chroot in it (executed by a script).

		# ./nyx/chroot-nyx
		
*Note: Now you are in the chroot environment, any command you run is executed the final, definitve Nyx system*

* Install the package manager the "dirty way" in order to track all installed base system packages. Later on, scratchpkg will be reinstalled and will get tracked. 

		# pushd nyx/scratchpkg
		# ./INSTALL.sh
		# popd

* Configure scratchpkg.conf (configuration file) in accordance to your needs.

		# nano /etc/scratchpkg.conf

* If you copy sources fetched by the toolchain before, run this to copy it it to the right places.

		# cp -Rv nyx/src/* /var/cache/scratchpkg/sources

* Copy ports into right places (those are the files you prepared earlier).

		# cp -Rv nyx/ports/{core,extra,git,lxde,wip,xfce4,xorg} /usr/ports

* Installing base system (baseinstall is a script provided by scratchpkg). By default, baseinstall script will install rc-init (my custom bsd/Slackware style init) but you can replace the init with `lfs-bootscripts` (sysvinit provided by LFS). Run `scratch -s lfsbootscripts` to search available daemon if using `lfs-bootscripts`.
	
		# ./nyx/baseinstall

* Exit chroot and remove temporary toolchain. (removing the temporary toolchain is considered mandatory because next chroot script will depend on it. You may naturally still tar it into a file and copy it somewhere else).

		# exit
		# rm -fr /mnt/lfs/tools
		
#### Re-chroot into Nyx system to configure

* Chroot in the final Nyx system by running the 'chroot' script.

		./nyx/chroot /mnt/lfs

* Make system bootable, install kernel. You can get the kernel at [kernel.org](https://www.kernel.org/) or use the one that fetched earlier by `fetch-sources` script. (its recommended to placed kernel sources in /usr/src directory)

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
		# ln -svf /usr/share/zoneinfo/Asia/Kuala_Lumpur /etc/localtime
		# useradd -m -G users,wheel,video,audio -s /bin/bash emmett
		# passwd emmett
		
* Optionally edit configuration file that exist in `/etc/conf.d/` directory to suit your need.
		
* At this stage you may reboot and test your Nyx or you can continue and install needed packages like dhcpcd for networking, Xorg for gui and etc.

* Some basic tips to use the package manager (scratchpkg):
	- `scratch search <pattern>` - to find the available package in repository
	- `scratch install <package1> <package2> <...>` - to install packages, ex. 'scratch install xorg-meta plasma-meta networkmanager'
	- its recommended to install 'xorg-meta' before install other desktop environment.
	- for now in the repo has plasma, xfce4, lxde for the desktop environment, some window manager like dwm, openbox, fluxbox and twm, and some applications for daily use.
 

## Screenshot
#### xfce4
![image](https://github.com/emmett1/nyx/blob/master/screenshot/2018-03-12-132211_1360x768_scrot.png)
![image](https://github.com/emmett1/nyx/blob/master/screenshot/2018-03-12-132340_1360x768_scrot.png)
![image](https://github.com/emmett1/nyx/blob/master/screenshot/2018-03-12-133743_1360x768_scrot.png)

#### lxde
![image](https://github.com/emmett1/nyx/blob/master/screenshot/2018-03-12-133549_1360x768_scrot.png)
![image](https://github.com/emmett1/nyx/blob/master/screenshot/2018-03-12-133601_1360x768_scrot.png)
![image](https://github.com/emmett1/nyx/blob/master/screenshot/2018-03-12-133520_1360x768_scrot.png)

#### kde5
![image](https://github.com/emmett1/nyx/blob/master/screenshot/2018-04-14-223158_1600x900_scrot.png)
![image](https://github.com/emmett1/nyx/blob/master/screenshot/2018-04-14-223353_1600x900_scrot.png)
