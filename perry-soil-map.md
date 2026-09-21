---
layout: bare
title: "Perry Soil Map"
permalink: /perry-soil-map/
---

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css">
<style>
#mapWrap{display:flex;gap:16px;align-items:flex-start;flex-wrap:wrap;margin-bottom:1.5em}#mapColumn{flex:2;min-width:320px}#map{height:520px;width:100%;border-radius:12px}#infoPanel,#legend{padding:12px 14px;border:1px solid rgba(0,0,0,.15);border-radius:12px;background:#fff}#infoPanel{flex:1;min-width:280px;max-width:460px;padding:12px 14px;border:1px solid rgba(0,0,0,.15);border-radius:12px;background:#fff;position:sticky;top:12px}#legend{margin-top:12px}#legendTitle{margin:0}.legend-header{display:flex;justify-content:space-between;align-items:center;gap:12px;margin-bottom:10px}.legend-header h4{margin:0}#resetLegendBtn{border:1px solid rgba(0,0,0,.2);background:#f7f7f7;border-radius:8px;padding:6px 10px;cursor:pointer;font-size:.9rem}#resetLegendBtn:hover{background:#ececec}.legend-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:8px 16px}.legend-item{display:flex;align-items:center;gap:8px;font-size:.95rem;padding:6px 8px;border-radius:8px;cursor:pointer;transition:background .2s ease,transform .2s ease,opacity .2s ease}.legend-item:hover{background:rgba(0,0,0,.05)}.legend-item.active{background:rgba(0,0,0,.08);font-weight:600}.legend-item.dimmed{opacity:.45}.legend-swatch{width:14px;height:14px;border-radius:50%;border:1.5px solid #000;flex:0 0 14px}.legend-label{color:inherit}.leaflet-popup-content{margin:10px 12px}.series-link{color:inherit;text-decoration:underline}.popup-img{width:160px;max-width:100%;height:auto;max-height:180px;border-radius:10px;cursor:zoom-in;display:block;margin-top:6px;object-fit:contain}.image-carousel{position:relative;width:100%;margin-top:6px}.carousel-images{position:relative;width:100%;border-radius:12px;overflow:hidden}.carousel-image{width:100%;height:auto;border-radius:12px;cursor:zoom-in;object-fit:contain;display:none}.carousel-image.active{display:block}.carousel-arrow{position:absolute;top:50%;transform:translateY(-50%);background:rgba(0,0,0,.5);color:#fff;border:none;font-size:24px;padding:8px 12px;cursor:pointer;border-radius:4px;z-index:10;transition:background .3s;line-height:1}.carousel-arrow:hover{background:rgba(0,0,0,.8)}.carousel-arrow.left{left:10px}.carousel-arrow.right{right:10px}.carousel-dots{position:absolute;bottom:10px;left:50%;transform:translateX(-50%);display:flex;gap:6px;z-index:10}.dot{width:8px;height:8px;border-radius:50%;background:rgba(255,255,255,.5);border:1px solid rgba(255,255,255,.8);cursor:pointer;transition:background .3s}.modal{display:none;position:fixed;inset:0;background:rgba(0,0,0,.85);z-index:9999;padding:24px}.modal-card{max-width:900px;margin:auto;background:#fff;padding:12px;border-radius:14px}.modal-card img{max-width:100%;display:block;margin:auto}.close-note{text-align:right;cursor:pointer}.soil-raster img{image-rendering:pixelated}.legend-note{font-size:0.8rem;color:#666;margin-top:8px}.surface-group{font-weight:700;margin-top:8px;display:block}.control-box{margin-top:10px;padding:10px 12px;border:1px solid rgba(0,0,0,.15);border-radius:10px;background:#fff}.control-box label{display:block;margin:4px 0}.opacity-row{display:flex;align-items:center;gap:10px}.opacity-row input{width:150px}.legend-ramp{display:flex;flex-direction:column;gap:4px}.legend-ramp-row{display:flex;align-items:center;gap:8px}.legend-ramp-swatch{display:inline-block;width:22px;height:14px;border:1px solid rgba(0,0,0,.25)}@media(max-width:700px){#mapWrap{display:block}#map{height:620px}#infoPanel{position:static;max-width:none}} </style>

<h1>Perry Soil Map</h1>
<p>This interactive map shows pedon observations at UGA GrandFarm in Perry, Georgia, together with ordinary-kriged predictions of soil chemistry and texture. Use the layer controls to compare surfaces and mapped soil boundaries.</p>

<div id="mapWrap">
  <div id="mapColumn">
    <div id="map"></div>
    <div id="legend" aria-label="Map legend">
      <div class="legend-header">
        <h4 id="legendTitle">Map legend</h4>
      </div>
      <div id="legendContent"></div>
      <p id="unitNote" class="legend-note"></p>
    </div>
  </div>

  <div id="infoPanel" aria-live="polite">
    <h3>Welcome</h3>
    <p class="muted">Click a pedon point to view its soil profile and field photo.</p>
  </div>
</div>

<div id="imgModal" class="modal" onclick="closeModal(event)">
  <div class="modal-card" onclick="event.stopPropagation()">
    <div class="close-note" onclick="closeModal()">Click outside or press ESC to close</div>
    <img id="modalImage" alt="Enlarged soil profile image">
    <p id="modalCaption"></p>
  </div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
<script>
const map = L.map('map').setView([32.43,-83.73],14);
map.createPane('soilRaster').style.zIndex = 200;
map.createPane('soilVector').style.zIndex = 400;
map.createPane('pedons').style.zIndex = 600;
L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', { attribution: 'Tiles © Esri' }).addTo(map);

const imgBase = '{{ "/assets/images/pedon_images/" | relative_url }}';
const fieldPhotoBase = 'https://github.com/madelynwillis3/research/releases/download/coastalplain-images-v1.0/perry_GA_point_';
const exclude = ['39','50','51','58','59','60','65','74'];

const seriesByPoint = {
  "P2":"Thursa","P3":"Faceville","P4":"Faceville","1":"Orangeburg","2":"Faceville","3":"Faceville","4":"Faceville","5":"Wagram","6":"Norfolk","7":"Bonneau","8":"Wagram","9":"Orangeburg","10":"Esto","11":"Norfolk","12":"Orangeburg","13":"Norfolk","14":"Orangeburg","15":"Dothan","16":"Lakeland","17":"Orangeburg","18":"Orangeburg","19":"Faceville","20":"Marvyn","21":"Norfolk","22":"Orangeburg","23":"Norfolk","24":"Norfolk","25":"Lakeland","26":"Benevolence","27":"Greenville","28":"Greenville","29":"Red Bay","30":"Faceville","31":"Faceville","32":"Norfolk","33":"Norfolk","34":"Johns","35":"Disturbed","36":"Orangeburg","37":"Faceville","38":"Faceville","40":"Greenville","41":"Orangeburg","42":"Dothan","43":"Norfolk","44":"Blanton","45":"Greenville","46":"Greenville","47":"Faceville","48":"Greenville","49":"Disturbed","52":"Leefield","53":"Orangeburg","54":"Faceville","55":"Faceville","56":"Faceville","57":"Greenville","61":"Greenville","62":"Greenville","63":"Lucy","64":"Greenville","66":"Faceville","67":"Faceville","68":"Greenville","69":"Faceville","70":"Faceville","71":"Greenville","72":"Orangeburg","73":"Disturbed","75":"Faceville","76":"Orangeburg","77":"Orangeburg","78":"Lucy","79":"Troup","80":"Troup"
};

const seriesColors = {
  Faceville: '#d73027', Orangeburg: '#c94c4c', Lucy: '#e76f51', Troup: '#f4a6a6', Greenville: '#8b0000', Leefield: '#d8c3a5', Blanton: '#8a7f73', Norfolk: '#f28c28', Dothan: '#d4a017', Johns: '#c2a878', 'Red Bay': '#5c0000', Benevolence: '#fa8072', Lakeland: '#d2a679', Wagram: '#d2a679', Bonneau: '#d3d3d3', Esto: '#c9a44b', Marvyn: '#a44a3f', Disturbed: '#808080'
};

const seriesLinks = Object.fromEntries(
  Object.keys(seriesColors)
    .filter(s => s !== 'Disturbed')
    .map(s => [s, `https://casoilresource.lawr.ucdavis.edu/sde/?series=${encodeURIComponent(s.toUpperCase())}#osd`])
);

const defaultMarkerStyle = { color: '#000', weight: 1.5, fillOpacity: 0.85, opacity: 1, radius: 6 };
const dimmedMarkerStyle = { fillOpacity: 0.15, opacity: 0.25, radius: 5 };
const highlightedMarkerStyle = { fillOpacity: 1, opacity: 1, radius: 8, weight: 2.5 };

const esc = v => String(v ?? '').replace(/[&<>"']/g, c => ({ '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;','\'':'&#39;' }[c]));

let markerEntries = [];
let activeSeries = null;
let modalImage = null;
let modalCaption = null;

function seriesHTML(s) {
  return seriesLinks[s] ? `<a class="series-link" target="_blank" rel="noopener noreferrer" href="${seriesLinks[s]}">${esc(s)}</a>` : esc(s);
}

function updateLegendState() {
  document.querySelectorAll('.legend-item').forEach(item => {
    const isActive = activeSeries === item.dataset.series;
    item.classList.toggle('active', isActive);
    item.classList.toggle('dimmed', !!activeSeries && !isActive);
  });
}

function openModal(src, label) {
  modalImage = src;
  document.getElementById('modalImage').src = src;
  document.getElementById('modalCaption').textContent = `Sample ${label}`;
  document.getElementById('imgModal').style.display = 'block';
  document.body.style.overflow = 'hidden';
}

function closeModal(event) {
  if (event && event.target && event.target.id !== 'imgModal') return;
  document.getElementById('imgModal').style.display = 'none';
  document.body.style.overflow = '';
}

document.addEventListener('keydown', function (e) {
  if (e.key === 'Escape') closeModal();
});

function renderSeriesLegend() {
  const counts = {};
  markerEntries.forEach(entry => {
    counts[entry.series] = (counts[entry.series] || 0) + 1;
  });

  const box = document.getElementById('legendContent');
  box.innerHTML = '';

  Object.keys(counts)
    .sort((a, b) => counts[b] - counts[a] || a.localeCompare(b))
    .forEach(series => {
      const item = document.createElement('button');
      item.type = 'button';
      item.className = 'legend-item';
      item.dataset.series = series;
      item.onclick = () => {
        activeSeries = activeSeries === series ? null : series;
        if (activeSeries) {
          markerEntries.forEach(entry => {
            entry.marker.setStyle(entry.series === activeSeries ? highlightedMarkerStyle : dimmedMarkerStyle);
          });
        } else {
          markerEntries.forEach(entry => entry.marker.setStyle(defaultMarkerStyle));
        }
        updateLegendState();
      };

      const swatch = document.createElement('span');
      swatch.className = 'legend-swatch';
      swatch.style.background = seriesColors[series] || '#808080';

      const label = document.createElement('span');
      label.className = 'legend-label';
      label.textContent = `${series} (${counts[series]})`;

      item.appendChild(swatch);
      item.appendChild(label);
      box.appendChild(item);
    });

  updateLegendState();
}

const pedons = L.layerGroup([], { pane: 'pedons' });

function addPedons() {
  Papa.parse('{{ "/assets/data/perry_FP_samples_80.csv" | relative_url }}', {
    download: true,
    header: true,
    complete: result => {
      result.data.forEach((row, i) => {
        const id = String(row['Point ID'] || '').trim();
        const lat = +row.y;
        const lng = +row.x;

        if (!id || exclude.includes(id) || Number.isNaN(lat) || Number.isNaN(lng)) return;

        const series = seriesByPoint[id] || id;
        const profile = `${imgBase}${id}.jpg`;
        const fieldPhoto = `${fieldPhotoBase}${id}.jpg`;

        const marker = L.circleMarker([lat, lng], {
          ...defaultMarkerStyle,
          fillColor: seriesColors[series] || '#888',
          pane: 'pedons'
        });

        marker.bindPopup(`
          <b>Series:</b> ${seriesHTML(series)}<br>
          <b>Point:</b> ${esc(id)}<br>
          <img src="${profile}" class="popup-img" onclick="openModal('${profile}', '${esc(id)}')">
        `);

        marker.on('click', () => {
          const panel = document.getElementById('infoPanel');
          panel.innerHTML = `
            <h3>${seriesHTML(series)}</h3>
            <p class="muted">Point ${esc(id)} • Lat: ${lat.toFixed(6)} • Lon: ${lng.toFixed(6)}</p>
            <div class="image-carousel">
              <div class="carousel-images">
                <img class="carousel-image active" src="${profile}" alt="Profile image" onclick="openModal('${profile}', '${esc(id)}')">
                <img class="carousel-image" src="${fieldPhoto}" alt="Field photo" onclick="openModal('${fieldPhoto}', '${esc(id)}')">
              </div>
            </div>
          `;
        });

        markerEntries.push({ marker, series });
        pedons.addLayer(marker);
        setTimeout(() => {
          marker.setStyle({ fillOpacity: 0.9, opacity: 1 });
        }, 300 + i * 20);
      });

      map.addLayer(pedons);
      renderSeriesLegend();
    }
  });
}

function vectorStyle(feature, kind) {
  if (kind === 'series') return { pane: 'soilVector', color: '#333', weight: 1, fillColor: feature.properties.fill || '#bbb', fillOpacity: 0.75 };
  if (kind === 'match') return { pane: 'soilVector', weight: 0, fillColor: feature.properties.fill || '#bbb', fillOpacity: 0.7 };
  if (kind === 'units') return { pane: 'soilVector', color: '#777', weight: 1, dashArray: '5,4', fill: false };
  return { pane: 'soilVector', color: '#000', weight: 2, fill: false };
}

function popup(feature, kind) {
  if (kind === 'series') {
    return `<b>Series:</b> ${esc(feature.properties.series)}<br><b>Musym:</b> ${esc(feature.properties.musym)}<br><b>Muname:</b> ${esc(feature.properties.muname)}`;
  }
  if (kind === 'match') {
    return `<b>Match:</b> ${esc(feature.properties.match_status)}<br><b>Field series:</b> ${esc(feature.properties.field_series)}<br><b>SSURGO series:</b> ${esc(feature.properties.ssurgo_series)}<br><b>SSURGO map unit:</b> ${esc(feature.properties.ssurgo_map_unit)}<br><b>Drainage:</b> ${esc(feature.properties.drainage)}<br><b>Slope:</b> ${esc(feature.properties.slope)}`;
  }
  return '';
}

async function loadMap() {
  try {
    const base = '{{ "/assets/" | relative_url }}';
    const manifest = await fetch(base + 'maplayers/manifest.json').then(r => r.json());
    document.getElementById('unitNote').textContent = manifest.unit_note || '';

    const rasterLayers = {};
    const vectorLayers = {};
    const activeRaster = { current: null };

    manifest.rasters.forEach(r => {
      const layer = L.imageOverlay(base + r.png, [[r.bounds.south,r.bounds.west],[r.bounds.north,r.bounds.east]], { opacity: 0.75, interactive: false, className: 'soil-raster', pane: 'soilRaster' });
      rasterLayers[r.id] = layer;
    });

    const surfaceControls = document.createElement('div');
    surfaceControls.className = 'control-box';
    surfaceControls.innerHTML = '<span class="surface-group">Continuous surfaces</span>';

    const opacityWrap = document.createElement('div');
    opacityWrap.className = 'opacity-row';
    opacityWrap.innerHTML = '<label>Opacity <input id="opacityControl" type="range" min="0" max="100" value="75"></label><output id="opacityValue">75%</output>';
    surfaceControls.appendChild(opacityWrap);

    manifest.rasters.forEach(r => {
      const label = document.createElement('label');
      const input = document.createElement('input');
      input.type = 'radio';
      input.name = 'surface';
      label.appendChild(input);
      label.appendChild(document.createTextNode(` ${r.title}${r.unit ? ` (${r.unit})` : ''}`));
      label.addEventListener('change', () => {
        if (activeRaster.current) map.removeLayer(activeRaster.current);
        activeRaster.current = rasterLayers[r.id];
        activeRaster.current.addTo(map);
        renderRasterLegend(r);
      });
      surfaceControls.appendChild(label);
    });

    const none = document.createElement('label');
    const noneInput = document.createElement('input');
    noneInput.type = 'radio';
    noneInput.name = 'surface';
    noneInput.checked = true;
    none.appendChild(noneInput);
    none.appendChild(document.createTextNode(' None'));
    none.addEventListener('change', () => {
      if (activeRaster.current) {
        map.removeLayer(activeRaster.current);
        activeRaster.current = null;
      }
      document.getElementById('legendContent').innerHTML = '';
      document.getElementById('legendTitle').textContent = 'Map legend';
    });
    surfaceControls.appendChild(none);

    const mapColumn = document.getElementById('mapColumn');
    mapColumn.appendChild(surfaceControls);

    document.getElementById('opacityControl').addEventListener('input', e => {
      const v = Number(e.target.value);
      document.getElementById('opacityValue').textContent = `${v}%`;
      if (activeRaster.current) {
        activeRaster.current.setOpacity(v / 100);
      }
    });

    const extraControls = document.createElement('div');
    extraControls.className = 'control-box';
    extraControls.innerHTML = '<span class="surface-group">Vector overlays</span>';

    for (const [key, value] of Object.entries(manifest.vectors)) {
      const label = document.createElement('label');
      const input = document.createElement('input');
      input.type = 'checkbox';
      label.appendChild(input);
      label.appendChild(document.createTextNode(` ${value.title}`));
      extraControls.appendChild(label);

      if (key === 'soil_series') {
        const layer = L.geoJSON(await fetch(base + value.file).then(r => r.json()), {
          style: feature => vectorStyle(feature, 'series'),
          onEachFeature: (feature, layer) => layer.bindPopup(popup(feature, 'series'))
        });
        vectorLayers[key] = layer;
        input.addEventListener('change', () => {
          if (input.checked) layer.addTo(map); else map.removeLayer(layer);
        });
      }

      if (key === 'ssurgo_matchup') {
        const layer = L.geoJSON(await fetch(base + value.file).then(r => r.json()), {
          style: feature => vectorStyle(feature, 'match'),
          onEachFeature: (feature, layer) => layer.bindPopup(popup(feature, 'match'))
        });
        vectorLayers[key] = layer;
        input.addEventListener('change', () => {
          if (input.checked) layer.addTo(map); else map.removeLayer(layer);
        });
      }

      if (key === 'ssurgo_units') {
        const layer = L.geoJSON(await fetch(base + value.file).then(r => r.json()), {
          style: feature => vectorStyle(feature, 'units')
        });
        vectorLayers[key] = layer;
        input.addEventListener('change', () => {
          if (input.checked) layer.addTo(map); else map.removeLayer(layer);
        });
      }

      if (key === 'site_border') {
        const layer = L.geoJSON(await fetch(base + value.file).then(r => r.json()), {
          style: feature => vectorStyle(feature, 'border')
        });
        vectorLayers[key] = layer;
        input.addEventListener('change', () => {
          if (input.checked) layer.addTo(map); else map.removeLayer(layer);
        });
      }
    }

    const pointsLabel = document.createElement('label');
    pointsLabel.innerHTML = '<input id="pointsToggle" type="checkbox" checked> Pedon points';
    extraControls.appendChild(pointsLabel);
    document.getElementById('pointsToggle').addEventListener('change', e => {
      if (e.target.checked) map.addLayer(pedons); else map.removeLayer(pedons);
    });

    mapColumn.appendChild(extraControls);
    addPedons();
  } catch (error) {
    console.error(error);
    document.getElementById('legendContent').textContent = 'Map layers could not be loaded.';
  }
}

function renderRasterLegend(r) {
  const box = document.getElementById('legendContent');
  box.innerHTML = '';

  const section = document.createElement('div');
  section.className = 'legend-ramp';

  for (let i = r.colors.length - 1; i >= 0; i--) {
    const row = document.createElement('div');
    row.className = 'legend-ramp-row';

    const swatch = document.createElement('span');
    swatch.className = 'legend-ramp-swatch';
    swatch.style.background = r.colors[i];

    const label = document.createElement('span');
    label.textContent = `${fmt(r.breaks[i])} – ${fmt(r.breaks[i + 1])}`;

    row.appendChild(swatch);
    row.appendChild(label);
    section.appendChild(row);
  }

  box.appendChild(section);
  document.getElementById('legendTitle').textContent = `${r.title}${r.unit ? ` (${r.unit})` : ''}`;
}

function fmt(v) {
  const a = Math.abs(v);
  if (a >= 100) return Number(v).toFixed(0);
  if (a >= 10) return Number(v).toFixed(1);
  return Number(v).toFixed(2);
}

loadMap();
</script>
