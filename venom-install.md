Venom Install Guide
=============

Requirements:
------------------
* Currently running linux system or any live linux
* Venom tarball
* Kernel tarball (optional if you want compile your own kernel)


Install:
--------
* Download Venom tarball [here](https://github.com/emmett1/venom/releases)
* Get kernel on [kernel.org](https://www.kernel.org/)
* Prepare the hard disk:

        # cfdisk /dev/sdaX
        # mkfs.ext4 -L venom /dev/sdaX
        # mkdir -p /mnt/venom && mount /dev/sdaX /mnt/venom

        
* Extract venom tarball:

        # tar xvJpf venom-<version>.txz -C /mnt/venom


* Chroot into venom:
 
        # mount -v --bind /dev /mnt/venom/dev
        # mount -vt devpts devpts /mnt/venom/dev/pts -o gid=5,mode=620
        # mount -vt proc proc /mnt/venom/proc
        # mount -vt sysfs sysfs /mnt/venom/sys
        # mount -vt tmpfs tmpfs /mnt/venom/run

        # [ -h /mnt/venom/dev/shm ] && mkdir -pv /mnt/venom/$(readlink /mnt/venom/dev/shm)
        
        # cp -L /etc/resolv.conf /mnt/venom/etc/

        # chroot /mnt/venom /usr/bin/env -i \
        HOME=/root \
        TERM="$TERM" \
        PS1='(venom chroot) \u:\w\$ ' \
        PATH=/bin:/usr/bin:/sbin:/usr/sbin /bin/bash --login

        
* Configure scratchpkg.conf (package manager's configuration file)

        # vim /etc/scratchpkg.conf
        
* Update ports

        # scratch sync
        
* Upgrade the package manager

        # scratch upgrade scratchpkg
        
* Update whole system

        # scratch sysup
        
* Install kernel

        # scratch install linux
        
     or if you want compile your own kernel

        # tar xvf linux-<version>.tar.xz -C /usr/src
        # cd /usr/src/linux-<version>
        # make menuconfig
        # make -j4
        # make modules_install
        # cp arch/x86/boot/bzImage /boot/vmlinuz-<kernel version>
        # cp System.map /boot
        
* Configure grub

        # grub-install /dev/sda
        # grub-mkconfig -o /boot/grub/grub.cfg
        
 * Configure fstab, comment unneeded line

        # vim /etc/fstab
        
 * Change root password (NOTE: this is important, you cant login with empty password)
 
        # passwd
        
 * Configure system (hostname, timezone, clock and etc)
 
        # vim /etc/rc.conf
        
 * Uncomment your locales and generate it
 
        # vim /etc/locales
        # genlocales
        
 * Configure `/etc/hosts`
 
        # vim /etc/hosts
        
 * Add your user (replace `emmett` with your username)
 
        # useradd -m -G users,wheel,video,audio -s /bin/bash emmett
        # passwd emmett
        
* Done, you can reboot now or install basic package first

* Run `scratch help` to see available scratchpkg options

* Use `scratch search <pattern>` to search available packages:

        # scratch search xorg


* Use `scratch install <package1> <package2> <package3> ...` to install package

        # scratch install dhcpcd xorg xfce4

* Run `rc -h/--help` to see availabe option for rc (script to control daemon)

* Available daemon is in `/etc/rc.d/` directory or run `rc list` and add needed daemon to run on boot in `/etc/rc.conf`

