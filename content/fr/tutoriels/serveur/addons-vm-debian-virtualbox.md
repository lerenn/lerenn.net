+++
Categories = [ "Serveur" ]
Date = "2015-03-23"
Tags = [ "Serveur", "Debian", "VirtualBox", "Addons" ]
Title = "Installer les Addons invité sur une VM Debian"
Description = "Installation des addons invités sur une VM Debian"
+++

Voici un court article sur les quelques instructions nécessaires à l''installation des addons invités sur une Debian permettant d''accéder à divers outils bien pratiques (affichage adapté, copier-collé bidirectionnel, etc.).
<!--more-->

    # En tant que root, mettez à jour votre système :
    apt-get update -y && apt-get upgrade -y

    # Installation des paquets nécessaires
    apt-get install build-essential module-assistant

    # Configuration du système pour la création de modules kernel :
    m-a prepare

    # Faites "Insérer l'image CD des additions invité..." dans le menu "Périphériques" de votre VM
    # Montez ensuite le cd en tapant la commande suivante (sauf si le CD s''est monté tout seul) :
    mount /media/cdrom

    # Lancer ensuite l'installation, avec la commande suivante, et suivez les instructions :
    sh /media/cdrom/VBoxLinuxAdditions.run
