clone 3 repo, scratchpkg, nyx and ports
	git clone https://github.com/emmett1/scratchpkg
	git clone https://github.com/emmett1/ports
	git clone https://github.com/emmett1/nyx

build toolchain must build using unprivilege users

prepare and mount partitions
	sudo mkdir /mnt/lfs
	sudo mkfs.ext4 /dev/sdaX
	sudo mount /dev/sdaX /mnt/lfs

prepare toolschain dir
	sudo mkdir -pv /mnt/lfs/tools
	sudo chown -Rv emmett:emmett /mnt/lfs
	sudo ln -sv /mnt/lfs/tools /

build the toolchain
	pushd nyx/toolchainscripts
	./fetch-sources
	./strap
	popd

change toolchains permision to root
	sudo chown -Rv 0:0 /mnt/lfs

prepare needed files
	mkdir /mnt/lfs/nyx
	cp scratchpkg ports nyx/sources /mnt/lfs/nyx


change to root user
	sudo su

make needed directories inside target nyx install then chroot
	./chroot-nyx

install scratchpkg
	pushd nyx/scratchpkg
	./INSTALL.sh
	popd

configure scratchpkg configuration to your need
	nano /etc/scratchpkg.conf

copy sources and ports into right places
	cp -Rv nyx/sources/* /var/cache/scratchpkg/sources
	cp -Rv ports/{core,extra,git,lxde,wip,xfce4,xorg} /usr/ports

now installing base system
	baseinstall

install an init to system (rc-init or lfs-bootscripts)
	scratch -i -p rc-init

make system bootable, install kernel
	tar -xvf linux-<version>.tar.xz -C /usr/src
	cd /usr/src/linux-<version>
	make menuconfig
	make -j6
	make modules_install
	cp arch/x86/boot/bzImage /boot/vmlinuz-<kernel version>
	cp System.map /boot

configure grub
	grub-install /dev/sda
	grub-mkconfig -o /boot/grub/grub.cfg

	set timeout=5

	menuentry "Nyx GNU/Linux" {
	    set root=(hd0,2)
	    linux /boot/vmlinuz root=/dev/sda2 ro quiet
	    }

exit chroot and remove temporary toolchain
	exit
	rm -fr /mnt/lfs/tools