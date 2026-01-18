---
layout: default
---
<!-- Get language slug from page url -->
{% assign current_language_slug = page.url | split: '/' | last %}
<!-- Detect pretty version of language slug -->
{% assign current_language_pretty = current_language %}
{% for post in site.categories["case-study"] %}
  {% for lang in post.language %}
    {% assign slugified_lang = lang | slugify: 'pretty' %}
    {% if slugified_lang == current_language_slug %}
      {% assign current_language_pretty = lang %}
      {% break %}
    {% endif %}
  {% endfor %}
{% endfor %}
<article class="post">

  <header class="post-header">
    <h1 class="post-title">{{ current_language_pretty | escape }} Case Studies</h1>
  </header>

  <div class="post-content">
      {% assign posts = site.categories["case-study"] | sort: "date" %}
      {% for post in posts %}
        {% for lang in post.language %}
          {% assign slugified_lang = lang | slugify: 'pretty' %}
          {% if slugified_lang == current_language_slug %}          
            <div class="row post-item" data-date="{{ post.date | date: "%F" }}">
              <h4 class="mt-0"><a href="{{ post.url }}">{{ post.title }}</a></h4>
              {% if post.date %}            
                <div class="text-start post-meta"><i class="bi-clock"></i> {{ post.date | date: "%-d %B %Y" }}</div>
              {% endif %}
              {% if post.author %}
                <div class="mt-1 text-start post-meta"><i class="bi-person"></i> {{ post.author }}</div>
              {% endif %}
              {% if post.language %}
                <div class="mt-1 text-start post-meta"><i class="bi-file-code"></i> {{ post.language }}</div>
              {% endif %}
              <div class="mt-1 text-start">
                {{ post.excerpt }}
              </div>
            </div>
            {% unless forloop.last %}<hr class="mb-2"/>{% endunless %}
          {% endif %}
        {% endfor %}
      {% endfor %}
  </div>
  <!-- todo: pagination? -->
</article>
