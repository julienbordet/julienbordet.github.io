---
title: Carnets — notes durables
layout: page
---

<div class="notes-index" id="notes-index">

  <h1>Réflexions &amp; notes</h1>
  <p class="sec-lede">Quelques réflexions posées sur le papier.</p>

  <div class="planches" role="list">
    {% for rubrique in site.data.rubriques %}
    {% assign nb = 0 %}
    {% for post in site.posts %}
      {% assign in_rubrique = false %}
      {% for c in post.categories %}
        {% if rubrique.categories contains c %}{% assign in_rubrique = true %}{% endif %}
      {% endfor %}
      {% if in_rubrique %}{% assign nb = nb | plus: 1 %}{% endif %}
    {% endfor %}
    <a class="planche" role="listitem" href="#{{ rubrique.slug }}" data-rubrique="{{ rubrique.slug }}" aria-expanded="false">
      {% assign vignette = rubrique.slug | prepend: 'vignettes/' | append: '.svg' %}
      {% include {{ vignette }} %}
      <span class="planche-t">{{ rubrique.titre }}</span>
      <span class="planche-n">{{ nb }} notes <span class="planche-fleche" aria-hidden="true">▾</span></span>
    </a>
    {% endfor %}
  </div>

  {% for rubrique in site.data.rubriques %}
  <section class="rubrique" id="{{ rubrique.slug }}">
    <div class="rubrique-head">
      <p class="kicker">{{ rubrique.titre }}</p>
      <p class="rubrique-desc">{{ rubrique.description }}</p>
    </div>
    <ul>
      {% for post in site.posts %}
        {% assign in_rubrique = false %}
        {% for c in post.categories %}
          {% if rubrique.categories contains c %}{% assign in_rubrique = true %}{% endif %}
        {% endfor %}
        {% if in_rubrique %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </section>
  {% endfor %}

  {% assign autres = "" | split: "" %}
  {% for post in site.posts %}
    {% assign matched = false %}
    {% for rubrique in site.data.rubriques %}
      {% for c in post.categories %}
        {% if rubrique.categories contains c %}{% assign matched = true %}{% endif %}
      {% endfor %}
    {% endfor %}
    {% unless matched %}{% assign autres = autres | push: post %}{% endunless %}
  {% endfor %}
  {% if autres.size > 0 %}
  <section class="rubrique rubrique-autres" id="autres">
    <div class="rubrique-head">
      <p class="kicker">Autres notes</p>
    </div>
    <ul>
      {% for post in autres %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endfor %}
    </ul>
  </section>
  {% endif %}

</div>

<script>
(function () {
  var index = document.getElementById('notes-index');
  if (!index) return;
  index.classList.add('js-planches');
  var planches = index.querySelectorAll('.planche');
  var rubriques = index.querySelectorAll('.rubrique');

  function ouvrir(slug, scroll) {
    rubriques.forEach(function (r) { r.classList.toggle('open', r.id === slug); });
    planches.forEach(function (p) {
      var actif = p.dataset.rubrique === slug;
      p.classList.toggle('active', actif);
      p.setAttribute('aria-expanded', actif ? 'true' : 'false');
    });
    if (scroll) {
      var cible = document.getElementById(slug);
      if (cible) cible.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }
  }

  function fermer() {
    rubriques.forEach(function (r) { r.classList.remove('open'); });
    planches.forEach(function (p) { p.classList.remove('active'); p.setAttribute('aria-expanded', 'false'); });
  }

  planches.forEach(function (p) {
    p.addEventListener('click', function (e) {
      e.preventDefault();
      var slug = p.dataset.rubrique;
      if (p.classList.contains('active')) {
        fermer();
        history.replaceState(null, '', location.pathname);
      } else {
        ouvrir(slug, true);
        history.replaceState(null, '', '#' + slug);
      }
    });
  });

  var hash = location.hash.replace('#', '');
  if (hash && document.getElementById(hash)) {
    ouvrir(hash, true);
  } else {
    ouvrir(planches.length ? planches[0].dataset.rubrique : '', false);
  }
})();
</script>
