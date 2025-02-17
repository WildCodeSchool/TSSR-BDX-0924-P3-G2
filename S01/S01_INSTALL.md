# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
- [Réfléchir à la configuration des équipements serveurs & clients (Rôles, OS, Configuration système)](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#ii-configuration-des-%C3%A9quipements-serveurs--clients)
- [Comment intégrer les utilisateurs à l'Active Directory](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#iii-int%C3%A9gration-des-utilisateurs-%C3%A0-lactive-directory)
- [Réfléchir à l'arborescence et à la création du domaine (Nom du domaine, Configuration du domaine, les OU & Groupes)](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#iv-arborescence-de-lad-et-du-domaine)
- [Fournir plan d'adressage réseau & schéma réseau](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#v-plan-dadressage-et-sch%C3%A9ma-r%C3%A9seau)
- [Mettre en place une nomenclature de nom](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#vi-nomenclature-de-nom)

## II. Configuration des équipements serveurs & clients

### a. Equipements serveurs
Pour ce projet, nous aurons besoin de :
- Serveur Windows pour les rôles AD-DS, DHCP & DNS au minimum
- Serveur Debian pour un serveur Web
- Serveur Debian ou Windows pour un serveur de messagerie

#### 1) Windows Server GUI
Ce serveur est un `domain controler`, qui aura pour fonction :
- AD-DS
- DHCP
- DNS

Il sera configuré de la façon suivante :
- Adresse IP : `10.10.7.10/16`
- Login : `Administrator`
- Mot de passe : `Azerty1*`
- Hostname : `Aquaman`

#### 2) Windows Server Core
Ce serveur est `domain controler`, qui aura pour fonction :
- AD-DS

Il sera configuré de la façon suivante :
- Adresse IP : `10.10.7.11/16`
- Login : `Administrator`
- Mot de passe : `Azerty1*`
- Hostname : `Zatanna`

Ce serveur aura la particularité d'avoir une réplication totale du Windows Server GUI.
#### 3) Debian 12 Server Core
Ce server devra être sur l'AD, et aura pour fonction :
- Gestion de parc GLPI

Il sera configuré de la façon suivante :
- Adresse IP : `10.10.7.12/16`
- Login : `root`
- Mot de passe : `Azerty1*`
- Hostname : `Shazam`
### b. Equipements clients
Pour ce projet, nous aurons besoin de :
- Client Ubuntu avec interface graphique pour de l'administration (au moins un par administrateur)
- Client Windows 10 pour les clients du projet

Cliquez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#guide-dinstallation-pour-ladministrateur) pour revenir en début de page.

## III. Intégration des utilisateurs à l'Active Directory
Notre équipe compte mettre en place un script pour intégrer les utilisateurs à l'AD. Ce script prendra en compte :
- Nom et prénom de l'utilisateur
- Le département de l'utilisateur
- Le service de l'utilisateur
- Création des login de l'utilisateur
- Création du mot de passe de l'utilisateur
- Numéro de mobile et fixe de l'utilisateur

Cliquez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#guide-dinstallation-pour-ladministrateur) pour revenir en début de page.

## IV. Arborescence de l'AD et du domaine
Le nom du domaine pour notre client sera `ecotech-solutions.lan`. 
### a. Unités d'organisations (OU)
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

### b. Les groupes
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

Cliquez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#guide-dinstallation-pour-ladministrateur) pour revenir en début de page.

## V. Plan d'adressage et schéma réseau
### a. Routeur principal
Suite aux données reçu, nous avons établis le futur réseau de l'entreprise. **Attention**, il peut être amenée à bouger. L'entreprise a un routeur avec pare-feu, ses interfaces sont :
- WAN : Permet de pouvoir accéder à Internet
    - Adresse IP : _10.0.0.3/29_
    - Passerelle : _10.0.0.1_
- DMZ : Permet de séparer et isoler des serveurs afin de renforcer la sécurité en limitant l'accès direct aux systèmes internes
    - Adresse IP : _10.12.0.0_
    - Broadcast : _10.12.255.254_
    - Réseau : _10.12.0.0./16_
- LAN : Réseau interne de l'entreprise pour les utilisateurs
    - Adresse IP : _10.10.0.0_
    - Broadcast : _10.10.255.254_
    - Réseau : _10.10.0.0./16_

### b. Plan d'adressage
Pour le plan d'adressage réseau, nous sommes parti sur la base d'un tableau qui sera rempli à chaque serveur / ordinateur rajouter sur le réseau.

#### 1) DMZ

| Nom du serveur / ordinateur | Adresse IP | Passerelle    | Broadcast     | Réseau    | Masque |
| --------------------------- | ---------- | ------------- | ------------- | --------- | ------ |
| Serveur Web                 | 10.12.0.1  | 10.12.255.254 | 10.12.255.255 | 10.12.0.0 | /16    |

*Ceci est donnée à titre d'exemple pour des raisons de sécurité.*

#### 2) LAN

| Nom du serveur / ordinateur | Adresse IP | Passerelle    | Broadcast     | Réseau    | Masque |
| --------------------------- | ---------- | ------------- | ------------- | --------- | ------ |
| Réseau client               | DHCP       | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |
| Administrateur              | 10.10.8.8  | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |

*Ceci est donnée à titre d'exemple pour des raisons de sécurité.*

### 3) Schéma réseau
![reseau](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/Ressources/Images/Réseau/reseauEcotechV2.png)

Cliquez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S01/S01_INSTALL.md#guide-dinstallation-pour-ladministrateur) pour revenir en début de page.

## VI. Nomenclature de nom
### a. Les serveurs
Pour la nomenclature de nom, nous sommes parti sur la base d'un tableau qui sera rempli à chaque serveur / ordinateur rajouté sur le réseau.

| Type | ID   | Nom             | Fonction                 | Rôle             | Hostname |
| ---- | ---- | --------------- | ------------------------ | ---------------- | -------- |
| VM   | 10XX | G2-Routeur-PF01 | Routeur firewall PfSense | Sécurité réseau  |          |
| VM   | 10XX | G2-Serveur-DCXX | Domain Controler         | AD-DS, DNS, DHCP | Superman |

*Ceci est donnée à titre d'exemple pour des raisons de sécurité.*

### b. Les ordinateurs
Les ordinateurs de chez Ecotech Solutions sont déjà nommés, vous pourrez trouver la liste [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/Ressources/ordinateur.pdf).
