# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
- Mise en place d'un serveur de gestion des mises à jours WSUS
- Mise en place d'un troisième `Domain Controler` puis ajout du rôle FSMO
- Mise en place d'un VPN site à site avec la société BillU
- Mise en place de dossiers partagés entre les utilisateurs des sociétés Ecotech Solutions & BillU

## II. Mise en place d'un serveur de gestion des mises à jours WSUS
### a. Configuration générale du serveur WSUS
WSUS est un **outils** interne à Windows Server. Notre serveur WSUS aura comme caractéristiques :
- Réseaux LAN, Adresse réservé via DHCP : `10.10.7.15/16`
- Systèmes d'exploitation : Windows Server avec interface graphique
- Un premier disque dur pour le système de 32Go
- Un second disque dur pour WSUS de 68Go
- RAM de 4Go
- Processeurs de 2 cœurs 

Dans le processus d'installation du serveur WSUS, nous allons le lier à notre domaine AD. Il faut donc que le serveur WSUS et le serveur DC01 (où se trouve l'AD-DS) puisse communiquer.
Nous aurons aussi besoin :
- D'une machine serveur Windows pour tester et vérifier la remontée vers WSUS
- D'une machine cliente Windows pour tester et vérifier la remontée vers WSUS

### b. Installation du rôle WSUS
Avant de démarrer l'installation du rôle **WSUS**, il faudra tout d'abord :
- Modifier le nom de votre machine **WHITECANARY**
	Pour cela, allez dans `Settings > System > About > Rename this PC`. Après redémarrez votre ordinateur pour appliquer les changements.
- Rajouter le serveur dans le domaine **ecotech-solutions.lan**
	Pour cela, allez dans `Settings > System > About > Rename this PC (advanced)`, dans l'onglet **Computer Name**, cliquez sur `Change` et sélectionnez `Domain` en mettant le nom de domaine **ecotech-solutions.lan**. Une fenêtre de connexion apparaitra où vous utiliserez les identifiants **Administrator** du domaine.

Pour cela, il faut aller dans la console **Server Manager**, , cliquez sur `Manage` puis sélectionner `Add Roles and Features`. Un **wizard** apparaîtra pour nous aider durant l'installation du rôle. Dans ce **wizard**, cliquez sur `Next` jusqu'à `Server Roles` où vous sélectionnerez les rôles suivants :
- Windows Server Update Services
- Web Server (IIS)
Puis cliquez sur `Next` jusqu'à `WSUS`, où vous sélectionnerez :
- Role Services : `WID Connectivity`/`WSUS Services`
- Content : Laisser la case coché puis mettez le chemin d'accès où seront installé toutes les mise à jours, soit **le second disque dédié à WSUS**.
	Attention, sous Windows les chemins d'accès se font avec des \.
Puis cliquez sur `Next` jusqu'à `Confirmation`, où vous cliquez sur `Install`. Pour finaliser l'installation du rôle, retournez sur la console **Server Manager** et en haut à droite vous aurez un panneau d'avertissement au niveau du drapeau. Cliquez dessus puis sur `Run post-installation tasks`.

Le rôle WSUS est maintenant installé !
### c. Configuration du rôle WSUS
Dans la console **Server Manager**, sélectionnez `Tools` puis `Windows Server Update Services`. Lors de la première utilisation de l'outils vous aurez un **Wizard** d'aide à la configuration :
- *Before you begin* : Cliquez sur `Next`
- *Microsoft Update Improvement Program* : Décochez la case et cliquez sur `Next`
- *Choose upstream server* : Sélectionnez `Synchronize from Microsoft Update`
- *Specify Proxy Server* : Ne rien remplir et cliquez sur `Next` sauf si vous avez un serveur proxy puis cliquez sur `Start Connecting` pour que le serveur WSUS se connecte sur les serveurs **Microsoft Updates** puis cliquez sur `Next`
- *Choose Languages* : Sélectionnez `English` & `French` puis cliquez sur `Next`
- *Choose Products* : Choisissez les produits nécessaires à votre infrastructure IT. Dans notre cas,  nous avons sélectionné : `Active Directory`/`Windows 10`/`Windows Server`. Attention, suivant le nombre de produit que vous choisissez la synchronisation peut être plus ou moins longue.
- *Choose Classification* : Sélectionnez `Critical Updates`/`Definition Updates`/`Security Updates`/`Updates` puis cliquez sur `Next`
- *Configure Sync Schedule* : Sélectionnez `Synchronize automatically` et choisissez une heure afin de ne pas perturber la production. Dans notre cas, nous avons choisis 02h00. Après choisissez le nombre de fois par jour où WSUS devra faire une synchronisation. Dans notre cas, nous avons choisis une seule synchronisation.
- *Finished* : Sélectionnez `Begin initial synchronisation` et cliquez sur `Finnish`.

Depuis la console WSUS, dans l'onglet `Synchronisation`, nous pouvons voir l'état de la synchronisation.
Vous pouvez toujours relancer ce **Wizard** en allant dans `Options` et tout en bas vous aurez `WSUS Server Configuration Wizard`.

Pour finaliser cette configuration, allez dans `Option > Automatic Approvals` et vérifiez si la case **Default Automatic Approval Rule** est bien coché. Si ce n'est pas le cas, faites-le.

La configuration de base de WSUS est maintenant terminée !

### d. Configuration de WSUS avec le domaine *ecotech-solutions.lan*
#### Modification de la méthode d'affectation des ordinateurs sur WSUS
Dans la console **WSUS**, allez dans `Option > Computers > Use Group Policy or registry settings on computers` puis cliquez sur `Apply > Ok`.
Maintenant, l'affectation des ordinateurs sur WSUS se fera depuis des GPO du serveur où se trouve notre AD.

#### Créer des groupes d'ordinateurs dans WSUS
Pour notre infrastructure, nous allons créer deux groupes d'ordinateurs :
- Pour les serveurs
- Pour les clients
Ces groupes seront nommés de la même manière que les GPO à laquelle ils seront liés, soit :
- GrpServerComputer
- GrpClientComputer

Pour créer des groupes d'ordinateurs, faites un clique droit sur `All Computers > Add Computer Group...`. Renseignez le nom, et cliquez sur `Add`. Une fois cela fait, vous aurez :  
![WSUS01](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS01.png)

#### Lier les PC et les serveurs à WSUS par GPO
Avant toute chose, sur le serveur où se trouve notre **Active Directory**, nous avons créé:
- Une OU `LabComputers` et dedans deux OU `Client`/`Serveur`
- Deux groupes `GrpServerComputer` & `GrpClientComputer` où nous ajouterons les serveurs et les clients dans leur groupe respectifs.
Après nous avons créée deux GPO, qui chacune seront placés dans les OU dédiés :
- `WSUS_Settings_GrpClientComputer`
	- *Configure Automatic Updates*
	- *Do not connect to any Windows Update Internet locations*
	- *Enable client-side targeting*
	- *Specify intranet Microsoft update service location*
	- *Turn off auto-restart for updates during active hours*
- `WSUS_Settings_GrpServerComputer`
	- *Configure Automatic Updates*
	- *Do not connect to any Windows Update Internet locations*
	- *Enable client-side targeting*
	- *Specify intranet Microsoft update service location*
	- *Turn off auto-restart for updates during active hours*

Pour éditer les GPO, nous avons fait un clique droit puis *Edit* sur la GPO cible puis nous sommes allés dans `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Update`. Puis, nous avons sélectionné la règle pour l'éditer.
##### Configure Automatic Updates
![WSUS02](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS02.png)
Puis cliquez sur `Apply > Ok`.

##### Do not connect to any Windows Update Internet locations
![WSUS03](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS03.png)
Puis cliquez sur `Apply > Ok`.

##### Enable client-side targeting
![WSUS04](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS04.png)
Puis cliquez sur `Apply > Ok`.

##### Specify intranet Microsoft update service location
L'url à mettre : `http://whitecanary.ecotech-solutions.lan:8530`. Le port 8530 est le port par défaut utilisé par WSUS.  
![WSUS05](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS05.png)
Puis cliquez sur `Apply > Ok`.

##### Turn off auto-restart for updates during active hours
![WSUS06](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS06.png)
Puis cliquez sur `Apply > Ok`.

#### Tester la GPO WSUS
Dans un premier temps, il faut aller sur le terminal d'un serveur et d'un client pour mettre à jours les GPO avec la commande `gpupdate /force`. Puis faire la commande `gpresult /r` pour vérifier que la GPO s'est bien appliqué.  
![WSUS07](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS07.png)  
![WSUS08](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS08.png)  
Attention, nous vous conseillons d'utiliser :
- Soit l'invité de commande (en administrateur)
- soit le terminal PowerShell (en administrateur)
Mais pas le terminal PowerShell ISE, où la commande ne fonctionne pas tout le temps.

Une fois la GPO mis, retournons sur le serveur WSUS pour vérifier que les machines soit visible.   
![WSUS09](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/11dd610963549d59ca4e32dde22a0f58f7c086aa/Ressources/Images/WSUS/WSUS09.png) 

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

