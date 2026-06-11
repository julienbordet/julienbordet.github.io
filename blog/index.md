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
