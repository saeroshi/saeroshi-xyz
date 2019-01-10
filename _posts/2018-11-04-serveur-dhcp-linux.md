---
layout: post
title:  "Serveur DHCP sous Linux"
date: 2018-11-04 16:42:00
tags:
- sysadmin
- linux
- dhcp
- debian
description: ''
color: 'rgb(38,50,56)'
cover: '/assets/cover/hello.gif'
---

> *Edition 2018-11-18 : Refonte article*

Ici on traitera la configuration d'un serveur DHCP sous Linux.

Le DHCP (Dynamique Host Configuration Protocol) est un protocole qui permet de configurer automatiquement les interfaces réseaux. Il permet de configurer : Adresse IP, Netmask, Gateway, NTP, Domain ...etc.
En bref le DHCP offre l'avantage d'effectuer la configuration réseau complète des machines qui en ont besoin. Ce qui évite d'avoir à le faire soi-même. L'automatisme...

Il existe un certain nombre de services permettant de faire, le plus connu étant [ISC DHCP](https://www.isc.org/downloads/dhcp/), celui que je vais décrire dans cet article. Mais par exemple il existe aussi [KEA DHCP](https://kea.isc.org/).

Prérequis
---------

Il y a quelques éléments à se mettre en tête quand on déploie un serveur DHCP :
- c'est NOUS qui faisons office de DHCP; Il est assez bête de mettre en place un DHCP sur un réseau où il en existe déjà un.
- Sauf cas particulier, la machine qui herbage le serveur DHCP doit avoir une configuration IP statique. Dans tous les cas l'interface qui est utilisée pour le DHCP doit avoir une IP statique.
Il ne sert à rien de suivre *bêtement* cet article sans comprendre, c'est juste contre-productif.

Configurations
--------------

Rapide exemple d'utilisation :

Attribution d'une IPv4 statique à l'interface que vous souhaitez utiliser. Ici nous uiliserons ens18.
```
# ip link set ens18 up
# ip address add 172.17.0.100/24 dev ens18
```
Plus d'info, voir (Configuration réseau)[/2018/11/07/memo-config-reseau-linux.html]

Le fichier de configuration se trouve dans `dhcpd.conf` *(peut différent selon la distribution utilisée)*.

```
default-lease-time 600;
max-lease-time 7200;
option domain-name-servers 1.1.1.1 9.9.9.9;
option domain-name "octo.lan";

subnet 172.17.0.0 netmask 255.255.255.0 {
	range 172.17.0.150 172.17.0.250;
}
```
Si on regarde de plus près ce qu'il se passe : le serveur DHCP assignera aux clients une adresse IP comprise entre 172.17.0.150 et 172.17.0.250 pour une durée de 600 secondes. Les clients peuvent spécifier une période de temps spécifique, dans ce cas, le temps d'allocation maximum est de 7200 secondes.
Le serveur attribuera également :
- un masque de sous réseau à 255.255.255.0
- des serveurs DNS, 172.17.0.2 et 9.9.9.9
- un suffixe DNS octo.lan


##### Reservation d'IP via adresse MAC
```
....

subnet 172.17.0.0 netmask 255.255.255.0 {
	range 172.17.0.150 172.17.0.250;
	host MICHOU-PC{
		hardware ethernet EF:72:6C:9A:29:2E;
		fixed-address 172.17.0.159;
	}
}
```
Nous avons défini pour l'host du nom `MICHOU-PC` avec la MAC `EF:72:6C:9A:29:2E` l'addresse IP `172.17.0.159`.

##### Le loadbalancing

```
failover peer "failover-octo" {
	primary;
	address 172.17.0.100;
	port 519;
	peer address 172.17.0.101;
	peer port 520;
	max-response-delay 60;
	max-unacked-updates 10;
	mclt 3600;
	split 128;
	load balance max seconds 3;
}
subnet 172.17.0.0 netmask 255.255.255.0 {
	pool{
		failover peer "failover-octo";
		range 172.17.0.150 172.17.0.250;
	}
}
```
Dans l'ordre :
- **[primay | secondary];** : Déclare si le serveur est primaire ou secondaire.
- **address** *address* : IP (ou nom DNS) d'écoute.
- **port** *port* : Port d'écoute.
- **peer address** *address* : IP (ou nom DNS) du second serveur.
- **peer port** *port* : Port du second serveur.
- **max-responsive-delay** *secondes* : Nombre de secondes avant de considérer que le second serveur est défaillant.
- **max-unacked-updates** *nombre* : Définit le nombre de BNDUPD qu'il peut envoyer avant de recevoir un BNDACK. 10 convient. 
- **mclt** *secondes* : Le temps que le serveur peut délivrer des contrats DHCP sans contacter son homologue. **A définir uniquement sur le primary**
- **split** *bits (0 à 255)* : Spécifie le taux de charge de chaque serveur, 128 étant la moitié (50/50).
- **load balance max seconds** *secondes* : Défini le nombre de secondes avant que le serveur prenne le relais par rapport à l'autre.
Il y en a d'autres, mais je ne les utilisent pas.

- **pool** *{Contient les infos du pool}*
- **failover peer** *Nom du peer* : Définit quel failover il utilisera.  

On peut voir que ce qui change, est uniquement le couple `address/port` et `peer address/peer port`, donc le serveur primary se connecte sur le port 520 du serveur secondary, et inversement, le serveur secondary se connecte sur le port 519 du serveur primary.

##### Autres

Il faut voir qu'il y a de nombreuses possibilités de configurations, dans mes configs je n'utilise pas le PXE, le NTP,...
Pour plus d'exemple de configuration je vous redirige vers un man [dhcpd.conf](https://linux.die.net/man/5/dhcpd.conf).

Il se peut que mes explications ne soient pas exactes.

Exemple avec Debian
-------------------

Donnons un exemple avec Debian 9 (Stretch).
Voici le cahier des charges :
- Pool DHCP de 172.31.1.150 à 172.31.1.250
- DNS 172.31.1.1 et 9.9.9.9
- Gateway 172.31.1.1
- suffix DNS à octo.lan
- Loadbalancing à 50/50
- Une réservation IP pour le Serveur AD-1 (MAC : 68-12-93-14-37-D6)

Sur debian le paquet fournissent DHCPD est `isc-dhcp-server`.

|Nom|Fonction|IPs|Paquets
|-|-|-|-|
|octo-dhcp-1|Serveur DHCP Primary|172.31.1.100|isc-dhcp-server|
|octo-dhcp-2|Serveur DHCP Secondary|172.31.1.101|isc-dhcp-server|
|octo-client|Utilisateur|dhcp|*Aucun*|
> Pour rappel, pour installer un paquet : apt-get install **isc-dhcp-server**

- Sur le `Primary` :
> /etc/dhcp/dhcpd.conf
```
default-lease-time 600;
max-lease-time 7200;

failover peer "failover-octo" {
	primary;
	address 172.31.1.100;
	port 540;
	peer address 172.31.1.101;
	peer port 541;
	max-response-delay 60;
	max-unacked-updates 10;
	mclt 3600;
	split 128;
	load balance max seconds 3;
}

subnet 172.31.1.0 netmask 255.255.255.0 {
	option domain-name-servers 172.31.1.1 9.9.9.9;
	option domain-name "octo.lan";
	option routers 172.31.1.1
	pool{
		failover peer "failover-octo";
		range 172.31.1.150  172.31.1.250;
		host AD-1{
			hardware ethernet 68:12:93:14:37:D6;
			fixed-address 172.17.0.159;
		}
	}
}
```
- Sur le `Secondary` :
> /etc/dhcp/dhcpd.conf
```
default-lease-time 600;
max-lease-time 7200;

failover peer "failover-octo" {
	secondary;
	address 172.31.1.101;
	port 541;
	peer address 172.31.1.100;
	peer port 540;
	max-response-delay 60;
	max-unacked-updates 10;
	split 128;
	load balance max seconds 3;
}

subnet 172.31.1.0 netmask 255.255.255.0 {
	option domain-name-servers 172.31.1.1 9.9.9.9;
	option domain-name "octo.lan";
	option routers 172.31.1.1
	pool{
		failover peer "failover-octo";
		range 172.31.1.150  172.31.1.250;
		host AD-1{
			hardware ethernet 68:12:93:14:37:D6;
			fixed-address 172.17.0.159;
		}
	}
}
```
- Commun au `Primary` et `Secondary`
> /etc/default/isc-dhcp-server
```
DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
INTERFACESv4="ens33"
```
Dans ce fichier de configuration il faut décommenter **DHCPDv4_CONF** et **INTERFACESv4**
- DHCPDv4_CONF correspond au chemin d'accès au fichier de configuration de DHCPD.
- INTERFACESv4 correspond à l'interface qui écoute les trames *DHCP DISCOVER*.

Le service ce nomme `isc-dhcp-server`, donc : `systemctl restart isc-dhcp-server`

Conclusion
----------

Pour finir, la dernière chose est qu'il faut savoir se documenter. Je vous laisse quelques docs supplémentaires.


*Source :*
- [https://www.isc.org/dhcp-manual-pages/](https://www.isc.org/dhcp-manual-pages/) - Doccumentation Officiel
- [http://www.delafond.org/traducmanfr/man/man8/dhcpd.8.html](http://www.delafond.org/traducmanfr/man/man8/dhcpd.8.html) - Man pour dhcpd - FR
- [http://www.delafond.org/traducmanfr/man/man5/dhcpd.conf.5.html](http://www.delafond.org/traducmanfr/man/man5/dhcpd.conf.5.html) - Man pour dhcpd.conf - FR
- [https://linux.die.net/man/5/dhcpd.conf](https://linux.die.net/man/5/dhcpd.conf) - Man pour dhcpd.conf - EN
- [https://wiki.debian.org/fr/DHCP_Server](https://wiki.debian.org/fr/DHCP_Server) - Documentation Debian
- [https://doc.ubuntu-fr.org/isc-dhcp-server](https://doc.ubuntu-fr.org/isc-dhcp-server) - Documentation ubuntu
