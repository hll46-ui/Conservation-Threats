---
layout: default
title: Resources
---
{% include nav.html %}
<style>
  .resource {
    --box-color: #5b8c5a;               
    margin-bottom: 2em;
    padding: 1em 1.25em;
    color: #1f2a2e;
    border: 2px solid var(--box-color);
    border-radius: 12px;
    background: color-mix(in srgb, var(--box-color) 14%, white);
  }
  .resource[hidden] { display: none; }
  .resource p { margin: 0.5em 0; }
  .resource a { color: #14506b; font-weight: 600; }

  .resource[data-category="article"]         { --box-color: #2f7f9e; } 
  .resource[data-category="journal-article"] { --box-color: #6b625b; } 
  .resource[data-category="report"]          { --box-color: #7b9e2f; } 
  .resource[data-category="legal-document"]  { --box-color: #1f7825; } 
</style>

Sources, readings, and tools that helped with this project.

{% assign cats = site.data.resources | map: "category" | uniq | sort %}

<div role="group" aria-label="Filter resources">
  <p>
    <label for="category-filter">Category:</label>
    <select id="category-filter">
      <option value="all">All categories</option>
      {% for c in cats %}
      <option value="{{ c | slugify }}">{{ c }}</option>
      {% endfor %}
    </select>
  </p>

  <p>
    <label for="place-filter">Place:</label>
    <select id="place-filter">
      <option value="all">All places</option>
    </select>
  </p>
</div>

<p id="result-count" aria-live="polite"></p>

<ul style="list-style:none; padding-left:0;">
  {% for r in site.data.resources %}
  <li class="resource"
      data-category="{{ r.category | slugify }}"
      data-place="{{ r.place | escape }}"
      data-text="{{ r.title | append: ' ' | append: r.note | escape }}">
    {% if r.url and r.url != "" %}
    <a href="{{ r.url }}">{{ r.title }}</a>
    {% else %}
    {{ r.title }}
    {% endif %}
    <p>Category: {{ r.category }}. Place: {{ r.place }}.</p>
    <p>{{ r.note }}</p>
    <small>Added by {{ r.added_by }}</small>
  </li>
  {% endfor %}
</ul>

<script>
document.addEventListener("DOMContentLoaded", function () {
  var PLACE_OPTIONS = [
    { group: "The Galápagos", label: "All of the Galápagos", keys: ["galapagos"] },

    { group: "Islands", label: "Baltra Island", keys: ["baltra", "seymour"] },
    { group: "Islands", label: "Isabela Island", keys: ["isabela"] },
    { group: "Islands", label: "Pinzón Island", keys: ["pinzon"] },
    { group: "Islands", label: "San Cristóbal Island", keys: ["san cristobal", "punta pitt", "schiavoni", "puerto baquerizo"] },
    { group: "Islands", label: "Santa Cruz Island", keys: ["santa cruz", "puerto ayora"] },

    { group: "Airports and protected areas", label: "The airports", keys: ["airport"], textKeys: ["biosecurity", "quarantine"] },
    { group: "Airports and protected areas", label: "Protected areas", keys: ["marine reserve", "world heritage", "in danger", "national park"] },

    { group: "Regions", label: "Mainland Ecuador", keys: ["mainland ecuador", "ecuador", "quito", "manta", "guayaquil"] },
    { group: "Regions", label: "Eastern Asia", keys: ["eastern asia", "china", "japan", "taiwan", "guangzhou", "beijing", "tokyo"] },
    { group: "Regions", label: "Western Europe", keys: ["western europe", "france", "united kingdom", "switzerland", "paris", "london", "gland"] }
  ];

  function norm(s) {
    return (s || "").toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "");
  }

  var catSelect = document.getElementById("category-filter");
  var placeSelect = document.getElementById("place-filter");
  var items = document.querySelectorAll(".resource");
  var count = document.getElementById("result-count");

  // Safety net: any place no option covers goes under "Other places".
  var extra = {};
  items.forEach(function (item) {
    var raw = item.getAttribute("data-place");
    raw.split(" - ").join(", ").split(", ").forEach(function (piece) {
      var p = norm(piece).trim();
      if (!p) { return; }
      var covered = PLACE_OPTIONS.some(function (o) {
        return o.keys.some(function (k) { return p.indexOf(k) !== -1; });
      });
      if (!covered) { extra[p] = piece.trim(); }
    });
  });
  Object.keys(extra).sort().forEach(function (p) {
    PLACE_OPTIONS.push({ group: "Other places", label: extra[p], keys: [p] });
  });

  // Build the grouped place dropdown.
  var groups = {};
  PLACE_OPTIONS.forEach(function (o, i) {
    if (!groups[o.group]) {
      var og = document.createElement("optgroup");
      og.label = o.group;
      placeSelect.appendChild(og);
      groups[o.group] = og;
    }
    var opt = document.createElement("option");
    opt.value = String(i);
    opt.textContent = o.label;
    groups[o.group].appendChild(opt);
  });

  function apply() {
    var cat = catSelect.value;
    var opt = placeSelect.value === "all" ? null : PLACE_OPTIONS[Number(placeSelect.value)];
    var shown = 0;
    items.forEach(function (item) {
      var okCat = cat === "all" || item.getAttribute("data-category") === cat;
      var place = norm(item.getAttribute("data-place"));
      var text = norm(item.getAttribute("data-text"));
      var okPlace = !opt ||
        opt.keys.some(function (k) { return place.indexOf(k) !== -1; }) ||
        (opt.textKeys && opt.textKeys.some(function (k) { return text.indexOf(k) !== -1; }));
      item.hidden = !(okCat && okPlace);
      if (!item.hidden) { shown++; }
    });
    count.textContent = shown === 0
      ? "No resources yet for this combination."
      : "Showing " + shown + " of " + items.length + " resources.";
  }

  catSelect.addEventListener("change", apply);
  placeSelect.addEventListener("change", apply);
  apply();
});
</script>
