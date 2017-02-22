+++
Categories = [ "Serveur" ]
Date = "2015-08-10"
Tags = [ "Serveur", "Raspberry Pi", "OpenVPN", "Ethernet", "Access Point" ]
Title = "Point d'accès OpenVPN Ethernet avec une RaspberryPi"
Description = "Tutoriel pour mettre en place un point d'accès OpenVPN par Ethernet avec une Raspberry Pi"
+++
Un accès Wifi peut parfois être médiocre et certains ports peuvent être bloqués.
Pour palier à ce problème, je vous propose donc une solution pour accéder à
internet par OpenVPN.
<!--more-->
### Pourquoi un tel équipement ?
Dans certaines résidences universitaires, l'accès à internet est assuré par un
réseau Wifi à la portée médiocre et aux ports bloqués par un proxy. Ne pouvant
bouger mon équipement près de la porte, la borne wifi se trouvant dans le couloir,
et ayant toujours le problème de ports à régler, l'idée fut de transformer ma
Raspberry Pi en point d'accès OpenVPN Ethernet.

![Schéma de la connexion](/img/tutoriels/raspberry-pi-openvpn-ethernet-access-point/schema-wifi-residence.png)

La Raspberry placée à l'entrée de la pièce, l'accès à internet devrait être
accélérée par l'absence de parois (tel que mon réfrigérateur). Le client
OpenVPN, quant à lui, devrait nous permettre d'outrepasser la limitation de
ports imposée par le proxy de l'université.

### Prérequis

Nous aurons besoin de :

* 1 x Raspberry Pi (Naaan ?);
* 1 x Clé Wifi (compatible Linux, de préférence avec des avis positifs
concernant son utilisation sur Raspberry Pi);
* 1 x Câble Ethernet;
* 1 x Serveur OpenVPN (et par extension, un serveur si vous n'utilisez pas de
service mutualisé). Si vous n'en avez pas, je vous conseille les
[VPS de OVH](https://www.ovh.com/fr/vps/) et le
[tutoriel de Nicolargo](http://blog.nicolargo.com/2010/10/installation-dun-serveur-openvpn-sous-debianubuntu.html).

Bien entendu, vous pouvez avoir également 2 clés Wifi et remplacer l'Ethernet
par une des clefs pour faire de la Raspberry un répéteur wifi.

### Mise en place

Avant de commencer le tutoriel, je tiens à noter qu'il est possible d'inverser
le rôle du câble Ethernet et de la clé Wifi : Il vous suffira normalement
d'inverser `wlan0` et `eth0` quand vous les croiserez dans les
commandes/configurations.

**De plus, toutes les commandes que vous verrez dans ce tutoriel sont à exécuter
en tant que `root`.** Voici la commande à utiliser si vous ne l'êtes pas :

    sudo su

#### Organisation

Nous mettrons en place un serveur
[DHCP](https://fr.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol),
sur la Raspberry pour assurer la connexion automatique des appareils, et un
client OpenVPN. Ces éléments seront reliés de la façon suivante :

![Schéma de la connexion](/img/tutoriels/raspberry-pi-openvpn-ethernet-access-point/schema-interne-raspberry.png)

#### Configurer les interfaces eth0 et wlan0

Ouvrez donc le fichier `/etc/network/interfaces` avec votre éditeur de texte
préféré et recopiez la configuration suivante :

    auto lo
    iface lo inet loopback

    iface eth0 inet static
    address 172.17.0.1
    netmask 255.255.255.0

    allow-hotplug wlan0
    iface wlan0 inet manual
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Nous avons donc 3 interfaces : `lo`, `eth0` et `wlan0`.

* **lo**: Interface de loopback (réseau interne au système avec adresse
comme `127.0.0.1`) et nous ne nous en occuperons pas.
* **eth0**: Interface Ethernet que nous réglons de manière statique. Nous lui
attribuons la première adresse ip de notre futur réseau (soit `172.17.0.1`) avec
le masque associé (`255.255.255.0`).
* **wlan0**: Interface de la clef Wifi que nous allons régler avec les paramètres
se trouvant dans le fichier `/etc/wpa_supplicant/wpa_supplicant.conf` et qui
correspond au réseau Wifi auquel vous souhaitez vous connecter.

##### Configurations possibles de wlan0

Si vous êtes sur un réseau domestique (ex: Livebox), il s'agit très probablement
d'un réseau protégé par une clef WPA. Vous pouvez donc modifier le fichier
`/etc/wpa_supplicant/wpa_supplicant.conf` avec la configuration suivante (en
prenant soin de bien changer le nom du réseau et le mot de passe):

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    network={
        ssid="NomRéseau"
        scan_ssid=1
        key_mgmt=WPA-PSK
        psk="MotDePasse"
    }

Si jamais votre réseau n'est pas sécurisé en WPA-PSK, vous changez la valeur
du champ `key_mgmt` et adapter les paramètres en fonction (et
[Qwant](https://www.qwant.com/web) est votre ami).

#### Le DHCP, ou comment vous connecter sans difficulté à votre Raspberry

Il nous faut d'abord installer le paquet correspondant au serveur DHCP :

    apt-get install isc-dhcp-server

Ensuite allez modifier le fichier `/etc/dhcp/dhcpd.conf` pour changer les
valeurs `option domain-name` et `option domain-name-servers` par les valeurs
suivantes :

    option domain-name “mydomain.org”;
    option domain-name-servers 8.8.8.8, 8.8.4.4;

Puis ajouter la configuration du sous-réseau à la fin de ce même fichier :

    subnet 172.17.0.0 netmask 255.255.255.0 {
        range 172.17.0.2 172.17.0.254;
        option routers 172.17.0.1;
    }

Pour les explications : Nous réglons le sous-réseau `172.17.0.0` avec comme
masque `255.255.255.0`. Nous autorisons l'assignation d'adresse de 2 à 254 (`0`
étant l'adresse du réseau, `1` l'adresse de la raspberry et `255` l'adresse de
broadcast). L'adresse de routage est définie comme étant l'adresse de la
Raspberry.

Relancez ensuite le DHCP avec la commande suivante :

    service isc-dhcp-server restart

#### Le client OpenVPN

Maintenant, il faut installer le client OpenVPN :

    apt-get install openvpn

Copiez ensuite l'intégralité des fichiers de la configuration client dans le
dossier `/etc/openvpn` et relancer le client avec la commande suivante :

    /etc/init.d/openvpn restart

#### Redirection du flux et instauration du NAT

Pour gérer la redirection du flux et le NAT, copiez le script suivant dans le
nouveau fichier `/etc/init.d/start-access-point` :

    #!/bin/bash

    echo "Cleaning iptables rules"
    iptables -F
    iptables -t nat -F
    iptables -X

    echo "Launching OpenVPN Client"
    /etc/init.d/openvpn restart

    echo "Enabling NAT on tun0"
    iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE

    echo "Enabling forwarding"
    echo "1" > /proc/sys/net/ipv4/ip_forward

Autorisez l'exécution du fichier avec la commande `chmod +x` :

    chmod +x /etc/init.d/start-access-point

Puis, lancez la commande suivante afin que le script soit lancé à chaque
démarrage de la Raspberry :

    update-rc.d start-access-point defaults

Et pour finir, lancez le script :

    /etc/init.d/start-access-point

### Conclusion

Voilà ! C'est terminé ! Maintenant, vous pouvez brancher votre pc/console/switch
sur votre raspberry et vous devriez avoir accès à internet par votre Raspberry
et votre serveur OpenVPN. A vous les parties de StarCraft et les téléchargements
des images Linux par torrent (rien d'illégal, bien entendu !).

Si vous voulez vous en assurez, allez sur [monip.org](http://monip.org). Si
vous avez l'adresse de votre serveur, c'est que vous passez bien par le VPN.

### Sources

* [ronnutter.com](http://www.ronnutter.com/raspberry-pi-installing-dhcp-server/) (DHCP)
* [karlrupp.net](http://www.karlrupp.net/en/computer/nat_tutorial) (NAT)
