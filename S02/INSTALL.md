# Guide pour l'administrateur

## 1. Mise en place des serveurs

### a) Serveur Windows Server 2022  GUI 
Ce serveur aura pour rôle :
  * AD-DS
  * DHCP
  * DNS

Son compte par défaut est `Administrator` et son mot de passe `motdepasse`.

### b) Serveur Windows Serveur 2022 Core
Ce serveur aura pour rôle :
  * AD-DS

Son compte par défaut est `Administrator` et son mot de passe `motdepasse`.

### c) Serveur Linux Debian
Ce serveur sera soit en VM soit en CT (en ligne de commande seulement), il devra être sur le domaine AD-DS et accessible en SSH seulement par un groupe d'administrateur du domaine.
Gestion manuelle ou automatique par script Bash ?

## 2. Mise en place du AD-DS et de son arborescence

### a) Création d'un domaine AD
Les deux serveurs Windows sont des *Domain Controler* du domaine et ont une réplication complètement gérée.  
C'est quoi une *réplication* ?

### b) Création des OU
Gestion manuelle ou automatique par script PowerShell ?

### c) Création des groupes
Gestion manuelle ou automatique par script PowerShell ?

### d) Intégration des utilisateurs
Le client nous a remis la liste des utilisateurs, que vous retrouverez *ici* (**RAJOUTER LIEN**). Avec ce fichier, nous allons pouvoir :
  * Créer les comptes utilisateurs,
  * Placer ces comptes dans les groupes correspondant,
  * Placer ces comptes dans les OU de départements/services,
  * Mettre en place une gestion des managers **(C'est quoi ??)**
Gestion manuelle ou automatique par script PowerShell ?
