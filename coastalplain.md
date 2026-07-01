---
layout: default
title: "Coastal Plain Interactive Map"
permalink: /coastalplain/
---

# Mapping Soil Profiles on UGAGrandFarm

This interactive map displays pedon data, such as horizon depth (cm) and color at UGA GrandFarm in Perry, GA. Click a point to view a pop-up of the soil profile on the map and in the side panel. 

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
  #mapColumn {
    flex: 2;
    min-width: 320px;
  }
  #map {
    height: 520px;
    width: 100%;
    border-radius: 12px;
  }
  #legend {
    margin-top: 12px;
    padding: 12px 14px;
    border: 1px solid rgba(0,0,0,0.15);
    border-radius: 12px;
    background: #fff;
  }
  .legend-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    margin-bottom: 10px;
  }
  .legend-header h4 {
    margin: 0;
  }
  #resetLegendBtn {
    border: 1px solid rgba(0,0,0,0.2);
    background: #f7f7f7;
    border-radius: 8px;
    padding: 6px 10px;
    cursor: pointer;
    font-size: 0.9rem;
  }
  #resetLegendBtn:hover {
    background: #ececec;
  }
  .legend-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 8px 16px;
  }
  .legend-item {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.95rem;
    padding: 6px 8px;
    border-radius: 8px;
    cursor: pointer;
    transition: background 0.2s ease, transform 0.2s ease, opacity 0.2s ease;
  }
  .legend-item:hover {
    background: rgba(0,0,0,0.05);
  }
  .legend-item.active {
    background: rgba(0,0,0,0.08);
    font-weight: 600;
  }
  .legend-item.dimmed {
    opacity: 0.45;
  }
  .legend-swatch {
    width: 14px;
    height: 14px;
    border-radius: 50%;
    border: 1.5px solid #000;
    flex: 0 0 14px;
  }
  .legend-label {
    color: inherit;
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

  .leaflet-popup-content { margin: 10px 12px; }
  .series-link {
    color: inherit;
    text-decoration: underline;
  }

  .popup-img {
    width: 160px;
    max-width: 100%;
    height: auto;
    max-height: 180px;
    border-radius: 10px;
    cursor: zoom-in;
    display: block;
    margin-top: 6px;
    object-fit: contain;
  }

  .image-carousel {
    position: relative;
    width: 100%;
    margin-top: 6px;
  }

  .carousel-images {
    position: relative;
    width: 100%;
    border-radius: 12px;
    overflow: hidden;
  }

  .carousel-image {
    width: 100%;
    height: auto;
    border-radius: 12px;
    cursor: zoom-in;
    object-fit: contain;
    display: none;
  }

  .carousel-image.active {
    display: block;
  }

  .carousel-arrow {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    background: rgba(0, 0, 0, 0.5);
    color: white;
    border: none;
    font-size: 24px;
    padding: 8px 12px;
    cursor: pointer;
    border-radius: 4px;
    z-index: 10;
    transition: background 0.3s;
    line-height: 1;
  }

  .carousel-arrow:hover {
    background: rgba(0, 0, 0, 0.8);
  }

  .carousel-arrow.left {
    left: 10px;
  }

  .carousel-arrow.right {
    right: 10px;
  }

  .carousel-dots {
    position: absolute;
    bottom: 10px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 6px;
    z-index: 10;
  }

  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.5);
    border: 1px solid rgba(255, 255, 255, 0.8);
    cursor: pointer;
    transition: background 0.3s;
  }

  .dot.active {
    background: rgba(255, 255, 255, 1);
  }

  #imgModal {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.85);
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
    position: relative;
  }

  #imgModal .closeNote {
    text-align: right;
    font-size: 0.9rem;
    opacity: 0.75;
    margin-bottom: 6px;
    cursor: pointer;
  }

  #imgModal .modal-carousel {
    position: relative;
    width: 100%;
  }

  #imgModal .modal-carousel-images {
    position: relative;
    width: 100%;
  }

  #imgModal .modal-carousel-image {
    width: 100%;
    height: auto;
    border-radius: 12px;
    object-fit: contain;
    display: none;
  }

  #imgModal .modal-carousel-image.active {
    display: block;
  }

  #imgModal .carousel-arrow {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    background: rgba(0, 0, 0, 0.6);
    color: white;
    border: none;
    font-size: 32px;
    padding: 12px 16px;
    cursor: pointer;
    border-radius: 4px;
    z-index: 10;
    transition: background 0.3s;
  }

  #imgModal .carousel-arrow:hover {
    background: rgba(0, 0, 0, 0.9);
  }

  #imgModal .carousel-dots {
    position: static;
    margin-top: 12px;
    display: flex;
    justify-content: center;
    gap: 8px;
  }

  #imgModal .dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(0, 0, 0, 0.5);
    cursor: pointer;
    transition: background 0.3s;
  }

  #imgModal .dot.active {
    background: rgba(0, 0, 0, 0.8);
  }

  #imgModal .caption {
    margin: 10px 6px 0;
    opacity: 0.85;
    text-align: center;
  }
</style>

<div id="mapWrap">
  <div id="mapColumn">
    <div id="map"></div>
    <div id="legend" aria-label="Map legend">
      <div class="legend-header">
        <h4>Soil Series Legend</h4>
        <button id="resetLegendBtn" type="button">Show all</button>
      </div>
      <div class="legend-grid" id="legendGrid"></div>
    </div>
  </div>

  <div id="infoPanel" aria-live="polite">
    <h3>Welcome</h3>
    <p class="muted">Click any point on the map to view its soil profile and field photo.</p>
  </div>
</div>

<div id="imgModal" onclick="closeModalFn(event)">
  <div class="modalCard" onclick="event.stopPropagation()">
    <div class="closeNote" onclick="closeModalFn()">Click outside or press ESC to close</div>
    <div class="modal-carousel" id="modalCarousel">
      <div class="modal-carousel-images" id="modalCarouselImages"></div>
      <button class="carousel-arrow left" onclick="changeModalImage(-1)">‹</button>
      <button class="carousel-arrow right" onclick="changeModalImage(1)">›</button>
      <div class="carousel-dots" id="modalCarouselDots"></div>
    </div>
    <p class="caption" id="modalCaption"></p>
  </div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>

<script>
  var map = L.map('map').setView([32.43, -83.73], 14);

  L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
    attribution: 'Tiles © Esri'
  }).addTo(map);

  var imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';
  const fieldPhotoBase = 'https://github.com/madelynwillis3/research/releases/download/coastalplain-images-v1.0/perry_GA_point_';
  const exclude = ["39", "50", "51", "58", "59", "60", "65", "74"];
  const defaultMarkerStyle = {
    color: "#000000",
    weight: 1.5,
    fillOpacity: 0.85,
    opacity: 1,
    radius: 6
  };
  const dimmedMarkerStyle = {
    fillOpacity: 0.15,
    opacity: 0.25,
    radius: 5
  };
  const highlightedMarkerStyle = {
    fillOpacity: 1,
    opacity: 1,
    radius: 8,
    weight: 2.5
  };
  const seriesByPoint = {
    "P2": "Thursa",
    "P3": "Faceville",
    "P4": "Faceville",
    "1": "Orangeburg",
    "2": "Faceville",
    "3": "Faceville",
    "4": "Faceville",
    "5": "Wagram",
    "6": "Norfolk",
    "7": "Bonneau",
    "8": "Wagram",
    "9": "Orangeburg",
    "10": "Esto",
    "11": "Norfolk",
    "12": "Orangeburg",
    "13": "Norfolk",
    "14": "Orangeburg",
    "15": "Dothan",
    "16": "Lakeland",
    "17": "Orangeburg",
    "18": "Orangeburg",
    "19": "Faceville",
    "20": "Marvyn",
    "21": "Norfolk",
    "22": "Orangeburg",
    "23": "Norfolk",
    "24": "Norfolk",
    "25": "Lakeland",
    "26": "Benevolence",
    "27": "Greenville",
    "28": "Greenville",
    "29": "Red Bay",
    "30": "Faceville",
    "31": "Faceville",
    "32": "Norfolk",
    "33": "Norfolk",
    "34": "Johns",
    "35": "Disturbed",
    "36": "Orangeburg",
    "37": "Faceville",
    "38": "Faceville",
    "40": "Greenville",
    "41": "Orangeburg",
    "42": "Dothan",
    "43": "Norfolk",
    "44": "Blanton",
    "45": "Greenville",
    "46": "Greenville",
    "47": "Faceville",
    "48": "Greenville",
    "49": "Disturbed",
    "52": "Leefield",
    "53": "Orangeburg",
    "54": "Faceville",
    "55": "Faceville",
    "56": "Faceville",
    "57": "Greenville",
    "61": "Greenville",
    "62": "Greenville",
    "63": "Lucy",
    "64": "Greenville",
    "66": "Faceville",
    "67": "Faceville",
    "68": "Greenville",
    "69": "Faceville",
    "70": "Faceville",
    "71": "Greenville",
    "72": "Orangeburg",
    "73": "Disturbed",
    "75": "Faceville",
    "76": "Orangeburg",
    "77": "Orangeburg",
    "78": "Lucy",
    "79": "Troup",
    "80": "Troup"
  };
  const seriesColors = {
    "Faceville": "#d73027",
    "Orangeburg": "#c94c4c",
    "Lucy": "#e76f51",
    "Troup": "#f4a6a6",
    "Greenville": "#8b0000",
    "Leefield": "#d8c3a5",
    "Blanton": "#8a7f73",
    "Norfolk": "#f28c28",
    "Dothan": "#d4a017",
    "Johns": "#c2a878",
    "Red Bay": "#5c0000",
    "Benevolence": "#fa8072",
    "Lakeland": "#d2a679",
    "Wagram": "#d2a679",
    "Bonneau": "#d3d3d3",
    "Esto": "#c9a44b",
    "Marvyn": "#a44a3f",
    "Disturbed": "#808080"
  };
  const visibleSeriesCounts = {};
  Object.entries(seriesByPoint).forEach(([pointId, seriesName]) => {
    if (!exclude.includes(pointId)) {
      visibleSeriesCounts[seriesName] = (visibleSeriesCounts[seriesName] || 0) + 1;
    }
  });
  const seriesDisplayOrder = Object.keys(visibleSeriesCounts).sort((a, b) => {
    const countDiff = visibleSeriesCounts[b] - visibleSeriesCounts[a];
    return countDiff !== 0 ? countDiff : a.localeCompare(b);
  });
  const seriesLinks = Object.fromEntries(
    seriesDisplayOrder
      .filter(series => series !== "Disturbed")
      .map(series => [
        series,
        `https://casoilresource.lawr.ucdavis.edu/sde/?series=${encodeURIComponent(series.toUpperCase())}#osd`
      ])
  );

  const panel = document.getElementById("infoPanel");
  const imgModal = document.getElementById("imgModal");
  const modalCarouselImages = document.getElementById("modalCarouselImages");
  const modalCarouselDots = document.getElementById("modalCarouselDots");
  const modalCaption = document.getElementById("modalCaption");
  const legendGrid = document.getElementById("legendGrid");
  const resetLegendBtn = document.getElementById("resetLegendBtn");

  let currentModalIndex = 0;
  let modalImagesSources = [];
  let currentLabel = '';
  let activeSeries = null;
  let markerEntries = [];
  let panelCurrentIndex = 0;
  let targetLatLng = null;

  function escapeHtml(text) {
    return String(text)
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#39;');
  }

  function getSeriesLinkHTML(seriesName) {
    const safeSeriesName = escapeHtml(seriesName);
    const url = seriesLinks[seriesName];
    if (!url) return safeSeriesName;
    return `<a href="${url}" target="_blank" rel="noopener noreferrer" class="series-link">${safeSeriesName}</a>`;
  }

  function updateLegendState() {
    const items = legendGrid.querySelectorAll('.legend-item');
    items.forEach((item) => {
      const seriesName = item.dataset.series;
      const isActive = activeSeries === seriesName;
      item.classList.toggle('active', isActive);
      item.classList.toggle('dimmed', !!activeSeries && activeSeries !== seriesName);
      item.setAttribute('aria-pressed', isActive ? 'true' : 'false');
    });
  }

  function pulseMarker(marker) {
    marker.setStyle({ radius: 10, weight: 3 });
    setTimeout(() => marker.setStyle({ radius: highlightedMarkerStyle.radius, weight: highlightedMarkerStyle.weight }), 180);
    setTimeout(() => marker.setStyle({ radius: 10, weight: 3 }), 360);
    setTimeout(() => marker.setStyle({ radius: highlightedMarkerStyle.radius, weight: highlightedMarkerStyle.weight }), 540);
  }

  function applySeriesHighlight(seriesName) {
    activeSeries = seriesName;
    markerEntries.forEach(({ marker, series }) => {
      if (series === seriesName) {
        marker.setStyle({
          fillOpacity: highlightedMarkerStyle.fillOpacity,
          opacity: highlightedMarkerStyle.opacity,
          radius: highlightedMarkerStyle.radius,
          weight: highlightedMarkerStyle.weight
        });
        pulseMarker(marker);
      } else {
        marker.setStyle({
          fillOpacity: dimmedMarkerStyle.fillOpacity,
          opacity: dimmedMarkerStyle.opacity,
          radius: dimmedMarkerStyle.radius,
          weight: defaultMarkerStyle.weight
        });
      }
    });
    updateLegendState();
  }

  function resetSeriesHighlight() {
    activeSeries = null;
    markerEntries.forEach(({ marker }) => {
      marker.setStyle({
        fillOpacity: defaultMarkerStyle.fillOpacity,
        opacity: defaultMarkerStyle.opacity,
        radius: defaultMarkerStyle.radius,
        weight: defaultMarkerStyle.weight
      });
    });
    updateLegendState();
  }

  function renderLegend() {
    legendGrid.innerHTML = '';
    seriesDisplayOrder.forEach((seriesName) => {
      const item = document.createElement('button');
      item.type = 'button';
      item.className = 'legend-item';
      item.dataset.series = seriesName;
      item.setAttribute('aria-pressed', 'false');

      const swatch = document.createElement('span');
      swatch.className = 'legend-swatch';
      swatch.style.backgroundColor = seriesColors[seriesName] || '#808080';
      item.appendChild(swatch);

      const label = document.createElement('span');
      label.className = 'legend-label';
      label.textContent = `${seriesName} (${visibleSeriesCounts[seriesName]})`;
      item.appendChild(label);

      item.addEventListener('click', () => {
        if (activeSeries === seriesName) {
          resetSeriesHighlight();
        } else {
          applySeriesHighlight(seriesName);
        }
      });

      legendGrid.appendChild(item);
    });
    updateLegendState();
  }

  resetLegendBtn.addEventListener('click', resetSeriesHighlight);

  function openModal(images, label, startIndex = 0) {
    modalImagesSources = images;
    currentLabel = label;
    currentModalIndex = startIndex;
    modalCarouselImages.innerHTML = '';
    modalCarouselDots.innerHTML = '';

    images.forEach((imgSrc, index) => {
      const img = document.createElement('img');
      img.src = imgSrc;
      img.className = 'modal-carousel-image' + (index === startIndex ? ' active' : '');
      img.alt = `Image ${index + 1} for Sample ${label}`;
      modalCarouselImages.appendChild(img);
    });

    images.forEach((_, index) => {
      const dot = document.createElement('span');
      dot.className = 'dot' + (index === startIndex ? ' active' : '');
      dot.onclick = () => goToModalImage(index);
      modalCarouselDots.appendChild(dot);
    });

    modalCaption.textContent = `Sample ${currentLabel} - Image ${startIndex + 1} of ${images.length}`;
    imgModal.style.display = "block";
    document.body.style.overflow = "hidden";
  }

  function closeModalFn(event) {
    if (event && event.target !== imgModal && event.type === 'click') return;
    imgModal.style.display = "none";
    document.body.style.overflow = "";
  }

  function changeModalImage(direction) {
    const images = modalCarouselImages.querySelectorAll('.modal-carousel-image');
    const dots = modalCarouselDots.querySelectorAll('.dot');
    images[currentModalIndex].classList.remove('active');
    dots[currentModalIndex].classList.remove('active');
    currentModalIndex = (currentModalIndex + direction + images.length) % images.length;
    images[currentModalIndex].classList.add('active');
    dots[currentModalIndex].classList.add('active');
    modalCaption.textContent = `Sample ${currentLabel} - Image ${currentModalIndex + 1} of ${images.length}`;
  }

  function goToModalImage(index) {
    const images = modalCarouselImages.querySelectorAll('.modal-carousel-image');
    const dots = modalCarouselDots.querySelectorAll('.dot');
    images[currentModalIndex].classList.remove('active');
    dots[currentModalIndex].classList.remove('active');
    currentModalIndex = index;
    images[currentModalIndex].classList.add('active');
    dots[currentModalIndex].classList.add('active');
    modalCaption.textContent = `Sample ${currentLabel} - Image ${currentModalIndex + 1} of ${images.length}`;
  }

  document.addEventListener("keydown", (e) => {
    if (imgModal.style.display === "block") {
      if (e.key === "Escape") closeModalFn();
      if (e.key === "ArrowLeft") changeModalImage(-1);
      if (e.key === "ArrowRight") changeModalImage(1);
    }
  });

  function changePanelImage(direction) {
    const carousel = panel.querySelector('.image-carousel');
    if (!carousel) return;
    const images = carousel.querySelectorAll('.carousel-image');
    const dots = carousel.querySelectorAll('.dot');
    images[panelCurrentIndex].classList.remove('active');
    dots[panelCurrentIndex].classList.remove('active');
    panelCurrentIndex = (panelCurrentIndex + direction + images.length) % images.length;
    images[panelCurrentIndex].classList.add('active');
    dots[panelCurrentIndex].classList.add('active');
  }

  function goToPanelImage(index) {
    const carousel = panel.querySelector('.image-carousel');
    if (!carousel) return;
    const images = carousel.querySelectorAll('.carousel-image');
    const dots = carousel.querySelectorAll('.dot');
    images[panelCurrentIndex].classList.remove('active');
    dots[panelCurrentIndex].classList.remove('active');
    panelCurrentIndex = index;
    images[panelCurrentIndex].classList.add('active');
    dots[panelCurrentIndex].classList.add('active');
  }

  window.changePanelImage = changePanelImage;
  window.goToPanelImage = goToPanelImage;

  function setPanelContent(label, lat, lng, images) {
    const seriesName = seriesByPoint[label] || label;
    panelCurrentIndex = 0;
    panel.innerHTML = `
      <h3>${getSeriesLinkHTML(seriesName)}</h3>
      <p class="muted">Point ${label} | Lat: ${lat.toFixed(6)} | Lon: ${lng.toFixed(6)}</p>
      <div class="image-carousel">
        <div class="carousel-images">
          <img src="${images[0]}" class="carousel-image active" alt="Profile image for ${seriesName} (Point ${label})" onclick="openModal(${JSON.stringify(images).replace(/"/g, '&quot;')}, '${label}', 0)">
          <img src="${images[1]}" class="carousel-image" alt="Field photo for ${seriesName} (Point ${label})" onclick="openModal(${JSON.stringify(images).replace(/"/g, '&quot;')}, '${label}', 1)">
        </div>
        <button class="carousel-arrow left" onclick="changePanelImage(-1)">‹</button>
        <button class="carousel-arrow right" onclick="changePanelImage(1)">›</button>
        <div class="carousel-dots">
          <span class="dot active" onclick="goToPanelImage(0)"></span>
          <span class="dot" onclick="goToPanelImage(1)"></span>
        </div>
      </div>
      <p class="muted" style="margin-top:10px;">
        Tip: Use arrows to switch images. Click any image to enlarge.
      </p>
    `;
  }

  Papa.parse('{{ "/assets/data/perry_FP_samples_80.csv" | relative_url }}', {
    download: true,
    header: true,
    complete: function(results) {
      const markers = [];
      markerEntries = [];
      renderLegend();

      results.data.forEach(function(row) {
        if (!row.x || !row.y || !row["Point ID"]) return;

        const lat = parseFloat(row.y);
        const lng = parseFloat(row.x);
        const label = row["Point ID"].trim();
        const seriesName = seriesByPoint[label] || label;
        const fillColor = seriesColors[seriesName] || "#808080";

        if (exclude.includes(label)) return;
        if (isNaN(lat) || isNaN(lng)) return;
        if (label === "45") targetLatLng = [lat, lng];

        const profileImg = `${imgBase}${label}.jpg`;
        const fieldImg = `${fieldPhotoBase}${label}.jpg`;
        const images = [profileImg, fieldImg];

        const popupHTML = `
          <b>${getSeriesLinkHTML(seriesName)}</b><br>
          <span>Point ${label}</span><br>
          <img src="${profileImg}" class="popup-img"
               alt="Profile image for ${seriesName} (Point ${label})"
               onclick="openModal(${JSON.stringify(images).replace(/"/g, '&quot;')}, '${label}', 0)">
        `;

        const marker = L.circleMarker([lat, lng], {
          radius: defaultMarkerStyle.radius,
          color: defaultMarkerStyle.color,
          weight: defaultMarkerStyle.weight,
          fillColor: fillColor,
          fillOpacity: 0.0,
          opacity: 0
        });

        marker.bindPopup(popupHTML);
        marker.on("click", () => {
          setPanelContent(label, lat, lng, images);
          marker.openPopup();
        });

        markers.push(marker);
        markerEntries.push({ marker, series: seriesName });
      });

      const target = targetLatLng || [32.43, -83.73];
      map.flyTo(target, 15, { animate: true, duration: 1.6 });

      setTimeout(() => {
        const delayMs = 30;
        markers.forEach((m, i) => {
          setTimeout(() => {
            m.addTo(map);
            m.setStyle({ fillOpacity: 0.35, opacity: 0.5 });
            setTimeout(() => m.setStyle({ fillOpacity: 0.60, opacity: 0.75 }), 60);
            setTimeout(() => m.setStyle({ fillOpacity: defaultMarkerStyle.fillOpacity, opacity: defaultMarkerStyle.opacity }), 120);
          }, i * delayMs);
        });
      }, 1700);
    }
  });
</script>
