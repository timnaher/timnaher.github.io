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
  .pub-list li { margin-bottom: 0.7em; padding-bottom: 0.7em; border-bottom: 1px solid #eee; font-size: 0.95em; line-height: 1.5; }
  .pub-list li:last-child { border-bottom: none; }
  .pub-title { font-weight: 600; }
  .pub-title a { text-decoration: none; }
  .pub-authors { color: #555; }
  .pub-venue { font-style: italic; color: #666; }
  .pub-year { color: #888; }
</style>

<h2>Journal Articles</h2>
<ul class="pub-list">
{% assign manuscripts = site.publications | where: "category", "manuscripts" | sort: "date" | reverse %}
{% for post in manuscripts %}
  <li>
    <span class="pub-title">{% if post.paperurl %}<a href="{{ post.paperurl }}">{{ post.title }}</a>{% else %}{{ post.title }}{% endif %}</span><br/>
    <span class="pub-venue">{{ post.venue }}</span>, <span class="pub-year">{{ post.date | date: "%Y" }}</span>
  </li>
{% endfor %}
</ul>

<h2>Preprints</h2>
<ul class="pub-list">
{% assign preprints = site.publications | where: "category", "preprints" | sort: "date" | reverse %}
{% for post in preprints %}
  <li>
    <span class="pub-title">{% if post.paperurl %}<a href="{{ post.paperurl }}">{{ post.title }}</a>{% else %}{{ post.title }}{% endif %}</span><br/>
    <span class="pub-venue">{{ post.venue }}</span>, <span class="pub-year">{{ post.date | date: "%Y" }}</span>
  </li>
{% endfor %}
</ul>

<h2>Conference Presentations</h2>
<ul class="pub-list">
{% assign conferences = site.publications | where: "category", "conferences" | sort: "date" | reverse %}
{% for post in conferences %}
  <li>
    <span class="pub-title">{{ post.title }}</span><br/>
    <span class="pub-venue">{{ post.venue }}</span>, <span class="pub-year">{{ post.date | date: "%Y" }}</span>
  </li>
{% endfor %}
</ul>
