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
  #infoPanel img {
    width: 100%;
    height: auto;
    border-radius: 12px;
    cursor: zoom-in;
    object-fit: contain;
  }

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
    object-fit: contain;
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

  /* Image Carousel Styles */
  .image-carousel {
    position: relative;
    width: 100%;
    margin-top: 6px;
  }

  .carousel-images {
    position: relative;
    width: 100%;
    overflow: hidden;
  }

  .carousel-image {
    width: 100%;
    height: auto;
    border-radius: 10px;
    cursor: zoom-in;
    display: none;
    object-fit: contain;
    transition: opacity 0.3s ease;
  }

  .carousel-image.active {
    display: block;
  }

  /* Popup-specific carousel image sizing */
  .leaflet-popup .carousel-image {
    max-width: 160px;
    max-height: 180px;
  }

  /* Panel-specific carousel image sizing */
  #infoPanel .carousel-image {
    width: 100%;
    max-height: none;
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
    transition: background 0.2s ease;
    line-height: 1;
  }

  .carousel-arrow:hover {
    background: rgba(0, 0, 0, 0.75);
  }

  .carousel-arrow.left {
    left: 8px;
  }

  .carousel-arrow.right {
    right: 8px;
  }

  .carousel-dots {
    position: absolute;
    bottom: 8px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 6px;
    z-index: 10;
  }

  .carousel-dots .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.5);
    border: 1px solid rgba(0, 0, 0, 0.3);
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .carousel-dots .dot.active {
    background: rgba(255, 255, 255, 0.95);
    transform: scale(1.2);
  }

  .carousel-dots .dot:hover {
    background: rgba(255, 255, 255, 0.75);
  }
</style>

<div id="mapWrap">
  <div id="map"></div>

  <div id="infoPanel" aria-live="polite">
    <h3>Sample details</h3>
    <p class="muted">Click a sample point to view details here.</p>
    <p class="muted">Tip: click to enlarge.</p>
  </div>
</div>

<!-- Image modal -->
<div id="imgModal" role="dialog" aria-modal="true" aria-label="Enlarged sample image">
  <div class="modalCard">
    <div class="closeNote">Press <b>Esc</b> to close</div>
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

  // allow inline onclick in popup HTML + panel HTML
  window.openModal = openModal;

  // ---------- Carousel Navigation Functions ----------
  
  // Change image by direction (1 for next, -1 for previous)
  function changeImage(event, label, direction, context) {
    event.stopPropagation(); // Prevent popup close or other unwanted behaviors
    
    // Find the carousel container for this label and context
    const carousel = document.querySelector(`.image-carousel[data-label="${label}"][data-context="${context}"]`);
    if (!carousel) return;
    
    const images = carousel.querySelectorAll('.carousel-image');
    const dots = carousel.querySelectorAll('.dot');
    
    // Find current active image
    let currentIndex = -1;
    images.forEach((img, idx) => {
      if (img.classList.contains('active')) {
        currentIndex = idx;
      }
    });
    
    // Calculate next index (wrap around)
    let nextIndex = currentIndex + direction;
    if (nextIndex < 0) nextIndex = images.length - 1;
    if (nextIndex >= images.length) nextIndex = 0;
    
    // Update active classes
    images.forEach((img, idx) => {
      img.classList.toggle('active', idx === nextIndex);
    });
    dots.forEach((dot, idx) => {
      dot.classList.toggle('active', idx === nextIndex);
    });
  }
  
  // Jump directly to a specific image
  function goToImage(event, label, index, context) {
    event.stopPropagation();
    
    const carousel = document.querySelector(`.image-carousel[data-label="${label}"][data-context="${context}"]`);
    if (!carousel) return;
    
    const images = carousel.querySelectorAll('.carousel-image');
    const dots = carousel.querySelectorAll('.dot');
    
    // Update active classes
    images.forEach((img, idx) => {
      img.classList.toggle('active', idx === index);
    });
    dots.forEach((dot, idx) => {
      dot.classList.toggle('active', idx === index);
    });
  }
  
  // Open modal with the currently displayed carousel image
  function openModalFromCarousel(event, label, context) {
    event.stopPropagation();
    
    const carousel = document.querySelector(`.image-carousel[data-label="${label}"][data-context="${context}"]`);
    if (!carousel) return;
    
    const activeImage = carousel.querySelector('.carousel-image.active');
    if (activeImage) {
      openModal(activeImage.src, `Sample ${label}`);
    }
  }
  
  // Touch swipe support
  let touchStartX = 0;
  let touchEndX = 0;
  const swipeThreshold = 50; // minimum distance for a swipe
  
  document.addEventListener('touchstart', (e) => {
    const carousel = e.target.closest('.image-carousel');
    if (carousel) {
      touchStartX = e.changedTouches[0].screenX;
    }
  });
  
  document.addEventListener('touchend', (e) => {
    const carousel = e.target.closest('.image-carousel');
    if (carousel) {
      touchEndX = e.changedTouches[0].screenX;
      handleSwipe(carousel);
    }
  });
  
  function handleSwipe(carousel) {
    const swipeDistance = touchEndX - touchStartX;
    if (Math.abs(swipeDistance) < swipeThreshold) return;
    
    const label = carousel.dataset.label;
    const context = carousel.dataset.context;
    const direction = swipeDistance > 0 ? -1 : 1; // swipe right = previous, swipe left = next
    
    // Create a synthetic event
    const syntheticEvent = { stopPropagation: () => {} };
    changeImage(syntheticEvent, label, direction, context);
  }
  
  // Keyboard arrow key support for carousel
  document.addEventListener('keydown', (e) => {
    // Only handle arrow keys when not in an input field
    if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
    
    if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
      // Find if there's an active carousel in the panel
      const panelCarousel = document.querySelector('#infoPanel .image-carousel');
      if (panelCarousel) {
        const label = panelCarousel.dataset.label;
        const context = panelCarousel.dataset.context;
        const direction = e.key === 'ArrowLeft' ? -1 : 1;
        const syntheticEvent = { stopPropagation: () => {} };
        changeImage(syntheticEvent, label, direction, context);
        e.preventDefault(); // Prevent default arrow key behavior
      }
    }
  });
  
  // Make functions globally available
  window.changeImage = changeImage;
  window.goToImage = goToImage;
  window.openModalFromCarousel = openModalFromCarousel;

  function setPanelContent(label, lat, lng, imgSrc) {
    panel.innerHTML = `
      <h3>Sample ${label}</h3>
      <p class="muted">Lat: ${lat.toFixed(6)} | Lon: ${lng.toFixed(6)}</p>
      <div class="image-carousel" data-label="${label}" data-context="panel">
        <div class="carousel-images">
          <img src="${imgSrc}" class="carousel-image active" data-index="0" alt="Profile for Sample ${label}" onclick="openModalFromCarousel(event, '${label}', 'panel')">
          <img src="https://github.com/madelynwillis3/research/releases/download/coastalplain-images-v1.0/perry_GA_point_${label}.jpg" class="carousel-image" data-index="1" alt="Field photo for Sample ${label}" onclick="openModalFromCarousel(event, '${label}', 'panel')">
        </div>
        <button class="carousel-arrow left" onclick="changeImage(event, '${label}', -1, 'panel')">‹</button>
        <button class="carousel-arrow right" onclick="changeImage(event, '${label}', 1, 'panel')">›</button>
        <div class="carousel-dots">
          <span class="dot active" onclick="goToImage(event, '${label}', 0, 'panel')"></span>
          <span class="dot" onclick="goToImage(event, '${label}', 1, 'panel')"></span>
        </div>
      </div>
      <p class="muted" style="margin-top:10px;">
        Tip: click the image to enlarge. Use arrows to view multiple images.
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

        const popupHTML = `
          <b>Sample ${label}</b><br>
          <div class="image-carousel" data-label="${label}" data-context="popup">
            <div class="carousel-images">
              <img src="${imgSrc}" class="carousel-image active" data-index="0" alt="Profile for Sample ${label}" onclick="openModalFromCarousel(event, '${label}', 'popup')">
              <img src="https://github.com/madelynwillis3/research/releases/download/coastalplain-images-v1.0/perry_GA_point_${label}.jpg" class="carousel-image" data-index="1" alt="Field photo for Sample ${label}" onclick="openModalFromCarousel(event, '${label}', 'popup')">
            </div>
            <button class="carousel-arrow left" onclick="changeImage(event, '${label}', -1, 'popup')">‹</button>
            <button class="carousel-arrow right" onclick="changeImage(event, '${label}', 1, 'popup')">›</button>
            <div class="carousel-dots">
              <span class="dot active" onclick="goToImage(event, '${label}', 0, 'popup')"></span>
              <span class="dot" onclick="goToImage(event, '${label}', 1, 'popup')"></span>
            </div>
          </div>
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
          setPanelContent(label, lat, lng, imgSrc);
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
