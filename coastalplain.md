---
layout: default
title: "Coastal Plain Interactive Map"
permalink: /coastalplain/
---

# Mapping Soil Profiles on UGAGrandFarm

This interactive map shows field data points on UGA GrandFarm in Perry, GA. Click on a point to load the sample’s details in the side panel (larger + easier to read than a popup). Click the image to view it full-size.

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
  #infoPanel .btn {
    display: inline-block;
    padding: 7px 10px;
    border: 1px solid rgba(0,0,0,0.2);
    border-radius: 10px;
    background: #f7f7f7;
    cursor: pointer;
    font-size: 0.95rem;
  }
  #infoPanel img {
    width: 100%;
    height: auto;
    border-radius: 12px;
    cursor: zoom-in;
  }

  /* Modal */
  #imgModal {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.75);
    z-index: 9999;
    padding: 24px;
  }
  #imgModal .modalCard {
    max-width: 1100px;
    margin: 0 auto;
    background: #fff;
    border-radius: 14px;
    padding: 12px;
    position: relative;
  }
  #imgModal .closeBtn {
    position: absolute;
    top: 10px;
    right: 10px;
    padding: 7px 10px;
    border-radius: 10px;
    border: 1px solid rgba(0,0,0,0.2);
    background: #f7f7f7;
    cursor: pointer;
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
</style>

<div id="mapWrap">
  <div id="map"></div>

  <div id="infoPanel" aria-live="polite">
    <h3>Sample details</h3>
    <p class="muted">Click a sample point to view details here.</p>
    <p class="muted">Tip: click the photo to enlarge. Press <b>Esc</b> to close.</p>
  </div>
</div>

<!-- Image modal -->
<div id="imgModal" role="dialog" aria-modal="true" aria-label="Enlarged sample image">
  <div class="modalCard">
    <button class="closeBtn" id="closeModal">Close</button>
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

  // Base folder for images
  var imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';

  // List of non-sampled points:
  const exclude = ["39", "50", "51", "58", "60", "65", "74"];

  // ---------- Side panel + modal ----------
  const panel = document.getElementById("infoPanel");
  const imgModal = document.getElementById("imgModal");
  const modalImg = document.getElementById("modalImg");
  const modalCaption = document.getElementById("modalCaption");
  const closeModal = document.getElementById("closeModal");

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

  closeModal.addEventListener("click", closeModalFn);
  imgModal.addEventListener("click", (e) => { if (e.target === imgModal) closeModalFn(); });
  document.addEventListener("keydown", (e) => { if (e.key === "Escape") closeModalFn(); });

  // Make openModal callable from inline onclick
  window.openModal = openModal;

  function setPanelContent(label, lat, lng, imgSrc) {
    panel.innerHTML = `
      <h3>Sample ${label}</h3>
      <p class="muted">Lat: ${lat.toFixed(6)} | Lon: ${lng.toFixed(6)}</p>

      <div style="display:flex; gap:10px; flex-wrap:wrap; margin-bottom:10px;">
        <button class="btn" type="button" id="copyBtn">Copy coords</button>
        <button class="btn" type="button" id="openImgBtn">View full-size</button>
      </div>

      <img src="${imgSrc}" alt="Profile image for Sample ${label}" onclick="openModal('${imgSrc}', 'Sample ${label}')">

      <p class="muted" style="margin-top:10px;">
        Tip: click the image to enlarge. Press <b>Esc</b> to close.
      </p>
    `;

    // Wire buttons (re-created each click)
    document.getElementById("copyBtn").addEventListener("click", async () => {
      try {
        await navigator.clipboard.writeText(`Sample ${label} — ${lat}, ${lng}`);
        const btn = document.getElementById("copyBtn");
        btn.textContent = "Copied!";
        setTimeout(() => (btn.textContent = "Copy coords"), 900);
      } catch (err) {
        alert("Copy failed (browser permissions).");
      }
    });

    document.getElementById("openImgBtn").addEventListener("click", () => {
      openModal(imgSrc, `Sample ${label}`);
    });
  }

  // ---------- CSV parse + animated marker drop ----------
  let targetLatLng = null; // we'll still center on 45 for a good “middle”, but it’s not special

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
          fillOpacity: 0.0  // start invisible -> fade in during animation
        });

        marker.on("click", () => {
          setPanelContent(label, lat, lng, imgSrc);
        });

        markers.push(marker);
      });

      // Animated “polished” fly-in (slightly zoomed out from your prior 16)
      const target = targetLatLng || [32.43, -83.73];
      map.flyTo(target, 15, { animate: true, duration: 1.6 });

      // After fly finishes, add markers sequentially with a fade-in
      setTimeout(() => {
        const delayMs = 30; // bigger number = slower “rain in”
        markers.forEach((m, i) => {
          setTimeout(() => {
            m.addTo(map);
            // simple 3-step fade
            m.setStyle({ fillOpacity: 0.30 });
            setTimeout(() => m.setStyle({ fillOpacity: 0.60 }), 60);
            setTimeout(() => m.setStyle({ fillOpacity: 0.85 }), 120);
          }, i * delayMs);
        });
      }, 1700);
    }
  });
</script>
