# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
- Mise en place d'un serveur de gestion des mises à jours WSUS
- Mise en place d'un troisième `Domain Controler` puis ajout du rôle FSMO
- Mise en place d'un VPN site à site avec la société BillU
- Mise en place de dossiers partagés entre les utilisateurs des sociétés Ecotech Solutions & BillU

## II. Mise en place d'un serveur de gestion des mises à jours WSUS

## III. Mise en place d'un troisième `Domain Controler` puis ajout du rôle FSMO

Pour éviter qu’un seul serveur ne gère toutes les tâches importantes du domaine, nous avons réparti les rôles FSMO sur plusieurs contrôleurs de domaine. Au départ, un seul serveur détenait tous les rôles. Pour améliorer la stabilité et la sécurité du réseau, nous avons ajouté deux nouveaux DC et transféré certains rôles sur ces serveurs. Cela permet de mieux répartir la charge et d’éviter qu’une panne ne bloque tout le système.


Nous avons d'abord installé deux nouveaux serveurs et les avons promus en tant que contrôleurs de domaine supplémentaires au sein du domaine existant.
La réplication de l’Active Directory a été vérifiée pour s’assurer que tous les objets étaient bien synchronisés entre les DC.


Nous avons utilisé la commande netdom query fsmo pour identifier l’hôte actuel des rôles FSMO. 

![Ressources/Images/Capture d'écran 2025-02-05 154529.png](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/Ressources/Images/Capture%20d'%C3%A9cran%202025-02-05%20154529.png)


Ensuite, via les consoles graphiques, nous avons réparti les rôles comme suit :
DC1 du nom d'Aquaman détient les rôles Schéma Master, Maître des noms de domaine et PDC Emulator.
Le nouveau DC du nom de BANE gère les rôles Maître RID, qui attribue les identifiants uniques aux objets, et Maître d’infrastructure, qui assure la cohérence entre les domaines.
Et le dernier DC du nom de Joker détient le rôle de Maître d’infrastructure, responsable de la gestion des références d’objets entre domaines.

Pour se faire, il faut se rendre dans la console Utilisateurs et ordinateurs Active Directory (dsa.msc).
Clic droit sur le domaine, puis Maître d'opérations.
Aller dans l'onglet RID et choisir le nouveau DC qui accueillera le rôle puis cliquer sur change.

![Ressources/Images/Capture d'écran 2025-02-06 155048.png](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/Ressources/Images/Capture%20d'%C3%A9cran%202025-02-06%20155048.png)


Une vérification a été effectuée avec netdom query fsmo pour s’assurer que les rôles ont bien été transférés.

![Ressources/Images/Capture d'écran 2025-02-06 162843.png](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/Ressources/Images/Capture%20d'%C3%A9cran%202025-02-06%20162843.png)

## IV. Mise en place d'un VPN site à site avec la société BillU

## V. Mise en place de dossiers partagés entre les utilisateurs des sociétés Ecotech Solutions & BillU
