# Guide d'utilisation

## I. Supervision PRTG

### a. Comptes d'utilisation
L'administrateur de notre IT a mis en place un compte en lecture seulement pour que les techniciens ait un visuel sur le système de supervision. Pour accéder à ce compte, il faut se connecter (depuis n'importe quel machine du réseau) avec l'adresse `https://10.10.7.14/index.htm`. Puis avec vous connecter avec les codes suivants :
- Login Name : *Technicien*
- Password : *Azerty1*

### b. Visualisation de l'ensemble des équipements IT
Pour visualiser l'ensemble des équipements IT, il faut aller dans *Devices > All Devices*

### c. Visualisation de la Map
Pour visualiser la map de notre IT, il faut aller dans *Maps > Dashboard*.

## d. Les différentes alarmes 
**PRTG** propose différents alarmes :
- *!!* : Down, Capteur non opérationnel prioritaire
- *w* : Warning, Capteur opérationnel mais qui a besoin d'une attention particulière
- *✓* : Up, Capteur opérationnel
- *⏸* : Paused, Capteur en pause (soit par l'administrateur soit par PRTG lui-même)
- *?* : Unknown, Capteur inconnu ou en cours d'acquisition par PRTG

Le technicien peut résoudre certaines alarmes *Down* & *Warning* mais pour tout le reste, il devra passer par l'administrateur de l'IT.
