---
layout: single
permalink: /fr/shiny-aws-4/
title: "Déployer Shiny sur AWS - Comment envoyer l'application sur votre serveur ?"
description: "Apprenez à déployer par vous-même une application Shiny sur un serveur AWS. C'est simple, gratuit, et expliqué dans cet article ! Partie 4 : Après avoir créé le serveur et installé Shiny Server, on déploie l'application et on la rend accessible à tous !"
excerpt: "On poursuit la série sur 'Comment déployer une application Shiny sur AWS' avec ce 4e et dernier article où enfin, vous allez voir votre application apparaître sur les internets !"
date: 2018-12-14
lang: fr
categories: [shiny, fr]
ref: shiny-server-aws-part4
header:
    image: /assets/images/shiny-server-aws-part4/featured.jpg
---

{% include tableofcontents/shiny-server-aws-fr.markdown %}

Rappelons les étapes que nous avons déjà faites.

D'abord, on a créé une application Shiny et on l'a envoyée sur Github. Ensuite, on a créé un compte sur AWS pour ouvrir un serveur, sur lequel on a préparé le terrain en installant R et R Shiny.

À la fin du précédent article, je vous disais que vous avez *déjà* une appli Shiny qui tourne sur votre serveur !

En effet, elle est apparue en installant R Shiny.

## Comment accéder à l'application Shiny installée par défaut ?

Pour y accéder, deux étapes :

* Trouver l'adresse IP de votre serveur
* Ouvrir le port 3838 sur le pare-feu

Dès que vous activez votre serveur sur AWS, une adresse IP y est affectée automatiquement. 

Attention, celle-ci change dès que vous éteignez/rallumez le serveur ! Si vous voulez avoir une IP fixe, il faudra aller voir dans "Elastic IP" et allouer une adresse IP directement à votre serveur.

Dans votre dashboard sur EC2, il y a une colonne **IPv4 Public IP**.

Repérez-la et notez l'adresse IP.

<a href="{{ "/assets/images/shiny-server-aws-part4/ip_address.png" | absolute_url }}"><img src="{{ "/assets/images/shiny-server-aws-part4/ip_address.png" | absolute_url }}" width="100%"></a>

Chez moi, c'est *3.121.42.9*

Pour accéder à l'application Shiny, il suffit alors simplement de taper l'adresse IP dans votre navigateur préférée en spécifiant le port réservé à Shiny : 3838

{% highlight shell %}
3.121.42.9:3838
{% endhighlight %}

Sauf que... si je fais ça, la page ne se charge pas. 

![Firewall AWS]({{ "/assets/images/shiny-server-aws-part4/firewall.png" | absolute_url }}){: .align-center}

En effet : **Par défaut, AWS n'autorise que le port 22**, c'est-à-dire le port réservé à SSH, le protocole qu'on a utilisé dans l'article précédent pour se connecter au serveur via la console.

Pour changer ça, à partir du dashboard EC2 :

* Sélectionnez votre instance.
* Dans le cadre du bas, repérez la ligne *Security groups* et cliquez sur le premier lien.
* Dans la nouvelle fenêtre, dans le cadre du bas, cliquez sur "Inbound", le deuxième onglet. Vous voyez alors que seulement le port 22 apparaît.
* Cliquez sur "Edit", puis dans le pop-up, "Add Rule". Les paramètres à entrer sont les suivants :

| Type            | Protocol | Port Range | Source            | Description |
| --------------- | -------- | ---------- | ----------------- | ----------- |
| SSH             | TCP      | 22         | Custom: 0.0.0.0/0 |             |
| Custom TCP Rule | TCP      | 3838       | Custom: 0.0.0.0/0 |             |

Voici quelques screenshots si jamais vous êtes perdu :

<a href="{{ "/assets/images/shiny-server-aws-part4/open_port.png" | absolute_url }}"><img src="{{ "/assets/images/shiny-server-aws-part4/open_port.png" | absolute_url }}" width="100%"></a>

.

<a href="{{ "/assets/images/shiny-server-aws-part4/open_port-2.png" | absolute_url }}"><img src="{{ "/assets/images/shiny-server-aws-part4/open_port-2.png" | absolute_url }}" width="100%"></a>

À présent, on a ouvert le port 3838 et l'application est accessible !

Ré-essayons d'y accéder :

{% highlight shell %}
3.121.42.9:3838
{% endhighlight %}

Cette fois-ci...

![Default app]({{ "/assets/images/shiny-server-aws-part4/default_app.png" | absolute_url }}){: .align-center}

Boom !

Ça marche ! Top cool !

C'est une page html avec des widgets shiny sur le côté droit de la page.

D'ailleurs, vous noterez peut-être un message d'erreur dans un des widgets : "An error has occurred". C'est tout simplement un problème de package qui n'est pas installé.

Ça c'est chouette mais.. le but c'est de mettre notre application maintenant !

## Comment ajouter votre application sur votre serveur ?

Pour ça, on va devoir retourner dans la console sur notre serveur.

Si vous avez déjà oublié comment on faisait, retournez voir l'article précédent [Installer R et R Shiny sur le serveur]({{ "/fr/shiny-aws-3" | absolute_url }}).

Dans un premier temps, on va explorer un peu notre serveur.

On vient tout juste de voir que quand on installe Shiny Server pour la première fois, une application par défaut s'installe et nous souhaite la bienvenue !

Où se trouve cette application ?

Elle se trouve tout simplement dans le dossier `/srv/shiny-server/`, et c'est là que seront stockées toutes vos applications.

Tapez les commandes suivantes dans votre console :

{% highlight shell %}
$ cd /srv/shiny-server/
$ ls
index.html  sample-apps
{% endhighlight %}

Vous allez voir juste un seul dossier : `sample-apps`, et un fichier : `index.html`.

Le fichier `index.html` correspond à ce que nous avons vu lorsque nous sommes allés nous connecter sur la page web de notre serveur, et il fait appel à l'application qui se trouve dans `sample-apps`. Si on va voir dedans :

{% highlight shell %}
$ cd sample-apps/hello
$ ls
server.R  ui.R
{% endhighlight %}

On retrouve nos fidèles `server.R` et `ui.R` qui sont la base de toute application Shiny !

Par défaut, Shiny Server va aller chercher les applications qui se trouvent dans ce dossier. Il est possible de le changer à partir du fichier de config qui se trouve dans `/etc/shiny-server/shiny-server.conf` mais je vous le déconseille.

Ce qu'on va plutôt faire, c'est :

1. Télécharger notre application dans un dossier de travail.
2. Créer un raccourci dans `/srv/shiny-server/`.

C'est généralement la pratique qui est recommandée.

Alors au travail !

D'abord, on retourne dans le dossier de base :

{% highlight shell %}
$ cd
{% endhighlight %}

Ensuite, on **clone** notre application qui se trouve sur Github :

{% highlight shell %}
$ git clone https://github.com/Huitziii/movie-explorer.git
Cloning into 'movie-explorer'...
remote: Enumerating objects: 9, done.
remote: Total 9 (delta 0), reused 0 (delta 0), pack-reused 9
Unpacking objects: 100% (9/9), done.
Checking connectivity... done.
{% endhighlight %}

Parfait ! Cette simple commande nous a permis de télécharger l'appli sur le serveur.

Sauf que Shiny s'attend à ce que l'appli se trouve dans le dossier bien spécial `/srv/shiny-server/`. Donc on va créer un raccourci là-bas :

{% highlight shell %}
$ cd /srv/shiny-server
$ sudo ln -s ~/movie-explorer .
{% endhighlight %}

À présent, si on affiche la liste des fichiers, on va voir que notre raccourci a bien été créé :

{% highlight shell %}
$ ls
index.html  movie-explorer  sample-apps
{% endhighlight %}

D'ailleurs, au passage, je vous propose de supprimer l'appli installée par défaut :

{% highlight shell %}
$ sudo rm index.html
$ sudo rm -R sample-apps
{% endhighlight %}

Eh bien ça y est !

On a ajouté notre application dans le bon dossier, à présent elle devrait être accessible sur les internets.

Testons !

Si je tape `3.121.42.9:3838` dans la barre de mon navigateur, je tombe sur...

![Root folder]({{ "/assets/images/shiny-server-aws-part4/root.png" | absolute_url }}){: .align-center}

Oops, pas très joli ça. On va voir après comment le changer.

Cliquons sur le lien, et...

`ERROR: An error has occurred. Check your logs or contact the app author for clarification.`

Eh :(

...

Bon.

Il semblerait que le serveur reconnaisse bien qu'on a ajouté notre application, mais elle ne marche pas !

Pourquoi ?

## Comment configurer le Shiny Server ?

Il va falloir changer un petit peu la config.

Le fichier de configuration de Shiny Server se trouve dans `/etc/shiny-server/shiny-server.conf`. 

On va l'ouvrir et essayer de comprendre :

{% highlight shell %}
sudo nano /etc/shiny-server/shiny-server.conf
{% endhighlight %}

Le fichier ressemble à ça :

	# Instruct Shiny Server to run applications as the user "shiny"
	run_as shiny;

	# Define a server that listens on port 3838
	server {
	  listen 3838;

	  # Define a location at the base URL
	  location / {

	    # Host the directory of Shiny Apps stored in this directory
	    site_dir /srv/shiny-server;

	    # Log all Shiny output to files in this directory
	    log_dir /var/log/shiny-server;

	    # When a user visits the base URL rather than a particular application,
	    # an index of the applications available in this directory will be shown.
	    directory_index on;
	  }
	}

Je vais vous expliquer ligne par ligne comment le comprendre.




	J'explique d'abord où se trouve l'application par défaut.
	Je montre le fichier de configuration.
	Je montre comment télécharger l'application sur le serveur dans le dossier qu'on veut.
	Je montre comment télécharger les packages nécessaires pour l'utilisateur shiny.