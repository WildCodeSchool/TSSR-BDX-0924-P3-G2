# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
- Réfléchir à la configuration des équipements serveurs & clients (Rôles, OS, Configuration système)
- Comment intégrer les utilisateurs à l'Active Directory
- Réfléchir à l'arborescence et à la création du domaine (Nom du domaine, Configuration du domaine, les OU & Groupes)
- Fournir plan d'adressage réseau & schéma réseau
- Mettre en place une nomenclature de nom

## II. Configuration des équipements serveurs & clients

### 1) Equipements serveurs
Pour ce projet, nous aurons besoin de :
- Serveur Windows pour les rôles AD-DS, DHCP & DNS au minimum
- Serveur Debian pour un serveur Web
- Serveur Debian ou Windows pour un serveur de messagerie

#### a) Windows Server GUI
Ce serveur est un `domain controler`, qui aura pour fonction :
- AD-DS
- DHCP
- DNS

Il sera configuré de la façon suivante :
- Adresse IP : `10.10.7.10/16`
- Login : `Administrator`
- Mot de passe : `Azerty1*`
- Hostname : `Aquaman`

#### b) Windows Server Core
Ce serveur est `domain controler`, qui aura pour fonction :
- AD-DS

Il sera configuré de la façon suivante :
- Adresse IP : `10.10.7.11/16`
- Login : `Administrator`
- Mot de passe : `Azerty1*`
- Hostname : `Zatanna`

Ce serveur aura la particularité d'avoir une réplication totale du Windows Server GUI.
#### c) Debian 12 Server Core
Ce server devra être sur l'AD, et aura pour fonction :
- Gestion de parc GLPI

Il sera configuré de la façon suivante :
- Adresse IP : `10.10.7.12/16`
- Login : `root`
- Mot de passe : `Azerty1*`
- Hostname : `Shazam`
### 2) Equipements clients
Pour ce projet, nous aurons besoin de :
- Client Ubuntu avec interface graphique pour de l'administration (au moins un par administrateur)
- Client Windows 10 pour les clients du projet

## III. Intégration des utilisateurs à l'Active Directory
Notre équipe compte mettre en place un script pour intégrer les utilisateurs à l'AD. Ce script prendra en compte :
- Nom et prénom de l'utilisateur
- Le département de l'utilisateur
- Le service de l'utilisateur
- Création des login de l'utilisateur
- Création du mot de passe de l'utilisateur
- Numéro de mobile et fixe de l'utilisateur

## IV. Arborescence de l'AD et du domaine
Le nom du domaine pour notre client sera `ecotech-solutions.lan`. 
### 1) Unités d'organisations (OU)
- Communication
	- Communication interne
	- Relations médias
- Développement
	- Analyse & Conception
	- Développement
	- Recherche & Prototype
	- Test & Qualité
- Direction
- Direction des Ressources Humaines
- DSI
	- Exploitation
- Finance & Comptabilité
	- Finance
	- Fiscalité
	- Service comptabilité
- Service commercial
	- ADV
	- B2B
	- Service achat
	- Service client

### 2) Les groupes
- Communication
	- Communication interne
	- Relations médias
- Développement
	- Analyse & Conception
	- Développement
	- Recherche & Prototype
	- Test & Qualité
- Direction
- Direction des Ressources Humaines
- DSI
	- Exploitation
- Finance & Comptabilité
	- Finance
	- Fiscalité
	- Service comptabilité
- Service commercial
	- ADV
	- B2B
	- Service achat
	- Service client

## V. Plan d'adressage et schéma réseau
### 1) Routeur principal
Suite aux données reçu, nous avons établis le futur réseau de l'entreprise. **Attention**, il peut être amenée à bouger. L'entreprise a un routeur avec pare-feu, ses interfaces sont :
- WAN : Permet de pouvoir accéder à Internet
    - Adresse réseau : _10.0.0.3/29_
    - Passerelle : _10.0.0.1_
- DMZ : Permet de séparer et isoler des serveurs afin de renforcer la sécurité en limitant l'accès direct aux systèmes internes
    - Adresse réseau : _10.12.0.0/16_
    - Passerelle : _10.12.255.254_
- LAN : Réseau interne de l'entreprise pour les utilisateurs
    - Adresse réseau : _10.10.0.0/16_
    - Passerelle : _10.10.255.254_

### 2) Plan d'adressage
Pour le plan d'adressage réseau, nous sommes parti sur la base d'un tableau qui sera rempli à chaque serveur / ordinateur rajouter sur le réseau.
#### a) DMZ

| Nom du serveur / ordinateur | Adresse IP | Passerelle    | Broadcast     | Réseau    | Masque |
| --------------------------- | ---------- | ------------- | ------------- | --------- | ------ |
| Serveur Web                 | 10.12.0.1  | 10.12.255.254 | 10.12.255.255 | 10.12.0.0 | /16    |

*Ceci est donnée à titre d'exemple pour des raisons de sécurité.*
#### b) LAN

| Nom du serveur / ordinateur | Adresse IP | Passerelle    | Broadcast     | Réseau    | Masque |
| --------------------------- | ---------- | ------------- | ------------- | --------- | ------ |
| Réseau client               | DHCP       | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |
| Administrateur              | 10.10.8.8  | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |

*Ceci est donnée à titre d'exemple pour des raisons de sécurité.*
### 3) Schéma réseau
![reseau](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/Réseau/reseauEcotechV2.png)

## VI. Nomenclature de nom
### 1) Les serveurs
Pour la nomenclature de nom, nous sommes parti sur la base d'un tableau qui sera rempli à chaque serveur / ordinateur rajouté sur le réseau.

| Type | ID   | Nom             | Fonction                 | Rôle             | Hostname |
| ---- | ---- | --------------- | ------------------------ | ---------------- | -------- |
| VM   | 10XX | G2-Routeur-PF01 | Routeur firewall PfSense | Sécurité réseau  |          |
| VM   | 10XX | G2-Serveur-DCXX | Domain Controler         | AD-DS, DNS, DHCP | Superman |

*Ceci est donnée à titre d'exemple pour des raisons de sécurité.*
### 2) Les ordinateurs
Les ordinateurs de chez Ecotech Solutions sont déjà nommés, vous pourrez trouver la liste [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/ordinateur.pdf).
