---
layout: default
title: Interactive Soil Sampling Map
permalink: /coastalplain/
---

# Mapping Soil Profiles on UGAGrandFarm

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
var map = L.map('map').setView([32.4307, -83.7338], 15);
L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
  attribution: 'Tiles © Esri'
}).addTo(map);

Papa.parse('{{ "/assets/data/perry_FP_samples_80.csv" | relative_url }}', {
  download: true,
  header: true,
  complete: function(results) {
    results.data.forEach(function(row) {
      // Make sure x, y, and Point ID exist and are numbers
      if(row.x && row.y && row["Point ID"]) {
        var lat = parseFloat(row.y);
        var lng = parseFloat(row.x);
        var label = row["Point ID"];
        // Only add marker if coordinates are valid
        if (!isNaN(lat) && !isNaN(lng)) {
          L.marker([lat, lng])
            .addTo(map)
            .bindPopup(`<b>Sample ${label}</b>`);
        }
      }
    });
  }
});
</script>
