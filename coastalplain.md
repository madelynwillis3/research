---
layout: default
title: Interactive Soil Sampling Map
permalink: /coastalplain/
---

# Digital Soil Mapping Points

This interactive map shows field data points on UGA GrandFarm in Perry, GA. Hover over each marker to see the soil profile at that point.

<div id="map" style="height: 400px; margin-bottom: 2em;"></div>

This sampling campaign took place from May 2023 to May 2025 in Perry GA. This site is situated in the northern coastal plains, where the sandy foothills lie only  During this time, 76 soil profiles were extracted and analyzed for texture, color, horizon, depth, structure, and other identifying notes.

<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<!-- PapaParse for CSV parsing -->
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>

<script>
var map = L.map('map').setView([32.4379, -83.7362], 15);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  attribution: 'Map data © OpenStreetMap contributors'
}).addTo(map);

Papa.parse('{{ "/assets/data/soil_points_corrected.csv" | relative_url }}', {
  download: true,
  header: true,
  complete: function(results) {
    results.data.forEach(function(row) {
      if(row.x && row.y) {
        L.marker([parseFloat(row.y), parseFloat(row.x)]).addTo(map)
          .bindPopup(`<b>${row.PointLabel || row.Label || "Sample"}</b><br>
                      <b>pH:</b> ${row.pH.2 || ""}<br>
                      <b>Ca:</b> ${row.Ca || ""}<br>
                      <b>K:</b> ${row.K || ""}<br>
                      <b>P:</b> ${row.P || ""}`);
      }
    });
  }
});
</script>
