---
layout: single
permalink: /fr/extract-pdf/
title: "Comment extraire et analyser les données de fichiers PDF en R"
description: "R est un outil formidable pour faire du text mining. Encore faut-il pouvoir extraire le texte de vos documents PDF ! Cet article explique exactement comment faire, et aussi comment préparer vos données pour l'analyse !"
date: 2018-10-05
lang: fr
categories: [r, fr]
ref: extract-pdf
header:
    image: /assets/images/extract-pdf/featured.jpg
---

Besoin d'extraire les textes de fichiers PDF ?

Vous êtes au bon endroit !

*Note : Dans cet article, on traite des fichiers PDF qui sont lisibles par la machine. Si ce n'est pas votre cas, je recommande un usage préalable de Adobe Acrobat Pro qui le fera automatiquement pour vous. Ensuite, revenez ici.*

Dans cet article, on va apprendre à :

* Extraire le contenu d'un fichier PDF en R (deux techniques)
* Nettoyer le résultat afin de pouvoir lancer des analyses sémantiques

D'abord je vais expliquer les outils que j'utilise, et ensuite je vous montrerai comment faire sur des exemples.

## Pourquoi des fichiers PDF ?

Lorsque j'ai commencé à travaillé en freelance, j'ai fait pas mal de missions qui consistaient à simplement extraire les données contenues dans des fichiers PDF.

Mes clients avaient en général deux options : Soit le faire manuellement, soit trouver un moyen d'automatiser la tâche.

La première était en général pénible et coûteux (en temps ou en personnel) au vu du nombre de fichiers à extraire. Des milliers au minimum.

Du coup, je les aidais à mettre en place la deuxième option.

Par exemple, un client avait des milliers de factures qui avaient toutes la même structure et il voulait extraire :

* le nombre de produits vendus,
* le montant de chaque facture
* les données de ses clients

Sauf qu'avoir tout en PDF n'était pas pratique.

Lui, il voulait une spreadsheet propre où plus tard il pourra facilement voir qui a acheté quoi, combien, et faire des calculs. Même tout simplement faire sa compta.

Un autre exemple classique est cet autre client qui voulait analyser des documents officiels, par exemple des discours à l'ONU.

Pareil, tout était sous forme PDF, donc j'ai eu besoin de tout extraire.

Alors, par où on commence ?

## Deux techniques pour extraire le texte des fichiers PDF

### Première technique : `pdftools::pdf_text`

La première technique requier l'utilisation du package `pdftools` disponible sur le CRAN :

{% highlight R %}
install.packages("pdftools")
{% endhighlight %}

Un rapide coup d'œil à la [documentation](https://cran.rstudio.com/web/packages/pdftools/pdftools.pdf) vous montrera les quelques fonctions du package.

La plus importante est `pdf_text`. 

Dans cet article, je vais utiliser un article de l'ONU que vous pouvez trouver à l'adresse suivante : [https://github.com/Huitziii/crispy-pdf/raw/master/71_PV.62.pdf](https://github.com/Huitziii/crispy-pdf/raw/master/71_PV.62.pdf)

{% highlight R %}
library(pdftools)
download.file("https://github.com/Huitziii/crispy-pdf/raw/master/71_PV.62.pdf",
              "./71_PV.62.pdf")
text <- pdf_text("./71_PV.62.pdf")
{% endhighlight %}

La fonction `pdf_text` va directement importer le text brut sous la forme d'un vecteur de type *character* avec des espaces pour représenter l'espace vide et des `\n` pour les sauts de ligne.

Avoir toute la page dans un seul élément est pas vraiment super pratique.

Du coup, on va utiliser `strsplit` pour séparer les lignes les unes des autres :

{% highlight R %}
text2 <- strsplit(text, "\n")
head(text2[[1]])
# [1] "                 United Nations                                                                                            A /71/PV.62"  
# [2] "                 General Assembly                                                                                       Official Records"
# [3] "                 Seventy-first session"                                                                                                  
# [4] "                 62nd plenary meeting"                                                                                                   
# [5] "                 Tuesday, 13 December 2016, 10 a.m."                                                                                     
# [6] "                 New York"
{% endhighlight %}

Si vous voulez en savoir plus sur les fonctions du package `pdftools`, je vous recommande la lecture de [Introducing pdftools - A fast and portable PDF extractor](https://ropensci.org/blog/2016/03/01/pdftools-and-jeroen), un article écrit par l'auteur du package lui-même.

### Deuxième technique : Le package `tm`

`tm` c'est le package cible quand on veut faire de l'analyse sémantique en R

{% highlight R %}
install.packages("tm")
{% endhighlight %}

Le côté pratique, c'est qu'il va nous permettre d'importe notre document PDF en R tout en gardant sa structure intact. Et en plus, il sera dans un objet tout prêt pour les analyses à faire ensuite.

Il existe une fonction `readPDF`, mais elle ne permet pas *directement* de lire un fichier PDF comme dans la partie précédente.

À la place, elle va nous permettre de **créer notre propre fonction de lecteur PDF**. L'avantage, c'est qu'on peut utiliser le driver PDF qu'on veut.

Par défaut, il va utiliser `xpdf`, disponible sur [http://www.xpdfreader.com/download.html](http://www.xpdfreader.com/download.html).

Il faut :

* Télécharger l'archive à partir du site web (dans la section *Xpdf tools*)
* La dézipper
* S'assurer qu'elle est dans le PATH du système.

Ensuite, on peut créer notre fonction extracteur de PDF :

{% highlight R %}
library(tm)
read <- readPDF(control = list(text = "-layout"))
{% endhighlight %}

L'argument `control` permet de spécifier les paramètres comme si on était en ligne de commande. Donc là, c'est comme si on avait écrit `xpdf -layout` dans le shell.

À présent, on est prêt à importer le document PDF !

{% highlight R %}
document <- Corpus(URISource("./71_PV.62.pdf"), readerControl = list(reader = read))
doc <- content(document[[1]])
head(doc)
# [1] "                United Nations                                                                                              A /71/PV.62"  
# [2] "                General Assembly                                                                                         Official Records"
# [3] "                Seventy-first session"                                                                                                    
# [4] ""                                                                                                                                         
# [5] ""                                                                                                                                         
# [6] "                62nd plenary meeting"
{% endhighlight %}

La différence majeure par rapport à la première méthode est que de nouvelles lignes vides sont apparues, ce qui est plus fidèle au document original.

Par exemple, ça peut nous aider à savoir quand se termine l'en-tête.

Une autre différence est la manière dont les pages sont gérées. Avec cette méthode, on a directement tout le texte dans l'objet et les sauts de page sont symbolisés par des symboles `\f`. Précédemment, on avait une liste avec 1 page = 1 élément.

{% highlight R %}
page_breaks <- grep("\\f", doc)
doc[page_breaks[1]]
## [1] "\fA/71/PV.62\t13/12/2016"
{% endhighlight %}

On affiche ici la première ligne de la deuxième page, avec un `\f` affiché devant.

## Extraire l'information utile

On ne veut pas s'arrêter ici.

Une fois que vous avez le document PDF importé dans R, vous voulez extraire les morceaux de texte que vous intéresse, et vous débarrasser du reste.

C'est ce qu'on va faire tout de suite.

Pour y arriver, j'utilise des outils standards de manipulation de chaînes de caractère :

* Les fonctions `grep` et `grepl`
* Des fonctions basiques de manipulations de `str` (comme `str_split`)
* Le package `stringr`

L'objectif est d'extraire tous les discours des orateurs dans le document présenté plus tôt ([celui-là](https://github.com/Huitziii/crispy-pdf/raw/master/71_PV.62.pdf)), excepté que je ne veux pas les discours du président. 

Je sais que si je peux le faire sur un document, alors je pourrai ensuite faire rouler mon script sur les milliers de documents que j'ai téléchargé au préalable.

Voici les étapes :

1. Nettoyer les en-têtes et pieds de page sur toutes les pages.
2. Concaténer les deux colonnes ensemble.
3. Trouver les lignes avec les noms des orateurs.
4. Extraire les bonnes lignes.

Je vais utiliser des expressions régulières (regex) à plusieurs reprises dans le code.

Si vous ne savez pas du tout ce que c'est, je vous conseille une rapide recherche Google (regex tutorial) pour comprendre un peu. C'est essential à connaître dès que vous manipulez du texte, de toute façon.

Si vous connaissez juste un tout petit peu, ça suffira, je ne suis pas un grand expert non plus.

Je vous conseille aussi, si vous voulez apprendre, de suivre les étapes en même temps que moi sur R, afin de mieux appréhender ce qu'on fait.

### 1. Nettoyer les en-têtes et pieds de page

Avez-vous remarqué que chaque page contient des infos en haut et en bas ?

On trouve la date, l'identifiant du document, etc.

On va se débarrasser de ça.

{% highlight R %}
# Retire l'en-tête de la première page
president_row <- grep("^President:", doc)[1]
doc <- doc[(president_row + 1):length(doc)]

# Retire le pied de page de la première page
footer_row_1 <- grep("This record contains the text of speeches ", doc)[1]
footer_row_2 <- grep("\\f", doc)[1] - 1
doc <- doc[- c(footer_row_1:footer_row_2)]

# Retire les en-têtes sur toutes les autres pages
header_rows <- grep("^\\f", doc) # Rappel: \f symbolisent les sauts de page
doc[header_rows] <- "page" # Je place un marqueur qui me servira plus tard
doc <- doc[- (header_rows - 1)]
{% endhighlight %}

À présent, on a un document un peu plus propre.

La prochaine étape, c'est de rassembler les deux colonnes en une seule.

### 2. Concaténer les deux colonnes ensemble.

Mon idée (mais il y a d'autres manières de faire), c'est d'utiliser la fonction `str_split` pour séparer chaque ligne dès qu'il y a deux espaces consécutifs (ça montre que ce n'est pas une phrase normale).

Ensuite, puisqu'il y aura plein d'espaces ensemble au début des lignes, je détecte là où il y a du texte, là où il n'y en a pas, et je sélectionne seulement les éléments avec du texte.

Vous allez voir, c'est un peu compliqué, mais ça marche :


{% highlight R %}
library(stringr)
doc_split <- strsplit(doc, "  ") # Sépare chaque ligne dès qu'il y a deux espaces
doc_split <- lapply(doc_split, function(x) {
	# Pour chaque élément, extraire :
	#    - doc1 qui est la première colonne.
	#    - doc2 qui est la deuxième colonne.
    doc1 <- x[1:8][x[1:8] != ""][1] # La première partie de texte qui n'est pas vide
    if (is.na(doc1)) doc1 <- ""
    # doc2 prend la partie suivante de text non vide
    doc2 <- x[x != ""] 
    if (doc1 != "") doc2 <- doc2[-1]
    if (length(doc2) == 0) doc2 <- ""
    # Parfois il faut extraire un peu plus de texte
    # Je vais le donner soit à doc1 soit à doc2 selon la taille du texte
    while (sum(nchar(doc2)) > 65) {
        doc1 <- paste(doc1, doc2[1], collapse = " ")
        doc2 <- doc2[-1]
    }
    # On nettoie et on envoie
    doc2 <- paste(doc2, collapse = " ")
    doc1 <- str_trim(doc1) # stringr::str_trim retire les espaces avant et après
    doc2 <- str_trim(doc2)
    list(doc1 = doc1, doc2 = doc2)
})
doc1 <- sapply(doc_split, `[[`, 1) # Première colonne
doc2 <- sapply(doc_split, `[[`, 2) # Deuxième colonne
{% endhighlight %}

Maintenant on met tout ensemble, grâce au marqueur `page` que j'ai ajouté plus tôt :

{% highlight R %}
# Vector contenant les positions des sauts de page :
pages_rows <- c(0, which(doc1 == "page"), length(doc1))
doc <- c()
# Page par page, on remplit un nouveau vecteur :
for (i in 1:(length(pages_rows) - 1)) {
    doc <- c(doc, c(doc1[(pages_rows[i] + 1):pages_rows[i + 1]],
                    doc2[(pages_rows[i] + 1):pages_rows[i + 1]]))
}
doc <- doc[doc != "page"]
{% endhighlight %}

Maintenant qu'on a un vecteur tout propre, on peut extraire les discours.

### 3. Trouver les lignes avec les noms des orateurs

Là il faut regarder le document d'un peu plus près et trouver les patterns qui vont nous aider à repérer là où démarrent les discours.

C'est assez facile en fait parce que tous les noms des orateurs commencent par "Mr." ou "Mrs.". Et le président est toujours appelé "The President:" ou "The Acting President:".

On prend les lignes :

{% highlight R %}
speakers_rows <- grep("Mrs?\\..+\\(", doc)
president_rows <- c(grep("The President:", doc),
                    grep("The Acting President:", doc))
all_rows <- sort(c(speakers_rows, president_rows))
{% endhighlight %}

Maintenant c'est facile !

On sait où commencent les discours, et ils finissent toujours quand quelqu'un d'autre va parler :

{% highlight R %}
speeches <- list()
for (i in 1:(length(all_rows) - 1)) {
    start <- all_rows[i]
    if (!start %in% speakers_rows) next
    stop <- all_rows[i + 1] - 1
    
    speeches <- append(speeches, list(doc[start:stop]))
}
speeches[[11]]
##  [1] "Mr. Hallak (Syrian Arab Republic) (spoke in"              
##  [2] "Arabic): Yesterday in his statement (see A/71/PV.61),"    
##  [3] "my colleague the representative of the Republic of Korea" 
##  [4] "made unprecedented allegations about my country that"     
##  [5] "we have not read in any report and that have not appeared"
##  [6] "in any document. We ask our colleague to provide us"      
##  [7] "with further information concerning those allegations"    
##  [8] "and to indicate if they have been corroborated through"   
##  [9] "bilateral channels. We would have preferred it if, rather"
## [10] "than accusing us, our colleague from South Korea had"     
## [11] "dispelled and disavowed information referring to the"     
## [12] "existence of nuclear weapons in my country, which"        
## [13] "would constitute a flagrant violation of the Treaty on"   
## [14] "the Non-Proliferation of Nuclear Weapons."                
## [15] "Ms. Kharashun (Belarus) (spoke in Russian):"              
## [16] "I would just like to underscore in my statement the"      
## [17] "untiring commitment of Belarus to the international"      
## [18] "norms and standards concerning nuclear energy, as"        
## [19] "well as the priority nature for us of ensuring nuclear"   
## [20] "safety and security and transparency in carrying out"     
## [21] "the construction of our first nuclear power plant. We"    
## [22] "stand ready for dialogue with all international partners,"
## [23] "including our neighbours. For my country, which"          
## [24] "suffered the greatest impact of the Chernobyl, nuclear"   
## [25] "security is of primary importance."                       
## [26] "As I noted earlier in my statement, Belarus uses"         
## [27] "the tools provided by the International Atomic Energy"    
## [28] ""                                                         
## [29] "Agency to countries that are embarking on nuclear"        
## [30] ""                                                         
## [31] "programmes for the first time. We have hosted the"        
## [32] ""                                                         
## [33] "assessment missions of the Agency on numerous"            
## [34] ""                                                         
## [35] "occasions."                                               
## [36] ""
{% endhighlight %}

Boom !

On a extrait tous les discours dans une liste.

Maintenant, on peut analyser ce que raconte chaque représentant de son pays, comment ça évolue au fil des documents, au fil des années, selon les sujets abordés, etc.

Alors oui c'est vrai que pour ce document, on aurait pu le faire à la main.

Mais imaginez que vous ayez des milliers de documents comme ça... ça devient fastidieux. Impossible même !

C'est là que ça devient fun en fait.

Même si tous les documents se ressemblent, ils vont tous avoir leurs petites spécificités, leurs petites exceptions, etc. Peut-être que le format du document va évoluer au fil des années. Parfois il y a des fautes de frappe.

En fait, même avec cette extraction, ce n'est pas parfait et on pourrait l'améliorer.

Alors au travail !