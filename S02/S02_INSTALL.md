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
<P ALIGN="center"><IMG src="../Ressources/Images/Captures DC01/capture_edit_csv.png" width=1000></P>  

Les 2 fichiers ont été enregistrés dans un dossier \DSI créé à la racine du disque local du serveur DC01.  
On y trouve donc le fichier de données (Utilisateurs.csv) ainsi que le Script Powershell (AD-OU-Users.ps1).  
Voici le script ci dessous, il contient un bon nombre de commentaires (lignes commençant par #) qui vous permettront de mieux comprendre ses différentes actions.  

```powershell

#############################################################
#          Script pour créer une OU par département         #
#               et un Utilisateur par employé               #
#                  dans son OU respective                   #
#############################################################

# Importation des données du fichier CSV
$CSV = Import-Csv -Path "C:\DSI\Utilisateurs.csv" -Delimiter ","

# Extraction des différents dÃ©partements
$departements = $CSV | Select-Object -ExpandProperty Departement -Unique

# Création d'une OU par département
Foreach ($departement in $departements) {
    # Vérification si l'OU existe déja 
    If (Get-ADOrganizationalUnit -Filter "Name -eq '$departement'" -ErrorAction SilentlyContinue) {
        Write-Host "L'OU '$departement' existe déja." -ForegroundColor Yellow
    }
    Else {
        New-ADOrganizationalUnit -Name $departement -Path "DC=ecotech-solutions,DC=net"
        Write-Host "OU $departement créée avec succès." -ForegroundColor Green
    }
   
    # Récupération des services pour ce département
    $services = $CSV | Where-Object {$_.Departement -eq $departement} | Select-Object -ExpandProperty Service -Unique
   
    # Création des sous-OUs pour les services de ce département
    Foreach ($service in $services) {
        # Vérifie si le service n'est pas vide et n'est pas identique au département        
        if ($service -and $service -ne $departement) {  
            # Vérification si l'OU service existe déja 
            If (Get-ADOrganizationalUnit -Filter "Name -eq '$service'" -ErrorAction SilentlyContinue) {
                Write-Host "L'OU '$service' existe déja." -ForegroundColor Yellow
            }
            Else {
                New-ADOrganizationalUnit -Name $service -Path "OU=$departement,DC=ecotech-solutions,DC=net"
                Write-Host "OU $service créée avec succès." -ForegroundColor Green
            }
        }
    }
}

# Création d'un compte utilisateur dans leur OU respective
Foreach ($employee in $CSV) {
    $departement = $employee.Departement
    $service = $employee.Service
    $UserFirstName = $employee.Prenom
    $UserLastName = $employee.Nom
    $UserLogin = ($UserFirstName).Substring(0,1).ToLower() + "." + $UserLastName.ToLower()
    $UserPswd = "Changeme123!"
    $UserFonction = $employee.Fonction
    $UserMobile = $employee.'Téléphone portable'
    $UserPhone = $employee.'Téléphone fixe'

    # Définition du chemin de la crÃ©ation de l'utilisateur
    # et vérification si le service n'est pas vide et n'est pas identique au département (c'était le cas pour Développement)
    if ($service -and $service -ne $departement) {  
        $OUPath = "OU=$service,OU=$departement,DC=ecotech-solutions,DC=net"
    } else {
        $OUPath = "OU=$departement,DC=ecotech-solutions,DC=net"
    }

    # Vérification si l'OU existe avant de créer l'utilisateur
    if (Get-ADOrganizationalUnit -LDAPFilter "(distinguishedName=$OUPath)" -ErrorAction SilentlyContinue) {
        # Vérification si l'utilisateur existe déja , sinon création du compte dans son OU
        If (Get-ADUser -Filter {SamAccountName -eq $UserLogin}) {
            Write-Host "L'utilisateur $UserLogin n'a pas pu être créé car il existe déja." -ForegroundColor Yellow
        }
        Else {
            New-ADUser  -Path $OUPath `
                        -SamAccountName $UserLogin `
                        -GivenName $UserFirstName `
                        -Name "$UserFirstName $UserLastName" `
                        -Title $UserFonction `
                        -Enabled $true `
                        -MobilePhone $UserMobile `
                        -OfficePhone $UserPhone `
                        -EmailAddress "$UserLogin@ecotech-solutions.net" `
                        -AccountPassword (ConvertTo-SecureString $UserPswd -AsPlainText -Force) `
                        -ChangePasswordAtLogon $true `
                        -Surname $UserLastName `
                        -DisplayName "$UserFirstName $UserLastName"

            Write-Host "L'utilisateur '$UserLogin' a été créé dans l'OU '$departement'" -ForegroundColor Green
        }
    } else {
        Write-Host "L'OU $OUPath n'a pas été trouvée pour l'utilisateur $UserLogin" -ForegroundColor Red
    }
}


```
Ce script, nous permettra, après avoir extrait les données du fichier CSV de :
  - Créer une OU pour chaque département
  - Créer une sous OU pour chaque service de département
  - Crééer chaque compte utilisateur avec un mot de passe unique qu'il devra changer dès sa première connection.
  - Placer les comptes Utilisateurs dans les OU correspondantes à leur département, services.

### Exécution du Script

Avant de pouvoir exécuter un Script en powershell il faut autoriser leur exécution dans Powershell avec cette ligne de commande :  
    
    Set-ExecutionPolicy -ExecutionPolicy Unrestricted  

Ensuite il faudra cliquer sur "Yes to All" afin de rendre les fichier ps1 exécutables en lignes de commande :  

<P ALIGN="center"><IMG src="../Ressources/Images/Captures DC01/capture_powershell_1_executionpolicy.png" width=1000></P>  

Ensuite on exécute le script :  

<P ALIGN="center"><IMG src="../Ressources/Images/Captures DC01/capture_powershell_2_script_exec.png" width=400></P>  

On remarque qu'on a bien confirmation de la création des OU, des sous OU et ensuite des Utilisateurs.  
Nous vérifions ensuite dans la console du Server Manager avec l'outil Active Directory Users and Computers qui va nous permettre de vérifier la bonne arborescence de notre domaine : 
<P ALIGN="center"><IMG src="../Ressources/Images/Captures DC01/capture_arbo_AD_1.png" width=450>&nbsp&nbsp&nbsp&nbsp<IMG src="../Ressources/Images/Captures DC01/capture_arbo_AD_2.png" width=450></P>  

L'arborescence est bonne et les utilisateurs ont bien leur attibuts renseignés.  



## III. Serveur Debian sur le domaine de l'Active Directory
Une fois que le domaine et l'Active Directory bien établie, nous avons connecté notre serveur Debian dessus.
### a. Prérequis
#### Mettre à jour le serveur
```bash
sudo apt-get update && sudo apt-get upgrade -y
```
#### Configuration réseau
- Assurez vous que le serveur Debian peut communiquer avec le contrôleur de domaine Windows.
- Configurez le serveur Debian pour utiliser le DNS du serveur AD pour la résolution des noms
```bash
# Vérification du fichier /etc/resolv.conf
nameserver 10.10.7.210 # Adresse du serveur Windows

# Testez la résolution DNS
nslookup ecotechsolutions.lan

# Ce qui donne
Server :  10.10.7.210
Address : 10.10.7.210#53

Name : ecotechsolutions.lan
Address : 10.10.7.210
```
#### Heure synchronisée
- Assurez vous que l'heure du serveur Debian soit synchronisée avec celle du contrôleur de domaine, **c'est important pour Kerberos**.
```bash
# Installez chrony
sudo apt install chrony -y
# Modifiez le fichier /etc/chrony/chrony.conf pour ajouter l'adresse IP du serveur AD comme serveur NTP
server 10.10.7.210 iburst
# Redémarrez le service
sudo systemctl restart chrony
sudo systemctl status chrony
# Vérifiez la synchronisation
timedatectl
```
- Avec cette dernière commande, la ligne `System clock synchronized` passera en **yes**.
### b.  Serveur Debian
#### Installation des paquets nécessaires
```bash
sudo apt install -y realmd sssd sssd-tools adcli samba-common krb5-user packagekit
```
- `realmd` : détecte et joint les domaines
- `sssd`: gère l'authentification via les services du domaines
- `krb5-user` : configure **Kerberos** pour l'authentification
- `adcli`: permet de joindre le domaine AD
- `samba`: pour les fonctionnalités SMB nécessaires
#### Configuration de Kerberos
Pendant l'installation du paquet `krb5-user`, vous devrez entrer le **nom du domaine** (exemple : ECOTECHSOLUTIONS.LAN).
### Rejoindre le domaine
```bash
# Rechercher le domaine
sudo realm discover ecotechsolutions.lan

# Joindre le serveur au domaine
sudo realm join --user=Administrator ecotechsolutions.lan
# Le mot de passe à rentrer est celui du compte Administrateur de votre serveur Windows

# Vérifiez que la machine est bien ajoutée
sudo realm list
```
#### Configuration du SSSD
```bash
# Commande pour modifier le fichier
sudo nano /etc/sssd/sssd.conf

# Modifier le fichier comme suit 
[sssd] 
services = nss, pam 
domains = ecotechsolutions.lan

[domain/mondomaine.local] 
ad_domain = ecotechsolutions.lan 
krb5_realm = ECOTECHSOLUTIONS.LAN 
realmd_tags = manages-system joined-with-adcli 
cache_credentials = True 
id_provider = ad 
access_provider = ad

# Changez les permissions du fichier
sudo chmod 600 /etc/sssd/sssd.conf
sudo systemctl restart sssd
sudo systemctl status sssd
```
#### Configuration des utilisateurs AD pour se connecter
```bash
# Modifiez le fichier /etc/pam.d/common-session
session required pam_mkhomedir.so skel=/etc/skel umask=0077
```
- `session`, indique que la directive s'applique à la phase de session *utilisateur* lorsqu'il se connecte au système
- `required`, indique que l'exécution de ce module est obligatoire pour la réussite de la session.
- `pam_mkhomedir.so`, indique qu'il s'agit d'un module PAM spécifique qui permet de créer automatiquement le répertoire personnel d'un utilisateur s'il n'existe pas.
- `skel=/etc/skel`, spécifie que le contenu du répertoire doit être copié dans le nouveau répertoire personnel de l'utilisateur.
- `umask=0077`, définit les permissions par défaut pour les fichiers et répertoires crées dans le répertoire personnel :
	- **0** : Pas de restriction sur l'utilisateur propriétaire
	- **7** : Retirer tous les droits (lecture, écriture, exécution) pour le groupe
	- **7** : Retirer tous les droits (lecture, écriture, exécution) pour les autres utilisateurs
	- En résumé, cela rend le répertoire personnel accessible uniquement par son propriétaire
#### Test & validation
```bash
# Test de connexion utilisateur avec un utilisateur AD
su - "utilisateurAD"@ecotechsolutions.lan
# Lister les utilisateurs et groupes du domaine 
id utilisateurAD@ecotechsolutions.lan
getent passwd
getent group
```
