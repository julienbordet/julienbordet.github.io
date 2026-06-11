# bordet.info — conventions du site

Site personnel de Julien Bordet (directeur général), publié sur `www.bordet.info`.
Jekyll + GitHub Pages, design éditorial (palette crème/navy/or, Playfair Display + Inter).
Spec de référence : `docs/superpowers/specs/2026-06-11-refonte-editoriale-design.md`.

## Ton éditorial

- **Première personne, toujours.** Jamais de 3e personne pour parler de Julien
  (« Ce qu'un dirigeant lit… » est proscrit ; écrire « ce que je lis… »). Les thèses
  impersonnelles (« Une stratégie ne vaut que si… ») sont acceptables.
- **Sobriété.** Pas de référence statutaire ni de mise en scène du rang
  (« comités de direction » et autres marqueurs de statut sont à éviter). La
  crédibilité passe par la matière, pas par l'affichage.
- **Orthographe française complète** : tous les accents et caractères spéciaux,
  jamais d'équivalents ASCII.

## Rédiger une note de blog

Les notes vivent dans `_posts/`, nommées `AAAA-MM-JJ-titre.md`. Front matter type :

```yaml
---
layout: post
title: "Titre de la note"
date: 2026-01-15
category: [rubrique]
tags: [mot-clé, autre-mot-clé]
---
```

- **Contenu intemporel.** Les notes sont des « notes durables », pas un flux
  d'actualité. La date n'est PAS un critère de tri : l'index `/blog/` classe par
  thème. La date de publication s'affiche discrètement en pied d'article (gérée par
  le layout, rien à faire).
- **Catégorie → rubrique.** La `category` du front matter est rattachée à une des
  trois rubriques éditoriales via `_data/rubriques.yml` (Monde & stratégie /
  Sciences & calcul / Esprit & cognition). Pour ranger une note, choisir une
  `category` déjà mappée, ou ajouter le mapping dans `_data/rubriques.yml` — **ne
  jamais modifier le front matter d'une note juste pour la reclasser**, agir sur le
  mapping. Une note hors rubrique apparaît sous « Autres notes ».
- **Titres Markdown** : toujours un espace après les `#` (`## Titre`, jamais
  `##Titre` ni `### Titre ###`) — sinon kramdown affiche la ligne en texte brut.
- **Liens internes : toujours relatifs.** `/reading/`, `/parcours/`,
  `/blog/AAAA/MM/JJ/slug.html` — jamais `http://www.bordet.info/…`. Attention :
  Jekyll translittère les accents dans les URLs (`Modèle` → `Mode-le`,
  `les modèles` → `les-mode-les`) ; vérifier l'URL réelle dans `_site/` après build.
  (Le tag `{% post_url %}` échoue sur les noms de fichiers accentués avec cette
  version de Jekyll — utiliser le chemin relatif en dur.)

## Page Lectures (`reading/index.md`)

- **Markdown pur**, aucune balise HTML : `## Année`, puis `### Mois`, puis les livres.
  Le layout `reading` reconstruit seul la barre d'années et la navigation.
- **Format d'une entrée** : titre en gras, auteur entre parenthèses, puis commentaire.
  Exemple : `**Titre du livre** (Auteur) : mon commentaire.`
- Années en ordre décroissant (la plus récente en tête) ; mois décroissants dans
  chaque année. La barre d'années se met à jour automatiquement à partir des `## Année`.
- La phrase d'assise est dans le front matter (`lede:`), pas dans le corps.

## Build et vérification (machine de Julien)

Ruby 3.3 Homebrew + locale UTF-8 sont obligatoires (voir détails et pièges dans la
mémoire Claude du projet) :

```bash
export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH" LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
bundle exec jekyll serve --port 4000 --livereload   # ou build
```

## Git

- **Ne jamais `git add -A` / `git add .`** — toujours cibler explicitement les
  fichiers du site. Ne jamais stager les fichiers Claude/mémoire ni `.claude/`.
- Branche de travail : `feat/refonte-editoriale` (PR vers `master`).
