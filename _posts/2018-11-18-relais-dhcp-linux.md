---
layout: post
title:  "Relais DHCP sous linux"
date: 2018-11-18 17:40:00
tags:
- sysadmin
- linux
- debian
- dhcp
description: ''
color: 'rgb(38,50,56)'
cover: '/assets/cover/hello.gif'
---

Un agent relais DHCP redirige toutes les demandes DHCP et BOOTP depuis un sous-réseau sans serveur DHCP vers un ou plusieurs serveurs DHCP situés sur d'autres sous-réseaux.

Il n'est pas déconseillé de savoir utiliser un serveur DHCP (cf. [Serveur DHCP avec Linux](/serveur-dhcp-linux/))

Introduction
------------

Pour effectuer le relay DHCP il y a un outils très connu qui ce nomme `dhcrelay`.
Son principe est simple, il récupère les trame DHCP REQUEST et les renvoies a(ux) serveur(s) DHCP qui ce situe dans un autre réseau (les trâmes DHCP ne traverse pas les réseau *et heureusement*).

Utilisation
-----------

#### Cas 1 : Serveur relais classique

- Interface du relais : `ens18`
- IP du serveur DHCP : `172.31.1.100`

```
dhcrelay -4 -i ens18 172.31.1.100
```

#### Cas 2 : Plusieurs serveur DHCP

- Inteface du relais : `ens18`
- IP du premier serveur DHCP : `172.31.1.100`
- IP du second serveur DHCP : `172.31.1.101`

```
dhcrelay -4 -i ens18 172.31.1.100 172.31.1.101
```

#### Cas 3 : Le relais DHCP est le router*

- Interfaces du relais : `ens18` et `ens19`
- IP du premier serveur DHCP : `172.31.1.100`
- IP du second serveur DHCP : `172.31.1.101`

```
echo 1 > /proc/sys/net/ipv4/ip_forward
dhcrelay -4 -i ens18 -i ens19 172.31.1.100 172.31.1.101
```

#### Observation

On peut voir que la commande n'est pas compliqué, il faut définir toutes les interfaces qui inteviendra dans le relais des trâmes DHCP ainsi que toutes les IP des serveurs DHCP.

Exemple avec Debian
-------------------

J'aime bien prendre l'exemple avec cette distribiton car plus courament utlisé.
Cahier des charges :
- Le serveur joue le rôle de routeur
- Il y a 2 serveurs DHCP 172.31.1.100 et 172.31.1.101
- Le serveur dispose de 3 interfaces : `ens18` `ens19` et `ens20`
- L'interface `ens19` est une interface d'accès à internet.

Ce serveur fera donc office de passrelle entre les 2 réseaux qui isole les clients et les serveurs.

Sous debian le paquet qui contient dhcrelay est `isc-dhcp-relay`
> Pour rappel, pour installer un paquet : apt-get install **isc-dhcp-relay**

Dans un premier temps nous devons autorisé le forwarding (le routage de paquets entre les interfaces), pour ça il y a 2 méthodes :
- La méthode sale mais qui marche a coup sur : `echo 1 > /proc/sys/net/ipv4/ip_forward`
- La méthode propre mais fonctionne pas partout : 
    - Il faut éditer le fichier `/etc/sysctl.conf` et décommenter/ajouter la ligne `net.ipv4.ip_forward=1`
    - Une fois fait un petit `sysctl -p`

Une fois cela fait on configure dhcrelay
> /etc/default/isc-dhcp-relay

```
# L'IP du serveur DHCP
SERVERS="172.31.1.100 172.31.1.101"

# L'interface qui reçois les trâmes ET qui renvoie ces mêmes trâmes
INTERFACES="ens18 ens20"
```
Le service ce nomme `isc-dhcp-relay`, donc : `systemctl restart isc-dhcp-relay`

Conclusion
----------

La configuration du relay n'est pas spécialement compliqué. Mais une simple mauvaise config et ça marche pas.


*Source :*
- [https://www.isc.org/dhcp-manual-pages/](https://www.isc.org/dhcp-manual-pages/) - Doccumentation Officiel
- [https://manpages.debian.org/stretch/isc-dhcp-relay/dhcrelay.8.en.html](https://manpages.debian.org/stretch/isc-dhcp-relay/dhcrelay.8.en.html) - Man deban dhcrelay - FR
- [http://www.delafond.org/traducmanfr/man/man8/dhcrelay.8.html](http://www.delafond.org/traducmanfr/man/man8/dhcrelay.8.html) - Man dhcrelay - FR
- [https://linux.die.net/man/8/dhcrelay](https://linux.die.net/man/8/dhcrelay) - Man dhcrelay - EN
- [https://doc.ubuntu-fr.org/isc-dhcp-server#relais_dhcp](https://doc.ubuntu-fr.org/isc-dhcp-server#relais_dhcp) - Doccumentation Ubuntu
