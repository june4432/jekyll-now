---
layout: page
title: Tech
permalink: /tech/
---

<div class="posts">
  {% for post in site.posts %}
    <article class="post">

      <h1><a href="{{ site.baseurl }}{{ post.url }}/tech">{{ post.title }}</a></h1>

      <div class="entry">
        {{ post.excerpt }}
      </div>

      <a href="{{ site.baseurl }}{{ post.url }}/tech" class="read-more">Read More</a>
    </article>
  {% endfor %}
</div>
