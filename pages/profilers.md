---
title: Profiler Database
layout: page
permalink: /profilers/
slug: profilers
---

We have a database of quick-start guides that introduce and explain how to get started with a wide range of profilers, categorised by their targeted programming language and profiling style.

<!--TBC Profiler issue link, issue template not currently setup.-->
If your favourite profiler is missing, you can [submit a new quickstart guide on GitHub](https://github.com/sig-rpc/sig-rpc.github.io/issues/new?assignees=&labels=Profiler&projects=&template=new_profiler.yml&title=%5BNew%5D%3A+).

<h2>Languages</h2>
<!-- Create an array of unique languages, so that they can be listed for the user. -->
{% assign all_languages = "" %}
{% for profiler in site.profilers %}
  {% for language in profiler.language %}
    {% assign all_languages = all_languages | append: language | append: "," %}
  {% endfor %}
{% endfor %}
{% assign unique_languages = all_languages | split: "," | uniq | sort %}
<ul class="language-list">
{% for lang in unique_languages %}
  <li><a href="/profilers/{{ lang | slugify: 'pretty' }}/">{{ lang }}</a></li>
{% endfor %}
</ul>
