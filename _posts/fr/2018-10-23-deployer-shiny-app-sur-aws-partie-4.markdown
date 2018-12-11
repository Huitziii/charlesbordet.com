---
layout: single
permalink: /fr/shiny-aws-4/
title: "Déployer Shiny sur AWS - Comment envoyer l'application sur votre serveur ?"
description: "Apprenez à déployer par vous-même une application Shiny sur un serveur AWS. C'est simple, gratuit, et expliqué dans cet article ! Partie 4 : Après avoir créé le serveur et installé Shiny Server, on déploie l'application et on la rend accessible à tous !"
excerpt: "On poursuit la série sur 'Comment déployer une application Shiny sur AWS' avec ce 4e et dernier article où enfin, vous allez voir votre application apparaître sur les internets !"
date: 2018-12-08
lang: fr
categories: [shiny, fr]
ref: shiny-server-aws-part4
header:
    image: /assets/images/shiny-server-aws-part4/featured.jpg
toc: true
toc_sticky: true
---

{% include tableofcontents/shiny-server-aws-fr.markdown %}

Rappelons les étapes que nous avons déjà faites.

D'abord, on a créé une application Shiny et on l'a envoyée sur Github. Ensuite, on a créé un compte sur AWS pour ouvrir un serveur, sur lequel on a préparé le terrain en installant R et R Shiny.

À la fin du précédent article, je vous disais que vous avez *déjà* une appli Shiny qui tourne sur votre serveur !

En effet, elle est apparue en installant R Shiny.

## 1. Comment accéder à l'application Shiny installée par défaut ?

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

## 2. Comment ajouter votre application sur votre serveur ?

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

## 3. Comment configurer le Shiny Server ?

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

* La première ligne `run_as shiny` indique l'utilisateur Ubuntu qui est derrière le Shiny Server. Quand vous vous connectez au serveur via SSH, vous dirigez l'utilisateur `ubuntu`, celui par défaut. Quand on a installé Shiny Server, il a créé un nouvel utilisateur, qui s'appelle `shiny`, qui est celui qui va faire tourner le serveur. C'est important pour la suite.
* Ensuite, on a `listen 3838`, ça veut dire qu'il faut spécifier le port 3838 pour accéder à l'appli. C'est justement ce qu'on a fait quand on a tapé l'adresse dans notre navigateur.
* La ligne `site_dir /srv/shiny-server` indique l'endroit où se trouve les applis Shiny. Et effectivement, c'est là qu'on a placé la notre. En tout cas, on y a mis un raccourci.
* Finalement, `log_dir /var/log/shiny-server` spécifie l'endroit où se trouve les logs. Ça c'est super utile, puisque ça va nous permettre de comprendre pourquoi notre appli a planté. On va aller les voir juste après.
* La dernière ligne, `directory_index on`, comme le commentaire l'indique, permettre de montrer l'arborescence des dossiers à la racine. C'est ce qu'on voit quand je vais directement sur `3.121.42.9:3838`. Moi je trouve ça un peu moche, du coup j'ai tendance à le désactiver : `directory_index off`. En plus, c'est mieux si on préfère ne pas montrer toutes les applis dont on dispose.

Outre cette modification optionnelle pour la dernière ligne, je vous invite à rajouter une ligne tout en haut du fichier de config : `preserve_logs true;`.

Cette ligne permet de garder les logs en toute circonstance.

En effet, Shiny a la mauvaise habitude de supprimer les logs quand il considère qu'il ne s'est rien passé d'intéressant. Et parfois, notre appli bug, on veut voir les logs... et ils ne sont pas là ! Cette ligne permet d'éviter ce problème.

**Important** : Lorsque vous changez la config, il faut recharger le Shiny Server avec la commande suivante afin que les changements soient pris en compte :

{% highlight shell %}
$ sudo systemctl reload shiny-server
{% endhighlight %}

D'ailleurs, en parlant de logs... allons voir pourquoi notre appli plante !

## 4. Comment débugger une appli Shiny à partir des logs

Pour aller voir les logs :

{% highlight shell %}
$ cd /var/log/shiny-server
$ ls
movie-explorer-shiny-20181210-080534-46879.log rmd-shiny-20181023-144836-40079.log
{% endhighlight %}

Je vois deux fichiers dans la liste. 

Celui nommé `movie-explorer` correspond à notre application. L'autre est l'application de base par défaut.

Les chiffres dans les noms des fichiers correspondent à la date. Donc le premier fichier a été créé le 2018-12-10 à 08:05:34. Quand les fichiers de logs commencent à s'accumuler, c'est pratique pour s'y retrouver.

Si jamais vous ne voyez pas de fichier, essayez de relancer l'application. C'est le problème des logs qui disparaissent que je mentionnais juste avant. Mais il sera résolu avec la nouvelle config.

J'affiche les dernières lignes du fichier de log et je vois :

{% highlight shell %}
$ sudo tail movie-explorer-shiny-20181210-080534-46879.log
Warning message:
replacing previous import by ‘Rcpp::evalCpp’ when loading ‘later’ 

Listening on http://127.0.0.1:44533
Warning: Error in library: there is no package called ‘ggvis’
  48: stop
  47: library


Execution halted
{% endhighlight %}

Mais oui !

On n'a pas installé les libraries nécessaires pour le fonctionnement de l'appli.

C'est évident maintenant qu'on le voit.

Mais dites-vous que c'est une source d'erreur très fréquente. On rajoute une librairie dans une appli, tout roule bien en local, on met à jour, et là... ça plante, on comprend plus pourquoi. Juste parce qu'on n'a pas pensé à installer le package sur le serveur.

**Attention** : Comme le Shiny Server est géré par l'utilisateur `shiny`, il faut installer les packages soit avec l'utilisateur `shiny`, soit avec le superuser. Je vous conseille plutôt de le faire avec l'utilisateur `shiny`, de cette façon :

{% highlight shell %}
$ sudo su - shiny
$ R
> install.packages("ggvis")
{% endhighlight %}

Il va vous demander si vous voulez utiliser une bibliothèque locale, répondez que oui.

Une fois le package installé, ré-essayez de charger l'application. 

Cette fois-ci, vous allez voir l'appli apparaître, et très rapidement crash (elle devient grisée).

<a href="{{ "/assets/images/shiny-server-aws-part4/movie_explorer_crash.png" | absolute_url }}"><img src="{{ "/assets/images/shiny-server-aws-part4/movie_explorer_crash.png" | absolute_url }}" width="50%" class="align-center"></a>

Encore raté !

Si on retourne rapidement dans les logs, on s'aperçoit qu'un nouveau fichier s'est créé, et il nous indique :

{% highlight shell %}
$ sudo tail movie-explorer-shiny-20181211-083747-41773.log
The following objects are masked from ‘package:base’:

    intersect, setdiff, setequal, union

Warning: Error in : The dbplyr package is required to communicate with database backends.
  56: <Anonymous>
Error : The dbplyr package is required to communicate with database backends.


Execution halted
{% endhighlight %}

Il manque un autre package ! `dbplyr`.

**Note**: Après avoir installé le package pour l'utilisateur `shiny`, n'oubliez pas de revenir à l'utilisateur principal `ubuntu` en tapant `exit` dans la console.
{: .notice--warning}

Je vous laisse installer le package, vous savez comment faire, à présent. Bon, et puis autant vous le dire tout de suite, il faut aussi installer `RSQLite`

Une fois installés...

<a href="{{ "/assets/images/shiny-server-aws-part4/movie_explorer_final.png" | absolute_url }}"><img src="{{ "/assets/images/shiny-server-aws-part4/movie_explorer_final.png" | absolute_url }}" width="50%" class="align-center"></a>

Finalement !

On y arrive.

Après avoir :

* Créé votre serveur sur AWS
* Installé R et R Shiny dessus
* Envoyé votre application sur le serveur
* Débuggé l'appli à partir des logs
* Et configuré le serveur...

Enfin on peut jouer avec notre appli à partir de n'importe où dans le monde ! Et surtout, on peut la partager à d'autres.

Par contre, vous avez peut-être encore quelques questions, comme...

* Comment se débarrasser de cette adresse IP moche et la remplacer par www.mabelleurl.fr ?
* Comment restreindre l'accès à l'application avec un mot de passe ?
* Comment sécuriser l'application en HTTPS ?
* Ou encore, une fois que vous avez une bonne application qui tourne, comment améliorer la performance pour que des dizaines, voire des centaines d'utilisateurs, puissent y accéder simultanément ?

C'est ce que nous verrons dans les prochains articles...