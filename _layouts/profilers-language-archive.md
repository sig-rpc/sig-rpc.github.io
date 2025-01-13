---
layout: default
---
<!-- Get language slug from page url -->
{% assign current_language_slug = page.url | split: '/' | last %}
<!-- Detect pretty version of language slug -->
{% assign current_language_pretty = current_language %}
{% for profiler in site.profilers %}
  {% for lang in profiler.language %}
    {% assign slugified_lang = lang | slugify: 'pretty' %}
    {% if slugified_lang == current_language_slug %}
      {% assign current_language_pretty = lang %}
      {% break %}
    {% endif %}
  {% endfor %}
{% endfor %}
<article class="post">

  <header class="post-header">
    <h1 class="post-title">Profilers for {{ current_language_pretty | escape }}</h1>
  </header>

  <div class="post-content">
    <!-- Build arrays of all possible styles, and all styles in this language -->
    {% assign all_styles = "" %}    
    {% assign all_lang_styles = "" %}
    {% for profiler in site.profilers %}
      <!-- All styles -->
      {% for style in profiler.style %}
        {% assign all_styles = all_styles | append: style | append: "," %}
      {% endfor %}
      <!-- All styles in this lang -->
      {% for lang in profiler.language %}
        {% assign slugified_lang = lang | slugify: 'pretty' %}
        {% if slugified_lang == current_language_slug %}
          {% for style in profiler.style %}
            {% assign all_lang_styles = all_lang_styles | append: style | append: "," %}
          {% endfor %}
          {% break %}
        {% endif %}
      {% endfor %}
    {% endfor %}
    <!-- This could be improved by slugifying them to fix ~case differences -->
    {% assign all_styles = all_styles | split: "," | uniq %} <!-- No sort, prefer manual order available in profilers -->
    {% assign all_lang_styles = all_lang_styles | split: "," | uniq %}
    <!-- Build the style filter list -->  
    <div id="filters">    
      Styles:
      <a href="?filter=all">All</a>
      {%- for style in all_styles -%}
        {%- if all_lang_styles contains style -%}
          &nbsp;·&nbsp;<a href="?filter={{ style }}">{{ style | capitalize }}</a>
        {%- else -%}
          &nbsp;·&nbsp;<a class="disabled">{{ style | capitalize  }}</a>
        {%- endif -%}
      {%- endfor %}
    </div>
    <div id="profilers">
      {% assign profilers = site.profilers | sort: 'name' | sort: 'level'  %}
      {% for profiler in profilers %}
        {% for lang in profiler.language %}
          {% assign slugified_lang = lang | slugify: 'pretty' %}
          {% if slugified_lang == current_language_slug %}
            {% assign joined_styles = profiler.style | join: " " %}          
            <article class="post profiler" data-attributes="{{ joined_styles }}">
              <header class="post-header mb-2">
                <h2 class="mb-2">{{ profiler.name }}</h2>
                <p class="post-styles post-meta">
                <strong>Styles: </strong>
                  {% for style in profiler.style %}
                    {{ style }}{% unless forloop.last %}, {% endunless %}
                  {% endfor %}
                </p>
              </header>
              {{ profiler.excerpt }}
              <a href="{{ profiler.url }}">Read More</a>
              {% unless forloop.last %}<hr/>{% endunless %}
            </article>
          {% endif %}
        {% endfor %}
      {% endfor %}
    </div>
  </div>
</article>

<script>
  // Wait for the DOM to fully load
  document.addEventListener("DOMContentLoaded", () => {
    const profilers = document.querySelectorAll("#profilers .profiler");
    const filters = document.querySelectorAll("#filters a");

    // Function to filter profilers based on query string
    const filterProfilers = () => {
      // Get the current filter value from the URL
      const urlParams = new URLSearchParams(window.location.search);
      const filter = urlParams.get("filter");

      // Clear any existing <hr /> elements
      document.querySelectorAll("#profilers hr").forEach((hr) => hr.remove());

      if (!filter || filter === "all") {        
        // If "filter=all" or no filter is specified, show all profilers
        profilers.forEach((profiler) => (profiler.style.display = ""));
      } else {
        // Otherwise, show profilers that match the filter
        profilers.forEach((profiler) => {
          const attributes = profiler.dataset.attributes.split(" ");
          const isVisible = attributes.includes(filter);
          profiler.style.display = isVisible ? "" : "none";
        });
      }

      // Highlight the selected filter
      filters.forEach((filterLink) => {
        const linkFilter = filterLink.getAttribute("href")?.split("=")[1] ?? null;
        if ((filter && filter === linkFilter) || (!filter && linkFilter === "all")) {
          filterLink.classList.add("selected");
        } else {
          filterLink.classList.remove("selected");
        }
      });

      // Add <hr /> elements between visible profilers
      const visibleProfilers = Array.from(profilers).filter(
        (profiler) => profiler.style.display !== "none"
      );

      visibleProfilers.forEach((profiler, index) => {
        if (index < visibleProfilers.length - 1) {
          const hr = document.createElement("hr");
          profiler.insertAdjacentElement("afterend", hr);
        }
      });
    };

    // Update the URL when a filter is clicked
    filters.forEach((filterLink) => {
      filterLink.addEventListener("click", (event) => {
        event.preventDefault(); // Prevent default link behavior
        const newFilter = filterLink.getAttribute("href").split("=")[1];
        const newUrl = `${window.location.pathname}?filter=${newFilter}`;
        window.history.pushState({ path: newUrl }, "", newUrl); // Update URL
        filterProfilers(); // Apply the filter
      });
    });

    // Initial filtering when the page loads
    filterProfilers();

    // Reapply filtering when navigating through browser history
    window.addEventListener("popstate", filterProfilers);
  });
</script>

