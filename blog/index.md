---
title: Blog
layout: page
---
<div id="posts">

    {% for post in site.posts offset: 0 limit: 10 %}
    	<h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
	    <h5>{{ post.date | date: "%B %d, %Y" }}</h5>
	    {% if post.image %}
	    <p>
	    	<a href="{{ post.url }}"><img class="centered" src="/images/blog/{{post.image}}" alt=""></a>
    	</p>
    	{% endif %}
        <p>{{ post.excerpt }} </p>
        <p>	<a class="graybutton" href="{{ post.url }}">Continue reading</a></p>
        <br/>
        <hr/>
    {% endfor %}

</div>