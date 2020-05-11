# Teaching-HEIGVD-SRX-2020-Laboratoire-VPN
# Auteurs Michaël Da Silva, Christian Zaccaria

**Ce travail de laboratoire est à faire en équipes de 3 personnes**

**Pour ce travail de laboratoire, il est votre responsabilité de chercher vous-même sur internet, le support du cours ou toute autre source (vous avez aussi le droit de communiquer avec les autres équipes), toute information relative au sujet VPN, le logiciel eve-ng, les routeur Cisco, etc que vous ne comprenez pas !**

**ATTENTION : Commencez par créer un Fork de ce repo et travaillez sur votre fork.**

Clonez le repo sur votre machine. Vous pouvez répondre aux questions en modifiant directement votre clone du README.md ou avec un fichier pdf que vous pourrez uploader sur votre fork.

**Le rendu consiste simplement à répondre à toutes les questions clairement identifiées dans le text avec la mention "Question" et à les accompagner avec des captures. Le rendu doit se faire par une "pull request". Envoyer également le hash du dernier commit et votre username GitHub par email au professeur et à l'assistant**

**N'oubliez pas de spécifier les noms des membres du groupes dans la Pull Request ainsi que dans le mail de rendu !!!**


## Echéance 

Ce travail devra être rendu le dimanche après la fin de la 2ème séance de laboratoire, soit au plus tard, **le 11 mai 2020, à 23h59.**


## Introduction

Dans ce travail de laboratoire, vous allez configurer des routeurs Cisco émulés, afin de mettre en œuvre une infrastructure sécurisée utilisant des tunnels IPSec.

### Les aspects abordés

-	Contrôle de fonctionnement de l’infrastructure
-	Contrôle du DHCP serveur hébergé sur le routeur
-	Gestion des routeurs en console
-	Capture Sniffer avec filtres précis sur la communication à épier
-	Activation du mode « debug » pour certaines fonctions du routeur
-	Observation des protocoles IPSec


## Matériel

La manière la plus simple de faire ce laboratoire est dans les machines des salles de labo. Le logiciel d'émulation c'est eve-ng. Vous trouverez un [guide très condensé](files/Fonctionnement_EVE-NG.pdf) pour l'utilisation de eve-ng ici.

Vous pouvez faire fonctionner ce labo sur vos propres machines à condition de copier la VM eve-ng. A présent, la manière la plus simple d'utiliser eve-ng est de l'installer sur Windows (mais, il est possible de le faire fonctionner sur Mac OS et sur Linux...).

**Tuto d'installation** de la VM eve-ng : https://www.eve-ng.net/index.php/documentation/installation/virtual-machine-install/

**Récupération de la VM pré-configurée** (vous ne pouvez pas utiliser la versión qui se trouve sur le site de eve-ng) : vous la trouverez sur \\eistore1\cours\iict\SRX\LaboVPn

Il est conseillé de passer la VM en mode "Bridge" si vous avez des problèmes. Le mode NAT **devrait** aussi fonctionner.

Les user-password en mode terminal sont : "root" | "eve"

Les user-password en mode navigateur sont : "admin" | "eve"

Ensuite, terminez la configuration de la VM, connectez vous et récupérez l'adresse ip de la machine virtuelle.

Utilisez un navigateur internet (hors VM) et tapez l'adresse IP de la VM.


## Fichiers nécessaires 

Tout ce qu'il vous faut c'est un [fichier de projet eve-ng](files/eve-ng_Labo_VPN_SRX.zip), que vous pourrez importer directement dans votre environnement de travail.


## Mise en place

Voici la topologie qui sera simulée. Elle comprend deux routeurs interconnectés par l'Internet. Les deux réseaux LAN utilisent les services du tunnel IPSec établi entre les deux routeurs pour communiquer.

Les "machines" du LAN1 (connecté au ISP1) sont simulées avec l'interface loopback du routeur. Les "machines" du LAN2 sont représentées par un seul ordinateur.  

![Topologie du réseau](images/topologie.png)

Voici le projet eve-ng utilisé pour implémenter la topologie. Le réseau Internet (nuage) est simulé par un routeur. 

![Topologie eve-ng](images/topologie-eve-ng.png)


## Manipulations

- Commencer par importer le projet dans eve-ng.
- Prenez un peu de temps pour vous familiariser avec la topologie présentée dans ce guide et comparez-la au projet eve-ng. Identifiez les éléments, les interconnexions et les adresses IPs.
- À tout moment, il vous est possible de sauvegarder la configuration dans la mémoire de vos routeurs :
	- Au Shell privilégié (symbole #) entrer la commande suivante pour sauvegarder la configuration actuelle dans la mémoire nvram du routeur : ```wr```
	- Vous **devez** faire des sauvegardes de la configuration (exporter) dans un fichier - c.f. [document guide eve-ng](files/Fonctionnement_EVE-NG.pdf) 


### Vérification de la configuration de base des routeurs
Objectifs:

Vérifier que le projet a été importé correctement. Pour cela, nous allons contrôler certains paramètres :

- Etat des interfaces (`show interface`)
- Connectivité (`ping`, `show arp`)
- Contrôle du DHCP serveur hébergé sur R2


### A faire...

- Contrôlez l’état de toutes vos interfaces dans les deux routeurs et le routeur qui simule l'Internet - Pour contrôler l’état de vos interfaces (dans R1, par exmeple) les commandes suivantes sont utiles :

```
R1# show ip interface brief
R1# show interface <interface-name>
R1# show ip interface <interface-name>
```

Un « status » différent de `up` indique très souvent que l’interface n’est pas active.

Un « protocol » différent de `up` indique la plupart du temps que l’interface n’est pas connectée correctement (en tout cas pour Ethernet).

**Question 1: Avez-vous rencontré des problèmes ? Si oui, qu’avez-vous fait pour les résoudre ?**

---

**Réponse :**  Aucun problème n'a été rencontré lors de l'importation de la configuration de base. Toutes les adresses sont justes, et ce sur les bonnes interfaces.

![question1_R1](images/question1_R1.PNG)

![question1_R2](images/question1_R2.PNG)

---


- Contrôlez que votre serveur DHCP sur R2 est fonctionnel - Contrôlez que le serveur DHCP préconfiguré pour vous sur R2 a bien distribué une adresse IP à votre station « VPC ».


Les commandes utiles sont les suivantes :

```
R2# show ip dhcp pool 
R2# show ip dhcp binding
```

Côté station (VPC) vous pouvez valider les paramètres reçus avec la commande `show ip`. Si votre station n’a pas reçu d’adresse IP, utilisez la commande `ip dhcp`.

- Contrôlez la connectivité sur toutes les interfaces à l’aide de la commande ping.

Pour contrôler la connectivité les commandes suivantes sont utiles :

```
R2# ping ip-address
R2# show arp (utile si un firewall est actif)
```

Pour votre topologie il est utile de contrôler la connectivité entre :

- R1 vers ISP1 (193.100.100.254)
- R2 vers ISP2 (193.200.200.254)
- R2 (193.200.200.1) vers RX1 (193.100.100.1) via Internet
- R2 (172.17.1.1) et votre poste « VPC »

**Question 2: Tous vos pings ont-ils passé ? Si non, est-ce normal ? Dans ce cas, trouvez la source du problème et corrigez-la.**

---

**Réponse :**  Tous les pings ont passé sans problème.

*Ping entre R1 et son ISP*

![question2_ping_R2_ISP](images/question2_ping_R2_ISP.PNG)

​*Ping entre R2 et son ISP*

![question2_ping_R2_R1_via_ISP](images/question2_ping_R2_R1_via_ISP.PNG)

​*Ping entre R1 et R2 via leur ISP*

![question2_ping_R1_R2_via_ISP](images/question2_ping_R1_R2_via_ISP.PNG)

​*Ping entre R2 et R1 via leur ISP*

![question2_ping_R2_PC](images/question2_ping_R2_R1_via_ISP.PNG)

​*Ping entre R2 et le PC*

![question2_ping_R2_PC](images/question2_R2_PC.PNG)

---

- Activation de « debug » et analyse des messages ping.

Maintenant que vous êtes familier avec les commandes « show » nous allons travailler avec les commandes de « debug ». A titre de référence, vous allez capturer les messages envoyés lors d’un ping entre votre « poste utilisateur » et un routeur. Trouvez ci-dessous la commande de « debug » à activer.

Activer les messages relatif aux paquets ICMP émis par les routeurs (repérer dans ces messages les type de paquets ICMP émis - < ICMP: echo xxx sent …>)

```
R2# debug ip icmp
```
Pour déclencher et pratiquer les captures vous allez « pinger » votre routeur R1 avec son IP=193.100.100.1 depuis votre « VPC ». Durant cette opération vous tenterez d’obtenir en simultané les informations suivantes :

-	Une trace sniffer (Wireshark) à la sortie du routeur R2 vers Internet. Si vous ne savez pas utiliser Wireshark avec eve-ng, référez-vous au document explicatif eve-ng. Le filtre de **capture** (attention, c'est un filtre de **capture** et pas un filtre d'affichage) suivant peut vous aider avec votre capture : `ip host 193.100.100.1`. 
-	Les messages de R1 avec `debug ip icmp`.


**Question 3: Montrez vous captures**

---

**Screenshots :**  

​*Debug du routeur R1*

![question3_R1_debug](images/question3_R1_debug.PNG)

*Capture Wireshark*

![question3_wireshark](images/question3_wireshark.PNG)

---

## Configuration VPN LAN 2 LAN

**Il est votre responsabilité de chercher vous-même sur internet toute information relative à la configuration que vous ne comprenez pas ! La documentation CISCO en ligne est extrêmement complète et le temps pour rendre le labo est plus que suffisant !**

Nous allons établir un VPN IKE/IPsec entre le réseau de votre « loopback 1 » sur R1 (172.16.1.0/24) et le réseau de votre « VPC » R2 (172.17.1.0/24). La terminologie Cisco est assez « particulière » ; elle est listée ici, avec les étapes de configuration, qui seront les suivantes :

- Configuration des « proposals » IKE sur les deux routeurs (policy)
- Configuration des clefs « preshared » pour l’authentification IKE (key)
- Activation des « keepalive » IKE
- Configuration du mode de chiffrement IPsec
- Configuration du trafic à chiffrer (access list)
- Activation du chiffrement (crypto map)


### Configuration IKE

Sur le routeur R1 nous activons un « proposal » IKE. Il s’agit de la configuration utilisée pour la phase 1 du protocole IKE. Le « proposal » utilise les éléments suivants :

| Element          | Value                                                                                                        |
|------------------|----------------------------------------------------------------------------------------------------------------------|
| Encryption       | AES 256 bits    
| Signature        | Basée sur SHA-1                                                                                                      |
| Authentification | Preshared Key                                                                                                        |
| Diffie-Hellman   | avec des nombres premiers sur 1536 bits                                                                              |
| Renouvellement   | des SA de la Phase I toutes les 30 minutes                                                                           |
| Keepalive        | toutes les 30 secondes avec 3 « retry »                                                                              |
| Preshared-Key    | pour l’IP du distant avec le texte « cisco-1 », Notez que dans la réalité nous utiliserions un texte plus compliqué. |


Les commandes de configurations sur R1 ressembleront à ce qui suit :

```
crypto isakmp policy 20
  encr aes 256
  authentication pre-share
  hash sha
  group 5
  lifetime 1800
crypto isakmp key cisco-1 address 193.200.200.1 no-xauth
crypto isakmp keepalive 30 3
```

Sur le routeur R2 nous activons un « proposal » IKE supplémentaire comme suit :

```
crypto isakmp policy 10
  encr 3des
  authentication pre-share
  hash md5
  group 2
  lifetime 1800
crypto isakmp policy 20
  encr aes 256
  authentication pre-share
  hash sha
  group 5
  lifetime 1800
crypto isakmp key cisco-1 address 193.100.100.1 no-xauth
crypto isakmp keepalive 30 3
```

Vous pouvez consulter l’état de votre configuration IKE avec les commandes suivantes. Faites part de vos remarques :

**Question 4: Utilisez la commande `show crypto isakmp policy` et faites part de vos remarques :**

---

**Réponse :**  
R1
![R1 show](images/question4_r1.PNG)

R2
![R2 show](images/question4_r2.PNG)  

On peut voir que le routeur R2 a deux configurations pour son IKE tandis que R1 n'en a qu'une. Cela permet de mettre en place plusieurs "types" d'IKE et ainsi utiliser plusieurs VPN en parallèle ou selon certaines situations.  

---


**Question 5: Utilisez la commande `show crypto isakmp key` et faites part de vos remarques :**

---

**Réponse :**  
R1
![R1 show](images/question5_r1.PNG)

R2
![R2 show](images/question5_r2.PNG)  

Nos deux routeurs utilisent une même clé partagée (ici "cisco-1").   

---

## Configuration IPsec

Nous allons maintenant configurer IPsec de manière identique sur les deux routeurs. Pour IPsec nous allons utiliser les paramètres suivants :

| Paramètre      | Valeur                                  |
|----------------|-----------------------------------------|
| IPsec avec IKE | IPsec utilisera IKE pour générer ses SA |
| Encryption     | AES 192 bits                            |
| Signature      | Basée sur SHA-1                         |
| Proxy ID R1    | 172.16.1.0/24                           |
| Proxy ID R2    | 172.17.1.0/24                           |

Changement de SA toutes les 5 minutes ou tous les 2.6MB

Si inactifs les SA devront être effacés après 15 minutes

Les commandes de configurations sur R1 ressembleront à ce qui suit :

```
crypto ipsec security-association lifetime kilobytes 2560
crypto ipsec security-association lifetime seconds 300
crypto ipsec transform-set STRONG esp-aes 192 esp-sha-hmac 
  ip access-list extended TO-CRYPT
  permit ip 172.16.1.0 0.0.0.255 172.17.1.0 0.0.0.255
crypto map MY-CRYPTO 10 ipsec-isakmp 
  set peer 193.200.200.1
  set security-association idle-time 900
  set transform-set STRONG 
  match address TO-CRYPT
```

Les commandes de configurations sur R2 ressembleront à ce qui suit :

```
crypto ipsec security-association lifetime kilobytes 2560  
crypto ipsec security-association lifetime seconds 300  
crypto ipsec transform-set STRONG esp-aes 192 esp-sha-hmac  
  mode tunnel  
  ip access-list extended TO-CRYPT  
  permit ip 172.17.1.0 0.0.0.255 172.16.1.0 0.0.0.255  
crypto map MY-CRYPTO 10 ipsec-isakmp  
  set peer 193.100.100.1  
  set security-association idle-time 900  
  set transform-set STRONG  
  match address TO-CRYPT  
```

Vous pouvez contrôler votre configuration IPsec avec les commandes suivantes :

```
show crypto ipsec security-association
show crypto ipsec transform-set
show access-list TO-CRYPT
show crypto map
```

## Activation IPsec & test

Pour activer cette configuration IKE & IPsec il faut appliquer le « crypto map » sur l’interface de sortie du trafic où vous voulez que l’encryption prenne place. 

Sur R1 il s’agit, selon le schéma, de l’interface « Ethernet0/0 » et la configuration sera :

```
interface Ethernet0/0
  crypto map MY-CRYPTO
```

Sur R2 il s’agit, selon le schéma, de l’interface « Ethernet0/0 » et la configuration sera :

```
interface Ethernet0/0
  crypto map MY-CRYPTO
```


Après avoir entré cette commande, normalement le routeur vous indique que IKE (ISAKMP) est activé. Vous pouvez contrôler que votre « crypto map » est bien appliquée sur une interface avec la commande `show crypto map`.

Pour tester si votre VPN est correctement configuré vous pouvez maintenant lancer un « ping » sur la « loopback 1 » de votre routeur RX1 (172.16.1.1) depuis votre poste utilisateur (172.17.1.100). De manière à recevoir toutes les notifications possibles pour des paquets ICMP envoyés à un routeur comme RX1 vous pouvez activer un « debug » pour cela. La commande serait :

```
debug ip icmp
```

Pensez à démarrer votre sniffer sur la sortie du routeur R2 vers internet avant de démarrer votre ping, collectez aussi les éventuels messages à la console des différents routeurs. 

**Question 6: Ensuite faites part de vos remarques dans votre rapport. :**

---

**Réponse :**  
Capture Wireshark  
![Capture Wireshark, eth0/0 de R2](images/question6_wireshark.PNG)  
R1 en mode debug  
![Debug de R1](images/question6_R1_debug.PNG)  

On voit dans la capture Wireshark que la phase 1 de notre IKE a lieu avec le mode principal (Main Mode) et l'échange des 6 messages. La phase 2 a lieu avec le mode rapide (Quick Mode).  

Enfin, nos pings ICMP sont envoyés via notre connexion IPSec avec une encapsulation de ces données (ESP).

---

**Question 7: Reportez dans votre rapport une petite explication concernant les différents « timers » utilisés par IKE et IPsec dans cet exercice (recherche Web). :**

---

**Réponse :**  
IKE  
- lifetime = temps de validité de la SA de la phase 1  
- keepalive = temps entre chaque paquet DPD qui permettent de vérifier que les stations qui sont reliés par le tunnel soient toujours actives

IPSec   
- security-association lifetime seconds = temps avant que la SA de la phase 2 n'expire  
- security-association lifetime kilobytes = volume de traffic qui peut passer entre les pairs avant que la SA n'expire  
- security-association idle-time = temps qu'un pair peut être inactif et maintenir la SA de phase 2


---


# Synthèse d’IPsec

En vous appuyant sur les notions vues en cours et vos observations en laboratoire, essayez de répondre aux questions. À chaque fois, expliquez comment vous avez fait pour déterminer la réponse exacte (capture, config, théorie, ou autre).


**Question 8: Déterminez quel(s) type(s) de protocole VPN a (ont) été mis en œuvre (IKE, ESP, AH, ou autre).**

---

**Réponse :**  
IKE/ISAKMP - Nous avons mis en place le protocole IKE (ISAKMP) dans nos routeurs. On le sait grâce aux paquets capturés sur Wireshark qui sont aux nombres de 6 en mode principal et de 3 en mode agressif.  

![Protocole IKE](images/question8_wireshark_isakmp.PNG)  

De plus, sur la page officielle de Cisco concernant ISAKMP, on voit bien que ISAKMP est utilisé par IKEv2 pour la création de la SA.  

![Page Ciscio ISAKMP/IKE](images/question8_cisco_isakmp.PNG)  

On peut aussi le confirmer via la théorie donnée sur les VPN  

![Théorie de IKE - Phase 1 (mode principal)](images/question8_theorie_1.PNG)  

![Théorie de IKE - Phase 1 (mode agressif)](images/question8_theorie_2.PNG)  

ESP - Le protocole d'ESP est utilisé pour le chiffrement de nos données échangées. On peut le voir premièrement par les configurations des routeurs.  

![question8_R1_conf](images/question8_R1_conf.PNG)  

![question8_R2_conf](images/question8_R2_conf.PNG)  

Cette configuration de l'ESP se retrouve dans la documentation officielle de Cisco.  

![question8_ESP_Cisco](images/question8_ESP_Cisco.PNG)

De plus, dans notre capture Wireshark, on voit l'échange des paquets ICMP (ping) entre la station du routeur R2 et l'adresse Localhost du routeur R1 qui sont encapsulés par le protocole ESP.  

![question8_wireshark_esp](images/question8_wireshark_esp.PNG)  

---

**Question 9: Expliquez si c’est un mode tunnel ou transport.**

---
       
**Réponse :**  
Selon la configuration des routeurs R1 et R2, nous utilisons le mode tunnel.  

Config R1  
![question9_R1_conf](images/question9_R1_conf.PNG)  

Config R2  
![question9_R2_conf](images/question9_R2_conf.PNG)  

---


**Question 10: Expliquez quelles sont les parties du paquet qui sont chiffrées. Donnez l’algorithme cryptographique correspondant.**

---

**Réponse :**  

Comme vue en théorie, la partie du paquet qui est chiffré est le paquet IP original ainsi que l'en-queue "ESP trailer" (vu que ESP est utilisé pour l'IPSec).  

![](images/question1011_theorie.PNG)

Pour le chiffrement, nous utilisons l'algorithme AES avec une clé 192 bits.  

![question10_R1_conf](images/question10_R1_conf.PNG)  

![question10_ESP_Cisco](images/question10_ESP_Cisco.PNG)  

![question10_Cisco](images/question10_Cisco.PNG)  

---


**Question 11: Expliquez quelles sont les parties du paquet qui sont authentifiées. Donnez l’algorithme cryptographique correspondant.**

---

**Réponse :**  

Comme vue en théorie, la partie du paquet qui est authentifié est le paquet IP original ainsi que l'en-queue "ESP trailer" et l'en-tête ESP.  

![question1011_theorie](images/question1011_theorie.PNG)  

Pour l'authentification, nous utilisons une clé partagée, qui utilise SHA1 et HMAC-160.  

![question11_R1_conf](images/question11_R1_conf.PNG)  

![question11_ESP_Cisco](images/question11_ESP_Cisco.PNG)  

![question11_Cisco](images/question11_Cisco.PNG)  

---


**Question 12: Expliquez quelles sont les parties du paquet qui sont protégées en intégrité. Donnez l’algorithme cryptographique correspondant.**

---

**Réponse :**  

Pour l'intégrité du paquet, la même partie du paquet que pour son authentification est protégée. Un nouveau segment est ajouté au paquet: ESP auth.  

![question1011_theorie](images/question1011_theorie.PNG) 

Pour garantir cette intégrité, nous utilisons l'algorithme SHA1 et HMAC-160. 

![question11_R1_conf](images/question11_R1_conf.PNG)

![question12_Cisco](images/question12_Cisco.PNG)

---
