# Spécification design — Refonte éditoriale de bordet.info (Direction A)

Date : 2026-06-11
Statut : validé sur maquettes, en attente de relecture finale

## 1. Objectif et positionnement

Le site est la **carte de visite exécutive** de Julien Bordet (Directeur Général, direction
opérationnelle) : crédibiliser le profil auprès de chasseurs de têtes, conseils
d'administration, investisseurs et partenaires.

La différenciation repose sur deux piliers, identifiés par l'analyse concurrentielle
(Collison, Evans, Roger Martin — aucun dirigeant français ne tient de site personnel
soigné) :

1. **Une signature éditoriale** : design de revue économique haut de gamme, là où les
   sites de profils dirigeants sont soit des templates, soit des pages nues.
2. **Une profondeur intellectuelle assumée** : tagline-thèse en hero, section
   « Carnets & bibliothèque » qui transforme les pages existantes (lectures, physique,
   photos, notes) en preuve de curiosité — contenu intemporel, sans dates affichées.

## 2. Direction artistique

Registre « éditorial / presse économique » :

- **Le crème devient dominant, le navy devient l'encre.** Inversion de la logique
  actuelle (hero navy + photo + overlay), qui faisait template.
- **Asymétrie systématique** : plus aucun bloc centré hormis le pied de contact.
- **Filets typographiques** (hairlines 1 px) comme ossature visuelle : séparations de
  colonnes, soulignés de titres, lignes de chronologie.
- **Sections numérotées** : `01`, `02`, `03` en italique or, titre serif, filet courant
  jusqu'à la marge.
- **La carte stratégique en gravure vectorielle sépia** : le fond du hero est un
  **SVG dessiné** dans l'esprit des maquettes validées — lignes de côte en courbes,
  routes maritimes en pointillés reliant des points-ports, graticule discret, rose
  des vents — en traits `--sepia` (≈ 1 px, opacité ≈ 0,5) posés directement sur le
  papier crème. Très sobre : la carte est une texture symbolique, pas une image.
  L'inline SVG remplace l'image photo (`images/website/hero-bg.png`), qui est
  retirée du hero. Plus d'overlay sombre. Avantages : poids quasi nul, netteté
  parfaite à toutes densités, couleurs pilotées par les tokens CSS.

## 3. Palette (tokens CSS)

| Token | Valeur | Usage |
|---|---|---|
| `--paper` | `#F6F2EA` | fond dominant |
| `--paper-deep` | `#F1ECE0` | sections alternées (Parcours) |
| `--ink` | `#0D1F2D` | texte titres, encre principale ; fond du bloc contact |
| `--gold-deep` | `#765D28` | accents texte sur fonds clairs (kickers, numéros, soulignés) |
| `--gold` | `#C9A84C` | accents sur fond navy uniquement |
| `--sepia` | `#C4B083` | gravure de la carte, filets du hero |
| `--rule` | `#DDD5C4` | filets sur `--paper` (`#DCD3BE` sur `--paper-deep`) |
| `--body` | `#4A4639` | corps de texte |
| `--muted` | `#5C5749` / `#6B6452` | texte secondaire, italiques |
| `--cream-text` | `#F2EDDF` | texte sur navy |

Règle d'accessibilité : `--gold` (#C9A84C) ne sert **jamais** de couleur de texte sur
fond clair (contraste insuffisant) ; sur clair on utilise `--gold-deep`, qui passe
AA en petites capitales/labels.

## 4. Typographie

- **Playfair Display** : display et titres, italiques d'assise, tagline. Graisse 500
  pour le display (la 400 actuelle manque d'autorité, la 700 est trop noire).
- **Inter** : labels, kickers en petites capitales espacées (letter-spacing 0.1–0.22 em),
  corps de texte UI, chapôs.
- **Auto-hébergement** : WOFF2 sous-ensemblés (latin + latin-ext), `font-display: swap`,
  préchargement des deux graisses critiques. Suppression des requêtes Google Fonts.
- **Micro-typographie** : chiffres elzéviriens (`font-feature-settings: "onum"`) pour
  les années de la chronologie ; espaces fines insécables avant ponctuations doubles ;
  `text-wrap: balance` sur les titres ; justification abandonnée au profit du fer à
  gauche (le `text-align: justify` actuel crée des lézardes).

## 5. Structure de la page d'accueil

Cinq blocs, dans cet ordre :

1. **Header** — `JULIEN BORDET` en petites capitales à gauche ; navigation
   `PARCOURS · CARNETS · CONTACT` à droite ; filet inférieur. Cibles : `/parcours`,
   ancre `/#carnets`, ancre `/#contact`.
2. **Hero** — carte sépia en fond ; grille 1.4fr / 1fr :
   - colonne gauche : kicker `DIRECTEUR GÉNÉRAL — DIRECTION OPÉRATIONNELLE`, puis
     tagline-thèse en serif ≈ 36 px : **« Une stratégie ne vaut que si elle survit au
     contact du terrain. »**
   - colonne droite, calée en pied sur un filet vertical : chapô (« Vingt ans
     d'allers-retours entre la stratégie et sa mise en œuvre sur le terrain — en
     conseil comme à la direction de PME industrielles. ») + lien souligné or
     « Lire le parcours → ».
3. **01 — Ce que j'apporte** — trois colonnes de journal séparées par des filets
   verticaux (plus de cartes) : Direction opérationnelle / Transformation & systèmes /
   Stratégie & analyse. Titres serif, corps Inter.
4. **02 — Parcours** — fond `--paper-deep`. Chronologie de revue : années en petites
   capitales or dans une colonne de marge (120 px), organisme serif + rôle italique
   sur la ligne, hairline entre les entrées. Sur la landing : entrées resserrées
   (organisme + rôle), le détail vit sur `/parcours`. Lien « Parcours complet,
   2002 — 2026 → ».
5. **03 — Carnets & bibliothèque** — phrase d'assise en italique (« Quelques
   réflexions posées sur le papier. » — sobre, première personne ou impersonnel,
   jamais de 3e personne ni de référence statutaire type « comités de direction »)
   puis grille 2 × 2 à filets, chaque case avec sa **vignette gravée** (SVG trait
   `--gold-deep`, style de la carte du hero) : Monde & stratégie (rose des vents)
   → `/blog/#monde`, Sciences & calcul (orbites) → `/blog/#sciences`, Esprit &
   cognition (arborescence) → `/blog/#esprit`, Lectures (livre ouvert) →
   `/reading`. Kicker en petites capitales + intitulé serif. La grille gère les
   nombres impairs d'entrées (filet supprimé uniquement sur la dernière rangée).
6. **Contact + colophon** — bloc navy : kicker `CONTACT`, phrase d'invitation en
   italique serif, liens or soulignés (email protégé anti-spam conservé, LinkedIn) ;
   filet or, puis colophon `© 2026 JULIEN BORDET — NANTES` / `BORDET.INFO`.

## 6. Contenu intemporel — règle « sans dates »

Les contenus de Carnets & bibliothèque sont des **notes durables**, pas un blog :

- **Aucune date affichée** : suppression du rendu de la date dans les listings et dans
  le layout des articles (`_layouts/post.html`). La date reste dans le front matter
  (tri interne, sitemap) mais n'est plus rendue.
- **URLs inchangées** : le permalink actuel `/blog/:year/:month/:day/:title.html` est
  conservé pour ne casser ni liens entrants ni référencement. Les nouvelles notes
  pourront adopter un permalink thématique (`/notes/:title/`) sans toucher à l'existant.
- **Classement thématique en 3 rubriques éditoriales** : les 13 catégories
  historiques des posts sont consolidées via `_data/rubriques.yml` (le front matter
  des notes n'est pas modifié) en Monde & stratégie / Sciences & calcul / Esprit &
  cognition. La page Notes (`/blog/`) présente trois **planches gravées**
  cliquables (vignette SVG, intitulé, nombre de notes) ; au clic, la liste de la
  rubrique se déplie sous la grille (JavaScript léger ; sans JS, toutes les
  rubriques sont visibles et les ancres `#monde`, `#sciences`, `#esprit`
  fonctionnent). Une section « Autres notes » apparaît automatiquement si une
  future note n'appartient à aucune rubrique. Plus de flux antéchronologique.
- Le flux RSS (`feed.xml`) est conservé tel quel.

## 7. Pages intérieures

Toutes les pages adoptent le même langage (header/footer communs, papier crème,
filets, kickers) :

- **/parcours** : chronologie éditoriale complète (même grille que la section 02,
  avec descriptions).
- **Pages de notes** : layout `post` sans date, rubrique en petites capitales or
  au-dessus du titre serif, largeur de lecture ≈ 680 px, fer à gauche.
- **/reading, /photos, /physique, /iran, /mac** : habillage via les layouts communs ;
  pas de refonte de leur contenu dans ce périmètre.

## 8. Micro-interactions et finition

- Soulignés or animés : le filet s'étend de 0 à 100 % au survol (`background-size`,
  200 ms).
- Apparition douce des sections au scroll : `IntersectionObserver`, translation 8 px +
  fondu 250 ms, désactivée si `prefers-reduced-motion`.
- États de focus visibles (liserés `--gold-deep`) sur tous les liens.
- Pas de carrousel, pas de parallaxe, pas d'animation décorative.

## 9. Chaîne technique (GitHub Pages + Actions)

Bascule du build Pages « classique » vers **GitHub Actions** (workflow
`jekyll build` + `actions/deploy-pages`), ce qui libère :

- **Fonts** : sous-ensembles WOFF2 commités (générés une fois), préchargés.
- **Images** : le hero étant vectoriel, plus aucun retraitement photo n'est requis
  pour la page d'accueil. Étape optionnelle de compression AVIF/WebP des images
  existantes (blog, photos) si le besoin apparaît — hors chemin critique.
- **CSS** : un seul fichier minifié (Lightning CSS ou csso), plus de CSS inliné par
  `{% include %}` non minifié.
- **Analytics** : suppression du `ga.js` obsolète (UA mort depuis 2023), sans
  remplacement par défaut. L'ajout ultérieur d'un outil léger (GoatCounter) restera
  possible mais ne fait pas partie de cette refonte.
- **Qualité** : job de vérification des liens internes (lychee) en CI.

Budget de performance : Lighthouse ≥ 95 sur les quatre axes, zéro requête
tierce bloquante.

## 10. Hors périmètre

- Réécriture du contenu des notes existantes.
- Refonte du contenu de /reading, /photos, /physique (habillage seulement).
- Nouveau CMS, commentaires, newsletter.
- Changement d'hébergement ou de domaine.

## 11. Critères de succès

- La page d'accueil ne ressemble à aucun template : asymétrie, filets, numérotation,
  gravure sépia visibles dès le premier écran.
- Aucune date visible dans Carnets & bibliothèque ; URLs existantes intactes.
- Lighthouse ≥ 95 partout ; aucune requête Google Fonts/Analytics au runtime.
- Le site se lit correctement sur mobile (colonnes empilées, chronologie en marge
  réduite, hero asymétrique conservé).
