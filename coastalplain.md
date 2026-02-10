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

  /* Leaflet popup polish */
  .leaflet-popup-content { margin: 10px 12px; }

  /* Popup image: preserve aspect ratio (no stretching) */
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

  /* Carousel styles for side panel */
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

  /* Modal with carousel */
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
  <div id="map"></div>

  <div id="infoPanel" aria-live="polite">
    <h3>Welcome</h3>
    <p class="muted">Click any point on the map to view its soil profile and field photo.</p>
  </div>
</div>

<!-- Modal for enlarged image with carousel -->
<div id="imgModal" onclick="closeModalFn(event)">
  <div class="modalCard" onclick="event.stopPropagation()">
    <div class="closeNote" onclick="closeModalFn()">Click outside or press ESC to close</div>
    <div class="modal-carousel" id="modalCarousel">
      <div class="modal-carousel-images" id="modalCarouselImages">
        <!-- Images will be inserted here dynamically -->
      </div>
      <button class="carousel-arrow left" onclick="changeModalImage(-1)">‹</button>
      <button class="carousel-arrow right" onclick="changeModalImage(1)">›</button>
      <div class="carousel-dots" id="modalCarouselDots">
        <!-- Dots will be inserted here dynamically -->
      </div>
    </div>
    <p class="caption" id="modalCaption"></p>
  </div>
</div>

<!-- Leaflet JS + PapaParse for CSV -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>

<script>
  // ---------- Map base ----------
  var map = L.map('map').setView([32.43, -83.73], 14);

  L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
    attribution: 'Tiles © Esri'
  }).addTo(map);

  var imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';
  const fieldPhotoBase = 'https://github.com/madelynwillis3/research/releases/download/coastalplain-images-v1.0/perry_GA_point_';
  const exclude = ["39", "50", "51", "58", "60", "65", "74"];

  // ---------- Side panel + modal ----------
  const panel = document.getElementById("infoPanel");
  const imgModal = document.getElementById("imgModal");
  const modalCarouselImages = document.getElementById("modalCarouselImages");
  const modalCarouselDots = document.getElementById("modalCarouselDots");
  const modalCaption = document.getElementById("modalCaption");

  let currentModalIndex = 0;
  let modalImagesSources = [];
  let currentLabel = '';

  // Open modal with carousel
  function openModal(images, label, startIndex = 0) {
    modalImagesSources = images;
    currentLabel = label;
    currentModalIndex = startIndex;

    // Clear previous content
    modalCarouselImages.innerHTML = '';
    modalCarouselDots.innerHTML = '';

    // Add images
    images.forEach((imgSrc, index) => {
      const img = document.createElement('img');
      img.src = imgSrc;
      img.className = 'modal-carousel-image' + (index === startIndex ? ' active' : '');
      img.alt = `Image ${index + 1} for Sample ${label}`;
      modalCarouselImages.appendChild(img);
    });

    // Add dots
    images.forEach((_, index) => {
      const dot = document.createElement('span');
      dot.className = 'dot' + (index === startIndex ? ' active' : '');
      dot.onclick = () => goToModalImage(index);
      modalCarouselDots.appendChild(dot);
    });

    modalCaption.textContent = `Sample ${label} - Image ${startIndex + 1} of ${images.length}`;
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

  // Keyboard navigation for modal
  document.addEventListener("keydown", (e) => {
    if (imgModal.style.display === "block") {
      if (e.key === "Escape") closeModalFn();
      if (e.key === "ArrowLeft") changeModalImage(-1);
      if (e.key === "ArrowRight") changeModalImage(1);
    }
  });

  // Panel carousel functions
  let panelCurrentIndex = 0;

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

  // Make functions globally available
  window.changePanelImage = changePanelImage;
  window.goToPanelImage = goToPanelImage;

  function setPanelContent(label, lat, lng, images) {
    panelCurrentIndex = 0; // Reset to first image

    panel.innerHTML = `
      <h3>Sample ${label}</h3>
      <p class="muted">Lat: ${lat.toFixed(6)} | Lon: ${lng.toFixed(6)}</p>
      <div class="image-carousel">
        <div class="carousel-images">
          <img src="${images[0]}" class="carousel-image active" alt="Profile image for Sample ${label}" onclick="openModal(${JSON.stringify(images).replace(/"/g, '&quot;')}, '${label}', 0)">
          <img src="${images[1]}" class="carousel-image" alt="Field photo for Sample ${label}" onclick="openModal(${JSON.stringify(images).replace(/"/g, '&quot;')}, '${label}', 1)">
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

        const profileImg = `${imgBase}${label}.jpg`;
        const fieldImg = `${fieldPhotoBase}${label}.jpg`;
        const images = [profileImg, fieldImg];

        // Simple popup with just the first image (no carousel in popup)
        const popupHTML = `
          <b>Sample ${label}</b><br>
          <img src="${profileImg}" class="popup-img"
               alt="Profile image for Sample ${label}"
               onclick="openModal(${JSON.stringify(images).replace(/"/g, '&quot;')}, '${label}', 0)">
        `;

        const marker = L.circleMarker([lat, lng], {
          radius: 6,
          color: "#007BFF",
          fillColor: "#3399FF",
          fillOpacity: 0.0
        });

        marker.bindPopup(popupHTML);

        // Click updates side panel AND opens the small map popup
        marker.on("click", () => {
          setPanelContent(label, lat, lng, images);
          marker.openPopup();
        });

        markers.push(marker);
      });

      // Smooth animated fly-in (slightly zoomed out)
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
