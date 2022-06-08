# Déroulement:

**20** minutes en **3** sujets:

* Approche méthodologique 

* Synthèse des résultats

* Démo de l'API

# Introduction

Voici le contexte :
Nous travaillons sur un système embarqué de vision par ordinateur pour véhicule automatisé. 
Cette documentation traitera de la partie "**segmentation des images**".
le dataset sera alimenté par l'équipe "traitement des images".
Il faut garder à l'esprit que le système acquisition des images n'est pas stabilisé 
( ce qui impliquera une certaine "souplesse" dans le format d'entrée des images... )
Et son volume va aller en s'agrandissant, rapidement, donc un "**Batch Generator**" semble indispensable...

L'objectif sera de produire un service en ligne, via une **API simple**, prenant en entrée un identifiant d'image; 
et qui produira en sortie la segmentation de l'image du moteur d'inférence créé, avec l'image d'entrée.

# 1. Approche Méthodologique :

## Les métriques qui seront utilisées :

### Accuracy

la plus générale, correspondant à la **précision du modèle**.
Mais elle reste particulière.
Indépendamment du fait que notre problème soit un problème de classification binaire ou multi-classes, nous pouvons spécifier la métrique "précision" pour rendre compte de la précision.

### Coefficient Dice

$$
2*AirChevauchement / AirTotal
$$

<img title="" src="file:///./Ressources/prez/diceCoeff.png" alt="" data-align="center">

### le score IoU ou "l'intersection de l'union"

$$
AirIntersection / AirUnion
$$

<img title="" src="file:///./Ressources/prez/scoreIoU.png" alt="" data-align="center">

<img title="" src="file:///./Ressources/prez/formulaIoU.png" alt="" data-align="center">

## La méthode de recherche du modèle qui sera utilisé :

<img title="" src="file:///./Ressources/prez/methodo.svg" alt="" data-align="center">

*Pourquoi U-Net ?*
Déjà il faut prendre en compte que le sujet se porte sur la **Segmentation sémantique**
(on parle aussi de prédiction dense, car chaque pixel de l'image est soumis à prédiction),
tout à fait adapté au projet de véhicule autonome.
Nous sommes ici dans le domaine des algorithmes de **classification supervisé** et plus particulièrement dans la vision par ordinateur .
(Computer Vision)
**U-Net** est un modèle de réseaux de neurones artificiel dédié à cet "art" et plus particulièrement aux problèmes de segmentation.
Il est aussi très utilisé dans les diagnostiques médicaux, la cartographie satellite, les robots d'agriculture de précision...

### Commençons par un tester un model simple, l'UnetMini

<img title="" src="file:///./Ressources/prez/UNetMini.svg" alt="" data-align="center">

Déjà nous obtenons des résultats plutôt satisfaisants:

<img title="" src="file:///./Ressources/prez/unetMini.png" alt="" data-align="center">

### Deuxième étape, le model U-NET classique :

<img title="" src="file:///./Ressources/prez/UNet.svg" alt="" data-align="center">

Les résultats sont meilleurs et en beaucoup moins de temps :D

<img title="" src="file:///./Ressources/prez/unet.png" alt="" data-align="center">

### Enfin, VGG16 et U-Net

<img title="" src="file:///./Ressources/prez/VggUNet.svg" alt="" data-align="center">

En voici les résultats très encourageants:

<img title="" src="file:///./Ressources/prez/vggunet.png" alt="" data-align="center">

### Synthèse des tests sur les 3 Models

| Model      | Temps de Fit | IoU   | dice  | accuracy |
|:----------:|:------------:|:-----:|:-----:|:--------:|
| U-Net Mini | 9:18:00      | 0.729 | 0.842 | 0.877    |
| U-Net      | 1:49:00      | 0.740 | 0.850 | 0.882    |
| Vgg U-Net  | 3:43:00      | 0.764 | 0.865 | 0.899    |

le Model le plus concluant est donc l'inclusion de VGG16 dans la partie "descendante", l'encodeur, de l'architecture de U-Net avec un IoU de **76%**

## Changement de fonction de perte

### Testons la fonction "kullback_leibler_divergence"

Qu'est-ce que cette fonction :

En [théorie des probabilités](https://fr.wikipedia.org/wiki/Th%C3%A9orie_des_probabilit%C3%A9s "Théorie des probabilités") et en [théorie de l'information](https://fr.wikipedia.org/wiki/Th%C3%A9orie_de_l%27information "Théorie de l'information"), la **divergence de Kullback-Leibler**[1](https://fr.wikipedia.org/wiki/Divergence_de_Kullback-Leibler#cite_note-KullbackLeibler1951-1),[2](https://fr.wikipedia.org/wiki/Divergence_de_Kullback-Leibler#cite_note-Kullback1959-2) (ou **divergence K-L** ou encore **entropie relative**) est une [mesure de dissimilarité](https://fr.wikipedia.org/wiki/Divergence_(statistiques) "Divergence (statistiques)") entre deux [distributions](https://fr.wikipedia.org/wiki/Loi_de_probabilit%C3%A9 "Loi de probabilité") de probabilités.

Une perte de divergence de KL de 0 suggère que les distributions sont identiques. 
En pratique, le comportement de la divergence de KL est **très similaire à celui de l'entropie croisée**. 
Elle calcule la quantité d'informations perdues (en termes de bits) 
si la distribution de probabilité prédite est utilisée pour se rapprocher de la distribution de probabilité cible souhaitée.

<img title="" src="file:///./Ressources/prez/modloss.png" alt="" data-align="center">

## Test de la librairie "Albumentations"

Cette librairie m'a interpelée dans le sens où elle procure un large spectre d'outils permettant de mettre en œuvre le concept d'**augmentation d'images**.
En particulier, j'y ai découvert des outils spécifiques pour simuler le mauvais temps comme le brouillard, la pluie, la neige...

Voici le résultat de quelque un de nos testes :

L'effet miroir verticale :

<img title="" src="file:///./Ressources/prez/flip.png" alt="" data-align="center">

Ajout de brouillard :

<img title="" src="file:///./Ressources/prez/fog.png" alt="" data-align="center">

Simulation de pluie :

<img title="" src="file:///./Ressources/prez/rain.png" alt="" data-align="center">

Il est à noter que ces tests ont été réalisés avec un ajout de **10%** d'images générées selon les méthodes désignées.

## Dernier test avec l'ensemble des paramètres optimaux

Dernière expérimentation avec l'ensemble des paramètres optimaux
Ici nous avons un entrainement du Model VGG U-Net, <br>avec une fonction de perte "divergence de Kullback-Leibler" <br>et un ajout de 10% d'images générées par simulation de pluie.

<img title="" src="file:///./Ressources/prez/optimalIoU.png" alt="" data-align="center">

# 2. Synthèse des résultats

Ci-dessous l'évolution des scores d'IoU pour les 3 Models testés.

<img title="" src="file:///./Ressources/prez/iouDes3ModelsTests.png" alt="" data-align="center">

Ici, toujours l'évolution de du score IoU au cours de l'entrainement, pour le test de 2 fonctions de pertes. 
La Divergence KL est réputée meilleur pour les la classification de classes multiple, dense... 
Ce qui est adéquat pour notre problème de segmentation d'image qui est dense dans le sens 
où c'est chaque pixel qui doit être classifié parmi nos 8 classes! 
Cela se vérifie bien dans cette expérience.

<img title="" src="file:///./Ressources/prez/iouChangementLoss.png" alt="" data-align="center">

Les augmentations ne semblent pas apporter beaucoup...
Plusieurs hypothèses peuvent être prises en compte:

1. les images de validation sont réputées pour être prisent par temps clair ou nuageux, **mais pas**, à priori par temps de brouillard ou de pluie forte... 
   De plus elles proviennent uniquement d'Allemagne, donc l'effet miroir manque d’intérêt. En effet en Angleterre ou le sens de circulation est inversé, nous aurions peu être plus de succès? 
2. Qui dit, plus d'images, dit plus d'information à traiter. Peut-être qu'ici le paramètre "patience" aurait pu être augmenté et ainsi permettre d'entrainer bien plus d'Epochs?

<img title="" src="file:///./Ressources/prez/comparaisonAugment.png" alt="" data-align="center">

Nous voyons, en conclusion, que les travaux d'optimisations ont portés leurs fruits.
Nous avons une précision équivalente pour un temps d'entrainement plus long, 
mais nous gagnons **3%** au score IoU ce qui est une bonne chose dans le sens où cette métrique est mieux adaptée à notre problème.
De plus, avec de nouvelles images, plus diversifiées en pays et en météo, nous pourrions espérer encore améliorer notre Model !

<img title="" src="file:///./Ressources/prez/modelBasVsModelOpti.png" alt="" data-align="center">

En Tableau :

| Implémentation       | Temps de Fit | IoU   | Dice  | Accuracy |
|:-------------------- |:------------:|:-----:|:-----:|:--------:|
| Divergence KL        | 4:33:0       | 0.784 | 0.878 | 0.901    |
| ---                  | ---          | ---   | ---   | ---      |
| Sans augmentations   | 3:43:0       | 0.764 | 0.865 | 0.899    |
| Miroir Vertical      | 2:53:0       | 0.738 | 0.848 | 0.888    |
| Brouillard           | 3:37:0       | 0.751 | 0.857 | 0.890    |
| Pluie                | 4:24:0       | 0.773 | 0.872 | 0.893    |
| ---                  | ---          | ---   | ---   | ---      |
| Toutes optimisations | 10:38:0      | 0.793 | 0.884 | 0.899    |

En images :

Une prédiction avec un Vgg U-Net de base :

<img title="" src="file:///./Ressources/prez/maskModelBase.png" alt="" data-align="center">

Le masque donné :

<img title="" src="file:///./Ressources/prez/maskTruth.png" alt="" data-align="center">

Et pout conclure, la prédiction avec le Model optimisé :

<img title="" src="file:///./Ressources/prez/maskModelOpti.png" alt="" data-align="center">

# 3. La mise en production

<img src="file:///./Ressources/prez/Mise%20en%20production.svg" title="" alt="" data-align="center">

Flask est une librairie, dite "côté backend", c'est à dire côté serveur.
Celle-ci permet de générer des pages encodées en HTML/JavaScript/CSS à partir de "template" 
( quelle que chose que nous pourrions comparer à un squelette HTML ) 
et d'entrées/sorties interfacées en Python.
Dans sa plus simple expression, le package est constitué de 3 fichiers Python : 

    __init__.py
    app.py
    views.py

et d'un fichier html :

    index.html

## Mise en place d'une API avec Heroku :

Pour cela, il est nécessaire de créer un compte gratuit chez Heroku.
A partir de là, nous avons 2 options de déploiements:

* Passer par GitHub
* Passer par ce qui parait être le Docker Hub.

Nous prendrons la 2eme option car la 1er nous limitera à 500Mo d'espace disque distant et le module Tensorflow en consomme déjà 2 fois plus...

L'étape suivant consiste à utiliser leur exécutable, "heroku", et préparer quelques fichiers de configuration:

1. **Dockerfile** :
   
   ```dockerfile
   # récupération de l'image de départ
   FROM ubuntu:latest
   
   # Installation de pip
   RUN apt update && apt install -y python3-pip
   
   # Copie des fichiers vers le container
   ADD ./inferenceEngine/requirements.txt /tmp/requirements.txt
   
   # installation des dépendances Python
   RUN pip3 install --no-cache-dir -q -r /tmp/requirements.txt
   
   # Copie de notre code
   ADD ./inferenceEngine /opt/inferenceEngine/
   WORKDIR /opt/inferenceEngine
   
   # "$PORT" est défini par Heroku
   CMD gunicorn --bind 0.0.0.0:$PORT wsgi
   ```

2. **requirements.txt** (permettant d'installer les modules nécessaires au bon fonctionnement de notre API):
   
   ```requiements.txt
   flask
   gunicorn
   tensorflow
   numpy
   opencv-python-headless
   ```

Enfin il faudra lancer quelques lignes de commandes, afin de mettre en ligne, notre API. Ces lignes seront à lancer à la racine du dossier contenant notre projet :

![](C:\MainDoc\GitCode\lXpCenter\02InnovLab\FormationIngeIA\08Projet\Ressources\prez\dossierAPI.png)

Cette première commande permet de s'identifier au service Heroku:

```shell
heroku login
```

Ici nous nous identifions au service de Docker Hub :

```shell
heroku container:login
```

Maintenant nous allons créer un nouveau service distant. En d'autres termes, cette commande permet d'initialiser le système sur les serveurs d'Heroku, dans notre espace personnel.

```shell
heroku create
```

Il est temps, de charger nos fichiers sur le Docker Hub. A la commande précédente, il nous sera retourné un nom: "nom_projet", qu'il faudra renseigner à la place des Brackets :

```shell
heroku container:push web -a [nom du projet, retourné à la commande précédente]
```

Enfin nous n'avons plus qu'à publier, dans le sens de rendre accessible et publique, notre API :

```shell
heroku container:release web -a [nom_projet]
```

Il est à noter, qu'il est nécessaire pour cette procédure d'avoir **installé**, auparavant, sur son ordinateur, **Docker**. Les commandes seront aussi à lancer avec les droits **d'administrateur**.

Voici le code qui permettra de tester cette API:

```python
headers = {"Content-Type": "application/json"}
r = requests.post('https://sleepy-reaches-84880.herokuapp.com/rest/',
    data=srzImg, headers=headers)
mask_predicted = deSerializ(r.text)
```

Nous pouvons y apercevoir l'adresse de "EndPoint" constituée ainsi:

```url
https://[nom_projet].herokuapp.com/rest/
```

## Mise en place d'une Web App sur Microsoft Azure

Il est possible d'héberger, sur la plateforme Azure gratuitement, une page web active.
Ici nous allons mettre en place une application, sur la base de Flask, qui permettra de choisir un identifiant d'image, de la sérialiser, afin de la transmettre à notre API précédemment mise en œuvre.
Après retour de notre API, cette application Web affichera 3 images: 

* l'original dont nous devons prédire la segmentation selon nos 8 classes
* le masque fournit pour l'entrainement, montrant la segmentation tel que nous l'attendons
* la segmentation prédite par notre API

Comme prérequis à cette opération, il est nécessaire d'avoir Git, installé sur son poste de travail...
Voici les étapes permettant de mettre "en production" cette Web App:

1. Créer un service "Web App" Pour ceci le moteur de recherche intégré nous aidera à la trouver

<img src="file:///./Ressources/prez/rechercheWebApp.png" title="" alt="" data-align="center">

2. Paramétrer la création du service, en prenant soin de sélectionner "Free F1" dans le champ "sku and size":
   
   <img src="file:///./Ressources/prez/Create%20Web%20App%20-%20Microsoft%20Azure.png" title="" alt="" data-align="center">

3. Voici les étapes permettant d'utiliser Git avec notre Web App, afin de la publier:
   
   <img src="file:///./Ressources/prez/webapp%20Git.png" title="" alt="" data-align="center">
   
   <img src="file:///./Ressources/prez/webapp%20Git1.png" title="" alt="" data-align="center">
   
   <img src="file:///./Ressources/prez/webapp%20Git2.png" title="" alt="" data-align="center">
   
   Cette dernière étape, sur le portail Azure, montre comment récupérer les identifiants (2), quand ils seront demandés par l'interface de connexion aux services de GitHub

4. Ensuite utiliser un Shell pour lancer les commandes nécessaires à la transmission de notre site web dynamique vers GitHub:

```shell
git init
git add .
git commit -m "Initial commit"
git remote add azure https://p08webapp.scm.azurewebsites.net:443/P08webapp.git
git push azure master
```

Si tout c'est bien passé notre application web devrait être disponible à l'adresse:

```url
https://p08webapp.azurewebsites.net
```

# 4. Annexes

## Logiciels utilisés

* VSCodium

* diagrams.net

* MarkText

* Grammalecte

* FreePlane

* Gimp

* GreenShot

## Sites d'inspirations

* [How to Use Metrics for Deep Learning with Keras in Python](https://machinelearningmastery.com/custom-metrics-deep-learning-keras-python/)
* https://datascientest.com/u-net
* https://machinelearningmastery.com/tensorflow-tutorial-deep-learning-with-tf-keras/
* https://machinelearningmastery.com/convolutional-layers-for-deep-learning-neural-networks/
* https://saifgazali.medium.com/retina-blood-vessel-segmentation-using-vgg16-unet-7262f97e1695
* https://machinelearningmastery.com/loss-and-loss-functions-for-training-deep-learning-neural-networks/
* [Divergence de Kullback-Leibler — Wikipédia](https://fr.wikipedia.org/wiki/Divergence_de_Kullback-Leibler)
* [How to Choose Loss Functions When Training Deep Learning Neural Networks](https://machinelearningmastery.com/how-to-choose-loss-functions-when-training-deep-learning-neural-networks/)
* [Intersection over Union (IoU) for object detection - PyImageSearch](https://pyimagesearch.com/2016/11/07/intersection-over-union-iou-for-object-detection/)
* [Mask augmentation for segmentation - Albumentations Documentation](https://albumentations.ai/docs/getting_started/mask_augmentation/)
* [GitHub - UjjwalSaxena/Automold--Road-Augmentation-Library: This library augments road images to introduce various real world scenarios that pose challenges for training neural networks of Autonomous vehicles. Automold is created to train CNNs in specific weather and road conditions.](https://github.com/UjjwalSaxena/Automold--Road-Augmentation-Library)
* [Démarrage rapide : Déployer une application web Python (Django ou Flask) sur Azure - Azure App Service | Microsoft Docs](https://docs.microsoft.com/fr-fr/azure/app-service/quickstart-python?tabs=flask%2Cwindows%2Cazure-portal%2Cterminal-powershell%2Clocal-git-deploy%2Cdeploy-instructions-azportal%2Cdeploy-instructions-zip-curl#3---deploy-your-application-code-to-azure)
* [Déployer à partir d’un dépôt Git local - Azure App Service | Microsoft Docs](https://docs.microsoft.com/fr-fr/azure/app-service/deploy-local-git?tabs=portal)
