---
layout: default
title: "Coastal Plain Interactive Map"
permalink: /coastalplain/
---


# Mapping Soil Profiles on UGAGrandFarm

This interactive map shows field data points on UGA GrandFarm in Perry, GA. Click on each marker to see the soil profile at that point to reveal the horizons and their depths (in cm) as well as color. 

<div id="map" style="height: 400px; margin-bottom: 2em;"></div>

This site is situated in the northern coastal plains, where 75 soil profiles were extracted and analyzed for texture, color, horizon, depth, structure, and other identifying notes.

<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<!-- PapaParse for CSV parsing -->
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>

<script>
var map = L.map('map').setView([32.43, -83.73], 14);

L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
  attribution: 'Tiles © Esri'
}).addTo(map);

var imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';
const exclude = ["39", "50", "51", "58", "60", "65", "74"];

let sample45LatLng = null;

Papa.parse('{{ "/assets/data/perry_FP_samples_80.csv" | relative_url }}', {
  download: true,
  header: true,
  complete: function(results) {

    // 1) Build marker objects, but DON’T add to map yet
    const markers = [];

    results.data.forEach(function(row) {
      if (!row.x || !row.y || !row["Point ID"]) return;

      const lat = parseFloat(row.y);
      const lng = parseFloat(row.x);
      const label = row["Point ID"].trim();

      if (exclude.includes(label)) return;
      if (isNaN(lat) || isNaN(lng)) return;

      if (label === "45") sample45LatLng = [lat, lng];

      const m = L.circleMarker([lat, lng], {
        radius: 6,
        color: "#007BFF",
        fillColor: "#3399FF",
        fillOpacity: 0.0   // start invisible for a nicer “fade in”
      }).bindPopup(
        `<b>Sample ${label}</b><br>
         <img src="${imgBase}${label}.jpg" style="width:150px;max-height:150px;">`
      );

      markers.push(m);
    });

    // 2) Fly to the center target first (adjust zoom to taste)
    const target = sample45LatLng || [32.43, -83.73];
    map.flyTo(target, 15, { animate: true, duration: 1.6 });

    // 3) After the fly finishes, “rain in” markers
    //    Staggered add + fade-in by increasing fillOpacity
    setTimeout(() => {
      const delayMs = 20; // smaller = faster animation, larger = slower

      markers.forEach((m, i) => {
        setTimeout(() => {
          m.addTo(map);

          // quick fade-in (3 steps)
          m.setStyle({ fillOpacity: 0.3 });
          setTimeout(() => m.setStyle({ fillOpacity: 0.6 }), 60);
          setTimeout(() => m.setStyle({ fillOpacity: 0.8 }), 120);
        }, i * delayMs);
      });
    }, 1700); // start after flyTo finishes (match duration above)
  }
});
</script>

