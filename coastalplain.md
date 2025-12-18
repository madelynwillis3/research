---
layout: page
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
var map = L.map('map').setView([32.430719, -83.733385], 15);
// ESRI World Imagery (Satellite)
L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
  attribution: 'Tiles © Esri'
}).addTo(map);

// Base folder for images
var imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';
// List of non-sampled points:
const exclude = ["39", "50", "51", "58", "60", "65", "74"];
 
Papa.parse('{{ "/assets/data/perry_FP_samples_80.csv" | relative_url }}', {
  download: true,
  header: true,
  complete: function(results) {
    results.data.forEach(function(row) {
      if(row.x && row.y && row["Point ID"]) {
        var lat = parseFloat(row.y);
        var lng = parseFloat(row.x);
        var label = row["Point ID"].trim();
        if (exclude.includes(label)) return; // skip excluded points
        if (!isNaN(lat) && !isNaN(lng)) {
          L.circleMarker([lat, lng], {
            radius: 6,
            color: "#007BFF",
            fillColor: "#3399FF",
            fillOpacity: 0.8
          })
          .addTo(map)
          .bindPopup(
            `<b>Sample ${label}</b><br>
             <img src="${imgBase}${label}.jpg" style="width:150px;max-height:150px;">`
          );
        }
      }
    });
  }
});
</script>
