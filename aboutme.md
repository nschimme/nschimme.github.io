---
layout: page
title: About me
cover-img: /assets/img/cover.gif
share-description: A page about Nils Schimmelmann.
---
I am a father of two and an engineer in Houston, Texas. I write about computer hardware, programming, gaming, and outdoor travels. I am a hacker, shredder, gamer, cycler, foodie, hiker, aquarist, and smart home afficionado.

### Archive

{% assign years = site.posts
   | group_by_exp: "post", "post.date | date: '%Y'"
%}

{% for year in years %}
  <p>{{ year.name }}</p>

  <ul>
    {% for post in year.items %}
      <li>
        <a class='title' href='{{ post.url }}'>{{ post.title }}</a>
      </li>
    {% endfor %}
  </ul>
{% endfor %} 
