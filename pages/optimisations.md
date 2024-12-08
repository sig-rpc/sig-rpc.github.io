---
title: Optimisation Database
layout: page
permalink: /optimisations/
---

We have a database of simple optimisation patterns to look out for when profiling your code. These have been subdivided by language, and where appropriate by subcategory (such as library). Some are based on computer science theory and are common to most languages, whereas others are highly language specific.

<!--TBC Optimisation issue link, issue template not currently setup.-->
Many of these simple optimisation patterns are incidental finds, uncovered whilst profiling code. If you discover anything missing from our database, please help by [submitting it for inclusion via GitHub](https://github.com/sig-rpc/sig-rpc.github.io/issues/new?assignees=&labels=Optimisation&projects=&template=new_optimisation.yml&title=%5BNew%5D%3A+).

<h2>Languages</h2>
<!-- Create an array of unique languages, so that they can be listed for the user. -->
{% assign all_languages = "" %}
{% for optimisation in site.optimisations %}
  {% for language in optimisation.language %}
    {% assign all_languages = all_languages | append: language | append: "," %}
  {% endfor %}
{% endfor %}
{% assign unique_languages = all_languages | split: "," | uniq | sort %}
<ul class="language-list">
{% for lang in unique_languages %}
  <li><a href="/optimisations/{{ lang | slugify: 'pretty' }}/">{{ lang }}</a></li>
{% endfor %}
</ul>
