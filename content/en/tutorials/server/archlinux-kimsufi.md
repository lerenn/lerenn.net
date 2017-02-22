+++
Categories = [ "Server" ]
Date = "2017-02-05"
Tags = [ "Arch Linux", "Linux", "Kimsufi", "OVH" ]
Type = "Server"
Title = "Official Arch Linux install on Kimsufi server"
Description = "Install tutorial of the official Arch Linux image on a Kimsufi server"
+++

For privacy and security, it can be interesting to install the official version of
Arch Linux on a Kimsufi server. Moreover, you can have an up-to-date Linux kernel
(which is not the case for the one proposed by OVH).
<!--more-->

### Foreword

This tutorials is built essential on the post of [tetsumaki.net](https://tetsumaki.net/blog/article/2013-06-07-installation-chroot-arch-linux-sur-dedie-kimsufi.html),
but using an official image instead of a chroot proposed by the website.

### Download files from actual server as models
Install an Arch Linux version proposed by OVH on your Kimsufi server. This will
will avoid a complete rewriting of the server configuration. Then, from your PC,
execute those commands to download configurations :

    cd ~
    sftp root@<ip_serveur>
    get /etc/hostname
    get /etc/hosts
    get /etc/resolv.conf
    get /etc/netctl/ovh_net_eth0
    get /etc/ssh/sshd_config
    exit

### Verification and replacement of the interface name in config files

It seems that the Arch Linux version proposed by OVH has a systemd version below 197.
Consequently, we have to replace `eth0` by the new name.

    ssh root@<ip_serveur>
    udevadm test-builtin net_id /sys/class/net/eth0 2> /dev/null
    # ID_NET_NAME_PATH=enp1s0 <<< Display the interface name
    exit

Most of the time, this new name should be `enp1s0`.
You now have to copy the file on your computer and modify it :

    cd ~
    cp ovh_net_eth0 ovh_net_enp1s0
    vim ovh_net_enp1s0

Then, you can replace corresponding parts in the file :

    [...]
    Interface=enp1s0
    [...]
    IPCustom=('-6 route add 2001:41D0:8:40ff:ff:ff:ff:ff dev enp1s0' '-6 route add default via 2001:41D0:8:40ff:ff:ff:ff:ff')

### Prepare hard drive disk for the new install

Now, you have to :

  * Connect on your Kimsufi server administration interface
  * Set Netboot on `rescue-pro`
  * Restart server

Then, you can connect on the server with IDs sent by mail and execute the
following commands:

    # Connect to server
    ssh root@<ip_serveur>

    # Delete existing partitions
    parted
    rm 1
    rm 2
    rm 3

    # Create new partitions
    mkpart primary ext2 0% 200MB
    mkpart primary ext4 200MB -2GB
    mkpart primary linux-swap -2GB 100%
    set 1 boot on
    quit

    # Format partitions
    mkfs.ext2 -m0 /dev/sda1
    mkfs.ext4 -m0 /dev/sda2
    mkswap /dev/sda3
    swapon /dev/sda3

### Download Arch Linux chroot from official image

Now, we have to extract chroot from official image. But, there is not enough
space on rescue to execute these operations. So, we have to mount the hard drive
disk to download the image and extract the chroot.

    # Install dependencies
    apt-get update
    apt-get install -y squashfs-tools haveged

    # Mount main HDD
    mkdir /mnt/hdd
    mount /dev/sda2 /mnt/hdd
    cd /mnt/hdd

    # Download Arch Linux image (change date if needed)
    mkdir install
    cd install
    wget http://archlinux.polymorf.fr/iso/2017.02.01/archlinux-2017.02.01-dual.iso
    mkdir -p /mnt/archiso

    # Mount image and extract chroot from it
    mount -t iso9660 -o loop archlinux-2017.02.01-dual.iso /mnt/archiso
    cd /mnt/archiso/arch/x86_64
    unsquashfs -d /mnt/hdd/install/archlinux-live airootfs.sfs

    # Mount partitions needed for chroot
    ARCH_MINI=/mnt/hdd/install/archlinux-live
    cp /etc/resolv.conf "$ARCH_MINI/etc/" # Copie du DNS pour l'installation
    mount -B /proc "$ARCH_MINI/proc"
    mount -B /dev "$ARCH_MINI/dev"
    mount -B /sys "$ARCH_MINI/sys"

    # Mount partitions for final system
    ARCH_SYS="$ARCH_MINI/mnt"
    mount /dev/sda2 "$ARCH_SYS"
    mkdir -p "$ARCH_SYS"/var/{cache/pacman/pkg,lib/pacman} "$ARCH_SYS"/{dev,proc,sys,run,tmp,etc,boot,root}
    mount /dev/sda1 "$ARCH_SYS/boot"
    echo 'PS1="[\u@\h \W]\$ "' > "$ARCH_SYS/root/.bashrc"

### System install

Now that everything is ready for install, we just have to prepare pacman and
start install.

    haveged # Generate entropy for pacman keys generation
    chroot "$ARCH_MINI" /bin/bash
    pacman -Syy # Pacman synchronization
    pacman-key --init #  Keys init
    pacman-key --populate archlinux # Register public keys

    # System install
    pacstrap /mnt base syslinux openssh haveged arch-install-scripts
    exit # back on recovery
    exit # back on PC

### Upload data from computer

Now that we are on the PC again, launch these commands to transfer data to the server :

    scp hostname root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc
    scp hosts root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc
    scp resolv.conf root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc
    scp sshd_config root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc/ssh
    scp ovh_net_enp1s0  root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc/netctl

### New system configuration

Now, we have to configure the new system before reboot.

    # Reconnect to server
    ssh root@<ip_serveur>
    ARCH_SYS=/mnt/hdd/install/archlinux-live/mnt

    # Chroot in new system
    mount -B /proc "$ARCH_SYS/proc"
    mount -B /dev "$ARCH_SYS/dev"
    mount -B /sys "$ARCH_SYS/sys"
    chroot "$ARCH_SYS" /bin/bash

Add following text to the file `/etc/vconsole.conf` :

    KEYMAP=fr-pc

Add following text to the file `/etc/locale.conf` :

    LANG="fr_FR.UTF-8"
    LC_COLLATE=C

Decomment following lines in `/etc/locale.gen` :

    en_US.UTF-8 UTF-8
    en_US ISO-8859-1
    fr_FR ISO-8859-1
    fr_FR.UTF-8 UTF-8
    fr_FR@euro ISO-8859-15

Generate decommented locales :

    locale-gen

Change time by executing following commands :

    rm /etc/localtime
    ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime

Generate `/etc/fstab` :

    genfstab -p / >> /etc/fstab

Activate netctl and openssh :

    systemctl enable sshd.service
    netctl enable ovh_net_enp1s0

Install syslinux on boot sector :

    syslinux-install_update -iam

Modify `/boot/syslinux/syslinux.cfg` with following lines :

    [...]
    TIMEOUT 1
    [...]
    LABEL arch
        MENU LABEL Arch Linux
        LINUX ../vmlinuz-linux
        APPEND root=/dev/sda2 rw
        INITRD ../initramfs-linux.img

    LABEL archfallback
        MENU LABEL Arch Linux Fallback
        LINUX ../vmlinuz-linux
        APPEND root=/dev/sda2 rw
        INITRD ../initramfs-linux-fallback.img
    [...]

Change root password for your next connection :

    passwd

### Server reboot

Go back into the Kimsufi interface and reactivate boot on hard drive disk.
Then restart server from the interface.

You just have to reconnect to your server and enjoy !
