+++
Categories = [ "Serveur" ]
Date = "2017-02-05"
Tags = [ "Arch Linux", "Linux", "Kimsufi", "OVH" ]
Type = "Serveur"
Title = "Installation d'une Arch Linux officielle sur Kimsufi"
Description = "Tutoriel d'installation de l'image officielle Arch Linux sur un serveur Kimsufi"
+++

Pour des questions de vie privée/sécurité, il peut être intéressant d'installer
la version officielle d'Arch Linux sur un serveur Kimsufi. De plus, cela permet
d'avoir toujours une version du noyau Linux à jour (contrairement à la version
du noyau proposée par OVH).

<!--more-->

### Avant-propos

Ce tutoriel s'inspire essentiellement du post de [tetsumaki.net](https://tetsumaki.net/blog/article/2013-06-07-installation-chroot-arch-linux-sur-dedie-kimsufi.html), mais en utilisant cette fois une image officielle plutôt qu'un
chroot proposé par le site.

### Récupérer les fichiers Arch Linux du serveur actuel
Pour commencer, installez une version d'Arch Linux proposée par OVH sur votre
serveur Kimsufi. Cela permettra d'éviter d'avoir à refaire toute la configuration
sur le nouveau serveur. Ensuite, à partir de votre PC, exécutez les commandes
suivantes pour rapatrier les configurations :

    cd ~
    sftp root@<ip_serveur>
    get /etc/hostname
    get /etc/hosts
    get /etc/resolv.conf
    get /etc/netctl/ovh_net_eth0
    get /etc/ssh/sshd_config
    exit

### Vérification et remplacement du nom de l'interface

La version d'Arch Linux proposée par OVH comporte une version de systemd visiblement
inférieure à 197. Par conséquent, il nous faut remplacer `eth0` par le nouveau nom.

    ssh root@<ip_serveur>
    udevadm test-builtin net_id /sys/class/net/eth0 2> /dev/null
    # ID_NET_NAME_PATH=enp1s0 <<< Affiche le nom de l'interface
    exit

La plupart du temps, ce nouveau nom devrait être `enp1s0`.
Il vous faut donc copier le fichier présent sur le PC et le modifier :  

    cd ~
    cp ovh_net_eth0 ovh_net_enp1s0
    vim ovh_net_enp1s0

Vous pouvez ensuite remplacer les parties correspondantes dans le fichier :

    [...]
    Interface=enp1s0
    [...]
    IPCustom=('-6 route add 2001:41D0:8:40ff:ff:ff:ff:ff dev enp1s0' '-6 route add default via 2001:41D0:8:40ff:ff:ff:ff:ff')

### Préparer le disque pour la nouvelle installation

Il vous faut maintenant :

  * Se connecter sur l'interface d'administration de votre serveur Kimsufi
  * Régler le Netboot sur `rescue-pro`
  * Redémarrez le serveur.

Connectez vous ensuite avec les identifiants qui vous ont été transmis par mail
et exécutez les commandes suivantes :

    # Connexion au serveur
    ssh root@<ip_serveur>

    # Affichage et suppression des partitions existantes
    parted
    rm 1
    rm 2
    rm 3

    # Création des nouvelles partitions
    mkpart primary ext2 0% 200MB
    mkpart primary ext4 200MB -2GB
    mkpart primary linux-swap -2GB 100%
    set 1 boot on
    quit

    # Formatage des partitions
    mkfs.ext2 -m0 /dev/sda1
    mkfs.ext4 -m0 /dev/sda2
    mkswap /dev/sda3
    swapon /dev/sda3

### Récupération du chroot Arch Linux à partir de l'image officielle

Il nous faut maintenant extraire le chroot de l'image officielle. Cependant,
il n'y a pas assez de place sur le rescue pour faire toutes ces opérations.
Il faut donc monter le disque dur avant pour pouvoir ensuite télécharger l'image
et extraire le chroot.

    # Installation des prérequis
    apt-get update
    apt-get install -y squashfs-tools haveged

    # Montage du disque dur principal
    mkdir /mnt/hdd
    mount /dev/sda2 /mnt/hdd
    cd /mnt/hdd

    # Téléchargement de l'image Arch Linux (changer la date au besoin)
    mkdir install
    cd install
    wget http://archlinux.polymorf.fr/iso/2017.02.01/archlinux-2017.02.01-dual.iso
    mkdir -p /mnt/archiso

    # Montage de l'image et extraction du chroot de l'image
    mount -t iso9660 -o loop archlinux-2017.02.01-dual.iso /mnt/archiso
    cd /mnt/archiso/arch/x86_64
    unsquashfs -d /mnt/hdd/install/archlinux-live airootfs.sfs

    # Montage des partitions nécessaires au chroot
    ARCH_MINI=/mnt/hdd/install/archlinux-live
    cp /etc/resolv.conf "$ARCH_MINI/etc/" # Copie du DNS pour l'installation
    mount -B /proc "$ARCH_MINI/proc"
    mount -B /dev "$ARCH_MINI/dev"
    mount -B /sys "$ARCH_MINI/sys"

    # Montage des partitions pour le système final
    ARCH_SYS="$ARCH_MINI/mnt"
    mount /dev/sda2 "$ARCH_SYS"
    mkdir -p "$ARCH_SYS"/var/{cache/pacman/pkg,lib/pacman} "$ARCH_SYS"/{dev,proc,sys,run,tmp,etc,boot,root}
    mount /dev/sda1 "$ARCH_SYS/boot"
    echo 'PS1="[\u@\h \W]\$ "' > "$ARCH_SYS/root/.bashrc"

### Installation du système

Maintenant que tout est prêt pour l'installation, il ne reste plus qu'à préparer
pacman et lancer l'installation.

    haveged # Génération de l'entropy pour générer la clef de pacman
    chroot "$ARCH_MINI" /bin/bash
    pacman -Syy # Synchronization de pacman
    pacman-key --init # Initiatialisation des clefs
    pacman-key --populate archlinux # Enregistrement des clefs publiques

    # Installation du système
    pacstrap /mnt base syslinux openssh haveged arch-install-scripts
    exit # Retour sur le recovery
    exit # Retour sur le PC

### Transfert des données depuis le PC

Retournez sur votre PC et lancez les commandes suivantes pour transférer les
données sur le serveur.

    scp hostname root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc
    scp hosts root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc
    scp resolv.conf root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc
    scp sshd_config root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc/ssh
    scp ovh_net_enp1s0  root@<ip_serveur>:/mnt/hdd/install/archlinux-live/mnt/etc/netctl

### Configuration du nouveau système

Il faut maintenant confi## gurer le nouveau système avant de redémarrer la machine.

    # Reconnexion au serveur
    ssh root@<ip_serveur>
    ARCH_SYS=/mnt/hdd/install/archlinux-live/mnt

    # Chroot dans le nouveau système
    mount -B /proc "$ARCH_SYS/proc"
    mount -B /dev "$ARCH_SYS/dev"
    mount -B /sys "$ARCH_SYS/sys"
    chroot "$ARCH_SYS" /bin/bash


Ajoutez le texte suivant de le fichier `/etc/vconsole.conf` :

    KEYMAP=fr-pc

Ajoutez le texte suivante dans `/etc/locale.conf` :

    LANG="fr_FR.UTF-8"
    LC_COLLATE=C

Décommentez les lignes suivantes dans `/etc/locale.gen` :

    en_US.UTF-8 UTF-8
    en_US ISO-8859-1
    fr_FR ISO-8859-1
    fr_FR.UTF-8 UTF-8
    fr_FR@euro ISO-8859-15

Générez les locales décommentées :

    locale-gen

Changez le fuseau horaire en exécutant les commandes suivantes :

    rm /etc/localtime
    ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime

Générez le `/etc/fstab` :

    genfstab -p / >> /etc/fstab

Activez netctl et openssh :

    systemctl enable sshd.service
    netctl enable ovh_net_enp1s0

Installation de syslinux sur le secteur de démarrage :

    syslinux-install_update -iam

Modifiez `/boot/syslinux/syslinux.cfg` avec les lignes suivantes :

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

Changez le mot de passe root pour votre prochaine connexion :

    passwd

### Redémarrage du serveur

Retournez sur l'interface de Kimsufi et réactivez le boot sur le disque dur puis
redémarrez le serveur via l'interface.

Il ne vous reste plus qu'à vous connecter par SSH et le tour est joué !
