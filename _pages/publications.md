---
layout: archive
title: "Publications"
permalink: /publications/
author_profile: true
---

{% if site.author.googlescholar %}
  <div class="wordwrap">You can also find my articles on <a href="{{site.author.googlescholar}}">my Google Scholar profile</a>.</div>
{% endif %}

{% include base_path %}

<style>
  .pub-list { list-style: none; padding: 0; margin-top: 0.5em; }
  .pub-list li { margin-bottom: 0.5em; padding-bottom: 0.5em; border-bottom: 1px solid #eee; font-size: 0.95em; line-height: 1.5; }
  .pub-list li:last-child { border-bottom: none; }
  .pub-title a { text-decoration: none; }
</style>

{% assign all_pubs = site.publications | sort: "date" | reverse %}
{% capture current_year %}'None'{% endcapture %}

<ul class="pub-list">
{% for post in all_pubs %}
  {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
  {% if year != current_year %}
    </ul><h2 id="{{ year }}">{{ year }}</h2><ul class="pub-list">
    {% capture current_year %}{{ year }}{% endcapture %}
  {% endif %}
  <li>
    {% if post.paperurl %}<a href="{{ post.paperurl }}">{{ post.title }}</a>{% else %}{{ post.title }}{% endif %}
  </li>
{% endfor %}
</ul>
