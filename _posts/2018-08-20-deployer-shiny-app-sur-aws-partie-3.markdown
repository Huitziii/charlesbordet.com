---
layout: single
title: "Comment déployer une application Shiny sur AWS - Partie 3"
date: 2018-08-20
header:
    image: /assets/images/shiny-server-aws-part3-featured.png
---

{% include deployer-shiny-app-sur-aws-toc.markdown %}

La dernière fois, nous nous étions arrêtés à la création du serveur sur AWS. 

Dans cette nouvelle partie, nous allons apprendre à :

* accéder à ce serveur via SSH
* créer un accès plus facile sur le serveur
* installer R et R Shiny Server

## Comment accéder à votre nouveau serveur ?

Une fois votre *instance* créée, AWS vous propose un bouton **Connect**. Bouton qui fait apparaître la fenêtre suivante :

![Bouton Connect]({{ "/assets/images/shiny-server-aws-part3-connect.png" | absolute_url }}){: .align-center}

Ce sont les instructions pour se connecter à votre serveur.

**Pour les utilisateurs de Windows**

Si vous êtes sous Windows, vous aurez besoin de télécharger un client SSH, puisque l'OS n'en propose pas par défaut. Le plus couramment utilisé est PuTTY que vous pouvez télécharger directement à partir de [https://www.putty.org/]("https://www.putty.org/"). 

Ensuite, il va vous falloir transformer la clé que vous avez téléchargée dans l'article précédent (vous savez, le fichier avec l'extension *pem*). 

Le mieux est de suivre les instructions de AWS : [ici]("https://docs.aws.amazon.com/fr_fr/AWSEC2/latest/UserGuide/putty.html")

Mais en gros, les étapes sont :

1. Lancez PuTTYgen.
2. Choisissez RSA.
3. Allez chercher le fichier *pem* que vous avez téléchargé.
4. Enregistrez la clé privée sous le même nom que le fichier *pem* (l'extension sera différente, en *ppk*).

Ensuite, lancez PuTTY puis :

1. Dans Host Name (or IP address), rentrez la valeur qui se trouve au point 4 du screenshot ci-dessus (c'est *ec2-52-59-238-223.eu-central-1.compute.amazonaws.com* pour moi)
2. Port : 22
3. Connection type : SSH
4. Changez de menu et allez dans Connection / SSH / Auth, et chargez la clé *ppk* que vous avez enregistrée il y a quelques instants. 

Cliquez sur "Open", dites "Oui", et une fenêtre console va s'ouvrir ! 

Bravo, vous êtes à présent connecté à votre serveur ! 

**Pour les utilisateurs de Ubuntu ou Mac**

Si vous n'êtes pas sous Windows, c'est plus simple !

Il vous suffit d'ouvrir une console, de vous déplacer jusqu'au répertoire qui contient la clé *pem* puis de taper l'instruction donnée en exemple sur le screenshot.

Par exemple pour moi, c'est :

{% highlight shell %}
charles@blue:~$ cd Downloads/
charles@blue:~/Downloads$ ssh -i "tutorial.pem" ubuntu@ec2-52-59-238-223.eu-central-1.compute.amazonaws.com
{% endhighlight %}

Dites "yes" et vous serez connecté.

Si vous obtenez un message d'erreur "WARNING: UNPROTECTED PRIVATE KEY FILE!", alors il vous faudra changer les droits du fichier avec la commande suivante :

{% highlight shell %}
chmod 600 tutorial.pem
{% endhighlight %}

