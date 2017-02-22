+++
Categories = [ "Desktop" ]
Date = "2015-08-06"
Tags = [ "Fedora", "Hubic", "Installation" ]
Title = "Install Hubic on Fedora 22"
Description = "Hubic installation on Fedora 22"
+++

We'll see that, despite the lack of RPM package on OVH website, it's quite easy to
install Hubic on Fedora 22 and enjoy file sync.
<!--more-->

### What is it ?

Hubic is the cloud service proposed by [OVH](https://www.ovh.com/).
Like other american services as Google Drive or Dropbox, you can store 25GB of
data, free of charge. This service having a special offer for Lille university
(50GB free) and data being stored in France, I prefered to trust Hubic than
an american service.

### Prerequisite

We'll need to use `make` and this package is theorically already present in your
Fedora distribution. If not, enter this command :

    sudo dnf install make -y

Install mono packages with the help of the following command (if you've already
activated [RPM Fusion](http://rpmfusion.org/Configuration) repositories) :

    sudo dnf install mono-core mono-data-sqlite mono-data

### Installation

Download the `.tar.gz` file at this [address](http://mir7.ovh.net/ovh-applications/hubic/hubic-Linux/2.1.0/)
et extract it with the command `tar -zvxf`.

Then, browse into the freshly created Hubic directory and enter the next command to install Hubic :

    sudo make install

### Launching synchronization

Having carried out these steps, you just have to launch those two commands :

    hubic start
    hubic login adresse@email.tld /synchronized/dir

The first one launches Hubic service. The last one connects you to your account
(by entering your email address) to start synchronization in the specified directory.

If you have troubles with these steps, you can follow informations about usual problems
on [Hubic website](https://forums.Hubic.com/showthread.php?230-Hubic-Linux-sortie-de-la-version-b%EAta).

#### Sources

* [Hubic installation under Linux](https://forums.Hubic.com/showthread.php?230-Hubic-Linux-sortie-de-la-version-b%EAta)
* [Akrylik.com](http://www.akrylik.com/2014/11/11/Hubic-sous-linux/)
* [Troll Factory](http://trollfactory.fr/installer-Hubic-sur-fedora-19-717)
