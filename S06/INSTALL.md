# Réinstallation des Serveurs DC01 et DC02 

Suite a un problème avec le hostname de nos machines, nous avons du tout recommencer au niveau des Serveurs Contrôleurs de Domaine.  
Nous avons donc du réinstaller complètement 2 Serveurs Windows :  
  
- 1 VM DC01 (pour Contrôleur de Domaine 1)
    - **Hostname** : AQUAMAN
    - **OS :** *Windows Server 2022 Desktop*
    - **Login :** *Administrator*
    - **Mot de passe :** *Azerty1**
    - **Usage principal :** *Serveur Domain Controller 1*
    - **@IP/CIDR :** *10.10.7.210/16 (vmbr1055)*
    - **Services/Rôles/Logiciels :** *Rôle AD-DS, DNS, DHCP.*
  
- 1 VM DC02
    - **Hostname** : ZATANNA
    - **OS :** *Windows Server 2022 Core*
    - **Login :** *Administrator*
    - **Mot de passe :** *Azerty1**
    - **Usage principal :** *Serveur Domain Controller 1*
    - **@IP/CIDR :** *10.10.7.211/16 (vmbr1055)*
    - **Services/Rôles/Logiciels :** *Les mêmes que DC01 car ils seront répliqués*
