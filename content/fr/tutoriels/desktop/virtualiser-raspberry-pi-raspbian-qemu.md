+++
Title = "Virtualiser une Raspberry Pi, sous Linux"
Date = "2016-05-16"
Tags = [ "Virtualisation", "QEMU", "ARM", "Raspberry Pi", "Linux", "Raspbian" ]
Categories = [ "Virtualisation" ]
Description = "Virtualiser une Raspberry Pi, sous Linux"
Draft = "true"
+++

Développant actuellement un système de diffusion d'images à base de Raspberry Pi pour le BDE de mon école, je vais vous décrire toute la procédure pour virtualiser une Raspberry Pi (sous ARM) sur un Linux (sous amd64).
<!--more-->

![Rapsberry-Pi-Logo](/img/tutoriels/virtualiser-raspberry-pi-raspbian-qemu/raspberry-logo.png)

### Un problème : Repartir d'un système propre

Dans le cadre de mon projet, il est nécessaire de configurer le système pour faire certaines choses comme ouvrir automatiquement le navigateur et le mettre en plein écran au démarrage. Pour me faciliter la vie et celle de mes successeurs, j'ai pris le parti de développer un script d'installation avec [Ansible](https://fr.wikipedia.org/wiki/Ansible_%28logiciel%29).

Par conséquent, j'eus besoin de remettre à zéro régulièrement le système de ma Raspberry Pi pour tester ce script dans les meilleurs conditions. Sauf que réinstaller un système sur une carte SD à chaque besoin, c'est long ! Je me suis donc penché sur la possibilité d'emuler le système Raspbian sur mon ordinateur et c'est ce que je vais vous décrire dans ce tutoriel.

### Comment virtualiser ?

#### Virtualbox : Nope

Ayant l'habitude de virtualiser des distributions Linux pour pouvoir les tester, je voulais utiliser ma solution de toujours qu'est VirtualBox pour emuler de l'ARM.
Toutefois, il s'avère que VirtualBox ne permet de simuler que des systèmes i386 et amd64. Fausse route donc.

#### QEMU : La solution

Lors de mes cours à Telecom Lille, j'ai eu l'occasion de tester QEMU que wikipedia présentera mieux que moi :

> QEMU permet la virtualisation sans émulation, si le système invité utilise le même processeur que le système hôte, ou bien d'émuler les architectures des processeurs x86, **ARM**, PowerPC, Sparc, MIPS...
> [[Wikipedia](https://fr.wikipedia.org/wiki/QEMU)]

Parfait ! Il ne me reste plus qu'à essayer de transposer ceci à mon problème.

## Virtualiser

TODO

### Dépendances

* make gcc
* qemu

### Etapes

* Télécharger noyaux raspbian
* Compiler le noyau
