# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
- [Mise en place d'un serveur de gestion des mises à jours WSUS](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S08/S08_INSTALL.md#ii-mise-en-place-dun-serveur-de-gestion-des-mises-%C3%A0-jours-wsus)
- [Mise en place d'un troisième `Domain Controler` puis ajout du rôle FSMO](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S08/S08_INSTALL.md#iii-mise-en-place-dun-troisi%C3%A8me-domain-controler-puis-ajout-du-r%C3%B4le-fsmo)
- [Mise en place d'un VPN site à site avec la société BillU](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S08/S08_INSTALL.md#iv-mise-en-place-dun-vpn-site-%C3%A0-site-avec-la-soci%C3%A9t%C3%A9-billu)

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

Cliquez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S08/S08_INSTALL.md#guide-dinstallation-pour-ladministrateur) pour revenir en début de page.

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

Cliquez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S08/S08_INSTALL.md#guide-dinstallation-pour-ladministrateur) pour revenir en début de page.

## IV. Mise en place d'un VPN site à site avec la société BillU
La société **Ecotech Solutions** veut mettre en place une connexion VPN avec la société **BillU**. Les deux parties utilisant un routeur *pfSense*, la configuration de cette connexion en sera plus simple car nous utiliserons les mêmes protocoles et technologies.

Le routeur *pfSense* a plusieurs protocole de connexion VPN :
- IPsec
- OpenVPN
- L2TP

Dans un premier temps, avec l'administrateur de chez **BillU**, nous avons configuré une connexion **IPsec** mais cela ne fonctionnait pas.
Nous avons donc décidé de configurer la connexion avec **OpenVPN**.

Pour rappel, tout se fait depuis l'interface web du routeur **pfSense**, où nous y accédons via :
- *URL* : `http://10.10.255.254`
- *Login* : admin
- *Mot de passe* : P0se!don
### a. Configuration des certificats
#### Création de l'autorité de certificat
Une **autorité de certification** est une entité de confiance, elle permet de délivrer des *certificats numériques*. Ces *certificats numériques* permettront d'authentifier l'identité des utilisateurs.

Pour créer l'**autorité de certification** sur **pfSense**, il faut aller dans `System > Certificates > Authorithies`. Dans cette onglet, cliquez sur `+ Add`, puis remplissez les champs suivants :
- *Descriptive name* : CA-EcotechSolutions-OpenVPN
- *Method* : Create an existing Certificate Authority
- *Common Name* : ecotech-solutions
- *Country Code* : FR
- *State or Province* : Nouvelle Aquitaine
- *City* : Bordeaux
- *Organization* : Ecotech Solutions
 
Laissez le reste comme c'est, et cliquez sur `Save`. Voici ce que vous aurez une fois sauvegardé :
![VPN01](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN01.png)  

#### Création du certificat Server
Le **certificat Server** sous **pfSense** est un certificat numérique qui permet de s'authentifier auprès des clients.

Pour créer un **certificat Server** sur **pfSense**, il faut aller dans `System > Certificates > Certificates`. Dans cette onglet, cliquez sur `+Add/Sign`, puis remplissez les champs suivants :
- *Method* : Create an internal Certificate
- *Descriptive name* : VPN-SSL-REMOTE-ACCESS
- *Certificate authority* : CA-EcotechSolutions-OpenVPN
- *Common name* : vpn.ecotechsolutions.local
- *Certificate Type* : Server Certificate
- *Alternative Names (value)* : vpn.ecotechsolutions.local

Laissez le reste comme c'est, et cliquez sur `Save`. Voici ce que vous aurez une fois sauvegardé :
![VPN02](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN02.png)  

### b. Création d'un utilisateur OpenVPN
Créer un utilisateur local OpenVPN sous **pfSense** permet de gérer l'authentification des clients VPN.

Pour créer un **utilisateur** sur **pfSense**, il faut aller dans `System > User Manager`. Dans cette onglet, cliquez sur `+Add`, puis remplissez les champs suivants :
- *Username* : EcoSolVPN
- *Password* : Azerty1*
- *Full Name* : Ecotech Solutions User VPN
- *Certificate* : Cochez la case `Click to create a user certificate`
- *Descriptive name* : VPN-SSL-RA
- *Certificate authority* : CA-EcotechSolutions-OpenVPN

Laissez le reste comme c'est, et cliquez sur `Save`. Voici ce que vous aurez une fois sauvegardé :
**Utilisateur**  
![VPN03](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN03.png)  
**Certificat User**  
![VPN04](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN04.png)  

### c. Configuration du serveur OpenVPN
Maintenant nous allons pouvoir configurer notre serveur **OpenVPN** sur **pfSense**, pour cela il faudra aller dans `VPN > OpenVPN`. Dans l'onglet **Servers**, cliquez sur `+Add`, puis remplissez les champs suivants :
- *Description* : VPN EcotechSolutions to BillU
- *Server mode* : Remote Access (SSL/TLS + User Auth)
- *Local port* : 1195
- *Peer Certificate Authority* : CA-EcotechSolutions-OpenVPN
- *Server Certificate* : VPN-SSL-REMOTE-ACCESS
- *IPv4 Tunnel Network* : L'adresse du réseau VPN que le client de chez **BillU** utilisera une fois connecté à OpenVPN, soit `10.11.0.0/16`
- *IPv4 Local Network* : L'adresse du réseau LAN de chez **Ecotech Solutions**, soit `10.10.0.0/16`
- *Concurrent connections* : Le nombre de connexion VPN simultanés que vous autorisez, soit `10`
- *Dynamic IP* : Cochez la case `Allow connected client to retain their connections if their IP address changes.`
- *Topology* : Sélectionnez `net30 -- Isolated /30 network per client`
- *DNS Default Domain* : Cochez la case `Provide a default domain name to clients`
- *DNS Default Domain* : ecotech-solutions.lan
- *DNS Server enable* : Cochez la case `Provide a DNS server list to clients. Addresses may be IPv4 or IPv6`
- *DNS Server 1* : 10.10.7.10
- *Custom options* : auth-nocache

Laissez le reste comme c'est, et cliquez sur `Save`. Voici ce que vous aurez une fois sauvegardé :  
![VPN05](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN05.png)

Maintenant que la configuration est faite, il faut exporter cette configuration pour la transmettre à l'administrateur de la société **BillU**.
Dans un premier temps, il faut que **pfSense** est le paquet `openvpn-client-export`. Pour cela, il faut aller dans `System > Packet Manager > Available Packages`. Dans **Search term**, cherchez le paquet `openvpn-client-export` et cliquez sur `+Install`. Puis vérifiez dans **Installed Packages** que le paquet soit bien installé :  
![VPN06](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN06.png)  

Une fois cela fait, allez dans l'onglet `VPN > OpenVPN > Client Export`. Puis remplissez les champs suivant :
- *Remote Access Server* : VPN EcotechSolutions to BillU UDP4:1194
- *Host Name Resolution* : Interface IP Address
- *Additional configuration options* : auth-nocache

Laissez le reste comme c'est, et cliquez sur `Save as default`. Maintenant pour exporter le fichier, toujours sur la même page, cliquez sur `Bundled Configurations > Archive`. Le fichier ZIP téléchargé devra être envoyé, par moyen sécurisés, à l'administrateur de chez **BillU**, et l'administrateur de chez **BillU** vous enverra celui de sa configuration. Voici le contenu de l'archive ZIP téléchargée :  
![VPN07](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN07.png)  

### d. Configuration des règles de pare-feu pour OpenVPN
Nous aurons besoin de configurer :
- Le flux depuis le réseau WAN vers le réseau OpenVPN, cela permet aux clients VPN externes de se connecter au **serveur OpenVPN** ainsi que de sécuriser cette connexion.
- Le flux depuis le réseau OpenVPN vers le réseau LAN, cela permet aux clients VPN de travailler comme s'ils étaient **physiquement dans le réseau LAN** et d'accéder aux serveurs internes, fichiers, bases de données ou autre services situés dans le LAN.

#### Du WAN vers OpenVPN
Allez dans `Firewall > Rules > WAN `, puis cliquez `+ Add`. Sur cette nouvelle page, remplissez les champs suivants :
- *Action* : Pass
- *Interface* : WAN
- *Address Family* : IPv4
- *Protocol* : UDP
- *Source* : Any
- *Destination* : WAN Address
	- *Destination Port Range* : from OpenVPN (1194) to OpenVPN (1194)
- *Log* : Cochez la case `Log packets that are handled by this rule`
- *Description* : Autoriser le VPN SSL

Si cela ne fonctionne pas, modifiez les champs suivants :
- *Protocol* : Any
- *Destination* : Any

En faisant cela, la connexion se fera mais ne sera pas sécurisé.  
![VPN08](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN08.png)  

#### De OpenVPN vers le LAN
Allez dans `Firewall > Rules > OpenVPN `, puis cliquez `+ Add`. Sur cette nouvelle page, remplissez les champs suivants :
- *Action* : Pass
- *Interface* : OpenVPN
- *Address Family* : IPv4
- *Protocol* : UDP
- *Source* : Any
- *Destination* : LAN Address
	- *Destination Port Range* : from Any to Any
- *Log* : Cochez la case `Log packets that are handled by this rule`
- *Description* : Autoriser le VPN sur le LAN

Si cela ne fonctionne pas, modifiez les champs suivants :
- *Protocol* : Any
- *Destination* : Network `10.10.0.0/16`

Il est aussi possible de modifier **Destination** pour mettre `Address or Alias` et spécifié l'adresse d'une machine cible (comme un serveur de stockage pour partager des fichiers).

En faisant cela, la connexion se fera mais ne sera pas sécurisé.  
![VPN09](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN09.png)

### e. Test & Vérification de l'accès distant
Sur le fichier **S08_USERGUIDE.md**, vous pourrez trouver la marche à suivre pour se connecter au réseau OpenVPN, qui est tout aussi important pour un simple utilisateur.
Une fois la connexion établie, sur l'interface web de **pfSense**, allez dans `Status > OpenVPN` où vous aurez l'image suivante :  
![VPN10](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/9fe1eefd69c2c6ffe8c6a5d1eb1c91932bffc46d/Ressources/Images/VPN/VPN10.png)

Cliquez [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/main/S08/S08_INSTALL.md#guide-dinstallation-pour-ladministrateur) pour revenir en début de page.
