# Mise en place

### Combien y a-t-il d'adresses disponibles dans un `/24` ?

Il y a 256 adresses au total dont 254 disponibles. X.X.X.0 étant l'adresse réseau et X.X.X.255 l'adresse de broadcast.

### Combien y a-t-il d'adresses disponibles dans un `/30` ?

Il y a 2 adresses disponibles seulement.

### Quelle est l'utilité d'un `/30` ?

Un réseau en /30 permet de déclarer 2 adresses donc cela évoque de la clarté mais aussi une sécurité puisque il n'y aura que deux cartes sur ce réseau.

## Cartes réseaux
 Carte host-only ens34 (10.1.1.2/24) : 
```bash
[lukihd@client1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-ens34
TYPE=Ethernet
BOOTPROTO=static
NAME=ens34
DEVICE=ens34
ONBOOT=yes
IPADDR=10.1.1.2
NETMASK=255.255.255.0
```
Carte host-only ens38 (10.1.2.2/29) car /30 n'est pas dispo sur vmware 
```bash
[lukihd@client1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-ens38
TYPE=Ethernet
BOOTPROTO=static
NAME=ens38
DEVICE=ens38
ONBOOT=yes
IPADDR=10.1.2.2
NETMASK=255.255.255.248
```

## Connexion SSH

```bash
ssh lukihd@10.1.1.2
lukihd@10.1.1.2's password:
Last login: Thu Feb 14 15:41:00 2019 from client1
[lukihd@client1 ~]$
```

## Définir un nom de domaine
```bash
# définition du hostname 
[lukihd@client1 ~]$ sudo hostname client1.tp1.b2
# défintion permanente du hostname 
[lukihd@client1 ~]$ echo 'client1.tp1.b2' | sudo tee /etc/hostname
```

## Complétion du fichier hosts

```bash
[lukihd@client1 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.1.1.1    client1   client1.tp1.b2
10.1.2.1    client1   client1.tp1.b2
```

## Fonctionnement cartes réseaux

Carte NAT ens33
```bash
[lukihd@client1 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
Carte host-only ens34 (10.1.1.2/24)
```bash
[lukihd@client1 ~]$ ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=128 time=0.322 ms
64 bytes from 10.1.1.1: icmp_seq=2 ttl=128 time=0.150 ms
64 bytes from 10.1.1.1: icmp_seq=3 ttl=128 time=0.174 ms
^C
--- 10.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.150/0.215/0.322/0.077 ms
```
Carte host-only ens38 (10.1.2.2/29)
```bash
[lukihd@client1 ~]$ ping 10.1.2.1
PING 10.1.2.1 (10.1.2.1) 56(84) bytes of data.
64 bytes from 10.1.2.1: icmp_seq=1 ttl=128 time=0.338 ms
64 bytes from 10.1.2.1: icmp_seq=2 ttl=128 time=0.227 ms
64 bytes from 10.1.2.1: icmp_seq=3 ttl=128 time=0.187 ms
^C
--- 10.1.2.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.187/0.250/0.338/0.066 ms
```

### Expliquer le retour que vous obtenez (code retour HTTP `301`)

Le code 301 permet une redirection permanente vers une nouvelle page / domaine.

# Basics

## Explications des commande des routes

### Supprimer la route par défaut
```bash
[lukihd@client1 ~]$ sudo ip route del default
```

### Afficher les routes
```bash
[lukihd@client1 ~]$ ip r s
# route vers la Host-only ens34
10.1.1.0/24 dev ens34 proto kernel scope link src 10.1.1.2
# route vers la Host-only ens38
10.1.2.0/29 dev ens38 proto kernel scope link src 10.1.2.2
# route vers la Nat ens33
192.168.109.0/24 dev ens33 proto kernel scope link src 192.168.109.139
```

### Supprimer une route
La commande pour supprimer : 
```bash
[lukihd@client1 ~]$ sudo ip route del 10.1.1.0/24
Connection reset by 10.1.1.2 port 22
```
Le test :
```bash
PS C:\Users\lucas> ssh lukihd@10.1.1.2
ssh: connect to host 10.1.1.2 port 22: Connection timed out
```

### Remettre la route

```bash
[lukihd@client1 ~]$ ping 10.1.1.2
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.035 ms
```
## Table ARP

### Expliquer chaque ligne :
```bash
[lukihd@client1 ~]$ ip n s
# adresse en cache vers la carte ens33, elle est bientôt plus valide
192.168.109.2 dev ens33 lladdr 00:50:56:f5:57:a6 STALE
# adresse en cache vers la carte ens38, elle est encore valide
10.1.2.1 dev ens38 lladdr 00:50:56:c0:00:05 REACHABLE
```

### Vider la table
```bash
[lukihd@client1 ~]$ sudo ip neigh flush all
[lukihd@client1 ~]$ ip n s
10.1.2.1 dev ens38 lladdr 00:50:56:c0:00:05 REACHABLE
```

### Actualisation de la table
```bash
[lukihd@client1 ~]$ ip n s
10.1.2.1 dev ens38 lladdr 00:50:56:c0:00:05 REACHABLE
# cette nouvelle ligne est celle du dernier ping qui a était fait
10.1.1.1 dev ens34 lladdr 00:50:56:c0:00:04 REACHABLE
```

## Capture des trames
Après avoir récupérer les trames on les transmets à l'hôte avec la commande :
`scp lukihd@10.1.2.2:/home/lukihd/ping.pcap .\Downloads\`

# Communication simple entre deux machines

##
