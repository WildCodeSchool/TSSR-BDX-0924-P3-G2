# Guide d'installation pour l'administrateur
## I. Généralités
Pour cette semaine, nous avons :
  - Amélioré l'Active Directory en intégrant les utilisateurs dans l'Active Directory ainsi quie les OU
  - Intégrer le serveur DC02 (Windows server 2022 Core) sur le domaine de l'Active Directory

## II. L'Active Directory & les Utilisateurs
Cette semaine, nous avons reçu la liste des utilisateurs que trouverez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/utilisateur.pdf)  

Ce fichier va être converti au format CSV, ainsi ses données pourront être intégrées dans un script qui créera automatiquement :
- Les différentes OU (Unités d'Organisation) et sous OU dans L'AD.
En effet il créera une OU par département et une Sous OU par service.
- Chaque utilisateur de la liste
En y ajoutant automatiquement ses attributs (Prénom, Nom, Téléphone, email, etc.)

Le fichier au format CSV ressemble à ça :  
<P ALIGN="center"><IMG src="Ressources/Images/Captures DC01/capture_edit_csv.png" width=600></P>  

Les 2 fichiers ont été enregistrés dans un dossier \DSI créé à la racine du disque local su serveur DC01.  
On y trouve donc le fichier de données (Utilisateurs.csv) ainsi que le Script Powershell 
```powershell

```
Ce script, nous permettra de :
  - Créer les comptes utilisateurs
  - Placer les comptes dans les groupes correspondant
  - Placer les comptes dans les OU correspondants
  - Mettre en place une gestion des managers

## III. Serveur Debian sur le domaine de l'Active Directory
Une fois que le domaine et l'Active Directory bien établie, nous avons connecté notre serveur Debian dessus.
