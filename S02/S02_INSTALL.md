# Guide d'installation pour l'administrateur
## I. Généralités
Pour cette semaine, nous avons :
  - Amélioré l'Active Directory en intégrant les utilisateurs dans l'Active Directory
  - Intégrer le serveur DC02 (Windows server 2022 Core) sur le domaine de l'Active Directory

## II. L'Active Directory & les Utilisateurs
Cette semaine, nous avons reçu la liste des utilisateurs que trouverez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/utilisateur.pdf)
Pour pouvoir l'intégrer dans notre Active Directory, nous avons créer un script :
```powershell

```
Ce script, nous permettra de :
  - Créer les comptes utilisateurs
  - Placer les comptes dans les groupes correspondant
  - Placer les comptes dans les OU correspondants
  - Mettre en place une gestion des managers
## III. Serveur Debian sur le domaine de l'Active Directory
Une fois que le domaine et l'Active Directory bien établie, nous avons connecté notre serveur Debian dessus.
