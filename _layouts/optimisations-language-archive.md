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
    <div id="search">
      <input type="text" id="search-box" placeholder="Search... (beta)" />
    </div>
    <div id="optimisations">
      {% assign optimisations = site.optimisations | sort: 'name' | sort: 'level'  %}
      {% for optimisation in optimisations %}
        {% for lang in optimisation.language %}
          {% assign slugified_lang = lang | slugify: 'pretty' %}
          {% if slugified_lang == current_language_slug %}
            {% assign joined_subcategories = optimisation.subcategory | join: " " %}          
            <article class="post optimisation" data-attributes="{{ joined_subcategories }}">
              <header class="post-header mb-2">
                <h2 class="mb-2">{{ optimisation.name }}</h2>
                {% if optimisation.subcategory != empty %}                
                  <p class="post-subcategories post-meta">
                    <strong>Subcategory: </strong> 
                    {% for subcat in optimisation.subcategory %}
                      {{ subcat }}{% unless forloop.last %}, {% endunless %}
                    {% endfor %}
                  </p>
                {% endif %}
              </header>
              {{ optimisation.excerpt }}
              <a href="{{ optimisation.url }}">Read More</a>
              {% unless forloop.last %}<hr/>{% endunless %}
              <div class="full-text" style="display:none">
                {{ optimisation.content }}
              </div>
            </article>
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
    const searchBox = document.getElementById("search-box");

    // Function to filter optimisations based on query string
    const filterOptimisations = () => {
      // Get the current filter value from the URL
      const urlParams = new URLSearchParams(window.location.search);
      const filter = urlParams.get("filter");
      const searchText = searchBox.value.toLowerCase();

      // Clear any existing <hr /> elements
      document.querySelectorAll("#optimisations hr").forEach((hr) => hr.remove());

      optimisations.forEach((optimisation) => {
        const attributes = optimisation.dataset.attributes.split(" ");
        const body = optimisation.querySelector(".full-text").textContent.toLowerCase();

        const matchesFilter =
          !filter || filter === "none" || attributes.includes(filter);
        const matchesSearch = !searchText || body.includes(searchText);

        // Display profiler only if it matches both the filter and search text
        optimisation.style.display = matchesFilter && matchesSearch ? "" : "none";
      });

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

     // Apply filtering when the search box value changes
     searchBox.addEventListener("input", filterOptimisations);

    // Initial filtering when the page loads
    filterOptimisations();

    // Reapply filtering when navigating through browser history
    window.addEventListener("popstate", filterOptimisations);
  });
</script>

