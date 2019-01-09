---
layout: post
title:  "Mémo - Configurer le réseau sur GNU/Linux"
date: 2018-11-07 11:51:00
tags:
- sysadmin
- linux
- memo
- network
description: ''
color: 'rgb(38,50,56)'
cover: '/assets/cover/hello.gif'
---

*Petit mémo suite a des demandes*

Ce mémo servira a la configuration réseau de façon dynamique avec `ip` ou de façon statique sur divers distributions.

## 1 - Configuration dynamique
Anciennement c'était net-tools (ifconfig, route, arp,etc...)qui fournissait les outils nécessaire a la configuration du réseau, mais celui-ci a été remplacer par iproute2, ce qui, a l'heure ou j'écrit ces lignes, rend obselète et deprecated net-tools.

Aujourd'hui c'est le paquet iproute2 qui est installé par défaut sur la majeur parti des distributions Lignux (GNU/Linux).

```console
root@godai:~# ip --help
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { link | address | addrlabel | route | rule | neigh | ntable |
                   tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm |
                   netns | l2tp | fou | macsec | tcp_metrics | token | netconf | ila }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |
                    -h[uman-readable] | -iec |
                    -f[amily] { inet | inet6 | ipx | dnet | mpls | bridge | link } |
                    -4 | -6 | -I | -D | -B | -0 |
                    -l[oops] { maximum-addr-flush-attempts } | -br[ief] |
                    -o[neline] | -t[imestamp] | -ts[hort] | -b[atch] [filename] |
                    -rc[vbuf] [size] | -n[etns] name | -a[ll] | -c[olor]}
```
#### Jouer avec *ip*

- Voir les interfaces et leur(s) IP(s)
```console
root@godai:~# ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 82:3d:9d:89:23:21 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.2/30 brd 10.0.0.3 scope global ens18
       valid_lft forever preferred_lft forever
    inet6 fe80::803d:9dff:fe89:2321/64 scope link 
       valid_lft forever preferred_lft forever
```
- Ajouter/Supprimer une IP
```console
root@godai:~# ip addr add 192.168.10.10/24 dev ens18 # Ajouter
root@godai:~# ip addr del 192.168.10.10/24 dev ens33 # Supprimer
```
- Activer/Desactiver une interface
```console
root@godai:~# ip link set ens33 up # Activer
root@godai:~# ip link set ens33 down # Desactiver
```
- Les passerelle (gateway)
```console
root@godai:~# ip route add default via 192.168.10.1 # Ajout gateway par defaut
root@godai:~# ip route del default via 192.168.10.1 # Supprime cette même gateway
root@godai:~# ip route add 192.168.1.0/24 via 192.168.10.254 dev ens33 # Ajoute une route pour le 192.168.1.0/24 vers 192.168.10.254 pour l'interface ens33
```


## 2 - Configuration statique (platefile)
Faire la configuration dynamique c'est bien pour faire les tests mais ça suffit pas en production...
### Debian

> Dans /etc/network/interfaces
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

auto ens18
iface ens18 inet dhcp
```

On vois 2 interfaces, lo et ens18, la première est l'adresse de loopback, la seconde l'interface ethernet, c'est elle qui nous interesse.

Si on veux configurer notre réseau de plus haut, 192.168.10.10/24 avec une gateway 192.168.10.1
```
.....
auto ens18
iface ens18 inet static
    address 192.168.10.10
    netmask 255.255.255.0
    gateway 192.168.10.1
    dns-nameservers 9.9.9.9 1.1.1.1
.....
```
On a défini que on étais pas en **dhcp** mais en **static**, en paramettre nous avons l'**address**, le **netmask** et la **gateway**.
Il y a un certains nombre de paramettres possible en plus, par exemple **dns-nameservers** qui configure les resolver DNS.

Nous pouvons aussi ajouter des routes supplémentaire, par exemple pour notre route de tout à l'heure :
```
.....
auto ens18
iface ens18 inet static
    address 192.168.10.10
    netmask 255.255.255.0
    gateway 192.168.10.1
    dns-nameservers 9.9.9.9 1.1.1.1
    post-up ip route add 192.168.1.0/24 via 192.168.10.254 dev ens18
    pre-down ip route del 192.168.1.0/24 via 192.168.10.254 dev ens19
.....
```

#### Jouer avec networking

- stop/start de daemon
```
root@godai:~# systemctl stop networking # Stop
root@godai:~# systemctl start networking # Start
root@godai:~# systemctl restart networking # Restart
```
- Down/Up interfaces
```
root@godai:~# ifdown ens18 # Desactiver l'interface
root@godai:~# ifup ens18 # Activer l'interface
```

### CentOS & Co (Dérivées)
*TODO*
### Archlinux
*TODO*

## Conclusion

Avec tout ça vous avez la base.


*Source*
[iproute2](https://linux.die.net/man/8/ip)
[Networking](https://wiki.debian.org/NetworkConfiguration)