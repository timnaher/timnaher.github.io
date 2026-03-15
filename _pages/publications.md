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
  .pub-venue { font-style: italic; color: #666; }
  .pub-year { color: #888; }
  .pub-tag {
    display: inline-block;
    padding: 0.1em 0.45em;
    border-radius: 4px;
    font-size: 0.72em;
    font-weight: 500;
    margin-left: 0.3em;
    vertical-align: middle;
  }
  .pub-tag-journal { background: #e3f2fd; color: #1565c0; }
  .pub-tag-preprint { background: #fff3e0; color: #e65100; }
  .pub-tag-talk { background: #e8f5e9; color: #2e7d32; }
  .pub-tag-poster { background: #f3e5f5; color: #7b1fa2; }
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
    <span class="pub-title">{% if post.paperurl %}<a href="{{ post.paperurl }}">{{ post.title }}</a>{% else %}{{ post.title }}{% endif %}</span>
    {% if post.pub_tags %}{% for tag in post.pub_tags %}<span class="pub-tag pub-tag-{{ tag }}">{{ tag }}</span>{% endfor %}{% endif %}<br/>
    <span class="pub-venue">{{ post.venue }}</span>
  </li>
{% endfor %}
</ul>
