---
layout: post
title:  "Promiscuous mode"
date: 2018-11-18 18:31:00
tags:
- linux
- sysadmin
- network
description: ''
color: 'rgb(38,50,56)'
cover: '/assets/cover/hello.gif'
---

Le Promiscuous mode (traduit de temps en temps en « mode promiscuité »), en informatique, se réfère à une configuration de la carte réseau, qui permet à celle-ci d'accepter tous les paquets qu'elle reçoit, même si ceux-ci ne lui sont pas adressés. (cf. [Wikipédia](https://fr.wikipedia.org/wiki/Promiscuous_mode)).

Introduction
------------
Il y a 2 semaines je me suis cassé les dents sur un problème assez ... ardent...
J'explique le contexte :
Je devais installer un serveur ESXi 6.5 sur VMWare Workstation pour effectuer des tests :
- Installation ESXi sur VMWare Workstation : OK
- Création de la VM : OK
- Test de connectivité : Pas OK

De là je suis parti sur une séance de débugage ... Bien aidé par 2 profs, ils se reconnaîtront.

Débugage
--------

Bon de là j'avais plusieurs possibilités :
- Problèmes d'installation ESXi
- Problème réseau sur l'host (VMWare Workstation)
- Problème réseau sur l'ESXi (La VM)

Nomons les machines :
- saeronix (IP : 192.168.144.1) : L'host Principal
- ESXi (192.168.144.128) : La VM ESXi
- alpine (192.168.144.4) : La VM dans la VM

Commençons :
- Ping alpine -> ESXi : OK
- Ping ESXi -> saeronix : OK
- Ping saeronix -> w3.org : OK
- Ping alpine -> w3.org : Not OK
Ah ... Bah merde ...
- Ping ESXi -> w3.org : OK
Fuck le lien est bon pourtant... J'ai regardé, la configuration du vSwitch est bonne, les interfaces VMWare Workstation sont bonnes (de toute façon ESXi arrive a Ping ...)

Continuons :
- Ping Alpine -> saeronix : Not OK ....
Hum hum ...
Faisons un tableau :

|            |w3.org|Saeronix|ESXi|Alpine|
|------------|:----:|:------:|:--:|:----:|
|**w3.org**  |/     |OK      |OK  |NOT OK|
|**Saeronix**|OK    |OK      |OK  |NOT OK|
|**ESXi**    |OK    |OK      |OK  |OK    |
|**Alpine**  |Not OK|NOT OK  |OK  |OK    |


On est parti pour un TCPDUMP des familles sur saeronix pour voir si on reçoit quelque-chose ou si ça disparaît simplement :
```
[root@saeronix]$ tcpdump -i any src 192.168.144.4 or dst 192.168.144.4
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
18:11:21.626893 ARP, Request who-has saeronix tell 192.168.144.4, length 46
18:11:21.626913 ARP, Reply saeronix is-at 00:50:56:c0:00:02 (oui Unknown), length 28
18:11:22.627933 ARP, Request who-has saeronix tell 192.168.144.4, length 46
18:11:22.627978 ARP, Reply saeronix is-at 00:50:56:c0:00:02 (oui Unknown), length 28
18:11:23.640774 ARP, Request who-has saeronix tell 192.168.144.4, length 46
18:11:23.640793 ARP, Reply saeronix is-at 00:50:56:c0:00:02 (oui Unknown), length 28
^C
6 packets captured
7 packets received by filter
0 packets dropped by kernel
```
Miracle on reçoit des paquets :D.
Mais toujours pas de retour réel au niveau de mon ping ... Donc le problème vient du retour ...

Regardons de plus près ce qu'il ce passe :
- Saeronix reçois des paquet provenant de 192.168.144.4
- Saeronix réponds donc a l'interface ayant l'adresse mac 00:50:56:c0:00:02 (Alpine)

Si on réfléchi bien il y a un principe assez simple en réseau : Quand une carte réseau reçoit un paquet, elle regarde si c'est bien pour elle, si cela n'est pas pour elle, elle la drop.

Or l'adresse MAC de alpine est 00:50:56:c0:00:02... Donc... Bah les paquets sont drop.

Après recherche il s'avère que cela ce nomme le Promiscuous Mode ...
Le principe est simple : il premet que tout les paquets puissent arriver jusqu'à ESXi pour que ce soit **lui** qui traite le paquet en ayant connaissance des informations (VM, réseaux internes,...)

Résolution
-----------

Sous linux, pour activer le Promiscuous mode, l'utilisateurs doit avoir accès en écriture à l'interface (virtuel ou non).

2 choix s'offre à moi :
- Ajouter à l'utilisateur un groupe qui à la permission d'écriture sur l'interface.
- Autoriser tout le monde à écrire sur la carte à coup de chmod a+rw.

Dans mon cas je ne m'embête pas, le chmod la bonne interface et on en place plus.
```
chmod a+rw /dev/vmnet0
```
> *NOTE : `vmnet0` étant l'interface que j'utilise pour mon ESXi.*

Conclusion
----------

Sur Windows il n'y a pas de soucis, VMWare parvient a activer le Promiscuous Mode, or moi je suis sous Linux de ce fait, les services n'ont pas toujours les priviligège nécéssaire.


*Sources :*
- [https://kb.vmware.com/s/article/287](https://kb.vmware.com/s/article/287) - Activation du Promiscous Mode sur VMWare Workstation Linux
- [https://fr.wikipedia.org/wiki/Promiscuous_mode](https://fr.wikipedia.org/wiki/Promiscuous_mode) - Wikipédia
- [https://www.it-connect.fr/quest-ce-que-le-promiscious-mode/](https://www.it-connect.fr/quest-ce-que-le-promiscious-mode/) - IT-Connect