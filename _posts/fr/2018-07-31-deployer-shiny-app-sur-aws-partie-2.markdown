---
layout: single
permalink: /fr/shiny-aws-2/
title: "Comment déployer une application Shiny sur AWS - Partie 2"
date: 2018-07-31
lang: fr
ref: shiny-server-aws-part2
header:
    image: /assets/images/shiny-server-aws-part2/featured.jpg
---

{% include tableofcontents/shiny-server-aws-fr.markdown %}

La dernière fois, nous nous étions arrêtés à la création de l'application Shiny. Pour rappel, elle est disponible sur Github à l'adresse suivante : [https://github.com/Huitziii/movie-explorer]("https://github.com/Huitziii/movie-explorer")

Aujourd'hui, nous allons **créer un serveur AWS afin d'héberger notre application sur les internets**.

## Créer votre compte gratuitement sur AWS

Pour ça, la première étape est de vous créer un compte sur AWS. 

Rendez-vous d'abord sur [https://aws.amazon.com]("https://aws.amazon.com/") et créez un compte. Les étapes sont simples à suivre donc je vous laisse faire.

Notez qu'Amazon va vous demander un moyen de paiement, puisqu'à chaque fois que vous utiliserez un service payant, il faudra bien le régler.

Mais rassurez-vous, lorsque vous ouvrez un nouveau compte, certains services (limités, certes) sont gratuits afin que vous puissiez vous faire la main sans soucis !

À noter aussi qu'il est possible de réserver un serveur pour très peu d'argent. À titre informatif, mon site aujourd'hui est hébergé sur AWS et il ne m'en coûte que 10$ par mois. J'y héberge aussi mes applications Shiny, un RStudio Server, un Jupyter Notebook, et encore d'autres services.

## Créer votre premier serveur sur EC2

Une fois votre compte créé, identifiez-vous, et vous arriverez sur une page avec tous les services proposés par AWS :

![Services AWS]({{ "/assets/images/shiny-server-aws-part2/aws-services.jpg" | absolute_url }}){: .align-center}

Dans le champ de recherche en haut, tapez *EC2*.

![Kesako]({{ "/assets/images/shiny-server-aws-part2/kesako.jpg" | absolute_url }}){: .align-right}

EC2, ça veut dire Elastic Compute Cloud. En français, un nuage élastique qui fait des calculs.

En fait, il s'agit d'un service qui permet de louer des ressources d'un serveur afin de :

* créer des services web (comme héberger un site, ou ... une appli Shiny !)
* faire des calculs (comme faire tourner un algorithme de machine learning)
* stocker des données

AWS dispose d'un service plus adapté pour stocker des données, comme S3, mais quand on veut héberger un site ou faire tourner un modèle, il faut bien stocker les données et les fichiers du site quelque part !

Le gros atout de ce service, c'est qu'on peut très facilement changer la configuration du serveur.

Peut-être qu'aujourd'hui vous avez besoin d'une petite machine pas trop gourmande. Mais demain, quand le monde entier ira visiter votre appli Shiny, vous devrez augmenter la RAM ou la puissance du processeur. C'est possible en seulement quelques clics.

Ou bien imaginez que vous souhaitiez faire tourner un algorithme de Deep Learning mais votre petit laptop n'est pas assez puissant.

Louez une machine bien balèze, importez vos données, faites tourner l'algorithme, exportez vos résultats, éteignez la machine, et hop! le problème est réglé.

Vraiment pratique.

Bref, je parle, je parle, en attendant on a toujours pas de serveur. 

**Créons le serveur !**

Dans le champ de recherche, tapez *EC2* et entrez dans le service.

À gauche, dans le menu, cliquez sur Instances. Une instance, c'est un serveur.

Puis ensuite, cliquez sur *Launch Instance*

## Étape 1 : Choisir votre AMI

![Choisir votre AMI]({{ "/assets/images/shiny-server-aws-part2/ami.jpg" | absolute_url }}){: .align-center}

Une AMI, c'est une *Amazon Machine Image*, c'est-à-dire une sorte de serveur pré-packagé avec déjà plein de choses prêtes à l'intérieur. Il y a des AMIs préparées par Amazon, on peut en créer nous-même, et on peut même en louer à des entreprises tierces (par exemple avec des logiciels propriétaires à l'intérieur).

Nous, on va prendre une AMI de base, la *Ubuntu Server 16.04 LTS*.

Sélectionnez-là en cliquant sur Select. 

## Étape 2 : Choisir votre type d'instance

![Choisir votre type d'instance]({{ "/assets/images/shiny-server-aws-part2/instances.jpg" | absolute_url }}){: .align-center}

Le type d'instance correspond à la puissance de votre machine.

Est-ce que vous avez besoin d'une bête de course ou d'une petite machine peu puissante ?

Dans ce guide, inutile d'avoir une grosse machine. En fait, on va prendre le type *t2.micro* qui est le seul type d'instance éligible à l'offre gratuite d'AWS. Elle contient un processeur avec 1 seul cœur, seulement 1 GB de RAM, et une connexion internet pas ouf mais suffisante.

Pour en savoir plus sur les différents types d'instance qui existent, vous pouvez aller voir la page [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/) qui répertorie toute la liste.

Ensuite, cliquez sur *Next: Configure Instance Details*.

Dans l'écran suivant, on ne va rien modifier, donc cliquez directement sur *Next: Add Storage*

## Étape 3 : Choisir votre taille de disque dur

![Choisir votre taille de disque dur]({{ "/assets/images/shiny-server-aws-part2/ebs.jpg" | absolute_url }}){: .align-center}

Par défaut, AWS choisit de vous allouer 8 GB d'espace sur votre serveur.

À vous de décider si vous avez besoin de plus ou non.

Notez que pour rester dans l'offre gratuite d'AWS, vous devez allouer un maximum de 30 GB.

À présent, la configuration est terminée ! 

Vous pouvez cliquer sur *Review and Launch*.

Un écran va vous résumer les différentes options que vous avez choisi. Vous pouvez y jeter un œil puis valider le lancement de votre instance en cliquant sur *Launch* !

## Étape 4 : Créez une clé de sécurité

![Créez une clé de sécurité]({{ "/assets/images/shiny-server-aws-part2/keypairs.jpg" | absolute_url }}){: .align-center}

Après avoir cliqué sur *Launch*, AWS va vous demander de créer une paire de clés. Qu'est-ce que c'est que cette histoire de clés ?

Eh bien il s'agit tout simplement de protéger votre serveur à présent ! Comme il est connecté aux internets, s'il n'y avait pas de clé, tout le monde pourrait y accéder et mettre le bazar !

Pas cool.

Du coup, vous allez créer une clé d'accès sous la forme d'un fichier que vous allez garder sur votre ordinateur. À chaque fois que vous voudrez vous connectez sur votre instance, il faudra spécifier ce fichier et hop vous aurez accès !

Important : Ne perdez pas ce fichier. Sinon, vous risquez de vous voir refuser l'accès à votre propre serveur et vous devrez tout recommencer ! 

Choisissez un nom pour votre clé, et téléchargez-la.

## Félicitations, vous avez créé votre premier serveur !

Après quelques minutes, votre instance sera prête et apparaîtra dans la liste de vos instances.

![Nouvelle instance créée]({{ "/assets/images/shiny-server-aws-part2/newinstance.jpg" | absolute_url }}){: .align-center}

Dans le prochain chapitre, on va voir comment s'y connecter en SSH puis on installera R et R Shiny dessus ! Pour y accéder, cliquez ici :

[Partie 3 - Installer R et R Shiny sur votre nouveau serveur]({{ "/fr/shiny-aws-3" | absolute_url }})
