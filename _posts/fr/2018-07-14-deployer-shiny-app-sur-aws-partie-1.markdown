---
layout: single
permalink: /fr/shiny-aws-1/
title: "Comment déployer une application Shiny sur AWS - Partie 1"
date: 2018-07-14
lang: fr
ref: shiny-server-aws-part1
header: 
    image: /assets/images/shiny-server-aws-part1-featured.png
---

{% include shiny-server-aws-fr.markdown %}

Récemment, un de mes clients m'a demandé de lui créer une application en [R Shiny](https://shiny.rstudio.com/) qui permet aux utilisateurs finaux de visualiser tout un tas d'indicateurs et aussi d'interagir avec les données.

Plutôt la base pour une appli Shiny.

J'ai fait ma tambouille de mon côté, j'ai créé le dashboard, et le client était super content !

Sauf que vient toujours le moment où il faut déployer l'application sur les systèmes du client.

Et là, ça se complique.

![De Shiny à AWS]({{ "/assets/images/shiny-server-aws-part1-shiny-to-aws.png" | absolute_url }})

En tant que data scientist, je maîtrise plutôt bien le langage R et il m'arrive régulièrement de créer des dashboards en Shiny pour faire des reportings automatiques, de la data viz, ou même créer un produit où le client interagit directement avec la donnée.

Sauf que… créer un serveur où je peux héberger l'application, la configurer, et ensuite maintenir le serveur à jour, c'est un peu moins dans mes cordes.

## La méthode « galère »

Je vous avoue que la toute première fois où j'ai été confronté au problème, mon équipe et moi avons opté pour une solution toute autre que le déploiement sur un serveur.

J'ai presque honte d'admettre ce qu'on a fait…

**On a installé R et l'appli Shiny sur chaque poste individuel des utilisateurs.**

Voilà.

Oui oui.

Il a fallu aller sur chaque poste, installer R, et après on avait préparé un script BATCH qu'on mettait sur le bureau qui permettait à l'utilisateur de créer un serveur local sur son poste et de lancer l'appli.

Avec le recul, on avait été assez créatif sur le coup.

![Sad Face]({{ "/assets/images/shiny-server-aws-part1-sad-face.png" | absolute_url }}){: .align-right}

Mais cette solution est horrible !

* D'abord, il faut quand même un moyen de centraliser les données.
* Ensuite, ça prend un temps fou d'installer l'appli sur le poste de chaque utilisateur. 
* Et puis, niveau maintenance, c'est juste pas possible.

On n'avait guère le choix parce qu'on était dans une grosse structure avec des accès limités aux serveurs, donc on s'est débrouillé comme on pouvait.

Mais il faut savoir qu'il existe une bien meilleure méthode.

## La méthode « qui marche »

Il s'agit tout simplement de :

* Créer un serveur dans le cloud.
* Envoyer l'application sur le cloud.

Et c'est tout.

Bon, bon, ok, je dis *tout simplement*, mais ce n'est pas si simple.

Au début en tout cas, quand on ne sait pas comment faire.

Aujourd'hui, et dans cette série d'articles, **mon objectif est de vous apprendre exactement comment héberger une application Shiny sur un serveur.**

Alors, comment va-t-on s'y prendre ?

D'abord, on va utiliser AWS, c'est le petit nom de Amazon Web Services, c'est-à-dire tous les services Cloud proposés par Amazon.

Vous pourriez utiliser des services concurrents, comme Azure chez Microsoft, Digital Ocean, Rackspace, etc. 

Peu importe en fait.

Mais on va utiliser AWS parce que c'est ce que je sais utiliser.

Un point important aussi est que quand vous créez un nouveau compte sur AWS, vous avez accès à certains services gratuitement pendant un an.

De la sorte, tout ce qu'on va faire dans ce tutorial est faisable sans débourser un sou.

![Happy Face]({{ "/assets/images/shiny-server-aws-part1-happy-face.png" | absolute_url }}){: .align-right}

Les avantages de cette méthode :

* Vous pouvez déployer l'application sur le serveur de chez vous et votre client aura immédiatement accès.
* C'est facile à maintenir et mettre à jour une fois que tout est installé.
* Vous pouvez facilement moduler la puissance du serveur selon l'usage de l'application et le nombre d'utilisateurs.
* Vous avez la possibilité de sécuriser votre application pour en restreindre l'accès.

## À quoi s'attendre dans cette série d'articles ?

Mon but est de créer des articles de référence afin qu'à chaque fois que vous serez confronté à la problématique *Comment déployer mon appli sur un serveur ?*, vous puissiez venir ici et trouver la réponse.

Jusqu'à ce que vous reteniez comment faire.

Mais je sais d'expérience que c'est le genre de chose qu'on oublie vite, comme on le fait rarement..

Je vais donc expliquer chaque étape avec moults détails, plein de screenshots, des dessins rigolos, et même parfois des vidéos !

Quelles étapes ? Les voici :

1. **Créer une application Shiny.**
2. **Créer un serveur sur AWS.**
3. **Installer R et R Shiny sur votre nouveau serveur.**
4. **Déployer l'application sur le serveur.**

Ces étapes suffiront à obtenir un résultat plus ou moins satisfaisant, mais pas parfait.

En effet, vous devrez taper une adresse IP bizarre pour accéder à votre appli et en plus elle ne sera pas sécurisée.

C'est pourquoi je rajouterai les étapes suivantes en bonus :

5. **Extra : Créer un nom de domaine pour avoir une belle URL.**
6. **Extra : Sécuriser votre application en HTTPS.**
7. **Extra : Protéger votre application avec un mot de passe.**

Et là, on sera paré ! 

D'ailleurs, je vous propose de commencer directement avec l'étape 1 : **Créer une application Shiny**.

C'est pas exactement le but de cet article, du coup ce que je vous propose c'est d'utiliser une appli disponible dans la galerie du site de R Shiny.

J'ai pris celle de l'explorateur de films :

![Movie Explorer]({{ "/assets/images/shiny-server-aws-part1-movie-explorer.png" | absolute_url }}){: .align-center}

Et j'ai créé un repo sur Github pour que vous puissiez la prendre facilement : [https://github.com/Huitziii/movie-explorer]("https://github.com/Huitziii/movie-explorer")

Notez que vous aurez besoin de git et d'une console pour suivre ce tutoriel. Vous pouvez télécharger git à l'adresse suivante : [https://git-scm.com/downloads]("https://git-scm.com/downloads")

Cette fois-ci, on est prêts !

Pour continuer vers l'étape suivante, cliquez ici : 

[Partie 2 - Créer un serveur sur AWS]({{ "/deployer-shiny-app-sur-aws-partie-2" | absolute_url }})
