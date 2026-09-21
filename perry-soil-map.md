---
layout: bare
title: "Perry Soil Map"
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

  #mapWrap {
    display: flex;
    gap: 18px;
    align-items: flex-start;
    margin: 1.2rem 0 2rem;
  }

  #legendColumn {
    flex: 0 0 260px;
    width: 260px;
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
    flex: 1 1 auto;
    min-width: 320px;
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
    flex: 0 0 300px;
    width: 300px;
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

  .thumb {
    width: 68px;
    height: 68px;
    object-fit: cover;
    border-radius: 8px;
    border: 1px solid var(--panel-border);
    cursor: pointer;
    background: #f1f5f9;
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

  @media (max-width: 980px) {
    #mapWrap {
      flex-direction: column;
    }

    #legendColumn,
    #infoPanel {
      width: 100%;
      flex-basis: auto;
    }

    #legend {
      position: static;
    }
  }
</style>

<h1>Perry Soil Map</h1>
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
    <h3>Welcome</h3>
    <p class="muted">Click a pedon point to inspect the sample and photo set.</p>
    <div id="infoContent"></div>
  </aside>
</div>

<div id="imgModal" class="modal" aria-modal="true" role="dialog" onclick="closeModal(event)">
  <div class="modal-card" onclick="event.stopPropagation()">
    <button type="button" class="modal-close" aria-label="Close gallery" onclick="closeModal()">×</button>
    <img id="modalImage" src="" alt="Sample photo">
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
    P2: 'Thursa', P3: 'Faceville', P4: 'Faceville', '1': 'Orangeburg', '2': 'Faceville', '3': 'Faceville', '4': 'Faceville', '5': 'Wagram', '6': 'Norfolk', '7': 'Bonneau', '8': 'Wagram', '9': 'Orangeburg', '10': 'Wagram', '11': 'Faceville', '12': 'Grady', '13': 'Troup', '14': 'Lucy', '15': 'Faceville', '16': 'Norfolk', '17': 'Faceville', '18': 'Bonneau', '19': 'Wagram', '20': 'Blanton'
  };

  const seriesColors = {
    Faceville: '#d73027', Orangeburg: '#c94c4c', Lucy: '#e76f51', Troup: '#f4a6a6', Greenville: '#8b0000', Leefield: '#d8c3a5', Blanton: '#8a7f73', Norfolk: '#f28c28', Dothan: '#d4a017', Johns: '#c2a84d', Wagram: '#a6d96a', Bonneau: '#65a765', Thursa: '#3b7f61', Grady: '#6a5acd', Disturbed: '#7a7a7a'
  };

  const defaultMarkerStyle = { color: '#000', weight: 1.5, fillOpacity: 0.85, opacity: 1, radius: 6 };
  const dimmedMarkerStyle = { fillOpacity: 0.15, opacity: 0.25, radius: 5 };
  const highlightedMarkerStyle = { fillOpacity: 1, opacity: 1, radius: 7 };
  const seriesLinks = Object.fromEntries(
    Object.keys(seriesColors)
      .filter((s) => s !== 'Disturbed')
      .map((s) => [s, `https://casoilresource.lawr.ucdavis.edu/sde/?series=${encodeURIComponent(s.toUpperCase())}#osd`])
  );

  const esc = (v) => String(v ?? '').replace(/[&<>"']/g, (ch) => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[ch]));

  const map = L.map('map', { scrollWheelZoom: true }).setView([32.43, -83.73], 14);
  map.createPane('soilRaster').style.zIndex = 200;
  map.createPane('soilVector').style.zIndex = 400;
  map.createPane('pedons').style.zIndex = 600;

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
    style: () => ({ color: '#4d4d4d', weight: 1.5, opacity: 0.9, fillOpacity: 0 })
  }).addTo(map);
  const borderLayer = L.geoJSON(null, {
    pane: 'soilVector',
    style: () => ({ color: '#1f2937', weight: 2.5, opacity: 0.9, fillOpacity: 0 })
  }).addTo(map);

  const state = {
    manifest: null,
    activeSeries: null,
    activeRaster: null,
    visibleRasterIds: new Set(),
    soilSeriesVisible: true,
    matchVisible: true,
    boundaryVisible: true,
    pedonsVisible: true,
    siteBorderVisible: true
  };

  let markerEntries = [];
  let modalImage = '';

  function seriesHTML(series) {
    return seriesLinks[series] ? `<a class="series-link" target="_blank" rel="noopener noreferrer" href="${seriesLinks[series]}">${esc(series)}</a>` : esc(series);
  }

  function renderLegendSections() {
    const box = document.getElementById('legendContent');
    if (!box) return;

    const sections = [];

    if (state.activeRaster && state.visibleRasterIds.has(state.activeRaster)) {
      const r = state.manifest.rasters.find((layer) => layer.id === state.activeRaster);
      if (r) {
        const items = r.breaks.map((breakValue, idx) => {
          const color = r.colors[Math.min(idx, r.colors.length - 1)];
          return `<div class="legend-item" data-series="__raster__${r.id}"><span class="legend-swatch" style="background:${color};"></span><span>${breakValue}</span></div>`;
        });
        sections.push(`
          <div class="legend-section">
            <h5>${esc(r.title)}</h5>
            <div class="legend-item"><span class="legend-swatch" style="background:linear-gradient(90deg, ${r.colors.join(',')});"></span><span>${esc(r.unit || 'value')}</span></div>
            <div style="display:flex;flex-direction:column;gap:4px;margin-top:8px;">${items.join('')}</div>
          </div>
        `);
      }
    }

    if (state.soilSeriesVisible) {
      const counts = {};
      markerEntries.forEach((entry) => {
        const s = entry.series || 'Unknown';
        counts[s] = (counts[s] || 0) + 1;
      });
      const entries = Object.keys(counts).sort().map((series) => {
        const isActive = state.activeSeries === series;
        return `
          <div class="legend-item ${isActive ? 'active' : ''} ${state.activeSeries && !isActive ? 'dimmed' : ''}" data-series="${esc(series)}" onclick="selectSeries('${series.replace(/'/g, "\\'")}')">
            <span class="legend-swatch circle" style="background:${seriesColors[series] || '#777'}"></span>
            <span>${seriesHTML(series)} (${counts[series]})</span>
          </div>
        `;
      });
      sections.push(`
        <div class="legend-section">
          <h5>Mapped soil series</h5>
          ${entries.join('') || '<div class="legend-item"><span class="legend-swatch circle" style="background:#777"></span><span>None</span></div>'}
        </div>
      `);
    }

    if (state.matchVisible) {
      sections.push(`
        <div class="legend-section">
          <h5>SSURGO agreement</h5>
          <div class="legend-item"><span class="legend-swatch" style="background:#a3e635"></span><span>Matched</span></div>
          <div class="legend-item"><span class="legend-swatch" style="background:#fcd34d"></span><span>Partial/uncertain</span></div>
          <div class="legend-item"><span class="legend-swatch" style="background:#f87171"></span><span>Mismatch</span></div>
        </div>
      `);
    }

    if (state.boundaryVisible) {
      sections.push(`
        <div class="legend-section">
          <h5>Boundary lines</h5>
          <div class="legend-item"><span class="legend-swatch line" style="background:#4d4d4d;"></span><span>SSURGO units</span></div>
          <div class="legend-item"><span class="legend-swatch line" style="background:#1f2937;"></span><span>Site border</span></div>
        </div>
      `);
    }

    if (state.pedonsVisible) {
      sections.push(`
        <div class="legend-section">
          <h5>Pedon points</h5>
          <div class="legend-item"><span class="legend-swatch circle" style="background:#111827;"></span><span>Sample points</span></div>
        </div>
      `);
    }

    box.innerHTML = sections.join('');
    const note = state.manifest ? state.manifest.unit_note || '' : '';
    document.getElementById('unitNote').textContent = note;
  }

  function selectSeries(series) {
    state.activeSeries = (state.activeSeries === series) ? null : series;
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

  function infoTemplate(entry) {
    const label = entry.label || entry.point || 'sample';
    const html = [];
    html.push(`<p><strong>Pedon:</strong> ${esc(label)}</p>`);
    html.push(`<p><strong>Series:</strong> ${seriesHTML(entry.series || 'Unknown')}</p>`);
    html.push(`<p><strong>Location:</strong> ${esc(entry.lat || '')}, ${esc(entry.lng || '')}</p>`);
    html.push(`<p><strong>Depth:</strong> ${esc(entry.depth || 'n/a')}</p>`);
    if (entry.notes) html.push(`<p><strong>Notes:</strong> ${esc(entry.notes)}</p>`);
    const thumbs = [];
    if (entry.images && entry.images.length) {
      entry.images.forEach((src) => {
        thumbs.push(`<img class="thumb" src="${src}" alt="${esc(label)} photo" onclick="openModal('${src}', '${label}')">`);
      });
    }
    if (thumbs.length) {
      html.push(`<div class="thumbnail-list">${thumbs.join('')}</div>`);
    }
    return html.join('');
  }

  function setInfoPanel(entry) {
    const info = document.getElementById('infoContent');
    if (!entry) {
      info.innerHTML = '<p>Select a pedon point to view sample details.</p>';
      return;
    }
    info.innerHTML = infoTemplate(entry);
  }

  function openModal(src, label) {
    modalImage = src;
    document.getElementById('modalImage').src = src;
    document.getElementById('modalCaption').textContent = `Sample ${label}`;
    document.getElementById('imgModal').classList.add('open');
  }

  function closeModal(event) {
    if (event && event.target !== event.currentTarget && event.target !== document.getElementById('imgModal')) return;
    document.getElementById('imgModal').classList.remove('open');
  }

  document.addEventListener('keydown', (event) => {
    if (event.key === 'Escape') {
      document.getElementById('imgModal').classList.remove('open');
    }
  });

  function createMarkerStyle(entry) {
    const baseStyle = { ...defaultMarkerStyle };
    if (state.activeSeries && entry.series !== state.activeSeries) {
      return { ...baseStyle, ...dimmedMarkerStyle };
    }
    if (state.activeSeries && entry.series === state.activeSeries) {
      return { ...baseStyle, ...highlightedMarkerStyle, color: seriesColors[entry.series] || '#111827' };
    }
    return { ...baseStyle, color: seriesColors[entry.series] || '#111827' };
  }

  function addPedonMarkers(data) {
    pedonsLayer.clearLayers();
    markerEntries = [];
    data.forEach((row) => {
      const pointId = row['label'] || row['Point'] || row['point'] || row['Sample'] || '';
      const cleanId = String(pointId).trim();
      if (!cleanId || excluded.includes(cleanId)) return;

      const lat = Number(row.Latitude || row.latitude || row.lat);
      const lng = Number(row.Longitude || row.longitude || row.lng);
      if (!lat || !lng) return;

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
      const marker = L.circleMarker([lat, lng], createMarkerStyle(entry));
      marker.bindPopup(`<strong>${esc(cleanId)}</strong><br>${seriesHTML(series)}`);
      marker.on('click', () => setInfoPanel(entry));
      marker.addTo(pedonsLayer);
    });

    if (state.activeSeries) {
      markerEntries.forEach((entry) => {
        const marker = pedonsLayer.getLayers().find((layer) => layer.options && layer.getLatLng && layer.getLatLng().lat === entry.lat && layer.getLatLng().lng === entry.lng);
        if (marker) {
          const style = createMarkerStyle(entry);
          marker.setStyle(style);
        }
      });
    }

    renderLegendSections();
    if (!state.activeSeries) setInfoPanel(markerEntries[0] || null);
  }

  async function loadMap() {
    const manifest = await fetch(base + 'maplayers/manifest.json').then((response) => response.json());
    state.manifest = manifest;
    document.getElementById('unitNote').textContent = manifest.unit_note || '';

    const allLayers = manifest.rasters || [];
    const controls = document.getElementById('layerControls');
    controls.innerHTML = '';

    allLayers.forEach((r) => {
      const row = document.createElement('div');
      row.className = 'control-row';
      row.innerHTML = `
        <label>
          <input type="checkbox" data-layer-id="${r.id}" checked />
          <span>${esc(r.title)}</span>
        </label>
        <input type="range" min="0" max="1" step="0.05" value="0.8" data-opacity-id="${r.id}" aria-label="${esc(r.title)} opacity" />
      `;

      const checkbox = row.querySelector('input[type="checkbox"]');
      const slider = row.querySelector('input[type="range"]');
      const bounds = [[r.bounds.south, r.bounds.west], [r.bounds.north, r.bounds.east]];
      const overlay = L.imageOverlay(base + r.png, bounds, { pane: 'soilRaster', opacity: 0.8, interactive: false });
      rasterLayers[r.id] = { layer: overlay, title: r.title, opacity: 0.8 };
      overlay.addTo(map);
      state.visibleRasterIds.add(r.id);
      state.activeRaster = r.id;

      checkbox.addEventListener('change', () => {
        const visible = checkbox.checked;
        if (visible) {
          overlay.addTo(map);
          state.visibleRasterIds.add(r.id);
          state.activeRaster = r.id;
        } else {
          overlay.remove();
          state.visibleRasterIds.delete(r.id);
          if (state.activeRaster === r.id) {
            const next = Array.from(state.visibleRasterIds)[0] || null;
            state.activeRaster = next;
          }
        }
        renderLegendSections();
      });

      slider.addEventListener('input', () => {
        const value = Number(slider.value);
        overlay.setOpacity(value);
        rasterLayers[r.id].opacity = value;
        if (checkbox.checked) {
          state.activeRaster = r.id;
        }
        renderLegendSections();
      });

      controls.appendChild(row);
    });

    const seriesToggle = document.createElement('div');
    seriesToggle.className = 'control-row';
    seriesToggle.innerHTML = `
      <label><input type="checkbox" id="seriesToggle" checked /> <span>Mapped soil series</span></label>
    `;
    seriesToggle.querySelector('input').addEventListener('change', (event) => {
      state.soilSeriesVisible = event.target.checked;
      renderLegendSections();
    });
    controls.appendChild(seriesToggle);

    const matchToggle = document.createElement('div');
    matchToggle.className = 'control-row';
    matchToggle.innerHTML = `
      <label><input type="checkbox" id="matchToggle" checked /> <span>SSURGO agreement</span></label>
    `;
    matchToggle.querySelector('input').addEventListener('change', (event) => {
      state.matchVisible = event.target.checked;
      renderLegendSections();
    });
    controls.appendChild(matchToggle);

    const boundaryToggle = document.createElement('div');
    boundaryToggle.className = 'control-row';
    boundaryToggle.innerHTML = `
      <label><input type="checkbox" id="boundaryToggle" checked /> <span>Boundary lines</span></label>
    `;
    boundaryToggle.querySelector('input').addEventListener('change', (event) => {
      state.boundaryVisible = event.target.checked;
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
      pedonsLayer.eachLayer((layer) => layer.setStyle ? layer.setStyle({ opacity: event.target.checked ? 1 : 0, fillOpacity: event.target.checked ? 0.85 : 0 }) : null);
      pedonsLayer.getLayers().forEach((layer) => {
        if (layer.setStyle) layer.setStyle({ opacity: event.target.checked ? 1 : 0, fillOpacity: event.target.checked ? 0.85 : 0 });
      });
      renderLegendSections();
    });
    controls.appendChild(pedonToggle);

    fetch(base + 'maplayers/soil_series.geojson')
      .then((response) => response.json())
      .then((geojson) => {
        soilSeriesLayer.addData(geojson);
      });

    fetch(base + 'maplayers/ssurgo_matchup.geojson')
      .then((response) => response.json())
      .then((geojson) => {
        matchLayer.addData(geojson);
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
