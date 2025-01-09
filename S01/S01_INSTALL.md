# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
  - Etablit une nomenclature de nom
  - Créer un plan d'adressage réseau puis un schéma représentatif
  - Configurer les serveurs
	  - Windows
	  - Debian
## II. Nomenclature de nom
Nous avons établit une nomenclature de nom pour :
  - Les serveurs
  - Les ordinateurs
  - L'arborescence de notre domaine
### 1) Les serveurs
Nous avons un tableau continuellement mis à jours au fur et à mesure des semaines et des objectifs :

| Type | ID   | Nom             | Fonction                 | Rôle             | Hostname |
| ---- | ---- | --------------- | ------------------------ | ---------------- | -------- |
| VM   | 10XX | G2-Routeur-PF01 | Routeur firewall PfSense | Sécurité réseau  |          |
| VM   | 10XX | G2-Serveur-DCXX | Domain Controler         | AD-DS, DNS, DHCP | Superman |

Ce tableau est donnée à titre d'exemple.
### 2) Les ordinateurs
Les ordinateurs de chez Ecotech Solutions sont déjà nommés, vous pourrez trouver la liste [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/ordinateur.pdf).

### 3) L'arborescence de notre domaine
Le nom du domaine sera `ecotech-solutions.lan` et il aura l'arborescence suivante :
  - Unités d'organisations (OU) : 
	  - OU Départements > OU Services
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
		  - Finance & comptabilité
			  - Finance
			  - Fiscalité
			  - Service comptabilité
		  - Service Commercial
			  - ADV
			  - B2B
			  - Service achat
			  - Service client
  - Groupes (Grp) :
	  - Grp Départements / Grp Service
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
		  - Finance & comptabilité
			  - Finance
			  - Fiscalité
			  - Service comptabilité
		  - Service Commercial
			  - ADV
			  - B2B
			  - Service achat
			  - Service client
	  - Grp fonctionnels
	  - Grp de sécurité

Une fois l'arborescence créée, nous l'avons intégré dans l'Active Directory (AD).
## III. Réseau entreprise
Suite aux données reçu, nous avons établis le futur réseau de l'entreprise. **Attention**, il peut être amenée à bouger.
L'entreprise a un routeur avec parefeu, ses interfaces sont :
  - WAN : Permet de pouvoir accéder à Internet
	  - Adresse IP : *10.0.0.3/29*
	  - Passerelle : *10.0.0.1*
  - DMZ : Permet de séparer et isoler des serveurs afin de renforcer la sécurité en limitant l'accès direct aux systèmes internes
	  - Adresse IP : *10.12.0.0*
	  - Broadcast : *10.12.255.254*
	  - Réseau : *10.12.0.0./16*
  - LAN : Réseau interne de l'entreprise pour les utilisateurs
	  - Adresse IP : *10.10.0.0*
	  - Broadcast : *10.10.255.254*
	  - Réseau : *10.10.0.0./16*
### 1) Plan d'adressage réseau
Pour le plan d'adressage réseau, nous sommes parti sur la base d'un tableau qui sera rempli à chaque serveur / ordinateur rajouter sur le réseau.
#### a) DMZ

| Nom du serveur / ordinateur | Adresse IP | Passerelle    | Broadcast     | Réseau    | Masque |
| --------------------------- | ---------- | ------------- | ------------- | --------- | ------ |
| Serveur Web                 | 10.12.0.1  | 10.12.255.254 | 10.12.255.255 | 10.12.0.0 | /16    |
|                             |            |               |               |           |        |

#### b) LAN

| Nom du serveur / ordinateur | Adresse IP  | Passerelle    | Broadcast     | Réseau    | Masque |
| --------------------------- | ----------- | ------------- | ------------- | --------- | ------ |
| Réseau client               | DHCP        | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |
| Administrateur              | 10.10.7.1   | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |
| Windows Serveur GUI         | 10.10.7.210 | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |
| Windows Serveur Core        | 10.10.7.211 | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |
| Debian Server               | 10.10.7.203 | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |
| Technicien                  | 10.10.7.2   | 10.10.255.254 | 10.10.255.255 | 10.12.0.0 | /16    |

### 2) Schéma réseau
![reseau](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/Réseau/reseau_ecotech.png)

## IV. Configurer les serveurs
### 1) Windows serveur GUI
Ce serveur est un `domain controler`, qui aura pour fonction :
  - AD-DS
  - DHCP
  - DNS

Il sera configuré de la façon suivante :
  - Adresse IP : `10.10.7.210/16`
  - Login : `Administrator`
  - Mot de passe : `Azerty1*`
  - Hostname : `Aquaman`
### 2) Windows serveur Core
Ce serveur est `domain controler`, qui aura pour fonction :
  - AD-DS

Il sera configuré de la façon suivante :
  - Adresse IP : `10.10.7.211/16`
  - Login : `Administrator`
  - Mot de passe : `Azerty1*`
  - Hostname : `Zatanna`
### 3) Debian 12 Serveur
Ce server devra être sur l'AD, et aura pour fonction : 
  - Gestion de parc GLPI

Il sera configuré de la façon suivante :
  - Adresse IP : `10.10.7.203/16`
  - Login : `root`
  - Mot de passe : `Azerty1*`
  - Hostname : `DrManhattan`
