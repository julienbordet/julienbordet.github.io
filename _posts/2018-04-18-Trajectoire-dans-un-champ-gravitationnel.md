---
layout: post
title: "Trajectoire dans un champ gravitationnel"
date: 2018-04-18
category: [physique]
tags: [physique, gravitation]
---

Mes explorations du moment m'ont permis de "jouer" avec [Sagemath](http://www.sagemath.org/fr/) et [Jupyter](http://jupyter.org/), afin de simuler des systèmes gravitationnels peu complexes, mais pour lesquel il n'existe pas de solution analytique simple.

Le fichier Jupyter associé est disponible [ici](https://github.com/julienbordet/Divers/blob/master/Physique/Trajectoire%20dans%20un%20potentiel%20gravitationnel%20de%20plusieurs%20corps.ipynb), mais ce billet est destiné à le résumer.

<!--more-->

L'objectif de ce petit notebook jupyter est de montrer qu'il est aisé de calculer la trajectoire d'un corps dans un potentiel gravitationnel quelconque.

Pour rappel, le potentiel gravitationnel global $\phi$ va être égal à la somme de chacun des potentiels gravitationnels individuels \\(\phi_i\\) : \\(\phi = \sum \phi_i\\).

Le champ gravitationnel résultant va alors se calculer comme

$$\mathbf g = - \vec \nabla \phi$$

Dans la configuration considéré ici, on introduit 3 corps. Le premier est au centre du repère, le second à une distance \\(d\\) sur l'axe \\((Ox)\\) (coordonnées polaire \\((d,0)\\)) et le troisième afin des coordonnées polaires \\((d, 2\pi/3)\\).

Pour simplifier et pour favoriser une représentation graphique agréable, on ne simule pas ici le mouvement de chacun des trois corps, qu'on représente fixes dans le repère.

```
%display latex
reset()

d = 500 # Distance entre les deux corps
M1 = 1000 # Masse du premier corps
M2 = 500 # Masse du second corps
M3 = 1300 # Masse du troisième corps
t3 = 2 *pi / 3 # Angle en coordonnées polaire du 3ème corps
cont = 150

r = var('r');assume(r>0)
theta = var('theta')
x = var('x')
y = var('y')
```

Le champ gravitationnel en \\((r, \theta)\\) peut s'exprimer pour un corps de coordonnées polaires \\((d, \theta_0)\\) et de masse \\(M\\), :

$$\dfrac{M}{\sqrt{r^2 + d^2 - 2 rd \cos(\theta - \theta_0)}}$$

```
# Fonction phi, exprimant le potentiel gravitationnel en fonction
# de r et de theta
phi(r, theta) = -(M1 / r + M2/sqrt(r^2 + d^2 - 2*r*d*cos(theta)) + M3 / sqrt(r^2 + d^2 - 2*r*d*cos(theta - t3)))
#phi(r, theta) = -(M1 / r + M2/sqrt(r^2 + d^2 - 2*r*d*cos(theta)))
#phi(r, theta) = -(M1 / r)
```

```
cp = contour_plot(-phi(sqrt(x^2 + y^2), arctan2(y,x)), (x,-700,700),
             (y, -500, 500), fill = False, cmap='hsv', labels=True,
             contours=cont, region= sqrt(x^2 + y^2) - 100 )
```

Pour rappel, le gradient en coordonnées polaires s'écrit

$$
\nabla \phi = 
\begin{cases}
\dfrac{\partial \phi}{\partial r} \\
\dfrac{1}{r}\dfrac{\partial \phi}{\partial \theta} \\
\end{cases}
$$

```
grad_r(r, theta) = phi.diff(r)
```

```
grad_th(r, theta) = phi.diff(theta)/r
```

```
r0 = vector((-400, -400))
v0 = vector((0.5, 1.5))

position = [r0]
vitesse = [v0]

acc = []
iteration = 1000
time_slice = 1

for i in range(iteration):
    vit_actuelle = vitesse[-1]
    pos_actuelle = position[-1]
    
    pos_nouvelle = pos_actuelle + vit_actuelle*time_slice
    
    x_actuel = pos_actuelle[0].n()
    y_actuel = pos_actuelle[1].n()
    
    r_actuel = sqrt(x_actuel^2+y_actuel^2).n()
    th_actuel = arctan2(y_actuel, x_actuel).n()
            
    if (r_actuel > 700 or r_actuel <= 100):
        break

    g_r  = grad_r(r_actuel, th_actuel).n()
    g_th = grad_th(r_actuel, th_actuel).n()
    
    a = vector ( [ - g_r * cos(th_actuel) - g_th * sin(th_actuel),
                   - g_r * sin(th_actuel) - g_th * cos(th_actuel)])

    
    position.append(pos_nouvelle)        
    vitesse.append(vit_actuelle + a* time_slice)
    acc.append(a)
```

```
lp = list_plot(position, plotjoined=True)
Gp = Graphics()
Gp += cp
Gp += lp
Gp.show()
```

<img src="/images/blog/Champ-gravitationnel.png" />
