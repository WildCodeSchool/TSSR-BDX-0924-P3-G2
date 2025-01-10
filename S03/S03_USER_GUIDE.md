# Guide d'utilisation
## I. Généralités
Dans ce document, nous vous indiquerons comment utilisez :
  - Les GPO
  - Gestion de parc GLPI
## II. Les GPO

## III. Gestion de parc GLPI
### a) Rappel
Pour vous connecter à GLPI, vous avez besoin d'une machine avec interface graphique qui communique avec le Serveur `Shazam`. Après vous allez sur une page Web, et tapez : `http://10.10.7.203/glpi.lab.lan`, vous atteindrez cette page :
![8](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/GLPI/8.png)

Ici vous pouvez vous connecter avec différents comptes :
  - Administrateur
	  - Login : `glpi`
	  - Mot de passe : `glpi`
  - Technicien
	  - Login : `tech`
	  - Mot de passe : `tech`
  - Utilisateur
	  - Login : `normal`
	  - Mot de passe : `normal`
  - Postonly
	  - Login : `post-only`
	  - Mot de passe : `postonly`

### b) Compte Administrateur
Ce compte a pour rôle principal de pouvoir effectuer une gestion complète de GLPI.

Il a les permissions suivantes :
  - Accès à toutes les fonctionnalités, modules et données
  - Gestion des utilisateurs, des groupes, des profils et des droits
  - Configuration avancée du système (plugins, paramètres globaux, ...)
  - Création, modification, suppression de :
	  - Tickets
	  - Equipements
	  - Toutes autres données
  - Consultation et modification des données de tous les autres utilisateurs
Son utilisation typique est réservé aux responsables informatiques ou administrateurs systèmes qui gèrent l'outil GLPI dans sa globalité.
### c) Compte Technicien

### d) Compte Utilisateur

### e) Compte Postonly

