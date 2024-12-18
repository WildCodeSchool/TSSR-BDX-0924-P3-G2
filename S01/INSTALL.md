# Guide pour l'administrateur
## 1. Nomenclature des noms
### a) Le domaine
Le nom du domaine pour cette infrastructure sera `ecotech-solutions.lan`.

### b) Les unités d'organisation

### c) Les groupes de sécurités


### d) Les ordinateurs
#### Les serveurs
D'un point de vue **interne**, les serveurs seront nommé de la manière suivante :
  * Par une numérotation de 4 chiffres
  * Par le numéro de notre groupe
  * Par la fonction du serveur
  * S'il est, ou non, avec une interface graphique

Par exemple, un serveur Windows avec interface graphique pour de l'AD-DS aura pour nom : `1010-G2-ADDS-GUI`.  

D'un point de vue **externe**, les serveurs seront nommé de la manière suivante :
  * Par une numérotation de 4 chiffres (commune à celle de l'interne)
  * Par le numéro de notre groupe
  * Par le nom d'un héros de l'univers DC Comics

Par exemple, un serveur Debian aura pour nom : `Dr Manhattan`.  
Nous portons à votre attention que ce ne sont que des **exemples**.

#### Les clients
Comme le client a déjà des utilisateurs avec des ordinateurs, il nous a remis une liste avec la totalité des ordinateurs et leur nom. En voici un exemple : `PA41987`.
Seulement les chiffres changent, les deux lettres ne changent pas.


### e) Les utilisateurs
La convention de nommage pour un utilisateur se fera en fonction :
  * De son nom,
  * De son prénom,
  * De son département.

De rajouter le département où la personne travaille permettra d'éviter la problématique des homonymes.  
**Attention**, lorsque qu'un caractère spécial se trouve dans le nom ou prénom comme *Ekström* le *ö* deviendra *o*.  
Prenons l'exemple de **Aria Ekström** du département **Service Commercial** deviendra : `ekstrom-aria.sc`. 
Dans le cas où nous aurions deux **Aria Ekström** du département **Service Commercial**, alors l'utilisateur deviendra : `aria-ekstrom.sc`.

## 2. Listes des serveurs et des matériels nécessaires à l'élaboration de la futur infrastructure réseau
Pour ce projet, nous aurons besoin :
  * De serveurs :
    * Windows server avec pour rôles :
      * AD-DS
      * DHCP
      * DNS
    * Debian server avec pour rôles :
      * Serveur mail
      * Serveur web
  * De matériels :
    * Routeur
    * Switch

Cette liste est exhaustive et peut être amenée à être modifié.

## 3. Plan d'adressage réseau & schématique
Le réseau global du client est sur l'adresse `10.10.0.0/16`. Il est à noter que c'est l'adresse privée de l'entreprise et non l'adresse public.
### a) Plan d'adressage réseau
Pour pouvoir faire un plan d'adressage réseau, nous avons découpé le réseau global de l'entreprise en X sous-réseau :
  * Sous-réseau WAN
    * Il permettra de pouvoir accéder à Internet
  * Sous-réseau DMZ
    * Ce sous-réseau est séparé et isolé du réseau LAN et CORE de notre client afin de renforcer la sécurité en limitant l'accès direct aux systèmes internes.
    * Il sera utilisé pour héberger des services accessibles depuis l'extérieur (serveur web, mail)
  * Sous-réseau CORE
    * Il sera utilisé pour héberger les serveurs nécessaires au bon fonctionnement de l'entreprise (AD-DS, DHCP, ...)
  * Sous-réseau LAN
    * Il sera utilisé par les utilisateurs de l'entreprise.
    * Il est possible de divisé ce sous-réseau en plusieurs VLAN si cela sera nécessaire (VLAN Administrateur, VLAN Utilisateur, VLAN Guest, ...)
   
Ce-dessous un tableau résumant le découpage réseau avec les adresses :
| Nom | Interface | Adresse réseau | Adresse IP | Masque de sous-réseau | CIDR |
| --- | --- | --- | --- | --- | --- |
| Routeur Proxmox | vmbr2 (Internet) | 192.168.1.0 | 192.168.1.254 | 255.255.255.0 | 24 |
| Routeur Projet | G2-WAN | 192.168.1.0 | 192.168.1.253 | 255.255.255.0 | 24 |
| Routeur Projet | G2-DMZ | 10.10.1.0 | 10.10.1.254 | 255.255.255.0 | 24 |
| Routeur Projet | G2-CORE | 10.10.2.0 | 10.10.2.254 | 255.255.255.0 | 24 |
| Routeur Projet | G2-LAN | 10.10.3.0 | 10.10.3.254 | 255.255.255.0 | 24 |

### b) Plan schématique réseau
Maintenant que le plan d'adressage réseau, il nous est possible de la faire d'un point de vue schématique :
**INSERER IMAGE**
