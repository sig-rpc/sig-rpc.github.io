---
layout: default-wide
---

<div class="container">
  <!-- Hero, introduction + call to action -->
  <div class="row align-items-center">
      <div class="text-center text-lg-start">
        <h1 class="display-4 fw-bold lh-1 mb-3">Reasonable Performance Computing SIG</h1>
      </div>
  </div>
  <div class="row align-items-center g-lg-5 py-5">
    <div class="col-lg-8 text-center text-lg-start">
      <p class="fs-4">A community dedicated to the research, development and advocacy of performance best practices for those that work closely with software. We hope to ensure that all code achieves a minimum standard of reasonable performance, whereby all “easy performance wins” have been exhausted.</p>
    </div>
    <div class="col-md-10 mx-auto col-lg-4 border rounded-3 bg-light p-2">
      <p>Join our mailing list to keep informed of upcoming news, events and ways to get involved!</p>
      <a class="w-100 btn btn-lg btn-primary" href="https://groups.google.com/a/society-rse.org/g/sig-rpc" _target="blank">Sign up</a>
      <small class="text-muted p-t-1">Difficulty joining?<br/> Contact <a href="mailto:sig-rpc-managers@society-rse.org">sig-rpc-managers@society-rse.org</a></small>
    </div>
  </div>
  <div class="row">
    <div class="col-md-12 text-center">
      <p markdown="span">Learn more about [profiling](/profiling/), [optimisation](/optimisations/), [parallel computing](/parallel/) and [more](/resources/), or [contribute your own](https://github.com/sig-rpc/sig-rpc.github.io/issues/new/choose) tips and tricks!</p>
    </div>
  </div>
  <div class="row">
    <div class="col-md-8 text-center">
      <h2>Recent Blog Posts</h2>
      {% assign post_exists = false %}
      {% assign posts = site.posts | sort: "date" %}
      {% for post in site.posts limit:2 %}
          <div class="row post-item" data-date="{{ post.date | date: "%F" }}{% if event.to %}{{event.to | date: " %H:%M:%S"}}{% endif %}">
            <h4 class="mt-0"><a href="{{ post.url }}">{{ post.title }}</a></h4>
            {% if post.date %}            
              <div class="text-start post-meta"><i class="bi-clock"></i> {{ post.date | date: "%-d %B %Y" }}</div>
            {% endif %}
            {% if post.author %}
              <div class="mt-1 text-start post-meta"><i class="bi-person"></i> {{ post.author }}</div>
            {% endif %}
            <div class="mt-1 text-start">
              {{ post.excerpt }}
            </div>
          </div>
          {% unless forloop.last %}<hr class="mb-2"/>{% endunless %}
          {% assign post_exists = true %}
      {% endfor %}
      {% unless post_exists %}
          The blog is currently empty!
      {% endunless %}
    </div>
    <div class="col-md-4 text-center">
      <h2>Upcoming Events</h2>
      {% assign current_date = site.time | date: '%F' | date: '%s' %}
      {% assign events = site.events | sort: "date" %}
      {% assign event_exists = false %}
      <div class="event-listing event-upcoming">
      {% for event in events limit:2 %}
        {% assign event_date = event.date | date: '%s' %}
        {% if event_date >= current_date %}
          <div class="row event-item" data-date="{{ event.date | date: "%F" }}{% if event.to %}{{event.to | date: " %H:%M:%S"}}{% endif %}">
            <h4 class="mt-0"><a href="{{ event.url }}">{{ event.title }}</a></h4>
            <div class="text-start"><strong><i class="bi-clock"></i> {{ event.date | date: "%-d %B %Y" }}  {% if event.end-date %} to {{ event.end-date | date: "%-d %B %Y" }}{% endif %}{% if event.from %} - {{ event.from}}{% if event.to %}-{{event.to}}{% endif %}{% endif %}</strong></div>
            {% if event.location %}
              <div class="mt-1 text-start"><strong><i class="bi-pin-map"></i> {{ event.location }}</strong></div>
            {% endif %}
            {% if event.summary %}
              <div class="mt-1 text-start">
                <p>{{event.summary}}</p>
              </div>
            {% endif %}
          </div>
          {% unless forloop.last %}<hr class="mb-2"/>{% endunless %}
          {% assign event_exists = true %}
        {% endif %}
      {% endfor %}
      {% unless event_exists %}
          No events are currently scheduled, please join our <a href="https://groups.google.com/a/society-rse.org/g/sig-rpc">mailing list</a> to hear about new events.
      {% endunless %}
      </div>
    </div>
  </div>
</div>
