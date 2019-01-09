---
layout: post
title:  "Les clients DHCP"
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

Halloa,

Je doit vous dire un truc, j'étais en train d'écrire le petit mémo pour configurer un serveur DHCP sous Linux mais je me suis rendu compte que c'est pas forcement évident de savoir comment renouvelé un bail DHCP (j'expliquerai un jour comment ça fonctionne dans les détails ... Flemme).Voilà donc les commandes basique pour jouer "basiquement" avec le client DHCP =D.

## Sous Linux :
### DESCRIPTION
[dhclient](https://manpages.debian.org/stretch/isc-dhcp-client/dhclient.8.en.html) est le client DHCP de l'Internet Software Consortium, dhclient, fournit un moyen de configurer une ou plusieurs interfaces réseaux en utilisant le protocole DHCP, le protocole BOOTP, ou si ces protocoles échouent, en assignant une adresse statique.
Cela n'est pas le seul outil, mais c'est le plus courament utilisé.

#### Utilisation
`dhclient [ -4 | -6 ] [ -S ] [ -N [ -N... ] ] [ -T [ -T... ] ] [ -P [ -P... ] ] -R ] [ -i ] [ -I ] [ -4o6 port ] [ -D LL|LLT ] [ -p port-number ] [ -d ] [ -df duid-lease-file ] [ -e VAR=value ] [ -q ] [ -1 ] [ -r | -x ] [ -lf lease-file ] [ -pf pid-file ] [ --no-pid ] [ -cf config-file ] [ -sf script-file ] [ -s server-addr ] [ -g relay ] [ -n ] [ -nw ] [ -w ] [ -v ] [ --version ] [ if0 [ ...ifN ] ]`

### En bref
Je vous laisser aller regarder le [man](https://manpages.debian.org/stretch/isc-dhcp-client/dhclient.8.en.html) mais dans l'ensemble voici les commandes que j'utilise :
- `dhclient -r -v` : Permet de "Release the current lease", c'est à dire de liberer le bail en cours.
- `dhclient -v` : Permet de demandé un nouveau bail.
- `ip addr` : Obtenir les informations réseau actuelle.
Vous remarquerait que j'ajoute toujours `-v` qui permet d'activer le mode verbose, c'est a dire que dhclient parle un peu plus.


-------------

## Sous Windows
### DESCRIPTION
[ipconfig](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig) est un utilitaire en ligne de commande accessible sous Windows depuis la version NT du système et toujours disponible dans Windows 10. Il permet d’obtenir diverses informations liées aux cartes réseau et offre un contrôle dans une certaine limite sur les connexions TCP/IP actives. La fonction Ipconfig peut être exécutée seule ou avec des paramètres. Ceux les plus importants sont : `ipconfig /all`, `ipconfig /release` et `ipconfig /renew`.
### Syntax
`ipconfig [/allcompartments] [/all] [/renew [<Adapter>]] [/release [<Adapter>]] [/renew6[<Adapter>]] [/release6 [<Adapter>]] [/flushdns] [/displaydns] [/registerdns] [/showclassid <Adapter>] [/setclassid <Adapter> [<ClassID>]]`
### EN BREF
Vous pouvez allez voir la [doc](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig) aussi mais voici les commandes les plus importante :
- `ipconfig /release` : Permet de "Release the current lease", c'est à dire de liberer le bail en cours.
- `ipconfig /renew` : Permet de demandé un nouveau bail.
- `ipconfig /all` : Obtenir les informations réseau actuelle.
Je n'ai pas spécifié de commande de verbose car ipconfig est dejo assez verbeux.