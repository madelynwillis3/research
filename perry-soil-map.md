---
layout: bare
title: "Perry Soil Map"
permalink: /perry-soil-map/
---

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<style>
  #mapWrap {
    display: flex;
    gap: 16px;
    align-items: flex-start;
    flex-wrap: wrap;
    margin-bottom: 1.5em;
  }

  #legendColumn {
    flex: 0 0 260px;
    max-width: 260px;
    position: sticky;
    top: 12px;
    align-self: flex-start;
  }

  #mapColumn {
    flex: 1 1 520px;
    min-width: 320px;
  }

  #map {
    height: 520px;
    width: 100%;
    border-radius: 12px;
  }

  #infoPanel {
    flex: 1 1 300px;
    min-width: 280px;
    max-width: 460px;
    position: sticky;
    top: 12px;
    padding: 12px 14px;
    border: 1px solid rgba(0,0,0,0.15);
    border-radius: 12px;
    background: #fff;
  }

  #legend {
    padding: 12px 14px;
    border: 1px solid rgba(0,0,0,0.15);
    border-radius: 12px;
    background: #fff;
    margin-top: 0;
    max-height: calc(100vh - 40px);
    overflow-y: auto;
    font-size: 0.9rem;
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

  .legend-section {
    padding: 10px 0;
    border-top: 1px solid rgba(0,0,0,0.09);
  }

  .legend-section:first-child {
    border-top: 0;
    padding-top: 0;
  }

  .legend-section h5 {
    margin: 0 0 6px;
    font-size: 0.82rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.04em;
    color: #444;
  }

  .legend-section .sub {
    margin: 0 0 6px;
    font-size: 0.75rem;
    color: #666;
  }

  .legend-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 2px;
  }

  .legend-item {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.85rem;
    padding: 4px 6px;
    border-radius: 6px;
    cursor: pointer;
    border: 0;
    background: transparent;
    text-align: left;
    width: 100%;
    color: inherit;
  }

  .legend-item:hover,
  .legend-item.active {
    background: rgba(0,0,0,0.08);
  }

  .legend-item.dimmed {
    opacity: 0.45;
  }

  .legend-item.static {
    cursor: default;
  }

  .legend-item.static:hover {
    background: transparent;
  }

  .legend-label {
    color: inherit;
    flex: 1;
  }

  .legend-swatch {
    width: 14px;
    height: 14px;
    border: 1.5px solid #000;
    flex: 0 0 14px;
  }

  .legend-swatch.dot {
    border-radius: 50%;
  }

  .legend-swatch.box {
    border-radius: 3px;
    border-width: 1px;
  }

  .legend-swatch.line {
    height: 0;
    border: 0;
    border-top-width: 2.5px;
    border-top-style: solid;
    width: 20px;
    flex: 0 0 20px;
  }

  .ramp {
    display: flex;
    flex-direction: column;
    gap: 2px;
    margin-top: 4px;
  }

  .ramp-row {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.78rem;
    line-height: 1.2;
  }

  .ramp-swatch {
    display: inline-block;
    width: 16px;
    height: 12px;
    border: 1px solid rgba(0,0,0,0.25);
    flex: 0 0 16px;
  }

  .legend-note {
    font-size: 0.75rem;
    color: #666;
    margin-top: 8px;
  }

  .control-box {
    margin-top: 10px;
    padding: 10px 12px;
    border: 1px solid rgba(0,0,0,0.15);
    border-radius: 10px;
    background: #fff;
  }

  .surface-group {
    display: block;
    margin-bottom: 8px;
    font-weight: 700;
  }

  .control-box label {
    display: flex;
    align-items: center;
    gap: 8px;
    margin: 4px 0;
    font-size: 0.92rem;
  }

  .opacity-row {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 8px;
  }

  .opacity-row input {
    width: 150px;
  }

  .leaflet-popup-content {
    margin: 10px 12px;
  }

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
    display: block;
    margin-top: 6px;
    object-fit: contain;
    cursor: zoom-in;
  }

  .soil-raster img {
    image-rendering: pixelated;
  }

  .image-carousel {
    position: relative;
    width: 100%;
    margin-top: 8px;
  }

  .carousel-images {
    position: relative;
    width: 100%;
    border-radius: 12px;
    overflow: hidden;
  }

  .carousel-image {
    display: none;
    width: 100%;
    height: auto;
    border-radius: 12px;
    background: #f4f4f4;
    object-fit: contain;
    cursor: zoom-in;
  }

  .carousel-image.active {
    display: block;
  }

  .modal {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.85);
    z-index: 9999;
    padding: 24px;
  }

  .modal-card {
    max-width: 900px;
    margin: auto;
    background: #fff;
    padding: 12px;
    border-radius: 14px;
  }

  .modal-card img {
    max-width: 100%;
    display: block;
    margin: auto;
  }

  .close-note {
    text-align: right;
    cursor: pointer;
    color: #666;
    margin-bottom: 8px;
  }

  @media (max-width: 700px) {
    #mapWrap {
      display: block;
    }

    #legendColumn,
    #infoPanel {
      position: static;
      max-width: none;
      width: 100%;
      flex: 1 1 auto;
    }

    #map {
      height: 620px;
    }
  }
</style>

<h1>Perry Soil Map</h1>

<p>
  This interactive map shows pedon observations at UGA GrandFarm in Perry, Georgia,
  along with soil chemistry and texture surfaces. The map is intentionally unlisted
  and shared by direct link only.
</p>

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
  </div>

  <div id="infoPanel" aria-live="polite">
    <h3>Welcome</h3>
    <p class="muted">Click a pedon point to view its soil profile and field photo.</p>
  </div>
</div>

<div id="imgModal" class="modal" onclick="closeModal(event)">
  <div class="modal-card" onclick="event.stopPropagation()">
    <div class="close-note" onclick="closeModal()">Click outside or press ESC to close</div>
    <img id="modalImage" alt="Expanded soil profile image" />
    <p id="modalCaption"></p>
  </div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
<script>
  const map = L.map('map').setView([32.433, -83.729], 14);
  map.createPane('soilRaster').style.zIndex = 200;
  map.createPane('soilVector').style.zIndex = 400;
  map.createPane('pedons').style.zIndex = 600;

  L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
    attribution: 'Tiles © Esri'
  }).addTo(map);

  const base = '{{ "/assets/" | relative_url }}';
  const imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';
  const fieldPhotoBase = 'https://github.com/madelynwillis3/research/releases/download/coastalplain-images-v1.0/perry_GA_point_';
  const exclude = ['39', '50', '51', '58', '59', '60', '65', '74'];

  const seriesByPoint = {
    P2: 'Thursa', P3: 'Faceville', P4: 'Faceville', 1: 'Orangeburg', 2: 'Faceville', 3: 'Faceville', 4: 'Faceville', 5: 'Wagram', 6: 'Norfolk',
    7: 'Bonneau', 8: 'Wagram', 9: 'Orangeburg', 10: 'Esto', 11: 'Norfolk', 12: 'Orangeburg', 13: 'Norfolk', 14: 'Orangeburg', 15: 'Dothan',
    16: 'Lakeland', 17: 'Orangeburg', 18: 'Orangeburg', 19: 'Faceville', 20: 'Marvyn', 21: 'Norfolk', 22: 'Orangeburg', 23: 'Norfolk', 24: 'Norfolk',
    25: 'Lakeland', 26: 'Benevolence', 27: 'Greenville', 28: 'Greenville', 29: 'Red Bay', 30: 'Faceville', 31: 'Faceville', 32: 'Norfolk',
    33: 'Norfolk', 34: 'Johns', 35: 'Disturbed', 36: 'Orangeburg', 37: 'Faceville', 38: 'Faceville', 40: 'Greenville', 41: 'Orangeburg', 42: 'Dothan',
    43: 'Norfolk', 44: 'Blanton', 45: 'Greenville', 46: 'Greenville', 47: 'Faceville', 48: 'Greenville', 49: 'Disturbed', 52: 'Leefield',
    53: 'Orangeburg', 54: 'Faceville', 55: 'Faceville', 56: 'Faceville', 57: 'Greenville', 61: 'Greenville', 62: 'Greenville', 63: 'Lucy',
    64: 'Greenville', 66: 'Faceville', 67: 'Faceville', 68: 'Greenville', 69: 'Faceville', 70: 'Faceville', 71: 'Greenville', 72: 'Orangeburg',
    73: 'Disturbed', 75: 'Faceville', 76: 'Orangeburg', 77: 'Orangeburg', 78: 'Lucy', 79: 'Troup', 80: 'Troup'
  };

  const seriesColors = {
    Faceville: '#d73027',
    Orangeburg: '#c94c4c',
    Lucy: '#e76f51',
    Troup: '#f4a6a6',
    Greenville: '#8b0000',
    Leefield: '#d8c3a5',
    Blanton: '#8a7f73',
    Norfolk: '#f28c28',
    Dothan: '#d4a017',
    Johns: '#c2a878',
    'Red Bay': '#5c0000',
    Benevolence: '#fa8072',
    Lakeland: '#d2a679',
    Wagram: '#d2a679',
    Bonneau: '#d3d3d3',
    Esto: '#c9a44b',
    Marvyn: '#a44a3f',
    Disturbed: '#808080'
  };

  const seriesLinks = Object.fromEntries(
    Object.keys(seriesColors)
      .filter(function (s) { return s !== 'Disturbed'; })
      .map(function (s) {
        return [s, 'https://casoilresource.lawr.ucdavis.edu/sde/?series=' + encodeURIComponent(s.toUpperCase()) + '#osd'];
      })
  );

  const defaultMarkerStyle = { color: '#000', weight: 1.5, fillOpacity: 0.85, opacity: 1, radius: 6 };
  const dimmedMarkerStyle = { fillOpacity: 0.15, opacity: 0.25, radius: 5 };
  const highlightedMarkerStyle = { fillOpacity: 1, opacity: 1, radius: 8, weight: 2.5 };

  const pedonGroup = L.layerGroup([], { pane: 'pedons' });
  let markerEntries = [];
  let activeSeries = null;
  let activeRaster = null;

  function esc(value) {
    return String(value ?? '').replace(/[&<>"']/g, function (c) {
      return ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' })[c];
    });
  }

  function seriesHTML(series) {
    return seriesLinks[series]
      ? '<a class="series-link" target="_blank" rel="noopener noreferrer" href="' + seriesLinks[series] + '">' + esc(series) + '</a>'
      : esc(series);
  }

  function setLegendTitle(text) {
    document.getElementById('legendTitle').textContent = text;
  }

  function makeSection(title, subtitle, body) {
    const section = document.createElement('div');
    section.className = 'legend-section';

    const heading = document.createElement('h5');
    heading.textContent = title;
    section.appendChild(heading);

    if (subtitle) {
      const sub = document.createElement('p');
      sub.className = 'sub';
      sub.textContent = subtitle;
      section.appendChild(sub);
    }

    section.appendChild(body);
    return section;
  }

  function renderRasterLegend(raster) {
    const box = document.getElementById('legendContent');
    box.innerHTML = '';

    const grid = document.createElement('div');
    grid.className = 'ramp';

    for (let i = raster.colors.length - 1; i >= 0; i--) {
      const row = document.createElement('div');
      row.className = 'ramp-row';

      const swatch = document.createElement('span');
      swatch.className = 'ramp-swatch';
      swatch.style.background = raster.colors[i];

      const label = document.createElement('span');
      label.textContent = fmt(raster.breaks[i]) + '–' + fmt(raster.breaks[i + 1]);

      row.appendChild(swatch);
      row.appendChild(label);
      grid.appendChild(row);
    }

    const section = makeSection(raster.title + (raster.unit ? ' (' + raster.unit + ')' : ''), '', grid);
    box.appendChild(section);
    setLegendTitle('Legend');
  }

  function renderSeriesLegend() {
    const counts = {};
    markerEntries.forEach(function (entry) {
      counts[entry.series] = (counts[entry.series] || 0) + 1;
    });

    const box = document.getElementById('legendContent');
    box.innerHTML = '';

    const grid = document.createElement('div');
    grid.className = 'legend-grid';

    Object.keys(counts)
      .sort(function (a, b) {
        return counts[b] - counts[a] || a.localeCompare(b);
      })
      .forEach(function (series) {
        const item = document.createElement('button');
        item.type = 'button';
        item.className = 'legend-item';
        item.dataset.series = series;

        item.onclick = function () {
          activeSeries = activeSeries === series ? null : series;
          markerEntries.forEach(function (entry) {
            if (!entry.marker) return;
            if (activeSeries) {
              entry.marker.setStyle(entry.series === activeSeries ? highlightedMarkerStyle : dimmedMarkerStyle);
            } else {
              entry.marker.setStyle(defaultMarkerStyle);
            }
          });
          updateSeriesLegendState();
        };

        const swatch = document.createElement('span');
        swatch.className = 'legend-swatch dot';
        swatch.style.background = seriesColors[series] || '#808080';

        const label = document.createElement('span');
        label.className = 'legend-label';
        label.textContent = series + ' (' + counts[series] + ')';

        item.appendChild(swatch);
        item.appendChild(label);
        grid.appendChild(item);
      });

    const section = makeSection('Mapped soil series', '', grid);
    box.appendChild(section);
    updateSeriesLegendState();
  }

  function renderMatchupLegend() {
    const box = document.getElementById('legendContent');
    const grid = document.createElement('div');
    grid.className = 'legend-grid';

    ['Agree', 'Disagree'].forEach(function (status) {
      const item = document.createElement('div');
      item.className = 'legend-item static';

      const swatch = document.createElement('span');
      swatch.className = 'legend-swatch box';
      swatch.style.background = status === 'Agree' ? '#1B7837' : '#E8710A';

      const label = document.createElement('span');
      label.className = 'legend-label';
      label.textContent = status;

      item.appendChild(swatch);
      item.appendChild(label);
      grid.appendChild(item);
    });

    const section = makeSection('SSURGO agreement', '', grid);
    box.appendChild(section);
  }

  function renderBoundaryLegend() {
    const box = document.getElementById('legendContent');
    const grid = document.createElement('div');
    grid.className = 'legend-grid';

    const ssurgoItem = document.createElement('div');
    ssurgoItem.className = 'legend-item static';
    const ssurgoSwatch = document.createElement('span');
    ssurgoSwatch.className = 'legend-swatch line';
    ssurgoSwatch.style.borderTopColor = '#7a7a7a';
    const ssurgoLabel = document.createElement('span');
    ssurgoLabel.className = 'legend-label';
    ssurgoLabel.textContent = 'SSURGO map unit boundaries';
    ssurgoItem.appendChild(ssurgoSwatch);
    ssurgoItem.appendChild(ssurgoLabel);
    grid.appendChild(ssurgoItem);

    const siteItem = document.createElement('div');
    siteItem.className = 'legend-item static';
    const siteSwatch = document.createElement('span');
    siteSwatch.className = 'legend-swatch line';
    siteSwatch.style.borderTopColor = '#000';
    const siteLabel = document.createElement('span');
    siteLabel.className = 'legend-label';
    siteLabel.textContent = 'Site border';
    siteItem.appendChild(siteSwatch);
    siteItem.appendChild(siteLabel);
    grid.appendChild(siteItem);

    const section = makeSection('Boundary lines', '', grid);
    box.appendChild(section);
  }

  function renderPedonLegend() {
    const box = document.getElementById('legendContent');
    const grid = document.createElement('div');
    grid.className = 'legend-grid';

    const item = document.createElement('div');
    item.className = 'legend-item static';
    const swatch = document.createElement('span');
    swatch.className = 'legend-swatch dot';
    swatch.style.background = '#d95f0e';
    const label = document.createElement('span');
    label.className = 'legend-label';
    label.textContent = 'Pedon points';
    item.appendChild(swatch);
    item.appendChild(label);
    grid.appendChild(item);

    const section = makeSection('Pedon points', '', grid);
    box.appendChild(section);
  }

  function updateSeriesLegendState() {
    document.querySelectorAll('.legend-item[data-series]').forEach(function (item) {
      const isActive = activeSeries === item.dataset.series;
      item.classList.toggle('active', isActive);
      item.classList.toggle('dimmed', !!activeSeries && !isActive);
    });
  }

  function renderLegend() {
    const box = document.getElementById('legendContent');
    box.innerHTML = '';

    if (activeRaster) {
      renderRasterLegend(activeRaster.data);
    }

    if (isSeriesVisible()) {
      renderSeriesLegend();
    }

    if (isMatchVisible()) {
      renderMatchupLegend();
    }

    if (isBoundaryVisible()) {
      renderBoundaryLegend();
    }

    if (pedonVisible()) {
      renderPedonLegend();
    }

    if (!activeRaster && !isSeriesVisible() && !isMatchVisible() && !isBoundaryVisible() && !pedonVisible()) {
      box.innerHTML = '<div class="legend-section"><h5>Layers</h5><div class="legend-item static">No layers are currently visible.</div></div>';
    }
  }

  function openModal(src, label) {
    document.getElementById('modalImage').src = src;
    document.getElementById('modalCaption').textContent = 'Sample ' + label;
    document.getElementById('imgModal').style.display = 'block';
    document.body.style.overflow = 'hidden';
  }

  function closeModal(event) {
    if (event && event.target && event.target.id !== 'imgModal') return;
    document.getElementById('imgModal').style.display = 'none';
    document.body.style.overflow = '';
  }

  document.addEventListener('keydown', function (event) {
    if (event.key === 'Escape') closeModal();
  });

  function fmt(value) {
    const v = Number(value);
    if (Math.abs(v) >= 100) return v.toFixed(0);
    if (Math.abs(v) >= 10) return v.toFixed(1);
    return v.toFixed(2);
  }

  function addPedonPoints() {
    Papa.parse('{{ "/assets/data/perry_FP_samples_80.csv" | relative_url }}', {
      download: true,
      header: true,
      complete: function (result) {
        result.data.forEach(function (row, index) {
          const id = String(row['Point ID'] || '').trim();
          const lat = Number(row.y);
          const lng = Number(row.x);

          if (!id || exclude.includes(id) || Number.isNaN(lat) || Number.isNaN(lng)) {
            return;
          }

          const series = seriesByPoint[id] || id;
          const profileImage = imgBase + id + '.jpg';
          const fieldImage = fieldPhotoBase + id + '.jpg';

          const marker = L.circleMarker([lat, lng], {
            ...defaultMarkerStyle,
            fillColor: seriesColors[series] || '#888',
            pane: 'pedons'
          });

          marker.bindPopup(
            '<b>Series:</b> ' + seriesHTML(series) + '<br>' +
            '<b>Point:</b> ' + esc(id) + '<br>' +
            '<img src="' + profileImage + '" class="popup-img" onclick="openModal(\'' + profileImage + '\', \\'' + esc(id) + '\\')">'
          );

          marker.on('click', function () {
            const panel = document.getElementById('infoPanel');
            panel.innerHTML = '<h3>' + seriesHTML(series) + '</h3>' +
              '<p class="muted">Point ' + esc(id) + ' • Lat: ' + lat.toFixed(6) + ' • Lon: ' + lng.toFixed(6) + '</p>' +
              '<div class="image-carousel">' +
                '<div class="carousel-images">' +
                  '<img class="carousel-image active" src="' + profileImage + '" alt="Profile image" onclick="openModal(\'' + profileImage + '\', \\'' + esc(id) + '\\')">' +
                  '<img class="carousel-image" src="' + fieldImage + '" alt="Field photo" onclick="openModal(\'' + fieldImage + '\', \\'' + esc(id) + '\\')">' +
                '</div>' +
              '</div>';
          });

          markerEntries.push({ marker: marker, series: series });
          pedonGroup.addLayer(marker);

          setTimeout(function () {
            marker.setStyle({ fillOpacity: 0.9, opacity: 1 });
          }, 300 + index * 20);
        });

        map.addLayer(pedonGroup);
        renderLegend();
      }
    });
  }

  function isSeriesVisible() {
    const checkbox = document.getElementById('toggle-series');
    return !!(checkbox && checkbox.checked);
  }

  function isMatchVisible() {
    const checkbox = document.getElementById('toggle-matchup');
    return !!(checkbox && checkbox.checked);
  }

  function isBoundaryVisible() {
    const units = document.getElementById('toggle-units');
    const border = document.getElementById('toggle-border');
    return !!((units && units.checked) || (border && border.checked));
  }

  function pedonVisible() {
    const checkbox = document.getElementById('toggle-pedons');
    return !!(checkbox && checkbox.checked);
  }

  async function buildControls() {
    const manifest = await fetch(base + 'maplayers/manifest.json').then(function (response) { return response.json(); });
    document.getElementById('unitNote').textContent = manifest.unit_note || '';

    const rasterLayers = {};
    manifest.rasters.forEach(function (raster) {
      rasterLayers[raster.id] = L.imageOverlay(base + raster.png, [[raster.bounds.south, raster.bounds.west], [raster.bounds.north, raster.bounds.east]], {
        opacity: 0.75,
        interactive: false,
        className: 'soil-raster',
        pane: 'soilRaster'
      });
    });

    const surfaceBox = document.createElement('div');
    surfaceBox.className = 'control-box';
    surfaceBox.innerHTML = '<span class="surface-group">Continuous surfaces</span>';

    const opacityRow = document.createElement('div');
    opacityRow.className = 'opacity-row';
    opacityRow.innerHTML = '<label>Opacity <input id="opacityControl" type="range" min="0" max="100" value="75" /></label><output id="opacityValue">75%</output>';
    surfaceBox.appendChild(opacityRow);

    const noneLabel = document.createElement('label');
    const noneInput = document.createElement('input');
    noneInput.type = 'radio';
    noneInput.name = 'surface';
    noneInput.checked = true;
    noneLabel.appendChild(noneInput);
    noneLabel.appendChild(document.createTextNode(' None'));
    noneLabel.addEventListener('change', function () {
      if (activeRaster) {
        map.removeLayer(activeRaster.layer);
        activeRaster = null;
      }
      renderLegend();
    });
    surfaceBox.appendChild(noneLabel);

    manifest.rasters.forEach(function (raster) {
      const label = document.createElement('label');
      const input = document.createElement('input');
      input.type = 'radio';
      input.name = 'surface';
      label.appendChild(input);
      label.appendChild(document.createTextNode(' ' + raster.title + (raster.unit ? ' (' + raster.unit + ')' : '')));

      input.addEventListener('change', function () {
        if (activeRaster) {
          map.removeLayer(activeRaster.layer);
        }

        activeRaster = {
          layer: rasterLayers[raster.id],
          data: raster
        };
        activeRaster.layer.addTo(map);
        activeRaster.layer.setOpacity(Number(document.getElementById('opacityControl').value) / 100);
        renderLegend();
      });

      surfaceBox.appendChild(label);
    });

    document.getElementById('mapColumn').appendChild(surfaceBox);

    document.getElementById('opacityControl').addEventListener('input', function () {
      const value = Number(this.value);
      document.getElementById('opacityValue').textContent = value + '%';
      if (activeRaster) {
        activeRaster.layer.setOpacity(value / 100);
      }
    });

    const vectorBox = document.createElement('div');
    vectorBox.className = 'control-box';
    vectorBox.innerHTML = '<span class="surface-group">Vector overlays</span>';

    const vectorLayers = {};

    const soilSeries = manifest.vectors.soil_series;
    const seriesLayer = L.geoJSON(await fetch(base + soilSeries.file).then(function (response) { return response.json(); }), {
      style: function (feature) {
        return {
          color: '#333',
          weight: 1,
          fillColor: feature.properties.fill || '#bbb',
          fillOpacity: 0.75,
          pane: 'soilVector'
        };
      },
      onEachFeature: function (feature, layer) {
        layer.bindPopup('<b>Series:</b> ' + esc(feature.properties.series) + '<br><b>Musym:</b> ' + esc(feature.properties.musym) + '<br><b>Muname:</b> ' + esc(feature.properties.muname));
      }
    });
    vectorLayers.soil_series = seriesLayer;

    const matchupLayer = L.geoJSON(await fetch(base + manifest.vectors.ssurgo_matchup.file).then(function (response) { return response.json(); }), {
      style: function (feature) {
        return {
          weight: 0,
          fillColor: feature.properties.fill || '#bbb',
          fillOpacity: 0.7,
          pane: 'soilVector'
        };
      },
      onEachFeature: function (feature, layer) {
        layer.bindPopup(
          '<b>Match status:</b> ' + esc(feature.properties.match_status) + '<br>' +
          '<b>Field series:</b> ' + esc(feature.properties.field_series) + '<br>' +
          '<b>SSURGO series:</b> ' + esc(feature.properties.ssurgo_series) + '<br>' +
          '<b>SSURGO map unit:</b> ' + esc(feature.properties.ssurgo_map_unit) + '<br>' +
          '<b>Drainage:</b> ' + esc(feature.properties.drainage) + '<br>' +
          '<b>Slope:</b> ' + esc(feature.properties.slope)
        );
      }
    });
    vectorLayers.ssurgo_matchup = matchupLayer;

    const unitsLayer = L.geoJSON(await fetch(base + manifest.vectors.ssurgo_units.file).then(function (response) { return response.json(); }), {
      style: function () {
        return { color: '#777', weight: 1, dashArray: '5,4', fill: false, pane: 'soilVector' };
      }
    });
    vectorLayers.ssurgo_units = unitsLayer;

    const borderLayer = L.geoJSON(await fetch(base + manifest.vectors.site_border.file).then(function (response) { return response.json(); }), {
      style: function () {
        return { color: '#000', weight: 2, fill: false, pane: 'soilVector' };
      }
    });
    vectorLayers.site_border = borderLayer;

    const toggles = [
      { key: 'soil_series', title: 'Mapped soil series' },
      { key: 'ssurgo_matchup', title: 'Field series vs SSURGO' },
      { key: 'ssurgo_units', title: 'SSURGO map unit boundaries' },
      { key: 'site_border', title: 'Site border' },
      { key: 'pedons', title: 'Pedon points', defaultChecked: true }
    ];

    toggles.forEach(function (toggle) {
      const label = document.createElement('label');
      const input = document.createElement('input');
      input.type = 'checkbox';
      input.checked = !!toggle.defaultChecked;
      input.id = 'toggle-' + toggle.key.replace('_', '-');
      label.appendChild(input);
      label.appendChild(document.createTextNode(' ' + toggle.title));

      if (toggle.key === 'soil_series') {
        input.addEventListener('change', function () {
          if (this.checked) {
            seriesLayer.addTo(map);
          } else {
            map.removeLayer(seriesLayer);
          }
          renderLegend();
        });
      }

      if (toggle.key === 'ssurgo_matchup') {
        input.addEventListener('change', function () {
          if (this.checked) {
            matchupLayer.addTo(map);
          } else {
            map.removeLayer(matchupLayer);
          }
          renderLegend();
        });
      }

      if (toggle.key === 'ssurgo_units') {
        input.addEventListener('change', function () {
          if (this.checked) {
            unitsLayer.addTo(map);
          } else {
            map.removeLayer(unitsLayer);
          }
          renderLegend();
        });
      }

      if (toggle.key === 'site_border') {
        input.checked = true;
        input.addEventListener('change', function () {
          if (this.checked) {
            borderLayer.addTo(map);
          } else {
            map.removeLayer(borderLayer);
          }
          renderLegend();
        });
      }

      if (toggle.key === 'pedons') {
        input.addEventListener('change', function () {
          if (this.checked) {
            map.addLayer(pedonGroup);
          } else {
            map.removeLayer(pedonGroup);
          }
          renderLegend();
        });
      }

      vectorBox.appendChild(label);
    });

    document.getElementById('mapColumn').appendChild(vectorBox);

    if (document.getElementById('toggle-site-border')) {
      document.getElementById('toggle-site-border').checked = true;
      borderLayer.addTo(map);
    }
    if (document.getElementById('toggle-pedons')) {
      document.getElementById('toggle-pedons').checked = true;
      map.addLayer(pedonGroup);
    }

    if (document.getElementById('toggle-soil-series')) {
      document.getElementById('toggle-soil-series').checked = false;
    }
    if (document.getElementById('toggle-ssurgo-matchup')) {
      document.getElementById('toggle-ssurgo-matchup').checked = false;
    }
    if (document.getElementById('toggle-ssurgo-units')) {
      document.getElementById('toggle-ssurgo-units').checked = false;
    }

    renderLegend();
    addPedonPoints();
  }

  buildControls();
</script>
