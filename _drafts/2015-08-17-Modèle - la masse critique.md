---
layout: post
title: "Modèle : la masse critique"
date: 2015-08-17
category: [modèle]
tags: [modèle, physique, masse critique, modélisation, mathématiques]
---

Pour réfléchir plus efficacement, il est particulièrement efficace de s'appuyer sur des "modèles mentaux". J'exposais récemment la liste que j'utilise, et que j'essaye à la fois de pratiquer et d'étendre lorsque cela fait sens.

Pour pousser la réflexion un peu plus loin, quelques mots  aujourd'hui sur la notion de masse critique.

<!--more-->

La masse critique est un concept simple mais extrêmement puissant. 

Historiquement, la masse critique a été inventée dans le domaine de la physique nucléaire. C'est la quantité de combustible nucléaire à partir de laquelle une réaction et se maintient dans le temps, et ceci de manière potentiellement exponentielle. Si cette masse n'est pas atteinte, il ne se passe rien. A partir du moment où elle est dépassée les conséquences sont potentiellement énormes.

### Une courte modélisation

A partir de cet exemple, on peut réaliser une modélisation basique du phénomène de masse critique.

1. Une masse $M$ de matière réactive est insérée dans le réacteur
2. De manière spontanée, seule une partie de ces atomes produit des électrons, en dégageant de l'énergie. On imagine qu'un atome génère 2 électrons, avec une probabilité $\alpha$.

A chaque instant , le nombre d'électron produit  vaut 

$$e = 2 \alpha M dt$$

Une fois qu'une atome a émis ses électrons, il est considéré (pour simplifier) comme inactif. Il faut donc le retirer de la masse $M$ réactive.

$$dM = - e / 2$$


Mais si un électron heurte un autre atome (avec une probabilité $\beta$), il va générer une nouvelle émission suite, et donc va augmenter la masse réactive :

$$dM = \beta e$$


Au final, quand on prend en compte ces deux effets :

$$\left\{\begin{matrix}
e & = & 2 \alpha M dt \\
dM & =  & (\beta - 1/2) e
\end{matrix}\right.$$


On a alors

$$M = e^{(2 \beta - 1)\alpha t}$$


Si le facteur  est inférieur à 0, la masse va en diminuant, et la réaction n'est pas auto-entretenue. S'il vaut 0, la réaction est équilibrée. En revanche, si le facteur est positif, la réaction croit de manière exponentielle.

Ceci arrive donc si la probabilité de collision  est supérieure à 50%, et donc si la matière est suffisamment "dense" pour occasionner de nouvelles collisions.

###Remarques

* Il y a une faiblesse de raisonnement, car en fait la probabilité de collision  dépend de M (ou du moins de sa densité). Pour simplifier on considère cette probabilité constante à moyen-terme dans la réaction.
* La question de l'activité des atomes dans cette modélisation est ultra-simpliste, voir erronée...
* Pour construire ce type d'équation temporelle, il faut introduire un facteur . Celui-ci trouve naturellement sa place dans les équations de stock (comme évolue un stock - ici d'atomes - dans un temps donné). Les équations de flux sont considérées comme instantanées (en tout cas ici) et donc n'introduisent pas de dt (mais seulement éventuellement un retard).

### Des applications pratiques

Démarrage d'un produit.


#### Notes
