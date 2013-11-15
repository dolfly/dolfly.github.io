---
layout: index 
title: 灵步稚鹰
tagline: Supporting tagline
---
{% include JB/setup %}

{% for post in site.posts limit:10 %}
<div class="post-preview">
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    <div class="date">
        <a href="{{ post.url }}">{{ post.date | date:"%Y-%m-%d" }}</a>
    </div>
<p class="excerpt">{{ post.excerpt | strip_html | strip_newlines }}</p>
</div>
<hr />
{% endfor %}
