---
title: Physique
layout: page
---

<div id="posts">

    {% for post in site.categories.physique offset: 0 limit: 10 %}
    	<h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
	    <h5>{{ post.date | date: "%B %d, %Y" }}</h5>
	    {% if post.image %}
	    <p>
	    	<a href="{{ post.url }}"><img class="centered" src="/images/blog/{{post.image}}" alt="" width="450px" /></a>
	    </p>
	    {% endif %}
        <p>{{ post.excerpt }} </p>
        <p>	<a class="graybutton" href="{{ post.url }}">Suite</a></p>
        <br/>
        <hr/>
    {% endfor %}

	<p>
	<a class="greenbutton" href="/physique/archive/" title="an archive of all posts">Voir tous les posts &rarr;</a>
	</p>
	
</div>

