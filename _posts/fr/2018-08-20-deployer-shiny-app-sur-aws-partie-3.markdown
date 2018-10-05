---
layout: single
permalink: /fr/shiny-aws-3/
title: "Comment déployer une application Shiny sur AWS - Partie 3"
date: 2018-08-20
lang: fr
categories: shiny
ref: shiny-server-aws-part3
header:
    image: /assets/images/shiny-server-aws-part3/featured.jpg
---

{% include tableofcontents/shiny-server-aws-fr.markdown %}

La dernière fois, nous nous étions arrêtés à la création du serveur sur AWS. 

Dans cette nouvelle partie, nous allons apprendre à :

* accéder à ce serveur via SSH
* installer R et R Shiny Server

## Comment accéder à votre nouveau serveur ?

Une fois votre *instance* créée, AWS vous propose un bouton **Connect**. Bouton qui fait apparaître la fenêtre suivante :

![Bouton Connect]({{ "/assets/images/shiny-server-aws-part3/connect.jpg" | absolute_url }}){: .align-center}

Ce sont les instructions pour se connecter à votre serveur. À présent, on va séparer les instructions selon votre système d'exploitation.

**Pour les utilisateurs de Windows**

Si vous êtes sous Windows, vous aurez besoin de télécharger un client SSH, puisque l'OS n'en propose pas par défaut. Le plus couramment utilisé est PuTTY que vous pouvez télécharger directement à partir de [https://www.putty.org/]("https://www.putty.org/"). 	

Ensuite, il va vous falloir transformer la clé que vous avez téléchargée dans l'article précédent (vous savez, le fichier avec l'extension *pem*) :

![PuTTYgen]({{ "/assets/images/shiny-server-aws-part3/puttygen.jpg" | absolute_url }}){: .align-center}

1. Lancez PuTTYgen (tapez "PuTTYgen" dans votre menu démarrer)
2. Choisissez RSA.
3. Allez chercher le fichier *pem* que vous avez téléchargé (vous aurez besoin d'activer l'option permettant d'afficher tous les types de fichiers)
4. Enregistrez la clé privée sous le même nom que le fichier *pem* (l'extension sera différente, en *ppk*).

Ensuite, lancez PuTTY puis configurez-le avec les informations données par AWS :

![PuTTY config]({{ "/assets/images/shiny-server-aws-part3/putty-config.jpg" | absolute_url }}){: .align-center}

1. Dans Host Name (or IP address), rentrez la valeur qui se trouve au point 4 du screenshot ci-dessus (c'est *ec2-52-59-238-223.eu-central-1.compute.amazonaws.com* pour moi)
2. Port : 22
3. Connection type : SSH
4. Changez de menu et allez dans Connection / SSH / Auth, et chargez la clé *ppk* que vous avez enregistrée il y a quelques instants. 

Cliquez sur "Open", entrez "ubuntu" quand on vous demande **login as**, puis dites "Oui", et une fenêtre console va s'ouvrir ! 

Bravo, vous êtes à présent connecté à votre serveur ! 

![PuTTY console]({{ "/assets/images/shiny-server-aws-part3/putty-console.jpg" | absolute_url }}){: .align-center}

**Pour les utilisateurs de Ubuntu ou Mac**

Si vous n'êtes pas sous Windows, c'est beaucoup plus simple !

Il vous suffit d'ouvrir une console, de vous déplacer jusqu'au répertoire qui contient la clé *pem* puis de taper l'instruction donnée par AWS.

Par exemple pour moi, c'est :

{% highlight shell %}
charles@blue:~$ cd Downloads/
charles@blue:~/Downloads$ ssh -i "tutorial.pem" ubuntu@ec2-52-59-238-223.eu-central-1.compute.amazonaws.com
{% endhighlight %}

Dites "yes" et vous serez connecté.

Si vous obtenez un message d'erreur "WARNING: UNPROTECTED PRIVATE KEY FILE!", alors il vous faudra changer les droits du fichier avec la commande suivante :

{% highlight shell %}
chmod 400 tutorial.pem
{% endhighlight %}

## Comment installer Shiny Server ?

La prochaine étape, c'est d'installer Shiny Server.

Et un premier pré-requis, c'est évidemment d'installer R !

Généralement, les repos d'Ubuntu ne contiennent pas la dernière version de R. Du coup, on va directement ajouter le repo du CRAN de cette manière :

{% highlight R %}
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
sudo add-apt-repository 'deb [arch=amd64,i386] https://cran.rstudio.com/bin/linux/ubuntu xenial/'
{% endhighlight %}

**Note importante** : À l'heure où j'écris cet article, AWS propose toujours des serveurs équipés de Ubuntu 16.04 LTS (c'est-à-dire la version stable de avril 2016). Mais la version Ubuntu 18.04 LTS est sortie et il se pourrait qu'au moment où vous lisiez cet article, votre serveur soit équipé de 18.04. 

Vous pouvez facilement le vérifier en lisant le message d'accueil au moment où vous vous connectez. Chez moi, ça écrit :

{% highlight shell %}
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-1065-aws x86_64)
{% endhighlight %}

Si vous êtes sous Ubuntu 18.04, alors remplacez 'xenial' par 'bionic'

Une fois ces commandes lancées, le repo du CRAN est ajouté. On peut alors installer R :

{% highlight shell %}
sudo apt update
sudo apt install r-base-dev
{% endhighlight %}

Une fois R installé, et en préambule à l'installation de Shiny Server, il va falloir installer le package *shiny*.

Pour ça, lancez R en mode administrateur et installez le package :

{% highlight shell %}
sudo R
> install.packages("shiny")
{% endhighlight %}

Vous pouvez quitter R en tapant *q()* puis choisissez *n*.

Ok, on progresse. Il ne reste plus qu'à installer Shiny Server !

Il va falloir aller télécharger la dernière version d'abord. Rendez-vous sur le site de RStudio à l'adresse suivante : [https://www.rstudio.com/products/shiny/download-server/](https://www.rstudio.com/products/shiny/download-server/) et on va vous donner les instructions tout en bas de la page.

Pour moi, il s'agit de :

{% highlight shell %}
sudo apt-get install gdebi-core
wget https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.7.907-amd64.deb
sudo gdebi shiny-server-1.5.7.907-amd64.deb
{% endhighlight %}

A priori les deux dernières lignes seront différentes pour vous si une nouvelle version de Shiny Server sort entre-temps.

Une fois les commandes lancées, dites "oui", et c'est parti !

Ça y est ! 

Vous ne le savez pas encore, mais vous avez une application Shiny qui tourne sur votre serveur.

*Comment y accéder ?*

C'est ce qu'on verra dans la prochaine partie...

[Partie 4 - Déployer l’application sur le serveur]({{ "/fr/shiny-aws-4" | absolute_url }})