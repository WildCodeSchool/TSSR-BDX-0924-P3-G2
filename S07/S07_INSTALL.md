# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
- Mise en place d'un serveur de messagerie
- Mise en place d'un serveur de gestion de mot de passe

## II. Mise en place d'un serveur de messagerie  

### Création du CT
Pour mettre en place le serveur de messagerie, nous avons choisit de créer une VM de type conteneur. On a prit un serveur serveur Linux Ubuntu 24.04 en CLI. 
La création d'un CT est simple, dans proxmox il suffit d'abord de cliquer sur "Create CT" :  
![capture 1](../Ressources/Images/MORPHEUS_1.png)  
  
Ensuite on choisit le template (local template, et on le sélectionne):  
![capture 1](../Ressources/Images/MORPHEUS_2.png)  
  
On sélectionne bien notre noeud (node10), on saisit le CTIG (1065), le Hostname (MORPHEUS, pour serveur de MESSAGERIE), le ressource pool et le mot de passe.  
![capture 1](../Ressources/Images/MORPHEUS_3.png)  
  
On choisit ensuite le strorage, on remplace ceph-data par local-data et on choisit la taille du disque.  
![capture 1](../Ressources/Images/MORPHEUS_4.png)  

On choisit ensuite le nombre de processeur et la mémoire :    
![capture 1](../Ressources/Images/MORPHEUS_5.png) 
![capture 1](../Ressources/Images/MORPHEUS_6.png)  

Ici on définit directement l'IP de notre serveur en IP fixe (ici 10.10.7.13/16) avec en gateway celle de notre routeur PFsense.  
On choisit l'interface (chez nous celle du LAN, la vmbr 1055)  
On retire aussi le firewall de l'interface, nous avons notre propre parefeu.  
![capture 1](../Ressources/Images/MORPHEUS_7.png)   

Dans l'onglet DNS, on met notre domaine dans "DNS domain" et l'IP de notre serveur DNS dans "DNS servers".  
On pourrait rajouter aussi celui de google en le rajoutant après une virgule :  
`10.10.7.10, 8.8.8.8`  
![capture 1](../Ressources/Images/MORPHEUS_8.png)  

On vérifie bien chaque paramètre et on confirme en cliquant sur "Finish"    
![capture 1](../Ressources/Images/MORPHEUS_9.png)   

### Installation du logiciel de messagerie  
  
Ensuite on a mis a jour la liste des paquets ainsi que les paquets eux même avec la commande :  
`sudo apt update && sudo apt upgrade -y`
## III. Mise en place d'un serveur de gestion de mot de passe
