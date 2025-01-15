
## AD - Nouveau fichier d'utilisateurs à synchroniser dans l'AD  


L'entreprise **ECOTECH Solutions** nous a fourni un nouveau fichier `S06_EcoTechSolutions.xlsx` contenant des changements concernant l'architecture de l'entreprise :  
 - Certains utilisateurs sont nouveaux dans l'entreprise
 - D'autres sont partis des effectifs
 - La directrice commerciale Lana Wong quitte la société, remplacé par Marina Brun
 - Le département "Finance et Comptabilité" change de nom et s'appelle désormais  "Direction financière"
 - Dans ce département, le service "Fiscalité" disparaît, et les collaborateurs intègrent le service "Finance"
 - Des collaborateurs se sont marié et ont un nouveau nom.
 - Une nouvelle responsable B2B, Iko Loubert, vient d'arriver dans la société. Elle gérera le service "B2B" du département "Service Commercial".

       

Afin de traiter au mieux l'organisation de l'AD, nous avons décidé de gérer tout celà par script.
Nous avons donc converti le nouveau fichier `S06_EcoTechSolutions.xlsx` en `Utilisateurs-new.csv`.  
Ensuite nous avons réalisé un Script qui va :
- Importer la nouvelle liste des employés ainsi que leurs attributs et l'architecture des Départemens services
- Importer également toute la base AD présente dans notre serveur DC01
- Mettre à jour les OU (par Département) et sous OU (par services)
- Reformater la nomenclature de celles-ci en remplaçant les espaces par des tirets pour améliorer la compatibilité de l'AD.
- Mettre à jour les attributs des employés
- Créer une nouvelle OU "Comptes-desactives"
- identifier les comptes absents de la nouvelle liste mais présents dans l'AD afin de les désactiver et de les déplacer dans l'OU "Comptes-desactives"
- A la fin le script affichera le nombre de comptes désactivés par cette mise à jour

Voici le script en question :

```Powershell 
﻿#############################################################
#          Script pour créer une OU par département         #
#               et un Utilisateur par employé               #
#                  dans son OU respective                   #
#        Mis à jour par rapport à une nouvelle liste        #
#############################################################

# Ce Script Vérifiera que les OU, sous OU et les utilisateurs sont à jour par rapport aux nouvelles données de la liste.
# Si une OU est manquante il la créera, si une est présente mais plus nécessaire il la supprimera.
# Même si un utilisateurv est déjà présent il mettra à jour ses données.
# Si un Utilisateur est présent dans l'AD mais pas dans la nouvelle liste fournie, son compte sera désactivé et déplacé dans la nouvelle OU Comptes Désactivés




# Import du module Active Directory
Import-Module ActiveDirectory

# Fonction pour nettoyer les noms (suppression des accents et des espaces)
function Clean-String {
    param (
        [string]$texte
    )
    if ([string]::IsNullOrEmpty($texte)) {
        return $texte
    }
    
    # Remplacement des espaces par des tirets
    $texte = $texte -replace '\s+', '-'
    
    # Suppression des accents
    $normalizedString = $texte.Normalize([Text.NormalizationForm]::FormD)
    $stringBuilder = new-object Text.StringBuilder
    
    $normalizedString.ToCharArray() | ForEach-Object {
        if ([Globalization.CharUnicodeInfo]::GetUnicodeCategory($_) -ne [Globalization.UnicodeCategory]::NonSpacingMark) {
            [void]$stringBuilder.Append($_)
        }
    }
    return $stringBuilder.ToString()
}

# Importation du fichier CSV
$utilisateurs = Import-Csv -Path "C:\DSI\Utilisateurs-new.csv" -Delimiter ","

# Création de l'OU "Comptes désactivés" si elle n'existe pas
$racineOU = "DC=ecotech-solutions,DC=net"
$ouDesactive = "OU=Comptes-desactives," + $racineOU

if (-not (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$ouDesactive'" -ErrorAction SilentlyContinue)) {
    New-ADOrganizationalUnit -Name "Comptes-desactives" -Path $racineOU
    Write-Host "OU 'Comptes-desactives' créée" -ForegroundColor Green
}

# 1. Création des OUs principales (départements)
$departements = $utilisateurs | Select-Object -ExpandProperty Departement -Unique

foreach ($departement in $departements) {
    # Nettoyage du nom du département
    $departementNettoye = Clean-String $departement
    $ouPath = "OU=$departementNettoye," + $racineOU
    
    if (-not (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$ouPath'" -ErrorAction SilentlyContinue)) {
        New-ADOrganizationalUnit -Name $departementNettoye -Path $racineOU
        Write-Host "OU département '$departementNettoye' créée" -ForegroundColor Green
    }

    # 2. Création des sous-OUs (services) pour chaque département
    $services = $utilisateurs | Where-Object { $_.Departement -eq $departement } | Select-Object -ExpandProperty Service -Unique
    
    foreach ($service in $services) {
        if ($service -and $service -ne $departement) {
            # Nettoyage du nom du service
            $serviceNettoye = Clean-String $service
            $serviceOUPath = "OU=$serviceNettoye," + $ouPath
            
            if (-not (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$serviceOUPath'" -ErrorAction SilentlyContinue)) {
                New-ADOrganizationalUnit -Name $serviceNettoye -Path $ouPath
                Write-Host "OU service '$serviceNettoye' créée dans '$departementNettoye'" -ForegroundColor Green
            }
        }
    }
}

# 3. Traitement des utilisateurs
foreach ($utilisateur in $utilisateurs) {
    # Construction du login (première lettre du prénom + point + nom en minuscules)
    $login = ($utilisateur.Prenom).Substring(0,1).ToLower() + "." + $utilisateur.Nom.ToLower()
    
    # Définition du chemin de l'OU avec noms nettoyés
    $departementNettoye = Clean-String $utilisateur.Departement
    if ($utilisateur.Service -and $utilisateur.Service -ne $utilisateur.Departement) {
        $serviceNettoye = Clean-String $utilisateur.Service
        $ouChemin = "OU=$serviceNettoye,OU=$departementNettoye," + $racineOU
    } else {
        $ouChemin = "OU=$departementNettoye," + $racineOU
    }

    # Vérification si l'utilisateur existe
    $compteExistant = Get-ADUser -Filter {SamAccountName -eq $login} -ErrorAction SilentlyContinue

    if ($compteExistant) {
        # Mise à jour de l'utilisateur existant
        try {
            Set-ADUser -Identity $login -DisplayName "$($utilisateur.Prenom) $($utilisateur.Nom)" `
                       -GivenName $utilisateur.Prenom `
                       -Surname $utilisateur.Nom `
                       -Title $utilisateur.fonction `
                       -Department $utilisateur.Departement `
                       -Office $utilisateur.Site
            
            # Mise à jour des numéros de téléphone s'ils existent
            if ($utilisateur.'Telephone fixe') {
                Set-ADUser -Identity $login -OfficePhone $utilisateur.'Telephone fixe'
            }
            if ($utilisateur.'Telephone portable') {
                Set-ADUser -Identity $login -MobilePhone $utilisateur.'Telephone portable'
            }

            # Déplacement de l'utilisateur si nécessaire
            $utilisateurActuel = Get-ADUser -Identity $login -Properties DistinguishedName
            if ($utilisateurActuel.DistinguishedName -notlike "*$ouChemin") {
                Move-ADObject -Identity $utilisateurActuel.DistinguishedName -TargetPath $ouChemin
                Write-Host "Utilisateur $login déplacé vers $ouChemin" -ForegroundColor Yellow
            }

            Write-Host "Utilisateur $login mis à jour" -ForegroundColor Green
        }
        catch {
            Write-Host "Erreur lors de la mise à jour de $login : $_" -ForegroundColor Red
        }
    }
    else {
        # Création d'un nouvel utilisateur
        try {
            New-ADUser -Name "$($utilisateur.Prenom) $($utilisateur.Nom)" `
                      -SamAccountName $login `
                      -GivenName $utilisateur.Prenom `
                      -Surname $utilisateur.Nom `
                      -DisplayName "$($utilisateur.Prenom) $($utilisateur.Nom)" `
                      -Title $utilisateur.fonction `
                      -Department $utilisateur.Departement `
                      -Office $utilisateur.Site `
                      -Path $ouChemin `
                      -UserPrincipalName "$login@ecotech-solutions.net" `
                      -AccountPassword (ConvertTo-SecureString "Changeme123!" -AsPlainText -Force) `
                      -Enabled $true `
                      -ChangePasswordAtLogon $true

            # Ajout des numéros de téléphone si présents
            if ($utilisateur.'Telephone fixe') {
                Set-ADUser -Identity $login -OfficePhone $utilisateur.'Telephone fixe'
            }
            if ($utilisateur.'Telephone portable') {
                Set-ADUser -Identity $login -MobilePhone $utilisateur.'Telephone portable'
            }

            Write-Host "Utilisateur $login créé dans $ouChemin" -ForegroundColor Green
        }
        catch {
            Write-Host "Erreur lors de la création de $login : $_" -ForegroundColor Red
        }
    }
}

# 4. Désactivation des comptes qui ne sont plus dans le CSV
$tousLesUtilisateurs = Get-ADUser -Filter * -Properties Enabled, DistinguishedName |
    Where-Object { $_.DistinguishedName -notlike "*OU=Comptes-desactives*" -and $_.Enabled -eq $true }

foreach ($utilisateur in $tousLesUtilisateurs) {
    $login = $utilisateur.SamAccountName
    if ($login -ne "Administrator") {
        $existe = $false
        foreach ($csvUser in $utilisateurs) {
            $csvLogin = ($csvUser.Prenom).Substring(0,1).ToLower() + "." + $csvUser.Nom.ToLower()
            if ($csvLogin -eq $login) {
                $existe = $true
                break
            }
        }
        
        if (-not $existe) {
            try {
                # Désactivation et déplacement du compte
                Disable-ADAccount -Identity $login
                Move-ADObject -Identity $utilisateur.DistinguishedName -TargetPath $ouDesactive
                Set-ADUser -Identity $login -Description "Compte désactivé le $(Get-Date -Format 'dd/MM/yyyy')"
                Write-Host "Compte $login désactivé et déplacé vers 'Comptes-desactives'" -ForegroundColor Yellow
            }
            catch {
                Write-Host "Erreur lors de la désactivation de $login : $_" -ForegroundColor Red
            }
        }
    }
}

# Affichage du résumé
$nombreDesactives = (Get-ADUser -SearchBase $ouDesactive -Filter {Enabled -eq $false}).Count
Write-Host "`nRésumé des opérations :" -ForegroundColor Cyan
Write-Host "Nombre de comptes désactivés : $nombreDesactives" -ForegroundColor Cyan
```

A l'exécution on a un retour sur le powershell qui ressemble à ça :  

 <P ALIGN="center"><IMG src="../Ressources/Images/Captures DC01/capture_powershell_3_script_exec_new.png" width=700></P>   <BR>  
 <BR>
 On remarque que l'arborescence de l'AD est bonne aussi et que les espaces ainsi que les accents ont disparus.<BR><BR><BR>
 <P ALIGN="center"><IMG src="../Ressources/Images/Captures DC01/capture_arbo_AD_new.png" width=700></P>   <BR>  
