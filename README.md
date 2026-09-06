<h1>IC-Campus, projet de création d’une infrastructure</h1>

<h2>Présentation de l'infrastructure</h2>

<img src="img/archi-network.jpeg" alt="Schéma réseau 3-tiers IC-Campus" width="600">

Nous aurons le frontend et le backend chacun dans un réseau différent. Cette option apporte de la sécurité, en réduisant la surface possible d’attaque. La charge de travail est aussi répartie entre les deux serveurs. De plus, la maintenabilité est accrue et la structure réalisée permet de réduire les risques de bugs après la modification d’un module.

<h2>Création du Dockerfile Api</h2>

<img src="img/CaptureDockerfileApi.png " alt="Capture écran du docker file côté api" width="600">

Voici le Dockerfile que j’ai rédigé en suivant les instructions données. Nous avons une utilisation de l’image python:3.12-alpine. J’attribue ensuite les valeurs aux labels désignés.
Nous pouvons voir que je fais des mises à jours et ajoute trois paquets : libcrypto3, util-linux et curl. L’explication de la présence de libcrypto3 et util-linux viendra plus tard, curl, lui, nous permet de tester la connexion à la fonction health plus bas. 
Ensuite je vais copier le requirements.txt qui nous permettra d’installer les dépendances utiles lors du pip install juste en dessous.
Je rapatrie ensuite le code source et les données qui seront utilisées.
Viens la création de l’utilisateur apposer, j’accorde les droits aux dossiers indispensables et je le connecte.
Le volume /data est créé. Et je met en place le HEALTHCHECK qui permettra de vérifier la disponibilité du serveur et renverra un statut healthy lors de son bon fonctionnement.
Ensuite la commande pour lancer gunicorn, qui écoutera sur toutes les adresses du réseau à condition d’appeler le port 5000 et laissera la possibilité de deux connexions simultanées.

<h2>Audit de sécurité avec Trivy</h2>
   
Je fais la commande suivante :

<code>trivy image --severity HIGH,CRITICAL ic-campus-api:1.0</code>

Ainsi j'ai obtenu, lors de mes tests, des alertes sur les vulnérabilités liées aux versions de certaines dépendances. Voici donc l'explication de la présence des deux paquets spécifiques libcrypto3 et util-linux.
Cela me permets d'utiliser la dernière version stable de ces paquets et limiter les vulnérabilités détectées par Trivy.

<img src="img/CaptureTrivy.png " alt="Capture écran du résultat du scan de Trivy" width="600">

<h2>Création du Dockerfile web</h2>

<img src="img/CaptureDockerfileWeb.png " alt="Capture écran du docker file côté api" width="600">

Cette fois le Dockerfile est composé de deux stages : builder et runtime. Dans le premier stage je travaille avec node.js 20. Le déroulé du fichier est le suivant : je me positionne sur le dossier /app. Je copie les package*.json au même endroit. je réalise un npm clean install. Toujours via COPY je rapatrie les fichiers de configuration. Pour finir je lance un build de mon application.
Pour le deuxième stageje travaille avec nginx version 1.30.4. Je commence par mettre à jour mon serveur. Puis j'ajoute le paquet curl pour la même raison que dans la partie api. Viennent ensuite les labels. Je veux ensuite créer un utilisateur non root, je dois le créer et lui attribuer des droits. Je copie ensuite des fichiers indispensables au bon fonctionnement du serveur. Je connecte l'utilisateur icweb créé précédemment.
Puis je réalise un HEALTHCHECK qui contrôlera le bon fonctionnement de mon serveur. J'expose le port 8080.

