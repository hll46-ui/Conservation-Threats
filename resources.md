---
layout: default
title: Resources
---
{% include nav.html %}

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
      data-place="{{ r.place | escape }}">
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
  // Every place on the map. An item matches if its place text contains any keyword.
  var PLACE_OPTIONS = [
    { group: "The Galápagos", label: "All of the Galápagos", keys: ["galapagos"] },
    { group: "The Galápagos: islands", label: "Baltra Island", keys: ["baltra", "seymour"] },
    { group: "The Galápagos: islands", label: "Isabela Island", keys: ["isabela"] },
    { group: "The Galápagos: islands", label: "Pinzón Island", keys: ["pinzon"] },
    { group: "The Galápagos: islands", label: "San Cristóbal Island", keys: ["san cristobal", "punta pitt", "schiavoni", "puerto baquerizo"] },
    { group: "The Galápagos: islands", label: "Santa Cruz Island", keys: ["santa cruz", "puerto ayora"] },
    { group: "The Galápagos: airports", label: "Seymour Airport (Baltra)", keys: ["seymour"] },
    { group: "The Galápagos: airports", label: "San Cristóbal Airport", keys: ["san cristobal airport"] },
    { group: "The Galápagos: towns and sites", label: "Puerto Ayora", keys: ["puerto ayora"] },
    { group: "The Galápagos: towns and sites", label: "Punta Pitt", keys: ["punta pitt"] },
    { group: "The Galápagos: towns and sites", label: "Schiavoni Reef", keys: ["schiavoni"] },
    { group: "The Galápagos: protected areas", label: "Galápagos Marine Reserve", keys: ["marine reserve"] },
    { group: "The Galápagos: protected areas", label: "World Heritage Site", keys: ["world heritage"] },
    { group: "The Galápagos: protected areas", label: "In Danger listing", keys: ["in danger"] },
    { group: "Mainland Ecuador", label: "All of mainland Ecuador", keys: ["ecuador", "quito", "manta", "guayaquil"] },
    { group: "Mainland Ecuador", label: "Quito", keys: ["quito"] },
    { group: "Mainland Ecuador", label: "Manta", keys: ["manta"] },
    { group: "Mainland Ecuador", label: "Guayaquil", keys: ["guayaquil"] },
    { group: "Eastern Asia", label: "All of Eastern Asia", keys: ["eastern asia", "china", "japan", "taiwan", "guangzhou", "beijing", "tokyo"] },
    { group: "Eastern Asia", label: "Guangzhou, China", keys: ["guangzhou"] },
    { group: "Eastern Asia", label: "Beijing, China", keys: ["beijing"] },
    { group: "Eastern Asia", label: "Tokyo, Japan", keys: ["tokyo"] },
    { group: "Eastern Asia", label: "Taiwan", keys: ["taiwan"] },
    { group: "Western Europe", label: "All of Western Europe", keys: ["western europe", "france", "united kingdom", "switzerland", "paris", "london", "gland"] },
    { group: "Western Europe", label: "Paris, France", keys: ["paris"] },
    { group: "Western Europe", label: "London, United Kingdom", keys: ["london"] },
    { group: "Western Europe", label: "Gland, Switzerland", keys: ["gland"] }
  ];

  function norm(s) {
    return (s || "").toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "");
  }

  var catSelect = document.getElementById("category-filter");
  var placeSelect = document.getElementById("place-filter");
  var items = document.querySelectorAll(".resource");
  var count = document.getElementById("result-count");

  // Safety net: any place text that no option above covers goes under "Other places".
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

  // Build the grouped dropdown.
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
    var keys = placeSelect.value === "all" ? null : PLACE_OPTIONS[Number(placeSelect.value)].keys;
    var shown = 0;
    items.forEach(function (item) {
      var okCat = cat === "all" || item.getAttribute("data-category") === cat;
      var text = norm(item.getAttribute("data-place"));
      var okPlace = !keys || keys.some(function (k) { return text.indexOf(k) !== -1; });
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
