# Documentation générale

## 1. Présentation du projet
Le projet consiste à créer une infrastructure réseau pour la société **EcoTech Solutions**. Durant ce projet, nous mettons en pratique les compétences suivantes :
  * Analyser les besoins techniques et fonctionnels
  * Mettre en place des objectifs et réaliser un suivi
  * Créer, gérer, faire évoluer une architecture réseau
  * Réaliser un projet avec une équipe de 3 personnes
  * Documenter toutes les étapes

Le projet est découpé en 12 sprints, chaque sprint a une durée d'une semaine. En début de sprint, des objectifs sont données. En fin de sprint, un bilan associé à une review est faite.

## 2. Objectif final
Actuellement, le client **EcoTech Solutions** a une infrastructure basique avec :
  * Borne WiFi
  * Box FAI

Le client souhaite modernier son infrastructure pour avoir :
  * Une meilleure performance
  * Une meilleure sécurité
  * Une meilleure gestion du réseau

C'est pourquoi, nous allons restructurer toute l'infrastructure du client avec :
  * Des sous-réseaus (LAN, DMZ)
  * Des machines dédiées (Serveur Windows, Serveur Web, ...)
  * Implémentation d'équipements réseau (routeur, switch, ...)

## 3. Les sprints

### a) Sprint 1
#### Membres du groupe et leurs rôles
Product Owner | Scrum Master | TSSR
--- | --- | ---
Julien Normand | Sybill Gribonval | Mindy Setham

#### Objectifs du sprint par membre 
- Julien Normand
  - Réfléchir à la configuration des équipements serveurs & clients (Rôles, OS, Configuration système).
  - Comment intégrer les utilisateurs à l'Active Directory
- Mindy Setham
  - Réfléchir à l'arborescence et à la création du domaine (Nom du domaine, Configuration du domaine, les OU & Groupes)
- Sybill Gribonval
  - Fournir plan d'adressage réseau & schéma réseau
  - Mettre en place une nomenclature de nom

### b) Sprint 2
#### Membres du groupe et leurs rôles
Product Owner | Scrum Master | TSSR
--- | --- | ---
Julien Normand | Sybill Gribonval | Mindy Setham

#### Objectifs du sprint par membre 
- Julien Normand
  - Création des serveurs Windows & Debian
  - Intégration des utilisateurs via script PowerShell
- Mindy Setham
  - Création du domaine et de l'Active Directory
  - Gestion de l'arborescence de l'Active Directory (OU & Groupes)
- Sybill Gribonval
  - Absente suite à un arrêt de travail

### c) Sprint 3
#### Membres du groupe et leurs rôles
Product Owner | Scrum Master | TSSR
--- | --- | ---
Julien Normand | Sybill Gribonval | Mindy Setham

#### Objectifs du sprint
- Julien Normand
  - Réinstallation de l'Active Directory
  - Finaliser l'intégration des utilisateurs via script PowerShell
- Mindy Setham
  - Mises en place de GPO de sécurité
  - Mises en place de GPO standard
- Sybill Gribonval
  - Intégration du serveur Debian sur l'AD
  - Installation et configuration de GLPI (Synchronisation AD, Inclusion des objets AD, Mise en place d'un système de ticketing)

### d) Sprint 4 à 6
#### Membres du groupe et leurs rôles
Product Owner | Scrum Master | TSSR
--- | --- | ---
Mindy Setham | Sybill Gribonval | Julien Normand

#### Objectifs du sprint
- Julien Normand
  - Gestion d'un firewal PfSense (Règle de pare-feu, Utilisation des principes **Deny All**/**Allow All**/**Least Privilege**/**Order of rules**)
  - Mise à jour de l'Active Directory 
- Mindy Setham
  - Mettre en place des dossiers réseaux pour les utilisateurs (Stockage des données sur un volume spécifique, Sécurité de partage des dossiers par groupe AD, Mappage)
- Sybill Gribonval
  - Installation et configuration d'un serveur de supervision PRTG avec mise en place d'un tableau de bord

### e) Sprint 7
#### Membres du groupe et leurs rôles
Product Owner | Scrum Master | TSSR
--- | --- | ---
Sybill Gribonval | Julien Normand | Mindy Setham

#### Objectifs du sprint
- Julien Normand
  - Mise en place d'un serveur de messagerie **iRedMail**
- Mindy Setham
  - 
- Sybill Gribonval
  - 

### e) Sprint 8
#### Membres du groupe et leurs rôles
Product Owner | Scrum Master | TSSR
--- | --- | ---
Julien Normand | Mindy Setham| Sybill Gribonval

#### Objectifs du sprint
- Julien Normand
  - 
- Mindy Setham
  - 
- Sybill Gribonval
  - Partenariat d'entreprise avec BillU pour la mise en place d'un VPN site-à-site.

## 4. Présentation finale

## 5. Difficultées rencontrées

## 6. Solutions trouvées

## 7. Améliorations possibles
