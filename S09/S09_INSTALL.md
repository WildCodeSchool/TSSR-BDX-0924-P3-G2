# Guide d'installation pour l'administrateur

## I. Généralités
Pour cette semaine, nous avons :
- Mise en place d'un serveur de téléphonie sur IP
- Mise en place d'un serveur Web
- Mise en place d'un PRA suite à une panne d'électricité

## II. Mise en place d'un serveur de téléphonie sur IP

## III. Mise en place d'un serveur Web
### 1) Installation et configuration

Pour réaliser l'objectif de mise en place d'un serveur Web pour la société Ecotech-Solutions, nous avons opté pour la création d'une machine serveur Ubuntu 24.04. Pour cela, nous avons créé la VM à partir d'un conteneur, la manipulation pour la créer a été la suivante :
- Create CT
- Choisi les différents paramètres
- Interface : DMZ (vmbr1070)
- Nœud 10 (le notre bien sûr, plus particulièrement celui pour le G2)
- Template CT ubuntu-24.04-standard_24.04-2_amd64.tar.zst
- Disk : 15 G
- Ram : 2048 M
- Cœurs : 4
- IP fixe : 10.12.0.2/16
- Gateway (celle du routeur sur l'interface DMZ) : 10.12.255.254
  
Après sa création, nous nous sommes identifié en `root` et effectué une mise à jours des paquets :  
```bash
apt update && apt upgrade -y
```
![capture 1](../Ressources/Images/capture_wonderwoman_1.png)  

Ensuite, nous avons installé le paquet `Apache2` qui nous servira à contenir notre site web.  

```bash
apt install apache2 -y
```

Une fois installé, nous avons réglé les règles du pare-feu pfSense pour pouvoir accéder aux pings et au protocole HTTP depuis de LAN vers la DMZ.  

![capture 2](../Ressources/Images/capture_wonderwoman_3.png)  

Ensuite, nous avons configuré une connexion SSH pour pouvoir administrer le serveur `WONDERWOMAN`.  
Nous avons créé une petite page d'accueil indiquant simplement que le site était actuellement en construction et invitant juste les visiteurs à envoyer un mail à la société via le courriel contact@ecotech-solutions.lan.  

Pour faire cela, nous avons simplement modifié le contenu du fichier de la page par défaut d'Apache, soit le fichier `/etc/var/www/html/index.html`.  
Voici le contenu du fichier :  
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ecotech-Solutions - En Construction</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            text-align: center;
            background-color: #f4f4f4;
            color: #333;
        }

        h1 {
            color: #4CAF50;
            font-size: 2.5rem;
            margin-bottom: 1rem;
        }

        p {
            font-size: 1.2rem;
            margin-bottom: 2rem;
        }

        a {
            color: #4CAF50;
            text-decoration: none;
            font-weight: bold;
            font-size: 1.2rem;
        }

        a:hover {
            text-decoration: underline;
        }

        footer {
            position: absolute;
            bottom: 1rem;
            font-size: 0.9rem;
            color: #777;
        }
    </style>
</head>
<body>
    <h1>Bienvenue sur Ecotech-Solutions</h1>
    <p>Notre site est actuellement en construction. Merci de votre patience.</p>
    <p>Pour toute demande, veuillez nous écrire à l'adresse suivante :</p>
    <a href="mailto:contact@ecotech-solutions.lan">contact@ecotech-solutions.lan</a>
    <footer>
        &copy; 2025 Ecotech-Solutions. Tous droits réservés.
    </footer>
</body>
</html>


```

Aperçu de la page web (accessible via l'IP du serveur soit 10.12.0.2) :  

![capture 3](../Ressources/Images/capture_wonderwoman_2.png)  

Ensuite nous avons bien sûr configuré un nouvel email sur notre serveur de messagerie afin que le mail contact@ecotech-solutions.lan existe.  
Nous sommes donc allé sur le Webmail d'administration à partir d'une VM du LAN.  
L'adresse d'administration étant `http://MORPHEUS.ecotech-solutions.lan/IredAdmin` ou bien `http://10.10.7.13/Iredadmin`.  
Ensuite, il faut simplement cliquer sur `Add`  > `User` et renseigner les champs.  

![capture 3](../Ressources/Images/capture_mail_1.png)    

Ensuite, depuis l'interface web d'une autre VM cliente, je me suis connecté avec mon mail j.normand@ecotech-solutions.lan.  
Pour se connecter à l'interface webmail il suffit de rentrer dans la barre d'adresses du navigateur `http://10.10.7.13/mail` ou bien `http://MORPHEUS.ecotech-solutions.lan/mail`.  
Puis, après avoir rentré mes identifiants, j'ai envoyé un mail de test à contact@ecotech-solutions.lan.  
  
![capture 3](../Ressources/Images/capture_mail_2.png)    

Enfin, depuis une autre VM encore j'ai pu constater que le nouveau mail fonctionnait bien !  

![capture 3](../Ressources/Images/capture_mail_3.png)    

## IV. Mise en place d'un PRA suite à une panne d'électricité
