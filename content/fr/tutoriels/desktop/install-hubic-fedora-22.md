+++
Categories = [ "Desktop" ]
Date = "2015-08-06"
Tags = [ "Fedora", "Hubic", "Installation" ]
Title = "Installer Hubic sous Fedora 22"
Description = "Installation de Hubic sous Fedora 22"
+++

Nous allons voir ici que, malgré l'absence de paquet RPM sur le site de OVH, il
est assez facile d'installer Hubic sur Fedora 22 afin de profiter de la
synchronisation des fichiers. Suivez le guide !
<!--more-->

### Qu'est-ce que c'est ?

Hubic est le service de cloud proposé par [OVH](https://www.ovh.com/).
A l'instar d'autres services américains comme Google Drive ou Dropbox, il
vous permet de sauvegarder jusqu'à 25Go de vos fichiers gratuitement.
Ce service ayant une offre spéciale pour le campus de Lille (50Go gratuits) et
les données étant stockées en France, j'ai préféré confier mes données à Hubic.

### Prérequis

Nous aurons besoin de `make` et cette commande est en théorie déjà présente dans
votre distribution Fedora. Si ce n'est pas le cas, entrez cette commande :

    sudo dnf install make

Installez les paquets mono à l'aide de la commande suivante (en ayant au
préalable activé [RPM Fusion](http://rpmfusion.org/Configuration)) :

    sudo dnf install mono-core mono-data-sqlite mono-data

### Installation

Téléchargez le fichier `.tar.gz` se trouvant à cette
[adresse](http://mir7.ovh.net/ovh-applications/hubic/hubic-Linux/2.1.0/) et
extrayez le à l'aide de `tar -zvxf`.

Naviguez ensuite jusqu'au dossier Hubic fraîchement décompressé et tapez la
commande suivante pour installer Hubic :

    sudo make install

### Lancement de la synchronisation

Une fois ces étapes effectuées, il ne vous reste plus qu'à lancer les deux
commandes :

    hubic start
    hubic login adresse@email.tld /dossier/synchronisé

La première démarre le service Hubic. La seconde vous permettra de vous
connecter avec votre compte (en entrant votre adresse mail comme ci-dessus) pour
lancer la synchronisation dans le dossier indiqué.

Si vous rencontrez des problèmes à la suite de ces étapes, je vous conseille de
suivre les informations à propos des problèmes courants sur le
[site de Hubic](https://forums.Hubic.com/showthread.php?230-Hubic-Linux-sortie-de-la-version-b%EAta).

#### Sources

* [Billet sur l'installation sous Linux sur le site de Hubic](https://forums.Hubic.com/showthread.php?230-Hubic-Linux-sortie-de-la-version-b%EAta)
* [Akrylik.com](http://www.akrylik.com/2014/11/11/Hubic-sous-linux/)
* [Troll Factory](http://trollfactory.fr/installer-Hubic-sur-fedora-19-717)
