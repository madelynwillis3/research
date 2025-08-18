---
layout: default
title: Interactive Soil Sampling Map
permalink: /coastalplain/
---

# Digital Soil Mapping Points

This interactive map shows field data points. Hover over each marker to see an image and description.

<div id="map" style="height: 400px; margin-bottom: 2em;"></div>

After the map, you can add more information about your sampling or research here.

<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<script>
  // Sample points array
  const points = [
    {
      lat: 40.71,
      lng: -74.00,
      name: "New York",
      img_url: "https://upload.wikimedia.org/wikipedia/commons/thumb/4/47/New_york_times_square-terabass.jpg/320px-New_york_times_square-terabass.jpg",
      desc: "Urban soil sampling site."
    },
    {
      lat: 34.05,
      lng: -118.24,
      name: "Los Angeles",
      img_url: "https://upload.wikimedia.org/wikipedia/commons/thumb/1/1e/Los_Angeles_at_night.jpg/320px-Los_Angeles_at_night.jpg",
      desc: "Coastal soil study area."
    }
  ];

  // Initialize map
  var map = L.map('map').setView([37, -95], 4);

  // Add OpenStreetMap tiles
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: 'Map data © OpenStreetMap contributors'
  }).addTo(map);

  // Add markers
  points.forEach(function(pt) {
    L.marker([pt.lat, pt.lng])
      .addTo(map)
      .bindPopup(
        `<b>${pt.name}</b><br>
         <img src="${pt.img_url}" style="width:150px;"><br>
         ${pt.desc}`
      );
  });
</script>
