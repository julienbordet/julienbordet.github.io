---
layout: post
title: "Modèles : la masse critique, principe et intérêt"
date: 2015-08-17
category: [modèle]
tags: [modèle, physique, masse critique, modélisation, mathématiques]
---

Pour réfléchir plus efficacement, il est particulièrement efficace de s'appuyer sur des "modèles mentaux". J'exposais récemment la liste que j'utilise, et que j'essaye à la fois de pratiquer et d'étendre lorsque cela fait sens.

Pour pousser la réflexion un peu plus loin, quelques mots aujourd'hui sur la notion de masse critique.

<!--more-->

### Un concept simple mais puissant ###
La masse critique est un concept simple mais extrêmement puissant. 

Historiquement, la masse critique a été inventée dans le domaine de la physique nucléaire. C'est la quantité de combustible à partir de laquelle la réaction nucléaire se maintient dans le temps. Si cette masse n'est pas atteinte, il ne se passe rien, ou quasiment : le système va rapidement retourner à un système inerte.

Mais à partir du moment où cette limite est dépassée, les conséquences sont potentiellement énormes. Soit, si le système est maîtrisée, la combustion nucléaire est établie et simplement auto-entretenue dans le temps. Soit, si on ne met pas en place de système de contrôle, la réaction augmente de manière exponentielle.

Le principe appliqué est alors simple : la "combustion" nucléaire produit des électrons, qui eux-mêmes vont générer d'autres réactions de combustion. Néanmoins chaque production d'électron "consomme" du combustible : la question est donc de savoir si les nouvelles réactions générées sont suffisamment nombreuses pour compenser cette consommation. Si le combustible est suffisamment abondant, les collisions entre les électrons et ce dernier sont suffisamment fréquences : on a atteint la masse critique et le système est auto-entretenu.

### Des applications au quotidien ###

Si cette usage du concept semble au premier abord spécifique, il est rapide de voir qu'on peut appliquer ce principe dans de nombreux domaines.

Imaginons par exemple le lancement d'un produit de type réseau social, reposant principalement sur le bouche à oreille. On a alors quasiment le comportement décrit ci-dessus. Si un utilisateur est content du réseau, il le recommande à ses connaissances. Pour qu'il le recommande, il faut qu'il retrouve un nombre suffisant de ses proches sur l'outil. C'est l'équivalent d'avoir une probabilité suffisante de collision entre l'électron et le combustible nucléaire. Si c'est le cas, alors l'audience du réseau va augmenter de manière exponentielle, comme on l'a vu avec Facebook à l'époque.

De manière plus générale, tout système[^1] qui dispose d'une boucle de rétroaction positive (qui tend à transmettre le changement, et non à stabiliser le système) et dont l'effet de transmission est contrôlé par un paramètre de type "masse"  dans le système débouche potentiellement sur un effet "masse critique" : propagation d'un virus dans une population, dégradation de l'ambiance dans une usine...

### Quel intérêt ? ###

L'intérêt principal d'intégrer ce concept dans la boite à outil de notre raisonnement est qu'il nous pousse à envisager que le monde puisse ne pas être linéaire et qu'il évolue de manière simple et prévisible pour nos capacités cognitives limitées. Il incite plutôt à rechercher les signes classiques d'un environnement, ou système, qui tend à s'autoalimenter, et qui risque de se comporter de manière non linéaire, c'est-à-dire exponentielle, avec des conséquences potentielles échappant au contrôle.

Avec un peu d'entraînement et une pensée systémique[^2] développée, il est discernable dans tous les domaines, depuis l'ingénierie, la stratégie, la sociologie ou encore la biologie. 

### [Technique] Une courte modélisation ###
**Un effet d'emballement**

A partir de cet exemple, on peut réaliser une modélisation basique du phénomène de masse critique, à l'aide de quelques outils mathématiques simples (n'ayez pas peur, cela reste très basique, et il est possible de sauter les équations), et certaines (grosses) approximation.

On imagine un nouvel outil, appelé *FaceBelier*, disposant d'un nombre \\(A\\) d'utilisateurs. Chaque utilisateur va recommander l'outil à 2 de ses connaissances s'il a au moins 4 connexions sur le réseau, c'est-à-dire s'il y a  rencontré 4 personne qu'il connait.

Plus l'audience de l'outil par rapport à la population totale \\(P\\) est grande, plus il y a de chance qu'un nouvel utilisateur parvienne à réaliser 4 connexions. On désigne par \\(p\\) cette probabilité. 

Par ailleurs, encore faut-il que l'outil plaise ! On désigne alors le taux de satisfaction, qui conditionne la recommandation à 2 connaissances, par le paramètre \\(\alpha\\).

L'effet d'augmentation de la base d'utilisateur de l'outil du fait des recommandations vaut alors, dans un lapse de temps \\(\mathrm{d}t\\) :

$$dA = p \cdot \alpha \cdot A  \mathrm{d}t$$

Cependant il ne faut pas se leurrer : certains utilisateurs quittent le navire, et arrêtent d'utiliser l'outil. Dans un lapse de temps \\(\mathrm{d}t\\), on désigne par \\(\beta\\) la probabilité pour que cela arrive. L'effet sur l'audience est alors :

$$dA = - \beta \cdot A \mathrm{d} t$$

En prenant en compte les deux effets recommandations et perte d'utilisateurs, on obtient : 

$$dA =  p \cdot \alpha \cdot A  \mathrm{d}t - \beta \cdot A \mathrm{d} t$$

où

$$\frac{dA}{A} = (p \alpha - \beta) \mathrm{d}t$$

Ce donne finalement l'évolution de l'outil de l'outil, *via* une intégration telle qu'on pouvait les faire en Terminale :

$$ A = e^{(p \alpha - \beta) \mathrm{d}t}$$

Dis avec des mots : la base utilisateur de *FaceBélier* dépend du taux de satisfaction (\\(\alpha\\)), de la probabilité de ne pas être tout seul sur l'outil (\\(p\\)) et du taux de départ. Si l'effet des deux premiers dépasse celui du dernier (c'est-à-dire si \\(p \alpha - \beta > 0\\)), alors la base va croître, et même de plus en plus vite. On aura alors une courbe utilisateur du type : 

{: style="font-size: 60%; text-align: center"} 
<img src="/images/blog/FaceBélier-croiss.jpg" width="350px" />

Si les deux se compensent (\\(p \alpha - \beta = 0\\)), on obtient une base stable :

{: style="font-size: 60%; text-align: center"} 
<img src="/images/blog/FaceBélier-plat.jpg" width="350px" />

Si au contraire il y a plus de départ que d'arrivée (\\(p \alpha - \beta < 0\\)), la base décroît :

{: style="font-size: 60%; text-align: center"} 
<img src="/images/blog/FaceBélier-decroiss.jpg" width="350px" />

**La masse critique**

Et la masse critique dans tout cela ? Intuitivement on va se dire pour *FaceBélier*, atteindre la masse critique, c'est atteindre suffisamment d'utilisateurs pour qu'il soit très probable qu'ils retrouvent quelqu'un de leur entourage et qu'ils soient tentés de rester, puis de recommander l'outil.

Pour cela, il faut que le nombre d'utilisateurs soit significatif au regard de la population totale. Lorsque la masse critique sera atteindre, la probabilité de retrouver une connaissance sera suffisamment importante pour que la courbe d'évolution des utilisateurs soit en croissance exponentielle, comme on l'a vu là haut.[^3]

#### Remarques ####

* Beaucoup de raisonnement sont extrêmement simplificateurs ici, l'objectif n'étant pas de décrire une réalité physique, mais un principe de réflexion. 

#### Notes ###
[^1]: Voir [Thinking in Systems: a Primer, Donella H. Meadows](http://www.amazon.com/Thinking-Systems-Donella-H-Meadows/dp/1603580557)
[^2]: Voir [Thinking in Systems: a Primer, Donella H. Meadows](http://www.amazon.com/Thinking-Systems-Donella-H-Meadows/dp/1603580557)
[^3]: Il y a ici une grosse astuce. En effet pour obtenir l'expression de l'évolution des utilisateurs, on a "supposé" que le paramètre \\(p\\) ne variait pas en fonction de la base utilisateur \\(A\\), sinon l'expression aurait été moins simple. Mais pour le propos, on a considéré que localement ce paramètre est "constant" et qu'il évolue de manière "suffisamment" moins rapide que A pour qu'on prenne cette hypothèse.
