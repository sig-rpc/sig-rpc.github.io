---
layout: default
---
<!-- Get language slug from page url -->
{% assign current_language_slug = page.url | split: '/' | last %}
<!-- Detect pretty version of language slug -->
{% assign current_language_pretty = current_language %}
{% for optimisation in site.optimisations %}
  {% for lang in optimisation.language %}
    {% assign slugified_lang = lang | slugify: 'pretty' %}
    {% if slugified_lang == current_language_slug %}
      {% assign current_language_pretty = lang %}
      {% break %}
    {% endif %}
  {% endfor %}
{% endfor %}
<article class="post">

  <header class="post-header">
    <h1 class="post-title">{{ current_language_pretty | escape }} Optimisation Patterns</h1>
  </header>

  <div class="post-content">
    <!-- Build arrays of all possible subcategories in this language --> 
    {% assign all_lang_subcategories = "" %}
    {% for optimisation in site.optimisations %}
      {% for lang in optimisation.language %}
        {% assign slugified_lang = lang | slugify: 'pretty' %}
        {% if slugified_lang == current_language_slug %}
          {% for subcat in optimisation.subcategory %}
            {% assign all_lang_subcategories = all_lang_subcategories | append: subcat | append: "," %}
          {% endfor %}
          {% break %}
        {% endif %}
      {% endfor %}
    {% endfor %}
    <!-- This could be improved by slugifying them to fix ~case differences -->
    {% assign all_lang_subcategories = all_lang_subcategories | split: "," | uniq | sort %}
    <!-- Build the style filter list -->
    {% if all_lang_subcategories != empty %}
      <div id="filters">    
        Subcategory:
        <a href="?filter=none">None</a>
        {%- for subcat in all_lang_subcategories -%}
            &nbsp;·&nbsp;<a href="?filter={{ subcat }}">{{ subcat | capitalize  }}</a>
        {%- endfor %}
      </div>
    {% endif %}
    <div id="optimisations">
      {% assign optimisations = site.optimisations | sort: 'name' | sort: 'level'  %}
      {% for optimisation in optimisations %}
        {% for lang in optimisation.language %}
          {% assign slugified_lang = lang | slugify: 'pretty' %}
          {% if slugified_lang == current_language_slug %}
            {% assign joined_subcategories = optimisation.subcategory | join: " " %}          
            <div class="optimisation" data-attributes="{{ joined_subcategories }}">
              <h2>{{ optimisation.name }}</h2>
              {% if optimisation.subcategory != empty %}
                <small>Subcategory: 
                  {% for subcat in optimisation.subcategory %}
                    <code>{{ subcat }}</code>{% unless forloop.last %},{% endunless %}
                  {% endfor %}
                </small>
              {% endif %}
              {{ optimisation.excerpt }}
              <a href="{{ optimisation.url }}">Read More</a>
              {% unless forloop.last %}<hr/>{% endunless %}
            </div>
          {% endif %}
        {% endfor %}
      {% endfor %}
    </div>
  </div>
  <!-- todo: pagination? -->
</article>

<script>
  // Wait for the DOM to fully load
  document.addEventListener("DOMContentLoaded", () => {
    const optimisations = document.querySelectorAll("#optimisations .optimisation");
    const filters = document.querySelectorAll("#filters a");

    // Function to filter optimisations based on query string
    const filterOptimisations = () => {
      // Get the current filter value from the URL
      const urlParams = new URLSearchParams(window.location.search);
      const filter = urlParams.get("filter");

      // Clear any existing <hr /> elements
      document.querySelectorAll("#optimisations hr").forEach((hr) => hr.remove());

      if (!filter || filter === "none") {        
        // If "filter=none" or no filter is specified, show all optimisations
        optimisations.forEach((optimisation) => (optimisation.style.display = ""));
      } else {
        // Otherwise, show optimisations that match the filter
        optimisations.forEach((optimisation) => {
          const attributes = optimisation.dataset.attributes.split(" ");
          const isVisible = attributes.includes(filter);
          optimisation.style.display = isVisible ? "" : "none";
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

      // Add <hr /> elements between visible optimisations
      const visibleOptimisations = Array.from(optimisations).filter(
        (optimisation) => optimisation.style.display !== "none"
      );

      visibleOptimisations.forEach((optimisation, index) => {
        if (index < visibleOptimisations.length - 1) {
          const hr = document.createElement("hr");
          optimisation.insertAdjacentElement("afterend", hr);
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
        filterOptimisations(); // Apply the filter
      });
    });

    // Initial filtering when the page loads
    filterOptimisations();

    // Reapply filtering when navigating through browser history
    window.addEventListener("popstate", filterOptimisations);
  });
</script>

