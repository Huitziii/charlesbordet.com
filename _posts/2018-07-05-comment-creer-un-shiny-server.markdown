---
layout: single
title: "Comment créer un shiny server avec AWS"
date: 2018-07-05 18:21:15 +0200
---

On vous demande de créer une application Shiny, et vous faites un joli dashboard avec plein de fonctionnalités qui remplit le cahier des charges qu'on vous a donné, et même plus encore !

Sauf que maintenant, il va falloir partager votre appli à toute votre équipe, ou à l'équipe de votre client.

Comment faire ?

Shiny, c'est super pratique pour lancer l'appli localement sur votre ordinateur. Mais quand il s'agit de le partager via un serveur... on s'aperçoit rapidement que c'est pas vraiment dans nos compétences.

Alors, c'est vrai que parfois on va avoir des devs dans l'équipe qui connaissent bien le fonctionnement des serveurs, mais ils ne savent pas forcément configurer un serveur Shiny, donc il va falloir leur montrer un minimum. 

Et si vous voulez maintenir l'appli, c'est bien de savoir un peu comment ça marche.

**Dans l'article d'aujourd'hui, on va apprendre comment créer un shiny serveur de A à Z en utilisant AWS**. Par étapes, on va :

* Créer une appli shiny toute simple.
* Créer un serveur via AWS.
* Envoyer l'appli sur le serveur grâce à Github.
* Paramétrer le serveur avec nginx.

En option, on pourra même se prendre un nom de domaine du style *exemple.fr* afin de pouvoir facilement partager l'application à votre client !

Prêt ?

# Créer une appli shiny toute simple

