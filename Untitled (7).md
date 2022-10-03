I. Exploration locale en solo

1. Affichage d'informations sur la pile TCP/IP locale

je fait ipconfig /all dans un terminale powershell:

Carte réseau sans fil Wi-Fi :

adresse MAC : 2C-6D-C1-37-81-FB
IP : 10.33.16.167

carte Ethernet Ethernet : 

adresse Mac : C0-18-03-59-30-32
on ne trouve pas d'adresse IP car on est pas connecté

j'affiche mon gateway avec la commande ipconfig :

gateway corespond a la passerelle par défaut donc :

10.33.19.254 

Déterminer la MAC de la passerelle :

Je fait Arp -a 

et je trouve mon adresse passerelle 10.33.19.254 qui a pour adresse physique 00-c0-e7-e0-04-4e

![](https://i.imgur.com/pLsvxNv.png)

2. Modifications des informations

A. Modification d'adresse IP (part 1)

j'ai modifié mon adresse IP et je fait ipconfig /all :

3. Modification d'adresse IP
```
 ipconfig /all
Carte réseau sans fil Wi-Fi :

   Suffixe DNS propre à la connexion. . . :
   Description. . . . . . . . . . . . . . : Intel(R) Wi-Fi 6 AX201 160MHz
   Adresse physique . . . . . . . . . . . : 2C-6D-C1-37-81-FB
   DHCP activé. . . . . . . . . . . . . . : Non
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::f5ca:7b36:a36a:f8cd%19(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.250(en double)
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Passerelle par défaut. . . . . . . . . : 10.33.19.254
   IAID DHCPv6 . . . . . . . . . . . : 170683841
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-29-6A-1A-C4-C0-18-03-59-30-32
   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       8.8.4.4
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé
   
   
   
   Carte Ethernet Ethernet :

   Suffixe DNS propre à la connexion. . . :
   Description. . . . . . . . . . . . . . : Realtek Gaming GbE Family Controller
   Adresse physique . . . . . . . . . . . : C0-18-03-59-30-32
   DHCP activé. . . . . . . . . . . . . . : Non
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::e1c2:fd45:f4f0:e17d%18(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.251(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Passerelle par défaut. . . . . . . . . :
   IAID DHCPv6 . . . . . . . . . . . : 163584003
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-29-6A-1A-C4-C0-18-03-59-30-32
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé
```

Vérifier à l'aide d'une commande que votre IP a bien été changée :

```
PS C:\Users\titim> ping  10.10.10.250

Envoi d’une requête 'Ping'  10.10.10.250 avec 32 octets de données :
Réponse de 10.10.10.250 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.250 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.250 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.250 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 10.10.10.250:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms
```
Déterminer l'adresse MAC de votre correspondant :
```
arp -a
Interface : 10.10.10.251 --- 0x12
  Adresse Internet      Adresse physique      Type
  10.10.10.250          b0-25-aa-47-c7-a4     dynamique
  10.10.10.255          ff-ff-ff-ff-ff-ff     statique
  169.254.139.228       b0-25-aa-47-c7-a4     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique![](https://i.imgur.com/vcmGB1k.png)
```
4. Utilisation d'un des deux comme gateway
![](https://i.imgur.com/6GdBA4S.png)



Tester l'accès internet :
```
PS C:\Users\titim> ping 1.1.1.1

Envoi d’une requête 'Ping'  1.1.1.1 avec 32 octets de données :
Réponse de 1.1.1.1 : octets=32 temps=20 ms TTL=55
Réponse de 1.1.1.1 : octets=32 temps=20 ms TTL=55
Réponse de 1.1.1.1 : octets=32 temps=21 ms TTL=55
Réponse de 1.1.1.1 : octets=32 temps=20 ms TTL=55

Statistiques Ping pour 1.1.1.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 20ms, Maximum
```

 Prouver que la connexion Internet passe bien par l'autre PC :
 
 PS C:\Users\titim> tracert 192.168.137.2

Détermination de l’itinéraire vers DESKTOP-0JKFI2T [192.168.137.2]
avec un maximum de 30 sauts :

  1     2 ms     1 ms     1 ms  DESKTOP-0JKFI2T [192.168.137.2]

Itinéraire déterminé.
 
 
 
 