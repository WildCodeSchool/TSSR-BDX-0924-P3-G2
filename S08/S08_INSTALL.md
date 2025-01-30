# Installation du Serveur Web  

## Sommaire  

# 1- Mise en place et configuration du serveur  

Pour réaliser l'objectif de mise en place d'un serveur Web pour la société Ecotech-Solutions, nous avons opté pour la création d'une machine serveur Ubuntu 24.04.
Pour ça on a créé la VM à partir d'un conteneur, la manipulation pour la créer a été la suivante :
- Create CT
- Choisi les différents paramètres
- Insterface : DMZ (vmbr1070)
- Noeud 10 (le notre bien sûr, plus particulièrement celui pour le G2)
- Template CT ubuntu-24.04-standard_24.04-2_amd64.tar.zst
- Disk : 15 G
- Ram : 2048 M
- Coeurs : 4
- IP fixe : 10.12.0.1/16 (définie en semaine 1)
- Gateway (celle du routeur sur l'interface DMZ) : 10.12.255.254
  
Après sa création, on s'est identifié en root et lancé une mise a jour des paquets :  
```bash
apt update && apt upgrade -y
```
![capture 1](/chemin/access/image)  

Ensuite nous avons intallé le paquet Apache2 qui nous servira à contenir notre site web.  

```bash
apt install apache2 -y
```

Une fois installé, nous avons réglé les règles du pare-feu PFSense pour pouvoir accéder aux pings et au protocole HTTP depuis de LAN vers la DMZ.  
