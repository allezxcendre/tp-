 Mettez en place une configuration réseau fonctionnelle entre les deux machines :
 
PS C:\windows\system32> netsh int ip set address "Ethernet" address=192.168.137.1 mask=255.255.252.0
 
 C:\Users\titim>ipconfig /all
  
  Carte Ethernet Ethernet :

   Suffixe DNS propre à la connexion. . . :
   Adresse IPv6 de liaison locale. . . . .: fe80::e1c2:fd45:f4f0:e17d%19
   Adresse IPv4. . . . . . . . . . . . . .: 192.168.137.1
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Passerelle par défaut. . . . . . . . . :

  
  Adresse IPv4. . . . . . . . . . . . . .: 192.168.137.1(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   l'adresse de réseau : 192.168.136.0
   l'adresse de Broadcast : 192.168.139.255
   
   Prouvez que la connexion est fonctionnelle entre les deux machines
   
   PS C:\windows\system32> ping 192.168.137.2

Envoi d’une requête 'Ping'  192.168.137.2 avec 32 octets de données :
Réponse de 192.168.137.2 : octets=32 temps=273 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=1 ms TTL=128

Statistiques Ping pour 192.168.137.2:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 273ms, Moyenne = 69ms
PS C:\windows\system32>

Wireshark it :

ping envoie en ICMP en type 8 = Echo Request
pong envoie en ICMP en type 0 = Echo Reply

[mon pcap](https://github.com/allezxcendre/tp-/blob/main/ping%20et%20pong.pcapng)
Check the ARP table :

PS C:\Users\titim\tp-> arp -a

Interface : 192.168.137.1 --- 0x13
  Adresse Internet      Adresse physique      Type
  **192.168.137.2         b0-25-aa-47-c7-a4**     dynamique

on retrouve avec son ip son adresse Physique

Et on fait pareil avec l'Ip de ynov.com

PS C:\Users\titim\tp-> arp -a

Interface : 10.33.17.18 --- 0x14
  Adresse Internet      Adresse physique      Type
  10.33.18.221          78-4f-43-87-f5-11     dynamique
  **10.33.19.254          00-c0-e7-e0-04-4e**     dynamique


 Manipuler la table ARP:
 
 
PS C:\Users\titim\tp-> arp -d
PS C:\Users\titim\tp-> arp -a

Interface : 192.168.56.1 --- 0x5
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 192.168.230.1 --- 0x12
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.137.1 --- 0x13
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.17.18 --- 0x14
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.75.1 --- 0x18
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
PS C:\Users\titim\tp-> ping 192.168.137.2

Envoi d’une requête 'Ping'  192.168.137.2 avec 32 octets de données :
Réponse de 192.168.137.2 : octets=32 temps=65 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 192.168.137.2:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 65ms, Moyenne = 17ms
PS C:\Users\titim\tp-> ping ynov.com

Envoi d’une requête 'ping' sur ynov.com [104.26.11.233] avec 32 octets de données :
Réponse de 104.26.11.233 : octets=32 temps=22 ms TTL=55
Réponse de 104.26.11.233 : octets=32 temps=23 ms TTL=55
Réponse de 104.26.11.233 : octets=32 temps=22 ms TTL=55
Réponse de 104.26.11.233 : octets=32 temps=21 ms TTL=55

Statistiques Ping pour 104.26.11.233:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 21ms, Maximum = 23ms, Moyenne = 22ms
PS C:\Users\titim\tp-> arp -a

Interface : 192.168.56.1 --- 0x5
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 192.168.230.1 --- 0x12
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.137.1 --- 0x13
  Adresse Internet      Adresse physique      Type
  192.168.137.2         b0-25-aa-47-c7-a4     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.33.17.18 --- 0x14
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.75.1 --- 0x18
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
PS C:\Users\titim\tp->

Wireshark it:

[pcap echange de ping](https://github.com/allezxcendre/tp-/blob/main/PING%20ARP.pcap)



 Wireshark it :
 
 [pcap DORA](https://github.com/allezxcendre/tp-/blob/main/okays.pcap)
 
 une ip a utliser :10.33.17.18
 ip de la passerelle :10.33.19.254
serveur dns :8.8.8.8




















































