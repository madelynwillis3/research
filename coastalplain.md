---
layout: default
title: "Coastal Plain Interactive Map"
permalink: /coastalplain/
---

# Mapping Soil Profiles on UGAGrandFarm

This interactive map shows field data points on UGA GrandFarm in Perry, GA. Click a point to load the sample’s details in the side panel. Click the image to view it larger and scroll.

This site is situated in the northern Coastal Plain, where 75 soil profiles were extracted and analyzed for texture, color, horizon, depth, structure, and other identifying notes.

<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

<style>
  /* Layout */
  #mapWrap {
    display: flex;
    gap: 16px;
    align-items: flex-start;
    flex-wrap: wrap;
    margin-bottom: 1.5em;
  }
  #map {
    height: 520px;
    flex: 2;
    min-width: 320px;
    border-radius: 12px;
  }
  #infoPanel {
    flex: 1;
    min-width: 280px;
    max-width: 460px;
    padding: 12px 14px;
    border: 1px solid rgba(0,0,0,0.15);
    border-radius: 12px;
    background: #fff;
    position: sticky;
    top: 12px;
  }
  #infoPanel h3 { margin: 0 0 6px; }
  #infoPanel .muted { margin: 0 0 10px; opacity: 0.8; font-size: 0.95rem; }
  #infoPanel img {
    width: 100%;
    height: auto;
    border-radius: 12px;
    cursor: zoom-in;
  }

  /* Modal (scrollable + constrained height) */
  #imgModal {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.75);
    z-index: 9999;
    padding: 24px;
    overflow-y: auto;
  }
  #imgModal .modalCard {
    max-width: 900px;
    margin: 0 auto;
    background: #fff;
    border-radius: 14px;
    padding: 12px;
  }
  #imgModal img {
    width: 100%;
    height: auto;
    border-radius: 12px;
  }
  #imgModal .caption {
    margin: 10px 6px 0;
    opacity: 0.85;
  }
  #imgModal .closeNote {
    text-align: right;
    font-size: 0.9rem;
    opacity: 0.75;
    margin-bottom: 6px;
  }
</style>

<div id="mapWrap">
  <div id="map"></div>

  <div id="infoPanel" aria-live="polite">
    <h3>Sample details</h3>
    <p class="muted">Click a sample point to view details here.</p>
    <p class="muted">Tip: click the photo to enlarge. Scroll to view the full profile.</p>
  </div>
</div>

<!-- Image modal -->
<div id="imgModal" role="dialog" aria-modal="true" aria-label="Enlarged sample image">
  <div class="modalCard">
    <div class="closeNote">Press <b>Esc</b> or click outside the image to close</div>
    <img id="modalImg" src="" alt="Enlarged sample image">
    <p class="caption" id="modalCaption"></p>
  </div>
</div>

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<!-- PapaParse for CSV parsing -->
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>

<script>
  // ---------- Map base ----------
  var map = L.map('map').setView([32.43, -83.73], 14);

  L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
    attribution: 'Tiles © Esri'
  }).addTo(map);

  var imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';
  const exclude = ["39", "50", "51", "58", "60", "65", "74"];

  // ---------- Side panel + modal ----------
  const panel = document.getElementById("infoPanel");
  const imgModal = document.getElementById("imgModal");
  const modalImg = document.getElementById("modalImg");
  const modalCaption = document.getElementById("modalCaption");

  function openModal(imgSrc, caption) {
    modalImg.src = imgSrc;
    modalCaption.textContent = caption || "";
    imgModal.style.display = "block";
    document.body.style.overflow = "hidden";
  }
  function closeModalFn() {
    imgModal.style.display = "none";
    modalImg.src = "";
    document.body.style.overflow = "";
  }

  imgModal.addEventListener("click", (e) => { if (e.target === imgModal) closeModalFn(); });
  document.addEventListener("keydown", (e) => { if (e.key === "Escape") closeModalFn(); });

  window.openModal = openModal;

  function setPanelContent(label, lat, lng, imgSrc) {
    panel.innerHTML = `
      <h3>Sample ${label}</h3>
      <p class="muted">Lat: ${lat.toFixed(6)} | Lon: ${lng.toFixed(6)}</p>
      <img src="${imgSrc}" alt="Profile image for Sample ${label}" onclick="openModal('${imgSrc}', 'Sample ${label}')">
      <p class="muted" style="margin-top:10px;">
        Tip: click the image to enlarge. Scroll to view the full profile.
      </p>
    `;
  }

  // ---------- CSV parse + animated marker drop ----------
  let targetLatLng = null;

  Papa.parse('{{ "/assets/data/perry_FP_samples_80.csv" | relative_url }}', {
    download: true,
    header: true,
    complete: function(results) {

      const markers = [];

      results.data.forEach(function(row) {
        if (!row.x || !row.y || !row["Point ID"]) return;

        const lat = parseFloat(row.y);
        const lng = parseFloat(row.x);
        const label = row["Point ID"].trim();

        if (exclude.includes(label)) return;
        if (isNaN(lat) || isNaN(lng)) return;

        if (label === "45") targetLatLng = [lat, lng];

        const imgSrc = `${imgBase}${label}.jpg`;

        const marker = L.circleMarker([lat, lng], {
          radius: 6,
          color: "#007BFF",
          fillColor: "#3399FF",
          fillOpacity: 0.0
        });

        marker.on("click", () => {
          setPanelContent(label, lat, lng, imgSrc);
        });

        markers.push(marker);
      });

      // Smooth animated fly-in
      const target = targetLatLng || [32.43, -83.73];
      map.flyTo(target, 15, { animate: true, duration: 1.6 });

      // Sequential marker fade-in
      setTimeout(() => {
        const delayMs = 30;
        markers.forEach((m, i) => {
          setTimeout(() => {
            m.addTo(map);
            m.setStyle({ fillOpacity: 0.30 });
            setTimeout(() => m.setStyle({ fillOpacity: 0.60 }), 60);
            setTimeout(() => m.setStyle({ fillOpacity: 0.85 }), 120);
          }, i * delayMs);
        });
      }, 1700);
    }
  });
</script>
