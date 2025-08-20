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
  // Initialize map with a suitable center and zoom for Perry, GA
  var map = L.map('map').setView([32.44, -83.73], 14);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: 'Map data © OpenStreetMap contributors'
  }).addTo(map);

  // Load CSV data
  Papa.parse('{{ "/data/soil_points_corrected.csv" | relative_url }}', {
    download: true,
    header: true,
    complete: function(results) {
      results.data.forEach(function(row) {
        // Only add marker if lat/lng are present
        if(row.y && row.x) {
          // Customize popup: Show PointLabel and (optional) an image if available
          var popupContent = `<b>Point ${row.PointLabel || row.Label}</b><br>
                              <b>pH:</b> ${row.pH.2 || ""}<br>
                              <b>Ca:</b> ${row.Ca || ""}<br>
                              <b>K:</b> ${row.K || ""}<br>
                              <b>Mg:</b> ${row.Mg || ""}<br>
                              <b>P:</b> ${row.P || ""}`;
          // If you have an image field, add here:
          // popupContent += `<br><img src="${row.img_url}" style="width:150px;">`;

          L.marker([parseFloat(row.y), parseFloat(row.x)])
            .addTo(map)
            .bindPopup(popupContent);
        }
      });
    }
  });
</script>
