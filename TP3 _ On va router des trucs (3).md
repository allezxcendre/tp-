# TP3 : On va router des trucs

Au menu de ce TP, on va revoir un peu ARP et IP histoire de **se mettre en jambes dans un environnement avec des VMs**.

Puis on mettra en place **un routage simple, pour permettre à deux LANs de communiquer**.

![Reboot the router](./pics/reboot.jpeg)

## Sommaire

- [TP3 : On va router des trucs](#tp3--on-va-router-des-trucs)
  - [Sommaire](#sommaire)
  - [0. Prérequis](#0-prérequis)
  - [I. ARP](#i-arp)
    - [1. Echange ARP](#1-echange-arp)
    - [2. Analyse de trames](#2-analyse-de-trames)
  - [II. Routage](#ii-routage)
    - [1. Mise en place du routage](#1-mise-en-place-du-routage)
    - [2. Analyse de trames](#2-analyse-de-trames-1)
    - [3. Accès internet](#3-accès-internet)
  - [III. DHCP](#iii-dhcp)
    - [1. Mise en place du serveur DHCP](#1-mise-en-place-du-serveur-dhcp)
    - [2. Analyse de trames](#2-analyse-de-trames-2)

## 0. Prérequis

➜ Pour ce TP, on va se servir de VMs Rocky Linux. 1Go RAM c'est large large. Vous pouvez redescendre la mémoire vidéo aussi.  

➜ Vous aurez besoin de deux réseaux host-only dans VirtualBox :

- un premier réseau `10.3.1.0/24`
- le second `10.3.2.0/24`
- **vous devrez désactiver le DHCP de votre hyperviseur (VirtualBox) et définir les IPs de vos VMs de façon statique**

➜ Les firewalls de vos VMs doivent **toujours** être actifs (et donc correctement configurés).

➜ **Si vous voyez le p'tit pote 🦈 c'est qu'il y a un PCAP à produire et à mettre dans votre dépôt git de rendu.**

## I. ARP

Première partie simple, on va avoir besoin de 2 VMs.

| Machine  | `10.3.1.0/24` |
|----------|---------------|
| `john`   | `10.3.1.11`   |
| `marcel` | `10.3.1.12`   |

schema
   john               marcel
  ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     │
  └─────┘    └───┘    └─────┘


> Référez-vous au [mémo Réseau Rocky](../../cours/memo/rocky_network.md) pour connaître les commandes nécessaire à la réalisation de cette partie.

### 1. Echange ARP

🌞**Générer des requêtes ARP**

- effectuer un `ping` d'une machine à l'autre
- [user@localhost ~]$ ip neigh show
[user@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.245 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.372 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=0.303 ms
^C
--- 10.3.1.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2082ms
rtt min/avg/max/mdev = 0.245/0.306/0.372/0.051 ms```

- observer les tables ARP des deux machines
- [user@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:15 REACHABLE
10.3.1.12 dev enp0s8 lladdr 08:00:27:da:cf:e7 STALE```

- repérer l'adresse MAC de `john` dans la table ARP de `marcel` et vice-versa
-    [user@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:15 REACHABLE
10.3.1.11 dev enp0s8 lladdr **08:00:27:9d:e8:c7** STALE
et
- [user@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:15 REACHABLE
10.3.1.12 dev enp0s8 lladdr **08:00:27:da:cf:e7** STALE```
- prouvez que l'info est correcte (que l'adresse MAC que vous voyez dans la table est bien celle de la machine correspondante)
- une commande pour voir la MAC de `marcel` dans la table ARP de `john`
 
[user@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:15 DELAY
10.3.1.12 dev enp0s8 lladdr 08:00:27:da:cf:e7 STALE

  - et une commande pour afficher la MAC de `marcel`, depuis `marcel`
  - [user@localhost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:da:cf:e7 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feda:cfe7/64 scope link
       valid_lft forever preferred_lft forever````

### 2. Analyse de trames

🌞**Analyse de trames**

- utilisez la commande `tcpdump` pour réaliser une capture de trame

[user@localhost ~]$ sudo tcpdump
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
11:30:05.068136 IP localhost.localdomain.ssh > 10.3.1.1.51076: Flags [P.], seq 2709476692:2709476752, ack 2351087599, win 501, length 60
11:30:05.068242 IP localhost.localdomain.ssh > 10.3.1.1.51076: Flags [P.], seq 60:204, ack 1, win 501, length 144
11:30:05.068320 IP 10.3.1.1.51076 > localhost.localdomain.ssh: Flags [.], ack 204, win 8190, length 0
11:30:05.068338 IP localhost.localdomain.ssh > 10.3.1.1.51076: Flags [P.], seq 204:240, ack 1, win 501, length 36
11:30:05.068363 IP localhost.localdomain.ssh > 10.3.1.1.51076: Flags [P.], seq 240:300, ack 1, win 501, length 60
11:30:05.068443 IP 10.3.1.1.51076 > localhost.localdomain.ssh: Flags [.], ack 300, win 8190, length 0
11:30:05.068452 IP localhost.localdomain.ssh > 10.3.1.1.51076: Flags [P.], seq 300:436, ack 1, win 501, length 136
11:30:05.068513 IP localhost.localdomain.ssh > 10.3.1.1.51076: Flags [P.], seq 436:472, ack 1, win 501, length 36
11:30:05.068668 IP 10.3.1.1.51076 > localhost.localdomain.ssh: Flags [.], ack 472, win 8189, length 0
^C
9 packets captured
19 packets received by filter
0 packets dropped by kernel````


capture wireshark :[https://github.com/allezxcendre/tp-/blob/main/tp3_arp.pcapng](https://)
- videz vos tables ARP, sur les deux machines, puis effectuez un `ping`
[user@localhost ~]$ sudo ip neigh flush all
[user@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.435 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.623 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=0.582 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=64 time=0.598 ms
^C
--- 10.3.1.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3051ms
rtt min/avg/max/mdev = 0.435/0.559/0.623/0.073 ms


🦈 **Capture réseau `tp3_arp.pcapng`** qui contient un ARP request et un ARP reply

> **Si vous ne savez pas comment récupérer votre fichier `.pcapng`** sur votre hôte afin de l'ouvrir dans Wireshark, et me le livrer en rendu, demandez-moi.

## II. Routage

Vous aurez besoin de 3 VMs pour cette partie. **Réutilisez les deux VMs précédentes.**

| Machine  | `10.3.1.0/24` | `10.3.2.0/24` |
|----------|---------------|---------------|
| `router` | `10.3.1.254`  | `10.3.2.254`  |
| `john`   | `10.3.1.11`   | no            |
| `marcel` | no            | `10.3.2.12`   |

> Je les appelés `marcel` et `john` PASKON EN A MAR des noms nuls en réseau 🌻

schema
   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘


### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

> Cette étape est nécessaire car Rocky Linux c'est pas un OS dédié au routage par défaut. Ce n'est bien évidemment une opération qui n'est pas nécessaire sur un équipement routeur dédié comme du matériel Cisco.

[user@localhost ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
[sudo] password for user:
Warning: ALREADY_ENABLED: masquerade
success
[user@localhost ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8 enp0s9
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: yes
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:



🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

- il faut taper une commande `ip route add` pour cela, voir mémo
- [user@localhost sysconfig]$ sudo ip route add default via 10.3.1.254 dev enp0s8
[sudo] password for user:
[user@localhost sysconfig]$ ip route
default via 10.3.1.254 dev enp0s8
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
10.3.1.0/24 dev enp0s8 proto kernel scope link src 10.3.1.254 metric 101
[user@localhost sysconfig]$```
- il faut ajouter une seule route des deux côtés
- une fois les routes en place, vérifiez avec un `ping` que les deux machines peuvent se joindre
[user@john ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=2.26 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=2.00 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.98 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=1.12 ms

--- 10.3.2.12 ping statistics ---
[user@marcel ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=1.70 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=1.84 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=63 time=1.94 ms

--- 10.3.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.702/1.825/1.938/0.096 ms
![THE SIZE](./pics/thesize.png)

### 2. Analyse de trames

🌞**Analyse des échanges ARP**

- videz les tables ARP des trois noeuds
- [user@router ~]$ sudo ip neigh flush all
[sudo] password for user:
[user@john ~]$ sudo ip neigh flush all
[sudo] password for user:
[user@marcel ~]$ sudo ip neigh flush all
[sudo] password for user:```
- effectuez un `ping` de `john` vers `marcel`
  - **le `tcpdump` doit être lancé sur la machine `john`**
  - [user@john ~]$ sudo tcpdump -i enp0s8 -w arp_john.pcapng
[sudo] password for orealyz:
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 263255 bytes
ping 10.3.2.12
^C61 packets captured
62 packets received by filter
0 packets dropped by kernel
- essayez de déduire un les échanges ARP qui ont eu lieu
  - en regardant la capture et/ou les tables ARP de tout le monde
- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur `marcel`
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange
- [user@marcel ~]$ sudo ip neigh flush all
- [user@john ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=2.26 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=2.00 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.98 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=1.12 ms
^C
--- 10.3.2.12 ping statistics ---
[user@marcel ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=1.70 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=1.84 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=63 time=1.94 ms

Par exemple (copiez-collez ce tableau ce sera le plus simple) :

| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | x         | `marcel` `AA:BB:CC:DD:EE` | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         | ?                       | x              | `marcel` `AA:BB:CC:DD:EE`    |
| ...   | ...         | ...       | ...                     |                |                            |
| ?     | Ping        | ?         | ?                       | ?              | ?                          |
| ?     | Pong        | ?         | ?                       | ?              | ?                          |

> Vous pourriez, par curiosité, lancer la capture sur `marcel` aussi, pour voir l'échange qu'il a effectué de son côté.

🦈 **Capture réseau `tp3_routage_marcel.pcapng`**

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

- ajoutez une carte NAT en 3ème inteface sur le `router` pour qu'il ait un accès internet
- ajoutez une route par défaut à `john` et `marcel`
  - vérifiez que vous avez accès internet avec un `ping`
  - le `ping` doit être vers une IP, PAS un nom de domaine
- donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser
  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`
  - puis avec un `ping` vers un nom de domaine

🌞**Analyse de trames**

- effectuez un `ping 8.8.8.8` depuis `john`
- capturez le ping depuis `john` avec `tcpdump`
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |     |
|-------|------------|--------------------|-------------------------|----------------|-----------------|-----|
| 1     | ping       | `marcel` `10.3.1.12` | `marcel` `AA:BB:CC:DD:EE` | `8.8.8.8`      | ?               |     |
| 2     | pong       | ...                | ...                     | ...            | ...             | ... |

🦈 **Capture réseau `tp3_routage_internet.pcapng`**

## III. DHCP

On reprend la config précédente, et on ajoutera à la fin de cette partie une 4ème machine pour effectuer des tests.

| Machine  | `10.3.1.0/24`      | `10.3.2.0/24` |
|----------|--------------------|---------------|
| `router` | `10.3.1.254`       | `10.3.2.254`  |
| `john`   | `10.3.1.11`        | no            |
| `bob`    | PAS POUR LE MOMENT | no            |
| `marcel` | no                 | `10.3.2.12`   |

```schema
   john               router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └─┬─┘    └─────┘    └───┘    └─────┘
   dhcp        │
  ┌─────┐      │
  │     │      │
  │     ├──────┘
  └─────┘
```

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

- installation du serveur sur `john`
- créer une machine `bob`
- faites lui récupérer une IP en DHCP à l'aide de votre serveur
  - utilisez le mémo toujours, section "Définir une IP dynamique (DHCP)"


🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :
  - une route par défaut
  - un serveur DNS à utiliser
- récupérez de nouveau une IP en DHCP sur `bob` pour tester :
  - `bob` doit avoir une IP
    - vérifier avec une commande qu'il a récupéré son IP
    - vérifier qu'il peut `ping` sa passerelle
  - il doit avoir une route par défaut
    - vérifier la présence de la route avec une commande
    - vérifier que la route fonctionne avec un `ping` vers une IP
  - il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms
    - vérifier avec la commande `dig` que ça fonctionne
    - vérifier un `ping` vers un nom de domaine

### 2. Analyse de trames

🌞**Analyse de trames**

- lancer une capture à l'aide de `tcpdump` afin de capturer un échange DHCP
- demander une nouvelle IP afin de générer un échange DHCP
- exportez le fichier `.pcapng`
- repérez, dans les trames DHCP observées dans Wireshark, les infos que votre serveur a fourni au client
  - l'IP fournie au client
  - l'adresse IP de la passerelle
  - l'adresse du serveur DNS que vous proposez au client

🦈 **Capture réseau `tp3_dhcp.pcapng`**





[user@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:15 REACHABLE
10.3.1.12 dev enp0s8 lladdr 08:00:27:da:cf:e7 STALE
