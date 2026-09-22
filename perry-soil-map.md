---
layout: bare
title: "UGA Grand Farm Taxonomic and Chemical Soil Map"
permalink: /perry-soil-map/
---

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css">
<style>
  :root {
    --panel-bg: rgba(255, 255, 255, 0.96);
    --panel-border: #dfe3e8;
    --text: #1d1d1f;
    --muted: #4b5563;
    --soft: #eef2f7;
    --accent: #2d6a4f;
    --accent-2: #6cae75;
    --shadow: 0 10px 22px rgba(17, 24, 39, 0.10);
  }

  /* The bare layout inherits Cayman's narrow .main-content, which was forcing
     the three columns to wrap (map ended up under the legend). Widen just this
     page and lay it out with an explicit grid instead of flex-basis maths. */
  .main-content,
  main#content {
    max-width: 1500px !important;
    width: 100% !important;
    box-sizing: border-box;
  }

  #mapWrap {
    display: grid;
    grid-template-columns: 250px minmax(0, 1fr) 380px;  /* legend | map | pedon panel */
    gap: 18px;
    align-items: start;
    margin: 1.2rem 0 2rem;
  }

  #legendColumn {
    min-width: 0;
  }

  #legend {
    background: var(--panel-bg);
    border: 1px solid var(--panel-border);
    border-radius: 12px;
    box-shadow: var(--shadow);
    padding: 14px 14px 10px;
    position: sticky;
    top: 12px;
  }

  .legend-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 10px;
    margin-bottom: 10px;
  }

  .legend-header h4 {
    margin: 0;
    font-size: 1.05rem;
    color: var(--text);
  }

  #legendContent {
    display: flex;
    flex-direction: column;
    gap: 12px;
    min-height: 80px;
  }

  .legend-section {
    border-top: 1px solid #e5e7eb;
    padding-top: 10px;
  }

  .legend-section:first-child {
    border-top: none;
    padding-top: 0;
  }

  .legend-section h5 {
    font-size: 0.78rem;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--muted);
    margin: 0 0 8px;
    font-weight: 700;
  }

  .legend-item {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.92rem;
    color: var(--text);
    margin: 5px 0;
    padding: 4px 6px;
    border-radius: 8px;
    cursor: pointer;
    transition: background 0.18s ease, opacity 0.18s ease;
  }

  .legend-item:hover {
    background: rgba(45, 106, 79, 0.05);
  }

  .legend-item.active {
    background: rgba(45, 106, 79, 0.07);
    box-shadow: inset 0 0 0 1px rgba(45, 106, 79, 0.12);
  }

  .legend-item.dimmed {
    opacity: 0.45;
  }

  .legend-swatch {
    width: 18px;
    height: 18px;
    border-radius: 4px;
    border: 1px solid rgba(0, 0, 0, 0.25);
    display: inline-block;
    flex: 0 0 18px;
  }

  .legend-swatch.line {
    width: 28px;
    height: 3px;
    border-radius: 2px;
    border: none;
    display: inline-block;
  }

  .legend-swatch.circle {
    border-radius: 50%;
    width: 12px;
    height: 12px;
    flex-basis: 12px;
  }

  .legend-note {
    margin: 10px 0 0;
    font-size: 0.78rem;
    color: var(--muted);
    line-height: 1.4;
  }

  #mapColumn {
    min-width: 0;             /* lets the grid column actually shrink */
  }

  /* vertical class ramp for the active raster */
  .ramp-row {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.82rem;
    line-height: 1.5;
    margin: 2px 0;
  }

  .ramp-chip {
    width: 20px;
    height: 13px;
    flex: 0 0 20px;
    border: 1px solid rgba(0, 0, 0, 0.25);
  }

  .legend-item.static {
    cursor: default;
  }

  .legend-item.static:hover {
    background: transparent;
  }

  #map {
    height: 560px;
    width: 100%;
    border-radius: 12px;
    border: 1px solid var(--panel-border);
    box-shadow: var(--shadow);
    background: #d8e8f3;
  }

  .layer-controls {
    margin-top: 12px;
    background: var(--panel-bg);
    border: 1px solid var(--panel-border);
    border-radius: 12px;
    box-shadow: var(--shadow);
    padding: 10px 12px;
  }

  .control-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    padding: 8px 0;
    border-bottom: 1px solid #edf2f7;
  }

  .control-row:last-child {
    border-bottom: none;
  }

  .control-row label {
    display: flex;
    align-items: center;
    gap: 8px;
    flex: 1;
    font-size: 0.92rem;
    color: var(--text);
  }

  .control-row input[type="checkbox"] {
    accent-color: var(--accent);
  }

  .control-row input[type="range"] {
    width: 120px;
    accent-color: var(--accent-2);
  }

  #infoPanel {
    min-width: 0;
    position: sticky;
    top: 12px;
    background: var(--panel-bg);
    border: 1px solid var(--panel-border);
    border-radius: 12px;
    box-shadow: var(--shadow);
    padding: 16px 16px 12px;
    min-height: 240px;
  }

  #infoPanel h3 {
    margin: 0 0 8px;
    font-size: 1.1rem;
  }

  #infoPanel .muted {
    color: var(--muted);
    margin: 0 0 12px;
    line-height: 1.5;
    font-size: 0.9rem;
  }

  #infoContent {
    font-size: 0.92rem;
    color: var(--text);
    line-height: 1.5;
  }

  #infoContent p {
    margin: 0 0 8px;
  }

  .info-row {
    margin-bottom: 8px;
  }

  .info-row strong {
    color: var(--text);
  }

  .thumbnail-list {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    margin-top: 10px;
  }

  /* map popup, 25% larger than before (220 -> 275 wide, 200 -> 250 tall) */
  .popup-img {
    display: block;
    margin-top: 6px;
    max-width: 275px;
    max-height: 250px;
    width: auto;
    border-radius: 6px;
    cursor: pointer;
    background: #f1f5f9;
  }

  .leaflet-popup-content { font-size: 0.95rem; }

  /* ---- side panel: sparse text + one carousel ---- */
  .pedon-id     { margin: 0 0 2px; font-weight: 700; font-size: 1rem; }
  .pedon-series { margin: 0 0 2px; font-size: 1.02rem; }
  .pedon-coords { margin: 0 0 12px; font-size: 0.8rem; color: var(--muted); font-variant-numeric: tabular-nums; }

  .image-carousel {
    position: relative;
    border-radius: 10px;
    overflow: hidden;
    background: #f1f5f9;
    border: 1px solid var(--panel-border);
  }

  /* tall enough for PORTRAIT profile photos, which were being squashed */
  .carousel-frame {
    position: relative;
    height: 440px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #0f172a;
  }

  .carousel-image {
    display: none;
    width: 100%;
    height: 440px;
    object-fit: contain;
    cursor: zoom-in;
  }

  .carousel-image.active { display: block; }
  .carousel-image.missing { display: none; }

  .carousel-arrow,
  .modal-arrow {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    border: none;
    background: rgba(15, 23, 42, 0.5);
    color: #fff;
    width: 30px;
    height: 44px;
    font-size: 1.4rem;
    line-height: 1;
    cursor: pointer;
    border-radius: 6px;
  }

  .carousel-arrow:hover,
  .modal-arrow:hover { background: rgba(15, 23, 42, 0.75); }
  .carousel-arrow.prev, .modal-arrow.prev { left: 6px; }
  .carousel-arrow.next, .modal-arrow.next { right: 6px; }

  .carousel-caption {
    text-align: center;
    font-size: 0.8rem;
    color: var(--muted);
    padding: 6px 0 2px;
    background: #fff;
  }

  .carousel-dots {
    display: flex;
    justify-content: center;
    gap: 6px;
    padding: 4px 0 8px;
    background: #fff;
  }

  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #cbd5e1;
    cursor: pointer;
  }

  .dot.active { background: var(--accent); }

  .modal-carousel { position: relative; }
  .modal-carousel-image { display: none; }
  .modal-carousel-image.active {
    display: block;
    max-width: min(88vw, 840px);
    max-height: 72vh;
    object-fit: contain;
    border-radius: 10px;
    background: #f8fafc;
  }

  .modal {
    position: fixed;
    inset: 0;
    background: rgba(15, 23, 42, 0.72);
    display: none;
    align-items: center;
    justify-content: center;
    z-index: 2000;
    padding: 24px;
  }

  .modal.open {
    display: flex;
  }

  .modal-card {
    position: relative;
    max-width: min(90vw, 900px);
    background: #fff;
    border-radius: 14px;
    padding: 16px 16px 12px;
    box-shadow: 0 18px 42px rgba(15, 23, 42, 0.35);
  }

  .modal-close {
    position: absolute;
    top: 8px;
    right: 10px;
    border: none;
    background: rgba(15, 23, 42, 0.08);
    border-radius: 50%;
    width: 32px;
    height: 32px;
    cursor: pointer;
    font-size: 1.25rem;
  }

  .modal img {
    display: block;
    max-width: min(88vw, 840px);
    max-height: 72vh;
    object-fit: contain;
    border-radius: 10px;
    background: #f8fafc;
  }

  .modal-caption {
    margin: 10px 0 0;
    font-size: 0.95rem;
    color: var(--text);
    text-align: center;
  }

  .close-note {
    font-size: 0.8rem;
    color: var(--muted);
    margin-top: 8px;
    text-align: center;
  }

  /* legend sections pop in when their layer is switched on */
  @keyframes legendPop {
    from { opacity: 0; transform: translateY(-6px) scale(0.98); }
    to   { opacity: 1; transform: none; }
  }

  .legend-section {
    animation: legendPop 0.22s ease-out both;
  }

  @media (prefers-reduced-motion: reduce) {
    .legend-section { animation: none; }
  }

  @media (max-width: 1240px) {
    #mapWrap {
      grid-template-columns: 230px minmax(0, 1fr);   /* pedon panel drops below */
    }
    #infoPanel {
      grid-column: 1 / -1;
      position: static;
    }
  }

  @media (max-width: 860px) {
    #mapWrap {
      grid-template-columns: 1fr;                    /* everything stacks */
    }
    #legend {
      position: static;
    }
  }
</style>

<h1>UGA Grand Farm Taxonomic and Chemical Soil Map</h1>
<p>This interactive map shows pedon observations at UGA GrandFarm in Perry, Georgia, together with ordinary-kriged predictions of soil chemistry and texture. Use the layer controls to compare surface soil measurements and the mapped soil series.</p>

<div id="mapWrap">
  <aside id="legendColumn">
    <div id="legend" aria-label="Map legend">
      <div class="legend-header">
        <h4 id="legendTitle">Legend</h4>
      </div>
      <div id="legendContent"></div>
      <p id="unitNote" class="legend-note"></p>
    </div>
  </aside>

  <div id="mapColumn">
    <div id="map"></div>
    <div id="layerControls" class="layer-controls" aria-label="Map layer controls"></div>
  </div>

  <aside id="infoPanel" aria-live="polite">
    <h3>Pedon Points</h3>
    <div id="infoContent">
      <p class="muted">Click a pedon point to view its soil profile and field photo.</p>
    </div>
  </aside>
</div>

<div id="imgModal" class="modal" aria-modal="true" role="dialog" onclick="closeModal(event)">
  <div class="modal-card" onclick="event.stopPropagation()">
    <button type="button" class="modal-close" aria-label="Close gallery" onclick="closeModal()">×</button>
    <div class="modal-carousel" id="modalCarousel"></div>
    <button type="button" class="modal-arrow prev" onclick="changeModalImage(-1)" aria-label="Previous photo">&#8249;</button>
    <button type="button" class="modal-arrow next" onclick="changeModalImage(1)" aria-label="Next photo">&#8250;</button>
    <div class="carousel-dots" id="modalCarouselDots"></div>
    <div id="modalCaption" class="modal-caption"></div>
    <div class="close-note">Click outside or press ESC to close</div>
  </div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
<script>
  const base = '{{ "/assets/" | relative_url }}';
  const imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';
  const fieldPhotoBase = 'https://github.com/madelynwillis3/research/releases/download/coastalplain-images-v1.0/perry_GA_point_';
  const excluded = ['39', '50', '51', '58', '59', '60', '65', '74'];

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

  const defaultMarkerStyle = { color: '#000', weight: 1.5, fillOpacity: 1, opacity: 1, radius: 7 };
  const dimmedMarkerStyle = { fillOpacity: 0.15, opacity: 0.25, radius: 5 };
  const highlightedMarkerStyle = { fillOpacity: 1, opacity: 1, radius: 8, weight: 2 };
  const seriesLinks = Object.fromEntries(
    Object.keys(seriesColors)
      .filter((s) => s !== 'Disturbed')
      .map((s) => [s, `https://casoilresource.lawr.ucdavis.edu/sde/?series=${encodeURIComponent(s.toUpperCase())}#osd`])
  );

  const esc = (v) => String(v ?? '').replace(/[&<>"']/g, (ch) => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[ch]));

  const map = L.map('map', { scrollWheelZoom: true }).setView([32.43, -83.73], 14);
  map.createPane('soilRaster').style.zIndex = 200;
  map.createPane('soilVector').style.zIndex = 400;
  map.createPane('pedons').style.zIndex = 650;   // above every overlay
  map.getPane('pedons').style.pointerEvents = 'auto';

  L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
    attribution: 'Tiles © Esri'
  }).addTo(map);

  const rasterLayers = {};
  const pedonsLayer = L.layerGroup([], { pane: 'pedons' }).addTo(map);
  const soilSeriesLayer = L.geoJSON(null, {
    pane: 'soilVector',
    style: (feature) => ({ color: '#333', weight: 1, fillColor: feature.properties.fill || '#bbb', fillOpacity: 0.72 })
  }).addTo(map);
  const matchLayer = L.geoJSON(null, {
    pane: 'soilVector',
    style: (feature) => ({ color: '#666', weight: 0.2, fillColor: feature.properties.fill || '#d8d8d8', fillOpacity: 0.68 })
  }).addTo(map);
  const boundaryLayer = L.geoJSON(null, {
    pane: 'soilVector',
    // dashed, so it reads as distinct from the solid site border
    style: () => ({ color: '#4d4d4d', weight: 1.5, opacity: 0.9, fillOpacity: 0, dashArray: '6,4' })
  }).addTo(map);
  const borderLayer = L.geoJSON(null, {
    pane: 'soilVector',
    style: () => ({ color: '#1f2937', weight: 2.5, opacity: 0.9, fillOpacity: 0 })
  }).addTo(map);

  const state = {
    manifest: null,
    activeSeries: null,
    activeRaster: null,          // at most ONE raster at a time
    rasterOpacity: 0.8,
    soilSeriesVisible: false,    // opens looking like the pedon map
    matchVisible: false,
    boundaryVisible: false,      // SSURGO unit lines start OFF, like the rest
    pedonsVisible: true,
    seriesPolyCounts: {},        // counted from soil_series.geojson
    matchCounts: {}              // counted from ssurgo_matchup.geojson
  };

  let markerEntries = [];

  function seriesHTML(series) {
    return seriesLinks[series] ? `<a class="series-link" target="_blank" rel="noopener noreferrer" href="${seriesLinks[series]}">${esc(series)}</a>` : esc(series);
  }

  function renderLegendSections() {
    const box = document.getElementById('legendContent');
    if (!box) return;

    const sections = [];

    // ---- 1. active raster: one row per CLASS, high value at top ------------
    // breaks has one more entry than colors: class i spans breaks[i]..breaks[i+1]
    if (state.activeRaster && state.manifest) {
      const r = state.manifest.rasters.find((layer) => layer.id === state.activeRaster);
      if (r) {
        const fmt = (v) => {
          const a = Math.abs(v);
          return a >= 100 ? v.toFixed(0) : a >= 10 ? v.toFixed(1) : v.toFixed(2);
        };
        const rows = [];
        for (let i = r.colors.length - 1; i >= 0; i--) {
          rows.push(`<div class="ramp-row">
            <span class="ramp-chip" style="background:${r.colors[i]}"></span>
            <span>${fmt(r.breaks[i])} &ndash; ${fmt(r.breaks[i + 1])}</span>
          </div>`);
        }
        sections.push(`
          <div class="legend-section">
            <h5>${esc(r.title)}${r.unit ? ' (' + esc(r.unit) + ')' : ''}</h5>
            ${rows.join('')}
          </div>
        `);
      }
    }

    // ---- 2. mapped soil series: counted from the POLYGONS, not the points --
    if (state.soilSeriesVisible && Object.keys(state.seriesPolyCounts).length) {
      const pal = (state.manifest && state.manifest.vectors.soil_series.palette) || {};
      const counts = state.seriesPolyCounts;
      const keys = Object.keys(counts).sort((a, b) =>
        (a === 'HTM') - (b === 'HTM') || counts[b] - counts[a] || a.localeCompare(b));
      const rows = keys.map((k) => `
        <div class="legend-item static">
          <span class="legend-swatch" style="background:${pal[k] || '#BDBDBD'}"></span>
          <span>${esc(k)} (${counts[k]})</span>
        </div>`);
      sections.push(`
        <div class="legend-section">
          <h5>Mapped soil series</h5>
          ${rows.join('')}
        </div>
      `);
    }

    // ---- 3. SSURGO agreement: two real classes, colours from the manifest --
    if (state.matchVisible && Object.keys(state.matchCounts).length) {
      const pal = (state.manifest && state.manifest.vectors.ssurgo_matchup.palette) || {};
      const c = state.matchCounts;
      const total = (c.Agree || 0) + (c.Disagree || 0);
      const rows = ['Agree', 'Disagree']
        .filter((k) => c[k] != null)
        .map((k) => `
          <div class="legend-item static">
            <span class="legend-swatch" style="background:${pal[k] || '#999'}"></span>
            <span>${k} (${c[k]})</span>
          </div>`);
      sections.push(`
        <div class="legend-section">
          <h5>SSURGO agreement</h5>
          ${rows.join('')}
          <p class="legend-note" style="margin-top:6px;">${c.Agree || 0} of ${total} polygons agree. Disagree includes polygons with no SSURGO overlap.</p>
        </div>
      `);
    }

    // ---- 4. boundaries: dashed vs solid must actually LOOK different ------
    // Site border is ALWAYS drawn and is a separate thing from the SSURGO
    // unit lines -- never lump the two together.
    {
      const rows = [
        '<div class="legend-item static"><span class="legend-swatch line" style="background:#1f2937;"></span><span>Site border</span></div>'
      ];
      if (state.boundaryVisible) {
        rows.unshift('<div class="legend-item static"><span class="legend-swatch line" style="background:repeating-linear-gradient(90deg,#4d4d4d 0 6px,transparent 6px 10px);"></span><span>SSURGO map unit</span></div>');
      }
      sections.push(`
        <div class="legend-section">
          <h5>Boundaries</h5>
          ${rows.join('')}
        </div>
      `);
    }

    // ---- 5. pedon points: the clickable series list ------------------------
    if (state.pedonsVisible) {
      const counts = {};
      markerEntries.forEach((entry) => {
        const s = entry.series || 'Unknown';
        counts[s] = (counts[s] || 0) + 1;
      });
      const keys = Object.keys(counts).sort((a, b) =>
        counts[b] - counts[a] || a.localeCompare(b));
      const rows = keys.map((series) => {
        const isActive = state.activeSeries === series;
        return `
          <div class="legend-item ${isActive ? 'active' : ''} ${state.activeSeries && !isActive ? 'dimmed' : ''}" data-series="${esc(series)}" onclick="selectSeries('${series.replace(/'/g, "\\'")}')">
            <span class="legend-swatch circle" style="background:${seriesColors[series] || '#777'}"></span>
            <span>${seriesHTML(series)} (${counts[series]})</span>
          </div>`;
      });
      sections.push(`
        <div class="legend-section">
          <h5>Pedon points (${markerEntries.length})</h5>
          ${rows.join('') || '<div class="legend-item static"><span>Loading&hellip;</span></div>'}
          ${state.activeSeries ? '<div class="legend-item" onclick="selectSeries(null)" style="font-size:.82rem;color:var(--accent);">&#8630; Show all series</div>' : ''}
        </div>
      `);
    }

    box.innerHTML = sections.join('') ||
      '<p class="legend-note">No layers visible. Turn one on to see its legend.</p>';

    // unit note only matters while a chemistry surface is showing
    const r = state.manifest && state.activeRaster
      ? state.manifest.rasters.find((layer) => layer.id === state.activeRaster) : null;
    // only the converted analytes, not pH or LBC (both are group "Chemistry"
    // but neither is converted)
    document.getElementById('unitNote').textContent =
      (r && r.unit === 'lbs/ac' && state.manifest) ? (state.manifest.unit_note || '') : '';
  }

  function selectSeries(series) {
    state.activeSeries = (series === null || state.activeSeries === series) ? null : series;
    // restyle the markers themselves, not just the legend rows
    pedonsLayer.getLayers().forEach((layer) => {
      if (layer._entry && layer.setStyle) layer.setStyle(createMarkerStyle(layer._entry));
    });
    updateSeriesHighlight();
    renderLegendSections();
  }

  function updateSeriesHighlight() {
    const all = document.querySelectorAll('.legend-item[data-series]');
    all.forEach((item) => {
      const series = item.getAttribute('data-series');
      const isActive = state.activeSeries && series === state.activeSeries;
      const isDimmed = !!state.activeSeries && !isActive;
      item.classList.toggle('active', isActive);
      item.classList.toggle('dimmed', isDimmed);
    });
  }

  // ---------------------------------------------------------------------
  // Side panel: a two-image carousel, as on the original coastalplain map.
  // Kept deliberately sparse -- point, series, coordinates, photos.
  // ---------------------------------------------------------------------
  const CAPTIONS = ['Soil profile', 'Field photo'];
  let panelIndex = 0;

  function carouselHTML(images, label) {
    const slides = images.map((src, i) => `
      <img class="carousel-image ${i === 0 ? 'active' : ''}" src="${src}" loading="lazy"
           alt="${esc(label)} ${esc(CAPTIONS[i] || 'photo')}"
           onclick="openModal(${i})" onerror="this.classList.add('missing')">`).join('');
    const dots = images.map((_, i) =>
      `<span class="dot ${i === 0 ? 'active' : ''}" onclick="goToPanelImage(${i})" role="button" aria-label="Photo ${i + 1}"></span>`).join('');
    return `
      <div class="image-carousel">
        <div class="carousel-frame">${slides}</div>
        ${images.length > 1 ? `
          <button type="button" class="carousel-arrow prev" onclick="changePanelImage(-1)" aria-label="Previous photo">&#8249;</button>
          <button type="button" class="carousel-arrow next" onclick="changePanelImage(1)" aria-label="Next photo">&#8250;</button>` : ''}
        <div class="carousel-caption" id="panelCaption">${esc(CAPTIONS[0])}</div>
        ${images.length > 1 ? `<div class="carousel-dots">${dots}</div>` : ''}
      </div>`;
  }

  function showPanelImage(i) {
    const frame = document.querySelector('#infoContent .image-carousel');
    if (!frame) return;
    const imgs = frame.querySelectorAll('.carousel-image');
    const dots = frame.querySelectorAll('.dot');
    if (!imgs.length) return;
    panelIndex = (i + imgs.length) % imgs.length;
    imgs.forEach((im, k) => im.classList.toggle('active', k === panelIndex));
    dots.forEach((d, k) => d.classList.toggle('active', k === panelIndex));
    const cap = document.getElementById('panelCaption');
    if (cap) cap.textContent = CAPTIONS[panelIndex] || `Photo ${panelIndex + 1}`;
  }
  function changePanelImage(dir) { showPanelImage(panelIndex + dir); }
  function goToPanelImage(i) { showPanelImage(i); }

  function infoTemplate(entry) {
    const label = entry.label || entry.point || 'sample';
    const lat = Number(entry.lat), lng = Number(entry.lng);
    return `
      <p class="pedon-id">Point ${esc(label)}</p>
      <p class="pedon-series">${seriesHTML(entry.series || 'Unknown')}</p>
      <p class="pedon-coords">${lat.toFixed(5)}, ${lng.toFixed(5)}</p>
      ${entry.images && entry.images.length ? carouselHTML(entry.images, label) : ''}`;
  }

  function setInfoPanel(entry) {
    const info = document.getElementById('infoContent');
    if (!entry) {
      info.innerHTML = '<p class="muted">Click a pedon point to view its soil profile and field photo.</p>';
      return;
    }
    currentEntry = entry;
    panelIndex = 0;
    info.innerHTML = infoTemplate(entry);
  }

  // ---------------------------------------------------------------------
  // Modal gallery: arrows, dots, keyboard
  // ---------------------------------------------------------------------
  let currentEntry = null;
  let modalIndex = 0;

  function openModal(startIndex) {
    if (!currentEntry || !currentEntry.images || !currentEntry.images.length) return;
    modalIndex = startIndex || 0;
    const wrap = document.getElementById('modalCarousel');
    wrap.innerHTML = currentEntry.images.map((src, i) => `
      <img class="modal-carousel-image ${i === modalIndex ? 'active' : ''}" src="${src}"
           alt="${esc(currentEntry.label)} ${esc(CAPTIONS[i] || 'photo')}">`).join('');
    document.getElementById('modalCarouselDots').innerHTML = currentEntry.images.length > 1
      ? currentEntry.images.map((_, i) =>
          `<span class="dot ${i === modalIndex ? 'active' : ''}" onclick="goToModalImage(${i})" role="button" aria-label="Photo ${i + 1}"></span>`).join('')
      : '';
    document.querySelectorAll('.modal-arrow').forEach((a) => {
      a.style.display = currentEntry.images.length > 1 ? '' : 'none';
    });
    syncModal();
    document.getElementById('imgModal').classList.add('open');
  }

  function syncModal() {
    const imgs = document.querySelectorAll('#modalCarousel .modal-carousel-image');
    const dots = document.querySelectorAll('#modalCarouselDots .dot');
    imgs.forEach((im, k) => im.classList.toggle('active', k === modalIndex));
    dots.forEach((d, k) => d.classList.toggle('active', k === modalIndex));
    const cap = document.getElementById('modalCaption');
    if (cap && currentEntry) {
      cap.textContent = `Point ${currentEntry.label} — ${CAPTIONS[modalIndex] || 'photo'}`;
    }
  }

  function changeModalImage(dir) {
    if (!currentEntry) return;
    const n = currentEntry.images.length;
    modalIndex = (modalIndex + dir + n) % n;
    syncModal();
  }
  function goToModalImage(i) { modalIndex = i; syncModal(); }

  function closeModal(event) {
    if (event && event.target !== event.currentTarget && event.target !== document.getElementById('imgModal')) return;
    document.getElementById('imgModal').classList.remove('open');
  }

  document.addEventListener('keydown', (event) => {
    const open = document.getElementById('imgModal').classList.contains('open');
    if (event.key === 'Escape') document.getElementById('imgModal').classList.remove('open');
    if (!open) return;
    if (event.key === 'ArrowLeft') changeModalImage(-1);
    if (event.key === 'ArrowRight') changeModalImage(1);
  });

  function createMarkerStyle(entry) {
    // pane MUST be set on the marker itself. L.layerGroup does not pass its
    // pane down to children, so without this the circleMarkers were rendered
    // in the default overlayPane alongside the polygons and disappeared
    // underneath the soil series / SSURGO fills.
    // stroke stays BLACK, the series colour is the FILL -- same as the
    // original coastalplain map. Setting `color` to the series colour removed
    // the black outline.
    const fill = seriesColors[entry.series] || '#808080';
    const baseStyle = { ...defaultMarkerStyle, pane: 'pedons', fillColor: fill, color: '#000' };
    if (state.activeSeries && entry.series !== state.activeSeries) {
      return { ...baseStyle, ...dimmedMarkerStyle };
    }
    if (state.activeSeries && entry.series === state.activeSeries) {
      return { ...baseStyle, ...highlightedMarkerStyle };
    }
    return baseStyle;
  }

  function addPedonMarkers(data) {
    pedonsLayer.clearLayers();
    markerEntries = [];
    data.forEach((row) => {
      // FIX: perry_FP_samples_80.csv has columns "Point ID", "Series", "x", "y".
      // The previous lookups (label/Point/Sample, Latitude/Longitude) matched
      // nothing, every row bailed out at the !lat check, and zero markers drew.
      const pointId = row['Point ID'] || row['PointID'] || row['Point'] || row['label'] || '';
      const cleanId = String(pointId).trim();
      if (!cleanId || excluded.includes(cleanId)) return;

      const lat = Number(row.y != null && row.y !== '' ? row.y : (row.Latitude || row.latitude || row.lat));
      const lng = Number(row.x != null && row.x !== '' ? row.x : (row.Longitude || row.longitude || row.lng));
      if (!Number.isFinite(lat) || !Number.isFinite(lng)) return;

      const series = seriesByPoint[cleanId] || seriesByPoint[cleanId.replace(/^0+/, '')] || 'Unknown';
      const entry = {
        point: cleanId,
        label: cleanId,
        lat,
        lng,
        series,
        depth: row.Depth || row.depth || 'n/a',
        notes: row.Notes || row.notes || '',
        images: [
          `${imgBase}${cleanId}.jpg`,
          `${fieldPhotoBase}${cleanId}.jpg`
        ].filter((src) => src)
      };

      markerEntries.push(entry);
      // lookup so a popup image can hand the right entry to the gallery
      window.__entries = window.__entries || {};
      window.__entries[cleanId] = entry;
      const marker = L.circleMarker([lat, lng], createMarkerStyle(entry));
      marker._entry = entry;           // so selectSeries() can restyle it later
      // popup mirrors the original coastalplain map: series link, point id,
      // and the profile photo, clickable to open the full gallery
      marker.bindPopup(`
        <b>${seriesHTML(series)}</b><br>
        <span>Point ${esc(cleanId)}</span><br>
        <img src="${entry.images[0]}" class="popup-img" loading="lazy"
             alt="Profile image for ${esc(series)} (Point ${esc(cleanId)})"
             onclick="setInfoPanel(window.__entries['${esc(cleanId)}']); openModal(0)"
             onerror="this.style.display='none'">
      `, { maxWidth: 325 });
      marker.on('click', () => setInfoPanel(entry));
      marker.addTo(pedonsLayer);
    });

    renderLegendSections();
    // leave the panel on its prompt until the user actually clicks a point
    setInfoPanel(null);
  }

  async function loadMap() {
    const manifest = await fetch(base + 'maplayers/manifest.json').then((response) => response.json());
    state.manifest = manifest;
    document.getElementById('unitNote').textContent = manifest.unit_note || '';

    const allLayers = manifest.rasters || [];
    const controls = document.getElementById('layerControls');
    controls.innerHTML = '';

    // ------------------------------------------------------------------
    // Surfaces are RADIO, not checkbox. Previously all 11 overlays were
    // added to the map at once and every box was checked, so the surfaces
    // stacked on top of each other and the legend described whichever one
    // happened to be last in the manifest (clay).
    // ------------------------------------------------------------------
    const surfaceWrap = document.createElement('div');
    surfaceWrap.className = 'control-row';
    surfaceWrap.innerHTML = '<strong style="font-size:.82rem;text-transform:uppercase;letter-spacing:.06em;">Prediction surface</strong>';
    controls.appendChild(surfaceWrap);

    function showRaster(id) {
      Object.keys(rasterLayers).forEach((key) => rasterLayers[key].layer.remove());
      state.activeRaster = id;
      if (id && rasterLayers[id]) {
        rasterLayers[id].layer.setOpacity(state.rasterOpacity);
        rasterLayers[id].layer.addTo(map);
      }
      renderLegendSections();
    }

    // "None" first, then one radio per surface, grouped
    const noneRow = document.createElement('div');
    noneRow.className = 'control-row';
    noneRow.innerHTML = '<label><input type="radio" name="surface" value="" checked /> <span>None</span></label>';
    noneRow.querySelector('input').addEventListener('change', () => showRaster(null));
    controls.appendChild(noneRow);

    let lastGroup = null;
    allLayers.forEach((r) => {
      const bounds = [[r.bounds.south, r.bounds.west], [r.bounds.north, r.bounds.east]];
      const overlay = L.imageOverlay(base + r.png, bounds, {
        pane: 'soilRaster', opacity: state.rasterOpacity, interactive: false, className: 'soil-raster'
      });
      rasterLayers[r.id] = { layer: overlay, title: r.title, group: r.group };
      // NOT added to the map here -- nothing shows until a radio is picked

      if (r.group && r.group !== lastGroup) {
        const h = document.createElement('div');
        h.className = 'control-row';
        h.innerHTML = `<span style="font-size:.74rem;text-transform:uppercase;letter-spacing:.06em;color:#6b7280;">${esc(r.group)}</span>`;
        controls.appendChild(h);
        lastGroup = r.group;
      }

      const row = document.createElement('div');
      row.className = 'control-row';
      row.innerHTML = `<label><input type="radio" name="surface" value="${r.id}" /> <span>${esc(r.title)}</span></label>`;
      row.querySelector('input').addEventListener('change', () => showRaster(r.id));
      controls.appendChild(row);
    });

    // one shared opacity slider, applied to whichever surface is active
    const opRow = document.createElement('div');
    opRow.className = 'control-row';
    opRow.innerHTML = '<label style="flex:1;">Surface opacity <input type="range" min="0" max="1" step="0.05" value="0.8" id="opacitySlider" aria-label="Surface opacity" /></label>';
    opRow.querySelector('input').addEventListener('input', (event) => {
      state.rasterOpacity = Number(event.target.value);
      if (state.activeRaster && rasterLayers[state.activeRaster]) {
        rasterLayers[state.activeRaster].layer.setOpacity(state.rasterOpacity);
      }
    });
    controls.appendChild(opRow);

    // FIX: these toggled only the legend, never the actual Leaflet layers.
    const seriesToggle = document.createElement('div');
    seriesToggle.className = 'control-row';
    seriesToggle.innerHTML = `
      <label><input type="checkbox" id="seriesToggle" /> <span>Mapped soil series</span></label>
    `;
    seriesToggle.querySelector('input').addEventListener('change', (event) => {
      state.soilSeriesVisible = event.target.checked;
      if (event.target.checked) soilSeriesLayer.addTo(map); else soilSeriesLayer.remove();
      renderLegendSections();
    });
    controls.appendChild(seriesToggle);

    const matchToggle = document.createElement('div');
    matchToggle.className = 'control-row';
    matchToggle.innerHTML = `
      <label><input type="checkbox" id="matchToggle" /> <span>SSURGO agreement</span></label>
    `;
    matchToggle.querySelector('input').addEventListener('change', (event) => {
      state.matchVisible = event.target.checked;
      if (event.target.checked) matchLayer.addTo(map); else matchLayer.remove();
      renderLegendSections();
    });
    controls.appendChild(matchToggle);

    const boundaryToggle = document.createElement('div');
    boundaryToggle.className = 'control-row';
    boundaryToggle.innerHTML = `
      <label><input type="checkbox" id="boundaryToggle" /> <span>SSURGO unit boundaries</span></label>
    `;
    // NOTE: this toggles the SSURGO unit boundaries ONLY. The site border is a
    // different thing entirely and is always on -- it is never removed and has
    // no checkbox.
    boundaryToggle.querySelector('input').addEventListener('change', (event) => {
      state.boundaryVisible = event.target.checked;
      if (event.target.checked) boundaryLayer.addTo(map); else boundaryLayer.remove();
      renderLegendSections();
    });
    controls.appendChild(boundaryToggle);

    const pedonToggle = document.createElement('div');
    pedonToggle.className = 'control-row';
    pedonToggle.innerHTML = `
      <label><input type="checkbox" id="pedonToggle" checked /> <span>Pedon points</span></label>
    `;
    pedonToggle.querySelector('input').addEventListener('change', (event) => {
      state.pedonsVisible = event.target.checked;
      if (event.target.checked) pedonsLayer.addTo(map); else pedonsLayer.remove();
      renderLegendSections();
    });
    controls.appendChild(pedonToggle);

    // All optional vectors start OFF. Only the site border stays on, and it
    // has no toggle at all.
    soilSeriesLayer.remove();
    matchLayer.remove();
    boundaryLayer.remove();

    fetch(base + 'maplayers/soil_series.geojson')
      .then((response) => response.json())
      .then((geojson) => {
        soilSeriesLayer.addData(geojson);
        const counts = {};
        geojson.features.forEach((f) => {
          const k = (f.properties && f.properties.series) || 'Unknown';
          counts[k] = (counts[k] || 0) + 1;
        });
        state.seriesPolyCounts = counts;
        renderLegendSections();
      });

    fetch(base + 'maplayers/ssurgo_matchup.geojson')
      .then((response) => response.json())
      .then((geojson) => {
        matchLayer.addData(geojson);
        const counts = {};
        geojson.features.forEach((f) => {
          const k = (f.properties && f.properties.match) || 'Unknown';
          counts[k] = (counts[k] || 0) + 1;
        });
        state.matchCounts = counts;
        renderLegendSections();
      });

    fetch(base + 'maplayers/ssurgo_units.geojson')
      .then((response) => response.json())
      .then((geojson) => {
        boundaryLayer.addData(geojson);
      });

    fetch(base + 'maplayers/site_border.geojson')
      .then((response) => response.json())
      .then((geojson) => {
        borderLayer.addData(geojson);
        // frame the site instead of sitting at a hardcoded zoom 14
        try { map.fitBounds(borderLayer.getBounds(), { padding: [24, 24] }); } catch (e) {}
      });

    Papa.parse('{{ "/assets/data/perry_FP_samples_80.csv" | relative_url }}', {
      download: true,
      header: true,
      complete: (result) => addPedonMarkers(result.data)
    });

    renderLegendSections();
  }

  loadMap();
</script>
