---
layout: post
title: "L'expansion de l'Univers impacte-t-elle la distance Terre-Lune ?"
date: 2016-11-16
category: [physique]
tags: [physique,cosmologie, relativité générale, énergie noire]
image: Lune.jpg
---

L'Univers, à large échelle, s'étend. C'est ce qu'à découvert Hubble dans les années 30. Mais ces résultats sont-ils valables à une échelle plus "locale", par exemple au sein du système solaire ? La Terre et la Lune s'éloignent-ils ? C'est l'objet de ce petit billet.

<!--more-->

### L'expansion de l'Univers, une déformation de l'espace-temps  ###

Parmi les découvertes clés de la cosmologie au XXème siècle, l'expansion de l'Univers occupe une place particulière. Observée pour la première fois par Edwin Hubble, c'est ce constat qui est à l'origine de la théorie du Big Bang : si toutes les galaxies (lointaines) s'éloignent les unes des autres, que se passe-t-il si on remonte dans le passé ? Elles convergent en un "point", qu'un des détracteurs de la théorie, Fredd Hoyle, a désigné pour s'en moquer sous le nom de  "Big Bang". Le nom est resté, et les théories de l'Univers stationnaires de son inventeur ont été oubliées.

Pour aller vite, l'expansion de l'Univers, telle qu'elle est comprise aujourd'hui, correspond à une déformation de l'espace-temps : ce ne sont pas les galaxies qui s'éloignent les unes des autres, mais l'espace-temps situé entre elle qui s'agrandit. J'en avais parlé dans mon [précédent billet](http://www.bordet.info/blog/2016/10/15/Mesurer-l-expansion-de-l-univers.html) sur la mesure de l'expansion de l'Univers.


### La vitesse d'expansion varie en fonction de la distance ###

L'effet de cette expansion est sensible à grande échelle. A l'échelle "locale", c'est-à-dire pour les étoiles d'une galaxie, ou pour les amas de galaxies, l'effet de la gravitation est le plus fort, et les galaxies peuvent rentrer en collision. Mais à plus grande échelle, entre des galaxies très lointaines, c'est bien l'effet d'éloignement qui prévaut.

Mais n'y a-t-il absolument aucun effet au niveau local ? Si on dit que toutes les distances s'accroissent, cela devrait aussi être le cas pour, par exemple, la distance Terre-Lune ou Soleil-Terre. Dans le même temps, les livres de référence sur la relativité générale (et notamment [Gravitation, Misner et al](https://www.amazon.fr/Gravitation-Charles-W-Misner/dp/0716703440)) affirme que c'est négligeable. Qu'en est-il ?

De nombreux papiers scientifiques étudient la question. Je voudrais juste apporter ici une explication simple, à l'aide uniquement de la gravitation newtonienne. Pour une vision (beaucoup) plus complète, on pourra se rapporter à [cette étude](https://arxiv.org/abs/0810.2712).

L'expansion de l'univers est décrite simplement en disant que la distance \\( R \\) entre deux objets évolue en fonction d'un paramètre d'échelle \\ (a(t) \\) par rapport à une valeur de référence \\( R_0 \\) :

\\[ R(t) = a(t) R_0 \\]

Le paramètre \\( a \\) est arbitraire : il décrit l'extension des distances par rapport à une référence. On prend généralement \\( a(t) = 1 \\) pour l'époque actuelle, et \\(R_0 \\) est alors la distance à l'heure actuelle.

On peut en déduire la vitesse d'éloignement :

\\[ \dot R = \dot a R_0 = \dot a \frac{R}{a}  = \frac{\dot a}{a} R \\]

\\[ \dot R = H R \tag{1} \\]

On voit maintenant apparaître la "constante" de Hubble \\(H_0 = \dot a / a\\), qui décrit la vitesse apparente d'éloignement en fonction de la distance \\(R\\). Je prend le terme *constante* entre guillemets, car la valeur évolue avec le temps (en fonction de la vitesse d'expansion de l'Univers). En revanche, elle est la même partout, à un instant donné. A l'heure actuelle, on estime qu'elle vaut environ \\(70 km.s^{-1}.Mpc^{-1}\\).

### Un effet de l'accélération complètement négligeable ? ###

Au niveau dynamique, on cherche maintenant à caractériser \\( \ddot R \\), la composante de l'accélération induite par l'expansion de l'Univers, pour voir si la forme usuelle de la gravité est modifiée.

\\[ \ddot R = \dot H R + H \dot R \\]

or

\\[ \dot H = \frac{\textrm{d}}{\textrm{d} t} \left ( \frac{\dot a }{a} \right ) = \frac{\ddot a}{a} - (\frac{\dot a}{a})^2 = \frac{\ddot a}{a} - H^2 \\] 

d'où 

\\[ \ddot R = \frac{\ddot a}{a} R \tag{2}\\]

on peut encore écrire le facteur \\(\ddot a / a \\) en fonction de la constante de Hubble et du facteur de décélération \\(q\\) :

\\[q = - \frac{\ddot a a}{ {\dot a}^2} = - \frac{\ddot a}{a} H^{-2}\\]

ce qui est intéressant est qu'il est possible de mesurer ce paramètre \\(q\\) (grâce aux équations de Friedmann, mais c'est une autre histoire). Comme on connait aussi la valeur actuelle de la constante de Hubble, on va pouvoir déterminer le coefficient reliant l'accélération et la distance.

\\[q \approx -0,6 \\]
\\[H_0 \approx 70 km.s^{-1}.Mpc^{-1} \\]

et donc \\(\frac{\ddot a }{a} \approx 3 \times 10^{-36} s^{-2}\\).

### Application sur le couple Terre-Lune ###

Si temporairement on ne considère que le couple Terre-Lune, séparé par une distance d'environ \\(d\\) de 360 000 km, obtient donc un facteur d'accélération de l'ordre de 10^-27 ! Si on le compare à l'accélération issue de la gravitation exercées par la Terre sur la Lune :

\\[a_{\textrm{grav}} = \frac{G M_T}{d^2} \approx 3 \times 10^{-3} m.s^{-2} \\]

L'effet est donc plus faible avec un ordre de grandeur de \\(10^{-24}\\) ! Bref, il est complètement négligeable à cette échelle.

### Mais un effet mesurable ! ###

Ici on parle d'accélération, mais qu'en est-il au niveau des distances ? Il s'avère que pour le couple Terre-Lune, l'effet est mesurable ! Pourquoi ? Tout simplement parce que nous sommes capables de mesurer cette distance de manière ultra-précise, grâce aux miroirs placés à la surface de notre satellite par les missions Apollo.

En supposant la constante de Hubble "constante" sur une période de 1 an (ce qui est tout à fait raisonnable), la résolution de l'équation différentielles \\(2\\) est de la forme :

\\[ R(t) = R_0 e^{\sqrt{\ddot a / a} t}\\] 

avec \\(R_0\\) la distance actuelle de la Terre à la Lune, pour \\(t = 0\\). Si on cherche à évaluer cette expression pour \\(t = \textrm{1 an}\\), on trouve :

\\[ R(t) - R(0) = 360000000 \, e^{\sqrt{3}*10^{-18} * 365 * 24 * 3600} - 360000000 \approx 2 \textrm{cm} \\]

Une valeur de 2 cm est tout à fait dans les cordes de nos instruments de mesure, puisque nous savons actuellement que notre satellite s'éloigne d'environ 3,8 cm / an. La majeure partie de cet éloignement est donc dû à l'effet de l'expansion de l'Univers !

On voit donc qu'il est possible, à l'échelle locale, de mesurer les effets de l'expansion de l'Univers, mais que dans le même temps les effets sur la gravitation sont complètement négligeables.

#### Remarques ####

* Bien entendu, ces calculs sont une version simplifiée de ce qu'on obtient en mobilisant la Relativité Générale. Néanmoins l'étude [Carrera et Giulini](https://arxiv.org/abs/0810.2712) mentionnée plus haut indique qu'on retrouve bien cette même expression en utilisant la RG.
* Il aura été possible de calculer R en utilisant directement la formule \\( (1) \\). On obtient grosso modo le même résultat. J'ai d'ailleurs utilisé cette propriété pour ne pas retenir de terme en \\(e^{-\alpha t}\\)
