---
title: Mac
layout: page
---

<div class="notes-index">

  <h1>Mac</h1>

  <ul class="listing">
    {% for post in site.categories.mac %}
    <li class="listing-item"><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>

</div>
