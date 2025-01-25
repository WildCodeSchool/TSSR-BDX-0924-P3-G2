# Guide d'installation pour l'administrateur
## I. Généralités
Pour cette semaine, nous avons :
  - Créer des GPO
	  - Sécurité
	  - Standard
  - Mise en place d'un serveur de gestion de parc GLPI
## II. Les GPO
### 1) Sécurité

### 2) Standard

Pour configurer la GPO standard pour mettre CHrome en navigateur par défaut, il faut:

Dans un premier temps se rendre sur le site: https://chromeenterprise.google/download/#download, et filtrer sur "Windows" en bas de l'écran

![9](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/Capture%20d'%C3%A9cran%202025-01-10%20105154.png)

Ensuite, cliquer sur le premier onglet "Quick start guide for Windows", pour ouvrir le guide de téléchargements des fichiers qui seront nécessaires à la configuration.

![10](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/Capture%20d'%C3%A9cran%202025-01-10%20105215.png)


Ensuite, il faut suivre l'installation en téléchargeant fichier "default application association", que l'on voit tout en bas 

![11](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/Capture%20d'%C3%A9cran%202025-01-10%20105238.png)


## III. Gestion de parc GLPI
### a. Prérequis
Il faut avoir :
  - Un serveur Debian (`Shazam`) avec le compte :
	  - Login : `root`
	  - Mot de passe : `Azerty1*`
	  - Adresse ip : `10.10.7.12`
  - Une machine administrateur dite local (`Atlas`) avec le compte :
	  - Login : `wilder`
	  - Mot de passer : `Azerty1*`
	  - Adresse ip : `10.10.8.8`
### b. Serveur Debian `Shazam`

#### Mettre à jours le serveur
```bash
apt update && apt upgrade -y
```

#### Installation des paquets
```bash
# Installation du paquet Apache
apt install apache2 -y
# Activation d'Apache au démarrage
systemctl enable apache2

# Installation de la base de donnée
apt install mariadbd-server -y

# Installation des paquets PHP
apt install ca-certificates apt-transport-https software-properties-common lsb-release curl lsb-release -y

# Installation des paquets PHP pour la synchronisation LDAP
sudo apt-get install php-ldap

# Ajout du dépôt pour PHP 8.1 :
wget -qO /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
curl -sSL https://packages.sury.org/php/README.txt | bash -x
apt update
# Installation de PHP 8.1
apt install php8.1 -y
# Installation des modules annexes
apt install php8.1 libapache2-mod-php -y
apt install php8.1-{ldap,imap,apcu,xmlrpc,curl,common,gd,mbstring,mysql,xml,intl,zip,bz2} -y
```

#### Configuration de MariaDB
Maintenant, installez le paquet MariaDB avec la commande `mysql_secure_installation`. A part `Change the root password?`, répondre `Y` à toutes les questions.
Pour le mot de passer `root`, nous avons mis : `Azerty1*`

Poursuivons sur la configuration de *MariaDB* avec la commande `mysql -u root -p`, puis mettre dans la configuration :
```mysql
# Nom de la base de donnée : glpidb
create database glpidb character set utf8 collate utf8_bin;

# Création du compte
grant all privileges on glpidb.* to glpi@localhost identified by "Azerty1*";
flush privileges
quit
```

#### Importation des ressources GitHub
```bash
# Téléchargement des sources
wget https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz

# Création du dossier pour glpi
mkdir /var/www/html/glpi.lab.lan

# Décompression du contenu téléchargé
tar -xzvf glpi-10.0.15.tgz

# Copie du dossier décompréssé vers le nouveau crée
cp -R glpi/* /var/www/html/glpi.lab.lan

# Suppression du fichier index.php dans /var/www/html
rm /var/www/html/index.html

# Droits d'accès aux fichiers
chown -R www-data:www-data /var/www/html/glpi.lab.lan
chmod -R 775 /var/www/html/glpi.lab.lan
```

#### Configuration de PHP
Editez le fichier `/etc/php/8.1/apache2/php.ini` avec nano, et modifiez les paramètres suivants :
```bash
memory_limit = 64M # <= à changer
file_uploads = on
max_execution_time = 600 # <= à changer
session.auto_start = 0
session.use_trans_sid = 0
```

Maintenant, vous pouvez redémarrer le serveur pour appliquer tous les changements.

### c. Administrateur local `Atlas`
#### Configuration de l'interface web
Depuis un navigateur web, allez sur la page internet : `http://10.10.7.12/glpi.lab.lan`, vous arriverez sur cette page :

![1](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/GLPI/1.png)
Et sélectionnez la langue `Français`, puis cliquez sur `Ok >`. Puis vous arriverez sur une seconde page pour les *copyrigth* où vous pourrez cliquer sur `Continuer >`. 

![3](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/GLPI/3.png)

Sur cette image, sélectionnez `Installer`. Puis remplissez la page d'après comme ci-dessous :
![4](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/GLPI/4.png)

Et cliquez sur `Continuer >`. Sur la page suivante, pensez à bien sélectionner la base de donnée **glpidb** puis cliquez sur `Continuer >` où une initialisation de la base de données sera effectuées. Une fois la base de donnée initialisée, cliquez sur `Continuer >` ou sur `Ok >` sur toutes les pages suivantes jusqu'à la fin de l'installation.

#### Connexion à l'interface Web
Pour vous connectez à GLPI, vous devez être sur cette page avec le lien `http://10.10.7.12/glpi.lab.lan` :

![8](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/Dev/Ressources/Images/GLPI/8.png)

Avec cette configuration, vous avez la possibilité de vous connecter sur ces comptes :
- Administrateur (`glpi`/`glpi`)
- Technicien (`tech`/`tech`)
- Utilisateur (`normal`/`normal`)
- Post-Only (`post-only`/`postonly`)

Pour le moment, nous utiliserons le compte Administrateur. Une fois connecté dessus, vous devrez :
- Modifier le mot de passe de chacun des comptes ci-dessus,
- Supprimer le fichier sur le serveur `Shazam` **/var/www/html/glpi.lab.lan/install/install.php**

### d. Synchronisation avec l'Active Directory
Sur l'interface web de GLPI, connecté sur le compte **Administrateur**, ouvrez le menu `Configuration` puis sélectionnez *Authentification > Annuaire LDAP > + Ajouter*. Dans **préconfiguration**, sélectionnez `Active Directory` puis configurez comme suit  et **sauvegarder** :
![9](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/3b0cfda541b4c9a82c63b7ecb80d7b9137678254/Ressources/Images/GLPI/9.png)

Une fois l'annuaire LDAP créé, allez dedans et sélectionnez `Tester` pour vérifier que la connexion à l'annuaire LDAP est fonctionnel. Si c'est le cas, vous aurez un message : ***Test réussi : Serveur principal Active Directory - ecotech-solutions.lan***.
### e. Inclusion des objets AD
Maintenant que la synchronisation est faite, nous allons pouvoir importer toute la base de donnée Active Directory (Utilisateurs, Groupes). Cela se fait toujours depuis l'interface web avec le compte administrateur et depuis le menu `Administration`.
#### Ajout d'utilisateur
Sélectionnez *Utilisateurs > Liaison annuaire LDAP > Importation de nouveaux utilisateurs > Mode expert*. Laissez les champs comme c'est et cliquez sur `Rechercher`, sélectionnez toute la liste et cliquez sur `Actions` pour importer comme suit :
![10](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/3b0cfda541b4c9a82c63b7ecb80d7b9137678254/Ressources/Images/GLPI/10.png)
Il ne reste plus qu'à envoyer ! Une fois fait et en retournant au menu *Utilisateurs*, vous verrez tous les nouveaux utilisateurs.

Pour l'ajout d'utilisateur supplémentaire, allez sur la documentation utilisateur en cliquant [ici](https://github.com/WildCodeSchool/TSSR-BDX-0924-P3-G2/blob/3b0cfda541b4c9a82c63b7ecb80d7b9137678254/S03/S03_USER_GUIDE.md).
#### Ajout de groupe
Sélectionnez *Groupes > Liaison Annuaire LDAP > Importation de nouveaux groupes*. Laissez les champs comme c'est et cliquez sur `Rechercher`, le reste est identique à l'importation des utilisateurs.
***Attention***, le groupe doit être attribué à un utilisateurs pour qu'il soit importer dans GLPI.

