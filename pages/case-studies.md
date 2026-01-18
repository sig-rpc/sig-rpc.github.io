---
title: Case Studies
layout: page
permalink: /case-studies/
excerpt_separator: <!--more-->
---

We are steadily building a collection of research software profiling case studies. If you're looking for real world examples of optimisations you might find in research code, you should read these.

If you have had success profiling and optimising your own research software, we would welcome a case study written from your perspective. The more examples we can share, the more clearly we can demonstrate the impact that profiling can have on research code.

If you are interested in contributing, you can [get in touch](mailto:sig-rpc-managers@society-rse.org) to discuss your experience with us, or submit an [issue](https://github.com/sig-rpc/sig-rpc.github.io/issues/new?template=BLANK_ISSUE) or a [pull request](https://github.com/sig-rpc/sig-rpc.github.io/pulls) directly.

<!--TBC Optimisation issue link, issue template not currently setup.-->
Many of these simple optimisation patterns are incidental finds, uncovered whilst profiling code. If you discover anything missing from our database, please help by [submitting it for inclusion via GitHub](https://github.com/sig-rpc/sig-rpc.github.io/issues/new?assignees=&labels=Optimisation&projects=&template=new_optimisation.yml&title=%5BNew%5D%3A+).

## Case Studies by Language
<!-- Create an array of unique languages, so that they can be listed for the user. -->
{% assign all_languages = "" %}
{% for post in site.categories["case-study"] %}
  {% for lang in post.language %}
    {% assign all_languages = all_languages | append: lang | append: "," %}
  {% endfor %}
{% endfor %}
{% assign unique_languages = all_languages | split: "," | uniq | sort %}
<ul class="language-list">
{% for lang in unique_languages %}
  <li><a href="/case-studies/{{ lang | slugify: 'pretty' }}/">{{ lang }}</a></li>
{% endfor %}
</ul>

## Recent Case Studies

{% assign posts = site.categories["case-study"] | sort: "date" %}
{% for post in posts limit:2 %}
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
{% endfor %}


