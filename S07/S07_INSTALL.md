# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
- Mise en place d'un serveur de messagerie
- Mise en place d'un serveur de gestion de mot de passe

## II. Mise en place d'un serveur de messagerie  

### Création du CT
Pour mettre en place le serveur de messagerie, nous avons choisit de créer une VM de type conteneur. On a prit un serveur serveur Linux Ubuntu 24.04 en CLI. 
![capture 1](../Ressources/Images/MORHEUS_1.png) 
![capture 1](../Ressources/Images/MORHEUS_2.png) 
![capture 1](../Ressources/Images/MORHEUS_3.png) 
![capture 1](../Ressources/Images/MORHEUS_4.png) 
![capture 1](../Ressources/Images/MORHEUS_5.png) 
![capture 1](../Ressources/Images/MORHEUS_6.png) 
![capture 1](../Ressources/Images/MORHEUS_7.png) 
![capture 1](../Ressources/Images/MORHEUS_8.png) 
![capture 1](../Ressources/Images/MORHEUS_9.png) 
La création d'un CT est simple, dans proxmox il suffit d'abord de cliquer sur "Create CT" : 

Ensuite on a mis a jour la liste des paquets ainsi que les paquets eux même avec la commande :  
`sudo apt update && sudo apt upgrade -y`
## III. Mise en place d'un serveur de gestion de mot de passe
