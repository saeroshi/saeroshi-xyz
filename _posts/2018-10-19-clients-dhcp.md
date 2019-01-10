---
layout: post
title:  "Clients DHCP"
date: 2018-10-19 16:35:00
tags:
- sysadmin
- linux
- dhcp
- windows
description: ''
color: 'rgb(38,50,56)'
cover: '/assets/cover/hello.gif'
---

Sur les divers OS existant il existe bon nombre de client (agent) DHCP, parfois c'est un outil complet, NetworkManager par exemple, parfois il faudra demander l'ouverture du bail DHCP manuellement, voici donc quelques outils.

## Sous Linux :
### Description
[dhclient](https://manpages.debian.org/stretch/isc-dhcp-client/dhclient.8.en.html) est le client DHCP de l'Internet Software Consortium, dhclient, fournit un moyen de configurer une ou plusieurs interfaces réseaux en utilisant le protocole DHCP, le protocole BOOTP, ou si ces protocoles échouent, en assignant une adresse statique.
C'est pas le seul outil, mais c'est le plus courament utilisé.

#### Utilisation
`dhclient [ -4 | -6 ] [ -S ] [ -N [ -N... ] ] [ -T [ -T... ] ] [ -P [ -P... ] ] -R ] [ -i ] [ -I ] [ -4o6 port ] [ -D LL|LLT ] [ -p port-number ] [ -d ] [ -df duid-lease-file ] [ -e VAR=value ] [ -q ] [ -1 ] [ -r | -x ] [ -lf lease-file ] [ -pf pid-file ] [ --no-pid ] [ -cf config-file ] [ -sf script-file ] [ -s server-addr ] [ -g relay ] [ -n ] [ -nw ] [ -w ] [ -v ] [ --version ] [ if0 [ ...ifN ] ]`

### En bref
Je vous laisse aller regarder le [man](https://manpages.debian.org/stretch/isc-dhcp-client/dhclient.8.en.html) mais dans l'ensemble voici les commandes que j'utilise :
- `dhclient -r [interface]` : Permet de libérer le bail en cours sur l'interface sélectionné.
- `dhclient [interface]` : Permet de demander un nouveau bail sur l'interface sélectionné.
> *NOTE : Vous pouvez ajouter l'option `-v` pour ajouter de la verbosité*

-------------

## Sous Windows :
### Description
[ipconfig](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig) est un utilitaire en ligne de commande accessible sous Windows depuis la version NT du système et toujours disponible dans les dernières version de Windows. Il permet d’obtenir diverses informations liées aux cartes réseau et offre un contrôle dans une certaine limite sur les connexions TCP/IP actives. La commande ipconfig peut être exécutée seule ou avec des paramètres.
Ceux les plus importants sont : `ipconfig /all`, `ipconfig /release` et `ipconfig /renew`.

### Utilisation
`ipconfig [/allcompartments] [/all] [/renew [<Adapter>]] [/release [<Adapter>]] [/renew6[<Adapter>]] [/release6 [<Adapter>]] [/flushdns] [/displaydns] [/registerdns] [/showclassid <Adapter>] [/setclassid <Adapter> [<ClassID>]]`
### En Bref
Vous pouvez aller voir la [doc](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig) aussi mais voici les commandes les plus importante :
- `ipconfig /release [interface]` : Permet de libérer le bail en cours sur l'interface sélectionné.
- `ipconfig /renew [interface]` : Permet de demander un nouveau bail sur l'interface sélectionné.
> *NOTE : ipconfig étant déjà assez verbeux par defaut, aucune option de verbosité n'est disponible.*