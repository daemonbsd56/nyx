venom Install Guide
=============

Requirements:
------------------
* Currently running linux system or any live linux
* venom tarball
* Kernel tarball


Install:
--------
* Download venom tarball.
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

        # chroot /mnt/venom /usr/bin/env -i \
        HOME=/root \
        TERM="$TERM" \
        PS1='(venom chroot) \u:\w\$ ' \
        PATH=/bin:/usr/bin:/sbin:/usr/sbin /bin/bash --login

        
* Configure scratchpkg.conf (package manager's configuration file)

        # vim /etc/scratchpkg.conf

        
* Update ports and system

        # scratch sync && scratch sysup

        
* Install kernel

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

        
 * Configure system (change emmett to your user name and Asia/Kuala_Lumpur to your country)

        # vim /etc/fstab
        # passwd
        # vim /etc/rc.conf
        # vim /etc/locales
        # vim /etc/hosts
        # useradd -m -G users,wheel,video,audio -s /bin/bash emmett
        # passwd emmett

        
* Done , you can reboot now or install basic package first

* Use 'scratch search <pattern>' to search available packages:

        # scratch search xorg


* Use 'scratch install <package1> <package2> <package3> ...' to install package

        # scratch install dhcpcd xorg xfce4


