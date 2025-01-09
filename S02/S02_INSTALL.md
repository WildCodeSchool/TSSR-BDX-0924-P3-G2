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

<P ALIGN="center"><IMG src="../Ressources/Images/Captures DC01/capture_powershell_2_script_exec.png" width=600></P>  

On remarque qu'on a bien confirmation de la création des OU, des sous OU et ensuite des Utilisateurs.  
Nous vérifions ensuite dans la console du Server Manager avec l'outil Active Directory Users and Computers qui va nous permettre de vérifier la bonne arborescence de notre domaine : 
<P ALIGN="center"><IMG src="../Ressources/Images/Captures DC01/capture_arbo_AD_1.png" width=500>&nbsp &nbsp<IMG src="../Ressources/Images/Captures DC01/capture_arbo_AD_2.png" width=500></P>  

L'arborescence est bonne et les utilisateurs ont bien leur attibuts renseignés.  



## III. Serveur Debian sur le domaine de l'Active Directory
Une fois que le domaine et l'Active Directory bien établie, nous avons connecté notre serveur Debian dessus.
