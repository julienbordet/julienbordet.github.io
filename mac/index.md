---
title: Physique
layout: page
---

<div id="posts">

    {% for post in site.categories.mac offset: 0 limit: 10 %}
    	<h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
        <h5>
                {% assign m = post.date | date: "%-m" %}
                {{ post.date | date: "%-d" }}
                {% case m %}
                  {% when '1' %}Janvier
                  {% when '2' %}Février
                  {% when '3' %}Mars
                  {% when '4' %}Avril
                  {% when '5' %}Mai
                  {% when '6' %}Juin
                  {% when '7' %}Juillet
                  {% when '8' %}Aout
                  {% when '9' %}Septembre
                  {% when '10' %}Octobre
                  {% when '11' %}Novembre
                  {% when '12' %}Décembre
                {% endcase %}
                {{ post.date | date: "%Y" }}
                &nbsp;({{ post.content | number_of_words }} mots)
        </h5>
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
	<a class="greenbutton" href="/mac/archive/" title="an archive of all posts">Voir tous les posts &rarr;</a>
	</p>
	
</div>

