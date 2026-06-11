# Refonte éditoriale bordet.info — Plan d'implémentation

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implémenter la direction éditoriale validée (spec `docs/superpowers/specs/2026-06-11-refonte-editoriale-design.md`) : papier crème dominant, gravure SVG sépia en hero, sections numérotées, chronologie de revue, carnets sans dates, fonts auto-hébergées, build GitHub Actions.

**Architecture:** Jekyll inchangé (layouts/includes), CSS assemblé par Liquid dans `css/main.css` à partir de `_includes/css/*.css` ; `landing.css` et `headfoot.css` sont remplacés par un nouvel `editorial.css` ; `common.css` est réécrit (tokens + base typographique). Le déploiement passe du build Pages « classique » à un workflow GitHub Actions (build + minification + lychee + deploy-pages).

**Tech Stack:** Jekyll (gem github-pages, Ruby Homebrew + bundler — voir mémoire projet : Ruby système trop vieux), Liquid, CSS vanilla, GitHub Actions, lightningcss-cli, lychee.

**Conventions de test :** pas de framework de test sur ce repo. Chaque tâche se vérifie par : (1) `bundle exec jekyll build` qui doit terminer sans erreur, (2) des `grep` précis sur `_site/`, (3) contrôle visuel via `bundle exec jekyll serve` aux jalons indiqués. Le « test qui échoue d'abord » correspond ici au grep de l'état attendu exécuté AVANT modification (il doit échouer/être vide), puis APRÈS (il doit réussir).

**Règle de staging (mémoire projet) :** ne jamais `git add -A` ; toujours cibler explicitement les fichiers du site. Ne jamais stager les fichiers mémoire/Claude.

---

### Task 0 : Branche de travail et build de référence

**Files:** aucun fichier modifié.

- [ ] **Step 0.1 : Créer la branche**

```bash
cd /Users/julien/Local/julienbordet.github.io
git checkout -b feat/refonte-editoriale
```

- [ ] **Step 0.2 : Vérifier que le build de référence passe**

```bash
bundle install
bundle exec jekyll build
```

Attendu : `done in X.XXX seconds.` sans erreur. Si `bundler` échoue, vérifier que le Ruby Homebrew est actif (`which ruby` → `/opt/homebrew/...`), pas le Ruby système 2.6.

---

### Task 1 : Fonts auto-hébergées (WOFF2)

**Files:**
- Create: `fonts/playfair-500.woff2`, `fonts/playfair-italic-400.woff2`, `fonts/inter-400.woff2`, `fonts/inter-500.woff2`, `fonts/inter-600.woff2`

- [ ] **Step 1.1 : Télécharger les sous-ensembles latin via google-webfonts-helper**

```bash
cd /Users/julien/Local/julienbordet.github.io
mkdir -p fonts /tmp/gwfh && cd /tmp/gwfh
curl -sL "https://gwfh.mranftl.com/api/fonts/playfair-display?download=zip&subsets=latin&variants=500,italic&formats=woff2" -o playfair.zip
curl -sL "https://gwfh.mranftl.com/api/fonts/inter?download=zip&subsets=latin&variants=regular,500,600&formats=woff2" -o inter.zip
unzip -o playfair.zip && unzip -o inter.zip && ls
```

Attendu : 5 fichiers `.woff2` (playfair-display `500` + `italic`, inter `regular` + `500` + `600`).

- [ ] **Step 1.2 : Renommer et déplacer dans le repo**

```bash
mv playfair-display-*-500.woff2      /Users/julien/Local/julienbordet.github.io/fonts/playfair-500.woff2
mv playfair-display-*-italic.woff2   /Users/julien/Local/julienbordet.github.io/fonts/playfair-italic-400.woff2
mv inter-*-regular.woff2             /Users/julien/Local/julienbordet.github.io/fonts/inter-400.woff2
mv inter-*-500.woff2                 /Users/julien/Local/julienbordet.github.io/fonts/inter-500.woff2
mv inter-*-600.woff2                 /Users/julien/Local/julienbordet.github.io/fonts/inter-600.woff2
ls -la /Users/julien/Local/julienbordet.github.io/fonts/
```

Attendu : 5 fichiers, chacun entre ~10 et ~50 Ko. Si l'API gwfh est indisponible, repli : télécharger les CSS Google Fonts avec un User-Agent Chrome et extraire les URLs du bloc `/* latin */`.

- [ ] **Step 1.3 : Vérifier que Jekyll copie le dossier**

```bash
cd /Users/julien/Local/julienbordet.github.io
bundle exec jekyll build && ls _site/fonts/
```

Attendu : les 5 `.woff2` présents dans `_site/fonts/`.

- [ ] **Step 1.4 : Commit**

```bash
git add fonts/
git commit -m "feat: fonts Playfair Display et Inter auto-hébergées (WOFF2 latin)"
```

---

### Task 2 : Tokens et base typographique (`common.css`)

**Files:**
- Modify: `_includes/css/common.css` (réécriture complète)

- [ ] **Step 2.1 : Test préalable (doit échouer)**

```bash
grep -c "paper" _includes/css/common.css
```

Attendu : `0` (le token n'existe pas encore).

- [ ] **Step 2.2 : Réécrire `_includes/css/common.css`**

Contenu complet du fichier :

```css
/* -------------------------------------------- *\
   Tokens — palette éditoriale (spec 2026-06-11)
\* -------------------------------------------- */

:root {
  --paper:        #F6F2EA;
  --paper-deep:   #F1ECE0;
  --ink:          #0D1F2D;
  --gold-deep:    #8A6D2F;
  --gold:         #C9A84C;
  --sepia:        #C4B083;
  --rule:         #DDD5C4;
  --rule-deep:    #DCD3BE;
  --body-color:   #4A4639;
  --muted:        #5C5749;
  --muted-italic: #6B6452;
  --cream-text:   #F2EDDF;
  --white:        #FFFFFF;
  /* rétrocompatibilité anciens noms (boutons, vieux contenus) */
  --navy:  #0D1F2D;
  --cream: #F6F2EA;
  --text:  #4A4639;
}

/* -------------------------------------------- *\
   Fonts auto-hébergées
\* -------------------------------------------- */

@font-face {
  font-family: 'Playfair Display';
  font-style: normal; font-weight: 500; font-display: swap;
  src: url('/fonts/playfair-500.woff2') format('woff2');
}
@font-face {
  font-family: 'Playfair Display';
  font-style: italic; font-weight: 400; font-display: swap;
  src: url('/fonts/playfair-italic-400.woff2') format('woff2');
}
@font-face {
  font-family: 'Inter';
  font-style: normal; font-weight: 400; font-display: swap;
  src: url('/fonts/inter-400.woff2') format('woff2');
}
@font-face {
  font-family: 'Inter';
  font-style: normal; font-weight: 500; font-display: swap;
  src: url('/fonts/inter-500.woff2') format('woff2');
}
@font-face {
  font-family: 'Inter';
  font-style: normal; font-weight: 600; font-display: swap;
  src: url('/fonts/inter-600.woff2') format('woff2');
}

/* -------------------------------------------- *\
   Reset & base
\* -------------------------------------------- */

* { margin: 0; padding: 0; box-sizing: border-box; }

html, body { height: 100%; }

body {
  font-family: 'Inter', Helvetica, Arial, sans-serif;
  font-weight: 400;
  font-size: 1.05em;
  color: var(--body-color);
  background-color: var(--paper);
}

/* -------------------------------------------- *\
   Typographie
\* -------------------------------------------- */

h1, h2 {
  font-family: 'Playfair Display', Georgia, serif;
  font-weight: 500;
  color: var(--ink);
  text-wrap: balance;
}

h1 { font-size: 2.2em; line-height: 1.15; margin-top: 0.2em; margin-bottom: 0.4em; }

h2 {
  font-size: 1.5em;
  margin-top: 1.2em;
  margin-bottom: 0.4em;
  border-bottom: 1px solid var(--rule);
  padding-bottom: 0.2em;
}

h3 {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: 1.15em;
  font-weight: 500;
  color: var(--ink);
  margin-top: 1.3em;
  margin-bottom: 0.3em;
}

h4 {
  font-family: 'Inter', Helvetica, Arial, sans-serif;
  font-weight: 400;
  font-size: 1em;
  color: var(--muted);
  margin-top: 0.5em;
  line-height: 1.6;
}

h5, h6 {
  font-family: 'Inter', Helvetica, Arial, sans-serif;
  font-size: 0.8em;
  color: var(--muted);
  margin-top: 0.5em;
  margin-bottom: 0.5em;
}

h6 { font-style: italic; }

p {
  margin-top: 1em;
  line-height: 1.7;
  text-align: left;
}

blockquote {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: 1.05em;
  color: var(--muted-italic);
  border-left: 2px solid var(--gold-deep);
  padding-left: 1em;
  margin: 1.2em 0 1.2em 1.5em;
  font-style: italic;
}

.onum { font-feature-settings: "onum"; }

pre {
  font-family: Monaco, Menlo, "Courier New", monospace;
  font-size: 14px;
  color: #333;
  background-color: #f5f1e8;
  border: 1px solid var(--rule);
  border-radius: 4px;
  white-space: pre-wrap;
  word-break: break-all;
  margin: 0 0 15px;
  padding: 10px;
}

code {
  font-family: Monaco, Menlo, "Courier New", monospace;
  font-size: 0.9em;
  color: #333;
  border-radius: 3px;
  border: 1px solid var(--rule);
  background-color: #f5f1e8;
  padding: 0 .3em;
}

pre code { border: none; font-size: 13px; }

/* -------------------------------------------- *\
   Liens et boutons
\* -------------------------------------------- */

a { color: var(--ink); text-decoration: none; }

a:hover {
  color: var(--gold-deep);
  text-decoration: underline;
  text-underline-offset: 3px;
  transition: 0.2s ease-in;
}

a:focus-visible {
  outline: 2px solid var(--gold-deep);
  outline-offset: 2px;
}

/* Lien éditorial — filet or animé */
.lk {
  display: inline-block;
  font-family: 'Inter', sans-serif;
  font-size: 0.85em;
  font-weight: 500;
  color: var(--ink);
  text-decoration: none;
  padding-bottom: 2px;
  border-bottom: 1px solid var(--gold-deep);
  background-image: linear-gradient(var(--gold-deep), var(--gold-deep));
  background-repeat: no-repeat;
  background-size: 0% 2px;
  background-position: left bottom;
  transition: background-size 0.2s ease;
}

.lk:hover { background-size: 100% 2px; color: var(--ink); text-decoration: none; }

.btn-primary {
  display: inline-block;
  background: var(--ink);
  color: var(--paper);
  font-family: 'Inter', sans-serif;
  font-weight: 500;
  font-size: 0.85em;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  border-radius: 2px;
  padding: 10px 22px;
  text-decoration: none;
  transition: background 0.2s ease-in, color 0.2s ease-in;
}

.btn-primary:hover { background: var(--gold-deep); color: var(--paper); text-decoration: none; }

.btn-outline {
  display: inline-block;
  background: transparent;
  color: var(--gold-deep);
  font-family: 'Inter', sans-serif;
  font-weight: 500;
  font-size: 0.85em;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  border: 1px solid var(--gold-deep);
  border-radius: 2px;
  padding: 9px 22px;
  text-decoration: none;
  transition: background 0.2s ease-in, color 0.2s ease-in;
}

.btn-outline:hover { background: var(--gold-deep); color: var(--paper); text-decoration: none; }

/* Rétrocompatibilité boutons existants */
.redbutton   { background: var(--gold-deep); border-radius: 3px; padding: 5px 10px; color: var(--paper); text-decoration: none; }
.greenbutton { background: var(--ink); border-radius: 3px; padding: 5px 10px; color: var(--paper); text-decoration: none; }
.graybutton  { background: #eee8da; border-radius: 3px; padding: 5px 10px; color: var(--muted); text-decoration: none; }
.graybutton:hover { background: var(--gold-deep); color: var(--paper); text-decoration: none; }

/* -------------------------------------------- *\
   Layout prose (pages intérieures, notes)
\* -------------------------------------------- */

#block { width: 100%; }

.prose {
  max-width: 680px;
  margin: 0 auto;
  padding: 40px 24px 64px;
  background-color: transparent;
}

/* -------------------------------------------- *\
   Listes, séparateurs, images
\* -------------------------------------------- */

ul, ol { margin-top: 1.1em; margin-bottom: 1em; }
ul li  { margin-left: 1.1em; line-height: 1.6em; }
ol li  { margin-left: 1.3em; line-height: 1.6em; }

hr { border: none; border-top: 1px solid var(--rule); margin: 36px 0; }

svg, figure { display: block; max-width: 100%; height: auto; }
figure { margin-bottom: 1.55rem; }
figure img { margin-bottom: .75rem; }

.centered { display: block; max-width: 100%; margin: auto; border-radius: 3px; }

.mosaic { width: 30%; display: block; margin-left: 2.5%; padding: 10px 0; float: left; }
.clear { clear: left; }

/* -------------------------------------------- *\
   D3js (anciens posts) — IMPORTANT : scopé à .post
   pour ne pas écraser la gravure SVG du hero
   (une règle CSS `path {}` globale primerait sur
   les attributs stroke="" du SVG)
\* -------------------------------------------- */

.post path { stroke: steelblue; stroke-width: 2; fill: none; }
.post .axis path, .post .axis line { fill: none; stroke: grey; stroke-width: 1; shape-rendering: crispEdges; }
#svgtree { margin: auto; }
```

- [ ] **Step 2.3 : Build et vérification**

```bash
bundle exec jekyll build
grep -c "\-\-paper" _site/css/main.css && grep -c "fonts/playfair-500.woff2" _site/css/main.css
```

Attendu : les deux greps retournent ≥ 1. Build sans erreur. (À ce stade l'ancienne landing est moche mais fonctionnelle — elle sera remplacée en Task 6.)

- [ ] **Step 2.4 : Commit**

```bash
git add _includes/css/common.css
git commit -m "feat: tokens éditoriaux, @font-face auto-hébergées, base typographique (fer à gauche, Playfair 500)"
```

---

### Task 3 : `default.html` — head allégé et script reveal

**Files:**
- Modify: `_layouts/default.html`

- [ ] **Step 3.1 : Test préalable**

```bash
grep -c "google-analytics\|fonts.googleapis" _layouts/default.html
```

Attendu : ≥ 2 (les deux sont encore présents).

- [ ] **Step 3.2 : Réécrire `_layouts/default.html`**

Contenu complet (on retire Google Fonts et Google Analytics, on ajoute les preloads et le script reveal ; MathJax et le JSON-LD sont conservés tels quels) :

```html
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>{{ page.title }} | {{ site.author }}</title>
    <link rel="alternate" type="application/rss+xml" title="Flux du blog" href="https://www.bordet.info/feed.xml" />
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    {% seo %}

    <!-- Fonts auto-hébergées : préchargement des graisses critiques -->
    <link rel="preload" href="/fonts/playfair-500.woff2" as="font" type="font/woff2" crossorigin>
    <link rel="preload" href="/fonts/inter-400.woff2" as="font" type="font/woff2" crossorigin>

    <!-- Custom CSS -->
    <link rel="stylesheet" href="/css/main.css" type="text/css" media="screen, projection" />

    <script type="text/javascript">
      window.MathJax = {
        tex: { packages: ['base', 'ams'] },
        loader: { load: ['ui/menu', '[tex]/ams'] }
      };
    </script>
    <script type="text/javascript" id="MathJax-script" async
            src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js">
    </script>

    <!-- JSON-LD Person -->
    <script type="application/ld+json">
    {
      "@context": "https://schema.org",
      "@type": "Person",
      "name": "Julien Bordet",
      "jobTitle": "Directeur Général",
      "url": "https://www.bordet.info",
      "email": "julien@bordet.info",
      "sameAs": [
        "https://www.linkedin.com/in/julienbordet/"
      ]
    }
    </script>
  </head>
  <body>
    {{ content }}
    <script>
      if (!window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
        document.documentElement.classList.add('js-reveal');
        window.addEventListener('DOMContentLoaded', function () {
          var io = new IntersectionObserver(function (entries) {
            entries.forEach(function (e) {
              if (e.isIntersecting) { e.target.classList.add('is-visible'); io.unobserve(e.target); }
            });
          }, { rootMargin: '0px 0px -10% 0px' });
          document.querySelectorAll('.reveal').forEach(function (el) { io.observe(el); });
        });
      }
    </script>
  </body>
</html>
```

- [ ] **Step 3.3 : Build et vérification**

```bash
bundle exec jekyll build
grep -c "google-analytics\|fonts.googleapis" _site/index.html ; grep -c "preload" _site/index.html
```

Attendu : premier grep `0` (exit code 1), second ≥ 2.

- [ ] **Step 3.4 : Commit**

```bash
git add _layouts/default.html
git commit -m "feat: head allégé — suppression GA et Google Fonts, preload WOFF2, script reveal sobre"
```

---

### Task 4 : Gravure SVG de la carte stratégique

**Files:**
- Create: `_includes/hero-map.svg`

- [ ] **Step 4.1 : Créer `_includes/hero-map.svg`**

Contenu complet (traits sépia via `currentColor` n'est pas nécessaire : couleurs en dur conformes au token `--sepia`) :

```html
<svg class="hero-map" viewBox="0 0 1200 480" preserveAspectRatio="xMidYMid slice" aria-hidden="true" focusable="false">
  <g fill="none" stroke="#C4B083" stroke-width="1">
    <path d="M-20,300 C160,236 300,344 460,284 C620,224 760,176 920,236 C1040,280 1140,208 1220,256"/>
    <path d="M-20,344 C160,284 300,388 460,332 C620,276 760,228 920,284 C1040,324 1140,260 1220,304" opacity="0.5"/>
    <path d="M-20,256 C160,196 300,300 460,244 C620,188 760,136 920,196 C1040,240 1140,168 1220,216" opacity="0.5"/>
    <path d="M140,390 Q520,110 980,176" stroke-dasharray="5 5"/>
    <path d="M230,420 Q620,300 1060,348" stroke-dasharray="5 5" opacity="0.7"/>
    <circle cx="140" cy="390" r="4" fill="#C4B083"/>
    <circle cx="980" cy="176" r="4" fill="#C4B083"/>
    <circle cx="1060" cy="348" r="3.5" fill="#C4B083"/>
    <circle cx="1090" cy="84" r="26"/>
    <circle cx="1090" cy="84" r="17" opacity="0.6"/>
    <path d="M1090,52 L1090,116 M1058,84 L1122,84"/>
    <path d="M80,40 L80,480 M300,20 L300,480 M600,20 L600,480 M900,20 L900,480" opacity="0.22"/>
    <path d="M0,120 L1200,120 M0,240 L1200,240 M0,360 L1200,360" opacity="0.22"/>
  </g>
</svg>
```

- [ ] **Step 4.2 : Commit**

```bash
git add _includes/hero-map.svg
git commit -m "feat: gravure vectorielle sépia de la carte stratégique (hero)"
```

(La vérification visuelle se fera en Task 6 quand le hero l'inclut.)

---

### Task 5 : Header, footer et styles éditoriaux (`editorial.css`)

**Files:**
- Create: `_includes/css/editorial.css`
- Modify: `_includes/header.html`, `_includes/footer.html`, `_layouts/page.html`, `css/main.css`
- Delete: `_includes/css/headfoot.css`

- [ ] **Step 5.1 : Réécrire `_includes/header.html`**

```html
<header class="header-page">
  <nav id="nav">
    <a class="site-title" href="/">Julien Bordet</a>
    <ul>
      <li><a href="/parcours" class="{% if page.url contains 'parcours' %}current{% endif %}">Parcours</a></li>
      <li><a href="/#carnets" class="{% if page.url contains 'blog' %}current{% endif %}">Carnets</a></li>
      <li><a href="/#contact">Contact</a></li>
    </ul>
  </nav>
</header>
```

- [ ] **Step 5.2 : Réécrire `_includes/footer.html`**

```html
<footer id="footer">
  <div class="footer-inner">
    <span>© 2026 Julien Bordet — Nantes</span>
    <span>
      <a href="&#109;&#97;&#105;&#108;&#116;&#111;&#58;&#106;&#117;&#108;&#105;&#101;&#110;&#64;&#98;&#111;&#114;&#100;&#101;&#116;&#46;&#105;&#110;&#102;&#111;">&#106;&#117;&#108;&#105;&#101;&#110;&#64;&#98;&#111;&#114;&#100;&#101;&#116;&#46;&#105;&#110;&#102;&#111;</a>
      &nbsp;·&nbsp;
      <a href="https://www.linkedin.com/in/julienbordet/" target="_blank" rel="noopener">LinkedIn</a>
    </span>
  </div>
</footer>
```

- [ ] **Step 5.3 : Corriger `_layouts/page.html`** (supprime les doubles wrappers `.header-page` / `#footer` qui dupliquaient les IDs)

```html
---
layout: default
---

{% include header.html %}

<div id="block">
	<div class="prose">
	{{ content }}
	</div>
</div>

{% include footer.html %}
```

- [ ] **Step 5.4 : Créer `_includes/css/editorial.css`**

Contenu complet :

```css
/* ============================================ *\
   Éditorial — header, footer, landing, kickers
\* ============================================ */

/* Header */
.header-page {
  background: var(--paper);
  border-bottom: 1px solid var(--rule);
}

.header-page #nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  max-width: 1080px;
  margin: 0 auto;
  padding: 18px 32px;
}

.header-page .site-title {
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: var(--ink);
  text-decoration: none;
}

.header-page .site-title:hover { color: var(--gold-deep); text-decoration: none; }

.header-page #nav ul { list-style: none; display: flex; gap: 28px; margin: 0; padding: 0; }
.header-page #nav li { margin: 0; line-height: 1; }

.header-page #nav li a {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 400;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--gold-deep);
  text-decoration: none;
  padding-bottom: 3px;
  background-image: linear-gradient(var(--gold-deep), var(--gold-deep));
  background-repeat: no-repeat;
  background-size: 0% 1px;
  background-position: left bottom;
  transition: background-size 0.2s ease, color 0.2s ease;
}

.header-page #nav li a:hover,
.header-page #nav li a.current {
  color: var(--ink);
  background-size: 100% 1px;
  text-decoration: none;
}

@media screen and (max-width: 600px) {
  .header-page #nav { padding: 14px 20px; }
  .header-page #nav ul { gap: 18px; }
}

/* Kickers — petites capitales or */
.kicker {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--gold-deep);
  margin: 0 0 14px;
  text-align: left;
}

/* Hero */
.hero-ed {
  position: relative;
  overflow: hidden;
  background: var(--paper);
}

.hero-ed .hero-map {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  opacity: 0.5;
}

.hero-ed-grid {
  position: relative;
  display: grid;
  grid-template-columns: 1.4fr 1fr;
  gap: 32px;
  max-width: 1080px;
  margin: 0 auto;
  padding: 96px 32px 84px;
}

.hero-ed-tagline {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(2em, 4.5vw, 3.1em);
  font-weight: 500;
  line-height: 1.12;
  color: var(--ink);
  margin: 0;
  text-wrap: balance;
}

.hero-ed-aside {
  border-left: 1px solid var(--sepia);
  padding-left: 24px;
  align-self: end;
}

.hero-ed-aside p {
  font-size: 0.85em;
  line-height: 1.65;
  color: var(--body-color);
  margin: 0 0 16px;
  text-align: left;
}

@media screen and (max-width: 700px) {
  .hero-ed-grid { grid-template-columns: 1fr; padding: 64px 24px 56px; }
}

/* Sections numérotées */
.sec { border-top: 1px solid var(--rule); padding: 56px 32px 64px; }
.sec-inner { max-width: 1080px; margin: 0 auto; }
.sec-deep { background: var(--paper-deep); }

.sec-head { display: flex; align-items: baseline; gap: 18px; margin-bottom: 36px; }

.sec-num {
  font-family: 'Playfair Display', Georgia, serif;
  font-style: italic;
  font-size: 1.15em;
  color: var(--gold-deep);
}

.sec-head h2 {
  font-size: 1.5em;
  font-weight: 500;
  margin: 0;
  border: none;
  padding: 0;
  color: var(--ink);
}

.sec-head .sec-rule { flex: 1; border-bottom: 1px solid var(--rule); }
.sec-deep .sec-head .sec-rule { border-color: var(--rule-deep); }

.sec-lede {
  font-family: 'Playfair Display', Georgia, serif;
  font-style: italic;
  font-size: 1.05em;
  color: var(--muted-italic);
  margin: -20px 0 32px;
  text-align: left;
}

/* 01 — trois colonnes de journal */
.cols-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; }

.cols-3 .col { padding: 0 26px; border-left: 1px solid var(--rule); }
.cols-3 .col:first-child { padding-left: 0; border-left: none; }
.cols-3 .col:last-child { padding-right: 0; }

.cols-3 h3 {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: 1.1em;
  font-weight: 500;
  color: var(--ink);
  margin: 0 0 10px;
}

.cols-3 p { font-size: 0.85em; line-height: 1.7; color: var(--muted); margin: 0; text-align: left; }

@media screen and (max-width: 700px) {
  .cols-3 { grid-template-columns: 1fr; }
  .cols-3 .col { padding: 18px 0; border-left: none; border-top: 1px solid var(--rule); }
  .cols-3 .col:first-child { border-top: none; padding-top: 0; }
}

/* 02 — chronologie de revue */
.chrono { display: grid; grid-template-columns: 130px 1fr; }

.chrono-years {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--gold-deep);
  padding: 16px 0;
  font-feature-settings: "onum";
}

.chrono-entry { padding: 14px 0; border-bottom: 1px solid var(--rule-deep); }
.chrono-entry.last { border-bottom: none; }

.chrono-entry .org {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: 1.05em;
  font-weight: 500;
  color: var(--ink);
}

.chrono-entry .role { font-style: italic; font-size: 0.9em; color: var(--muted-italic); }

.chrono-entry p { font-size: 0.85em; color: var(--muted); line-height: 1.65; margin: 6px 0 0; text-align: left; }

.lk-more { display: inline-block; margin-top: 26px; }

@media screen and (max-width: 700px) {
  .chrono { grid-template-columns: 1fr; }
  .chrono-years { padding: 16px 0 0; }
  .chrono-entry { padding-top: 4px; }
}

/* 03 — carnets & bibliothèque */
.carnets-grid { display: grid; grid-template-columns: 1fr 1fr; }

.carnet {
  display: block;
  padding: 18px 26px;
  text-decoration: none;
  border-bottom: 1px solid var(--rule);
}

.carnet:nth-child(odd) { padding-left: 0; }
.carnet:nth-child(even) { border-left: 1px solid var(--rule); }
.carnet:nth-last-child(-n+2) { border-bottom: none; }

.carnet .kicker { font-size: 10.5px; letter-spacing: 0.14em; margin: 0 0 6px; }

.carnet-t {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: 1em;
  color: var(--ink);
  transition: color 0.2s ease;
}

.carnet-t .arrow { color: var(--gold-deep); }
.carnet:hover { text-decoration: none; }
.carnet:hover .carnet-t { color: var(--gold-deep); }

@media screen and (max-width: 700px) {
  .carnets-grid { grid-template-columns: 1fr; }
  .carnet,
  .carnet:nth-child(odd),
  .carnet:nth-child(even) { padding-left: 0; border-left: none; border-bottom: 1px solid var(--rule); }
  .carnet:last-child { border-bottom: none; }
}

/* Contact + colophon (landing) */
.contact-ed { background: var(--ink); padding: 72px 32px 36px; text-align: center; }
.contact-ed .kicker { color: var(--gold); text-align: center; }

.contact-lede {
  font-family: 'Playfair Display', Georgia, serif;
  font-style: italic;
  font-size: 1.5em;
  line-height: 1.4;
  color: var(--cream-text);
  margin: 0 0 28px;
  text-align: center;
}

.contact-links-ed { font-family: 'Inter', sans-serif; font-size: 0.9em; text-align: center; margin: 0; }

.contact-links-ed a {
  color: var(--gold);
  text-decoration: none;
  border-bottom: 1px solid rgba(201, 168, 76, 0.5);
  padding-bottom: 2px;
}

.contact-links-ed a:hover { color: var(--cream-text); text-decoration: none; }
.contact-links-ed .sep { color: rgba(242, 237, 223, 0.35); margin: 0 16px; }

.colophon {
  max-width: 1080px;
  margin: 56px auto 0;
  padding-top: 18px;
  border-top: 1px solid rgba(201, 168, 76, 0.2);
  display: flex;
  justify-content: space-between;
  font-family: 'Inter', sans-serif;
  font-size: 10px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: rgba(242, 237, 223, 0.4);
}

/* Footer pages intérieures */
#footer { background: var(--ink); padding: 26px 32px; clear: both; }

#footer .footer-inner {
  max-width: 1080px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 8px;
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  letter-spacing: 0.06em;
  color: rgba(242, 237, 223, 0.45);
}

#footer a { color: var(--gold); text-decoration: none; }
#footer a:hover { color: var(--cream-text); text-decoration: none; }

/* Reveal au scroll */
.js-reveal .reveal { opacity: 0; transform: translateY(8px); transition: opacity 0.25s ease, transform 0.25s ease; }
.js-reveal .reveal.is-visible { opacity: 1; transform: none; }
```

- [ ] **Step 5.5 : Mettre à jour `css/main.css`** (editorial remplace headfoot ; landing reste temporairement pour l'ancienne home)

```css
---
---

{% include css/common.css %}
{% include css/editorial.css %}
{% include css/post.css %}
{% include css/syntax.css %}
{% include css/landing.css %}
```

- [ ] **Step 5.6 : Supprimer `headfoot.css`**

```bash
git rm _includes/css/headfoot.css
```

- [ ] **Step 5.7 : Build et vérification**

```bash
bundle exec jekyll build
grep -c "header-page" _site/css/main.css ; grep -c "footer-inner" _site/css/main.css ; grep -c "carnets" _site/blog/index.html
```

Attendu : deux premiers greps ≥ 1 ; le header rendu contient le lien `/#carnets` (vérifier : `grep -c "/#carnets" _site/blog/index.html` ≥ 1).

- [ ] **Step 5.8 : Contrôle visuel rapide**

```bash
bundle exec jekyll serve
```

Ouvrir `http://localhost:4000/blog/` : header crème en petites capitales, footer navy fin. (La home est encore l'ancienne — normal.)

- [ ] **Step 5.9 : Commit**

```bash
git add _includes/css/editorial.css _includes/header.html _includes/footer.html _layouts/page.html css/main.css
git commit -m "feat: header/footer éditoriaux, styles editorial.css, suppression headfoot.css"
```

---

### Task 6 : Page d'accueil éditoriale (`index.html`)

**Files:**
- Modify: `index.html` (réécriture complète), `_layouts/landing.html`, `css/main.css`
- Delete: `_includes/css/landing.css`

- [ ] **Step 6.1 : Mettre à jour `_layouts/landing.html`** (le bloc contact embarque son colophon → pas de footer)

```html
---
layout: default
---

{% include header.html %}

{{ content }}
```

- [ ] **Step 6.2 : Réécrire `index.html`**

Contenu complet :

```html
---
layout: landing
title: Julien Bordet
---

<!-- ========== HERO ========== -->
<section class="hero-ed">
  {% include hero-map.svg %}
  <div class="hero-ed-grid">
    <div class="hero-ed-main">
      <p class="kicker">Directeur Général — Direction opérationnelle</p>
      <h1 class="hero-ed-tagline">Une stratégie ne vaut que si elle survit au contact du terrain.</h1>
    </div>
    <div class="hero-ed-aside">
      <p>Vingt ans à faire ce trajet dans les deux sens — du conseil en infrastructures critiques à la direction de PME industrielles.</p>
      <a class="lk" href="/parcours">Lire le parcours →</a>
    </div>
  </div>
</section>

<!-- ========== 01 — CE QUE J'APPORTE ========== -->
<section class="sec reveal">
  <div class="sec-inner">
    <div class="sec-head">
      <span class="sec-num">01</span>
      <h2>Ce que j'apporte</h2>
      <span class="sec-rule"></span>
    </div>
    <div class="cols-3">
      <div class="col">
        <h3>Direction opérationnelle</h3>
        <p>Pilotage de PME industrielles en croissance — gestion d'équipes, arbitrages sous contrainte, maintien en condition opérationnelle de systèmes complexes.</p>
      </div>
      <div class="col">
        <h3>Transformation &amp; systèmes</h3>
        <p>Accompagnement de transformations d'envergure, déploiement d'innovations industrielles dans des délais courts.</p>
      </div>
      <div class="col">
        <h3>Stratégie &amp; analyse</h3>
        <p>Conseil en stratégie pour des infrastructures critiques, analyse géopolitique, interactions public/privé et commande publique.</p>
      </div>
    </div>
  </div>
</section>

<!-- ========== 02 — PARCOURS ========== -->
<section class="sec sec-deep reveal">
  <div class="sec-inner">
    <div class="sec-head">
      <span class="sec-num">02</span>
      <h2>Parcours</h2>
      <span class="sec-rule"></span>
    </div>
    <div class="chrono">
      <div class="chrono-years">2022 — actuel</div>
      <div class="chrono-entry"><span class="org">Progicar</span><span class="role"> — Directeur Général</span></div>
      <div class="chrono-years">2017 — 2022</div>
      <div class="chrono-entry"><span class="org">Stimio</span><span class="role"> — Directeur Général Délégué &amp; co-actionnaire</span></div>
      <div class="chrono-years">2011 — 2017</div>
      <div class="chrono-entry"><span class="org">EY</span><span class="role"> — Manager, stratégie &amp; infrastructures critiques</span></div>
      <div class="chrono-years">2008 — 2009</div>
      <div class="chrono-entry"><span class="org">MBA — Audencia</span><span class="role"> — International MBA stratégie &amp; finance</span></div>
      <div class="chrono-years">2004 — 2011</div>
      <div class="chrono-entry"><span class="org">Wavestone</span><span class="role"> — Consultant Senior, transformation</span></div>
      <div class="chrono-years">2002 — 2004</div>
      <div class="chrono-entry last"><span class="org">INTRINsec</span><span class="role"> — Manager &amp; Business Developer, cybersécurité</span></div>
    </div>
    <a class="lk lk-more" href="/parcours">Parcours complet, 2002 — 2026 →</a>
  </div>
</section>

<!-- ========== 03 — CARNETS & BIBLIOTHÈQUE ========== -->
<section class="sec reveal" id="carnets">
  <div class="sec-inner">
    <div class="sec-head">
      <span class="sec-num">03</span>
      <h2>Carnets &amp; bibliothèque</h2>
      <span class="sec-rule"></span>
    </div>
    <p class="sec-lede">Ce qu'un dirigeant lit, calcule et observe en dehors des comités de direction.</p>
    <div class="carnets-grid">
      <a class="carnet" href="/reading">
        <p class="kicker">Lectures</p>
        <span class="carnet-t">Bibliothèque — stratégie, histoire, sciences <span class="arrow">→</span></span>
      </a>
      <a class="carnet" href="/blog">
        <p class="kicker">Réflexions</p>
        <span class="carnet-t">Notes — géopolitique, technique, systèmes <span class="arrow">→</span></span>
      </a>
      <a class="carnet" href="/physique">
        <p class="kicker">Physique &amp; calcul</p>
        <span class="carnet-t">Carnets d'exploration scientifique <span class="arrow">→</span></span>
      </a>
      <a class="carnet" href="/photos">
        <p class="kicker">Photographies</p>
        <span class="carnet-t">Carnet visuel — voyages, lumières <span class="arrow">→</span></span>
      </a>
    </div>
  </div>
</section>

<!-- ========== CONTACT + COLOPHON ========== -->
<section class="contact-ed" id="contact">
  <p class="kicker">Contact</p>
  <p class="contact-lede">Échanger sur un projet, une opportunité,<br>ou simplement des idées.</p>
  <p class="contact-links-ed">
    <a href="&#109;&#97;&#105;&#108;&#116;&#111;&#58;&#106;&#117;&#108;&#105;&#101;&#110;&#64;&#98;&#111;&#114;&#100;&#101;&#116;&#46;&#105;&#110;&#102;&#111;">&#106;&#117;&#108;&#105;&#101;&#110;&#64;&#98;&#111;&#114;&#100;&#101;&#116;&#46;&#105;&#110;&#102;&#111;</a>
    <span class="sep">·</span>
    <a href="https://www.linkedin.com/in/julienbordet/" target="_blank" rel="noopener">LinkedIn</a>
  </p>
  <div class="colophon">
    <span>© 2026 Julien Bordet — Nantes</span>
    <span>bordet.info</span>
  </div>
</section>
```

- [ ] **Step 6.3 : Retirer `landing.css` de `css/main.css`**

```css
---
---

{% include css/common.css %}
{% include css/editorial.css %}
{% include css/post.css %}
{% include css/syntax.css %}
```

- [ ] **Step 6.4 : Supprimer `landing.css`**

```bash
git rm _includes/css/landing.css
```

- [ ] **Step 6.5 : Build et vérification**

```bash
bundle exec jekyll build
grep -c "hero-ed-tagline" _site/index.html
grep -c "hero-bg.png" _site/index.html _site/css/main.css
grep -c "id=\"carnets\"" _site/index.html
```

Attendu : `hero-ed-tagline` ≥ 1 ; `hero-bg.png` = 0 partout (exit 1) ; `id="carnets"` = 1.

- [ ] **Step 6.6 : Contrôle visuel complet**

```bash
bundle exec jekyll serve
```

Vérifier sur `http://localhost:4000/` : gravure sépia visible sous le hero, tagline asymétrique, 3 colonnes à filets, chronologie en marge, grille carnets 2×2, contact navy avec colophon. Tester en largeur mobile (DevTools ≤ 700 px) : tout s'empile proprement.

- [ ] **Step 6.7 : Commit**

```bash
git add index.html _layouts/landing.html css/main.css
git commit -m "feat: page d'accueil éditoriale — hero gravure, sections numérotées, carnets, contact colophon"
```

---

### Task 7 : Notes sans dates (`post.html` + `post.css`)

**Files:**
- Modify: `_layouts/post.html` (réécriture), `_includes/css/post.css` (réécriture)

- [ ] **Step 7.1 : Test préalable**

```bash
grep -c "disqus\|date_to_string\|number_of_words\|page.date" _layouts/post.html
```

Attendu : ≥ 3 (date, mots, Disqus encore présents).

- [ ] **Step 7.2 : Réécrire `_layouts/post.html`**

Contenu complet (plus de date affichée, plus de compteur de mots, plus de Disqus ; rubrique en kicker ; navigation précédent/suivant conservée sans dates) :

```html
---
layout: page
---

<article class="post">
	{% if page.categories and page.categories != empty %}
	<p class="kicker">{{ page.categories | join: " · " }}</p>
	{% endif %}

	<h1 class="post-title">{{ page.title }}</h1>

	{% if page.image %}
	<img src="/images/blog/{{ page.image }}" alt="" class="post-img" />
	{% endif %}

	{{ content }}

	<nav class="previous_next">
		{% if page.previous %}
		<a href="{{ page.previous.url }}" title="Note précédente">← précédente</a>
		{% endif %}
		{% if page.next %}
		<a href="{{ page.next.url }}" title="Note suivante">suivante →</a>
		{% endif %}
		<a class="lk" href="/blog/">Toutes les notes</a>
	</nav>
</article>
```

- [ ] **Step 7.3 : Réécrire `_includes/css/post.css`**

Contenu complet :

```css
/* ============================================ *\
   Notes (articles) et listes de notes
\* ============================================ */

.post .kicker { margin-bottom: 10px; }

.post .post-title {
  font-size: 1.9em;
  font-weight: 500;
  line-height: 1.2;
  margin: 0 0 0.8em;
}

.post pre { margin-top: 10px; }
.post code { margin-top: 10px; }

.post img,
.post img.centered,
.post .post-img {
  max-width: 100%;
  display: block;
  margin: 1.5em auto;
  border-radius: 3px;
}

.previous_next {
  display: flex;
  gap: 24px;
  align-items: baseline;
  flex-wrap: wrap;
  margin-top: 48px;
  padding-top: 18px;
  border-top: 1px solid var(--rule);
  font-size: 0.85em;
}

.previous_next a { color: var(--gold-deep); text-decoration: none; }
.previous_next a:hover { color: var(--ink); }

/* Index des notes par thème */
.notes-theme { margin-top: 2.2em; }
.notes-theme .kicker { margin-bottom: 4px; }
.notes-theme ul { margin-top: 0.4em; list-style: none; }
.notes-theme li { margin-left: 0; padding: 7px 0; border-bottom: 1px solid var(--rule); line-height: 1.5; }
.notes-theme li:last-child { border-bottom: none; }
.notes-theme li a { font-family: 'Playfair Display', Georgia, serif; font-size: 1.02em; color: var(--ink); }
.notes-theme li a:hover { color: var(--gold-deep); text-decoration: none; }

/* Archive (liste complète) */
.listing-item { list-style: none; padding: 7px 0; border-bottom: 1px solid var(--rule); }
.listing-item a { font-family: 'Playfair Display', Georgia, serif; color: var(--ink); }
.listing-item a:hover { color: var(--gold-deep); text-decoration: none; }

.meta, .categories, .tags, .archive-list, #tag_cloud {
  margin-left: auto;
  margin-right: auto;
  max-width: 680px;
  font-size: 90%;
  color: var(--muted);
}

/* ancien cours math13 */
.lecture small { color: var(--muted); font-family: 'Inter', Helvetica, Arial, sans-serif; font-size: 80%; }
```

- [ ] **Step 7.4 : Build et vérification**

```bash
bundle exec jekyll build
F=$(ls _site/blog/2021/01/23/*.html | head -1)
grep -c "disqus" "$F" ; grep -c "Janvier\|Février\|Décembre" "$F" ; grep -c "post-title" "$F"
```

Attendu : `disqus` = 0 (exit 1) ; mois = 0 (exit 1) ; `post-title` ≥ 1.

- [ ] **Step 7.5 : Commit**

```bash
git add _layouts/post.html _includes/css/post.css
git commit -m "feat: notes intemporelles — suppression dates, compteur de mots et Disqus, rubrique en kicker"
```

---

### Task 8 : Index des notes par thème + archive sans dates

**Files:**
- Modify: `blog/index.md` (réécriture), `blog/archive/index.html` (réécriture)

- [ ] **Step 8.1 : Réécrire `blog/index.md`**

Contenu complet (classement thématique via `site.categories`, plus de flux antéchronologique paginé, aucune date) :

```html
---
title: Carnets — notes durables
layout: page
---

<div class="notes-index">

  <h1>Réflexions &amp; notes</h1>
  <p class="sec-lede">Notes durables, classées par thème — géopolitique, technique, systèmes, sciences.</p>

  {% assign sorted_categories = site.categories | sort %}
  {% for category in sorted_categories %}
  <section class="notes-theme">
    <p class="kicker">{{ category[0] | capitalize }}</p>
    <ul>
      {% for post in category[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endfor %}
    </ul>
  </section>
  {% endfor %}

  {% assign uncategorized = site.posts | where_exp: "p", "p.categories == empty" %}
  {% if uncategorized.size > 0 %}
  <section class="notes-theme">
    <p class="kicker">Autres notes</p>
    <ul>
      {% for post in uncategorized %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endfor %}
    </ul>
  </section>
  {% endif %}

  <p style="margin-top: 2.5em;"><a class="lk" href="/blog/archive/">Liste complète des notes →</a></p>

</div>
```

- [ ] **Step 8.2 : Réécrire `blog/archive/index.html`**

Contenu complet (toutes les notes, sans dates, sans exclusion de catégorie) :

```html
---
layout: page
title: Toutes les notes
---

<div class="archive-list">
	<h1>Toutes les notes</h1>
	<ul class="listing">
		{% for post in site.posts %}
		<li class="listing-item">
			<a href="{{ post.url }}" class="title">{{ post.title }}</a>
		</li>
		{% endfor %}
	</ul>
</div>
```

- [ ] **Step 8.3 : Build et vérification**

```bash
bundle exec jekyll build
grep -c "notes-theme" _site/blog/index.html
grep -c "datetime=" _site/blog/archive/index.html
grep -rc "Janvier\|Février\|Mars 20" _site/blog/index.html
```

Attendu : `notes-theme` ≥ 3 (plusieurs thèmes) ; `datetime=` = 0 (exit 1) ; mois = 0 (exit 1).

- [ ] **Step 8.4 : Commit**

```bash
git add blog/index.md blog/archive/index.html
git commit -m "feat: index des notes par thème, archive sans dates"
```

---

### Task 9 : Page Parcours — chronologie éditoriale

**Files:**
- Modify: `parcours/index.md` (réécriture de la section « Parcours professionnel » en grille chrono ; le reste du contenu est conservé tel quel)

- [ ] **Step 9.1 : Réécrire `parcours/index.md`**

Contenu complet :

```html
---
title: Parcours — Julien Bordet
layout: page
---

# Julien Bordet

#### Ingénieur SI de formation (Télécom Sud Paris), MBA international (Audencia), 20 ans entre le conseil en transformation et la direction de PME industrielles. Aujourd'hui Directeur Général de Progicar. Intérêt marqué pour les enjeux de défense, de souveraineté technologique et de géopolitique — domaines dans lesquels mes compétences opérationnelles ont une application naturelle.

<a class="btn-primary" href="https://www.linkedin.com/in/julienbordet/" target="_blank" rel="noopener">LinkedIn</a>
&nbsp;
<a class="btn-outline" href="/about/CV-Julien_Bordet.pdf" target="_blank">Télécharger le CV (PDF)</a>

<hr>

## Compétences clés

- Pilotage de programmes multi-acteurs et gestion d'écosystèmes industriels complexes
- Optimisation opérationnelle et maintien en condition opérationnelle
- Compréhension des interactions public/privé et contraintes de la commande publique
- Conception et déploiement rapide d'innovations industrielles
- Développement international et partenariats stratégiques
- Sécurisation, déploiement et gouvernance de systèmes d'information critiques
- Pilotage stratégique et financier dans des contextes d'investissement long

<hr>

## Parcours professionnel

<div class="chrono">
  <div class="chrono-years">2022 — actuel</div>
  <div class="chrono-entry">
    <span class="org">Progicar</span><span class="role"> — Directeur Général</span>
    <p>Optimisation opérationnelle pour l'industrie automobile. Nantes. Pilote la conception de solutions logicielles avec 15+ grands donneurs d'ordre (distributeurs automobiles) ; développe une architecture modulaire conciliant exigences grands comptes et rationalisation industrielle ; conçoit une plateforme d'optimisation des flux de reconditionnement de flottes, réduisant les temps d'immobilisation de 30 % ; impulse une dynamique d'innovation agile au sein d'une PME tech de 30 personnes.</p>
  </div>
  <div class="chrono-years">2017 — 2022</div>
  <div class="chrono-entry">
    <span class="org">Stimio</span><span class="role"> — Directeur Général Délégué &amp; co-actionnaire</span>
    <p>Maintenance prédictive et optimisation industrielle. Nantes. Développe une solution de maintenance prédictive pour infrastructures ferroviaires, optimisant le MCO et la continuité de service d'une activité d'importance vitale ; mutualise des technologies entre ferroviaire haute exigence et bus grand public (technologies duales) ; élabore la stratégie de croissance et pilote deux levées de fonds (3 M€) ; met en place un écosystème d'innovation collaborative avec des partenaires industriels de tailles variées.</p>
  </div>
  <div class="chrono-years">2011 — 2017</div>
  <div class="chrono-entry">
    <span class="org">EY</span><span class="role"> — Manager, conseil en stratégie : infrastructures critiques, énergie, transport</span>
    <p>Missions centrées sur des enjeux de souveraineté et d'intérêt national. Élabore des plans stratégiques pour de grands ports maritimes et aéroports, intégrant les enjeux de sûreté, sécurité et cybersécurité d'infrastructures critiques ; conduit l'analyse stratégique pour la Région du projet de reprise de l'aéroport de Toulouse ; accompagne Bpifrance dans la sélection et l'évaluation de projets innovants à fort potentiel stratégique.</p>
  </div>
  <div class="chrono-years">2004 — 2011</div>
  <div class="chrono-entry">
    <span class="org">Wavestone</span><span class="role"> — Consultant Senior, conseil en transformation (2004 — 2008 / 2009 — 2011)</span>
    <p>Pilote des projets de transformation d'envergure (&gt;1 000 jours.homme) ; manage des équipes de 6 à 20 collaborateurs.</p>
  </div>
  <div class="chrono-years">2002 — 2004</div>
  <div class="chrono-entry last">
    <span class="org">INTRINsec</span><span class="role"> — Manager &amp; Business Developer, cybersécurité</span>
    <p>Paris &amp; Shanghai. Conduit des missions de cybersécurité opérationnelle (tests d'intrusion, architecture sécurisée, réponse à incident) ; mène une mission en Chine intégrant les enjeux de sécurité économique et de protection contre l'espionnage industriel.</p>
  </div>
</div>

<hr>

## Formation

**International MBA en stratégie et finance** — Audencia (2008-2009)

**Ingénieur en Système d'Information** — Télécom Sud Paris (1999-2003)

**Séminaire — Le renseignement : quelles adaptations pour quelles menaces ?** — Institut d'Études de Géopolitique Appliquée (2020)

<hr>

## Engagements

- Membre de **Chatham House**, think tank international de référence sur les questions de géopolitique et de sécurité
- Membre actif de l'association des anciens d'**Audencia** : mentorat et jury MBA
- Entraîneur bénévole de basketball

<hr>

## Centres d'intérêt

Géopolitique et défense · Monde iranien · Cybersécurité · Cosmologie · Psychologie cognitive · Piano · Basketball · Science-fiction

Je lis et parle le persan : زبان فارسی را صحبت می کنم

<hr>

## Contact

<a href="&#109;&#97;&#105;&#108;&#116;&#111;&#58;&#106;&#117;&#108;&#105;&#101;&#110;&#64;&#98;&#111;&#114;&#100;&#101;&#116;&#46;&#105;&#110;&#102;&#111;">&#106;&#117;&#108;&#105;&#101;&#110;&#64;&#98;&#111;&#114;&#100;&#101;&#116;&#46;&#105;&#110;&#102;&#111;</a>

[Clé publique GPG](/about/julienbordet-asc.gpg) pour les échanges sensibles.
```

- [ ] **Step 9.2 : Build et vérification**

```bash
bundle exec jekyll build
grep -c "chrono-entry" _site/parcours/index.html
```

Attendu : ≥ 5.

- [ ] **Step 9.3 : Contrôle visuel**

`bundle exec jekyll serve` → `http://localhost:4000/parcours/` : années en marge or, hairlines, descriptions lisibles, mobile OK.

- [ ] **Step 9.4 : Commit**

```bash
git add parcours/index.md
git commit -m "feat: page parcours en chronologie éditoriale"
```

---

### Task 10 : Workflow GitHub Actions (build, minification, liens, deploy)

**Files:**
- Create: `.github/workflows/pages.yml`
- Delete: `images/website/hero-bg.png`

- [ ] **Step 10.1 : Vérifier que `hero-bg.png` n'est plus référencé**

```bash
grep -rn "hero-bg" --include="*.html" --include="*.css" --include="*.md" --include="*.yml" . | grep -v _site | grep -v docs/
```

Attendu : aucune occurrence. Si une occurrence apparaît, la traiter avant suppression.

- [ ] **Step 10.2 : Supprimer l'image**

```bash
git rm images/website/hero-bg.png
```

- [ ] **Step 10.3 : Créer `.github/workflows/pages.yml`**

Contenu complet :

```yaml
name: Build and deploy

on:
  push:
    branches: [master]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.3'
          bundler-cache: true

      - name: Build Jekyll
        run: bundle exec jekyll build
        env:
          JEKYLL_ENV: production

      - name: Minify CSS
        run: npx --yes lightningcss-cli --minify _site/css/main.css -o _site/css/main.css

      - name: Check internal links
        uses: lycheeverse/lychee-action@v2
        continue-on-error: true
        with:
          args: --offline --no-progress _site

      - uses: actions/upload-pages-artifact@v3
        with:
          path: _site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 10.4 : Commit**

```bash
git add .github/workflows/pages.yml
git commit -m "feat: déploiement GitHub Actions — build Jekyll, minification CSS, vérification de liens"
```

- [ ] **Step 10.5 : ACTION MANUELLE pour Julien (à signaler dans le récap final)**

Après merge sur `master` : GitHub → repo `julienbordet.github.io` → Settings → Pages → « Build and deployment » → Source : **GitHub Actions** (au lieu de « Deploy from a branch »). Le fichier `CNAME` est copié dans `_site` par Jekyll, le domaine `www.bordet.info` reste actif.

---

### Task 11 : Vérifications finales

**Files:** aucun.

- [ ] **Step 11.1 : Build complet et greps de conformité spec**

```bash
bundle exec jekyll build
echo "--- aucune requête Google ---"
grep -rc "fonts.googleapis\|google-analytics" _site/index.html _site/parcours/index.html _site/blog/index.html || echo OK
echo "--- gravure présente ---"
grep -c "hero-map" _site/index.html
echo "--- aucune date visible dans les notes ---"
grep -rl "date_to_string\|(.*mots)" _site/blog/ || echo OK
echo "--- URLs notes inchangées ---"
ls _site/blog/2021/01/23/ | head -2
```

Attendu : OK / ≥ 1 / OK / les fichiers HTML existants listés.

- [ ] **Step 11.2 : Contrôle visuel final**

`bundle exec jekyll serve` : home, /parcours/, /blog/, une note (ex. `/blog/2021/01/23/macOSGatekeeper.html`), /blog/archive/. Desktop + mobile (DevTools).

- [ ] **Step 11.3 : Lighthouse (optionnel, recommandé)**

```bash
npx --yes lighthouse http://localhost:4000/ --quiet --chrome-flags="--headless" --only-categories=performance,accessibility,best-practices,seo --output=json | python3 -c "import json,sys; d=json.load(sys.stdin); print({k: round(v['score']*100) for k,v in d['categories'].items()})"
```

Attendu : ≥ 95 partout (le serveur local n'a pas la compression du CDN GitHub — tolérer 90+ en local sur performance).

- [ ] **Step 11.4 : Push et PR**

```bash
git push -u origin feat/refonte-editoriale
gh pr create --title "Refonte éditoriale — direction A" --body "Implémente la spec docs/superpowers/specs/2026-06-11-refonte-editoriale-design.md

- Hero gravure SVG sépia, tagline-thèse, mise en page asymétrique
- Sections numérotées 01/02/03, chronologie de revue, carnets sans dates
- Fonts auto-hébergées WOFF2, suppression GA et Google Fonts
- Index des notes par thème, archive sans dates, URLs inchangées
- Déploiement GitHub Actions (build + minify + lychee)

⚠️ Après merge : Settings → Pages → Source = GitHub Actions

🤖 Generated with [Claude Code](https://claude.com/claude-code)"
```

---

## Auto-revue du plan (spec → tâches)

| Exigence spec | Tâche |
|---|---|
| Palette tokens (§3) | Task 2 |
| Typo auto-hébergée, onum, fer à gauche (§4) | Tasks 1, 2 |
| Gravure SVG hero, retrait photo (§2) | Tasks 4, 6, 10 |
| Structure 5 blocs landing (§5) | Task 6 |
| Header/footer/colophon (§5) | Task 5 |
| Sans dates, URLs inchangées, classement thématique (§6) | Tasks 7, 8 |
| RSS conservé (§6) | aucun changement à `feed.xml` (vérifié Task 11) |
| /parcours chronologie (§7) | Task 9 |
| Pages notes kicker + 680 px (§7) | Tasks 2, 7 |
| Micro-interactions + reduced-motion (§8) | Tasks 3, 5 |
| Actions : fonts, minify, lychee, suppression GA (§9) | Tasks 3, 10 |
| Lighthouse ≥ 95 (§11) | Task 11 |
```
