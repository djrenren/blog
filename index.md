---
layout: default
title: John Renner 
---
<section>
<h2 class="section">MOST RECENT POST</h2>
{% for post in site.posts limit:1 %}
<div class="entry" href="#">
    <h1 class="title head-text"><a class="title-link" href="{{post.url}}">{{ post.title }}</a></h1>
    <div class="extras">
        <i class="far fa-calendar-alt"></i>
        {{ post.date | date_to_long_string }}
        <!-- <i class="far fa-clock"></i>
        {{ post.content | reading_time_as_i }} -->
    </div>
    <div class="excerpt">
     {{ post.excerpt }}
    </div>
</div>
{%endfor%}

</section>

<section>
<h2 class="section">PUBLICATIONS</h2>
{% for pub in site.data.publications %}
{% include publication.html 
    src=pub.src
    title=pub.title
    conference=pub.conference
    authors=pub.authors
    id=forloop.index
    bibtex=pub.bibtex
%}
{% endfor %}
</section>