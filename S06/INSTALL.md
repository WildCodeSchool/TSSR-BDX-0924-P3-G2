# Réinstallation des Serveurs DC01 et DC02 

Suite a un problème avec le hostname de nos machines, nous avons du tout recommancer au niveau des Serveurs Contrôleur de Domaine.  
Nous avons donc du réinstaller complètement 2 Serveurs Windows :  
- 1 VM DC01 (pour Contrôleur de Domaine 1)
    - Hostname : AQUAMAN
    - **OS :** *Windows Server 2022 Desktop*
    - **Propriétaire :** *Groupe 2 (EcoTetch Solutions)*
    - **Login :** *Administrator*
    - **Mot de passe :** *Azerty1**
    - **Usage principal :** *Serveur Domain Controller 1*
    - **@IP/CIDR :** *10.10.7.210/16 (vmbr1055)*

## Services/Rôles/Logiciels (détails)

- *Rôle AD-DS*
- *DNS*
- *DHCP*

## Source

-*Template-Windows-Server-2022*
