<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>GeoViewer 3D</title>
<script src="https://cesium.com/downloads/cesiumjs/releases/1.114/Build/Cesium/Cesium.js"></script>
<link href="https://cesium.com/downloads/cesiumjs/releases/1.114/Build/Cesium/Widgets/widgets.css" rel="stylesheet"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{background:#0d1117;color:#c9d1d9;font-family:'Segoe UI',sans-serif;height:100vh;overflow:hidden;display:flex;flex-direction:column}
#hdr{background:#161b22;border-bottom:1px solid #30363d;padding:8px 16px;display:flex;align-items:center;gap:10px;flex-shrink:0;flex-wrap:wrap}
#hdr h1{font-size:14px;font-weight:700;color:#58a6ff;letter-spacing:1px}
.hb{padding:5px 11px;border-radius:5px;border:1px solid #30363d;background:#21262d;color:#c9d1d9;font-size:11px;cursor:pointer;transition:.2s}
.hb:hover{border-color:#58a6ff;color:#58a6ff;background:#0d1f3c}
#main{display:flex;flex:1;overflow:hidden}
#side{width:270px;background:#161b22;border-right:1px solid #30363d;display:flex;flex-direction:column;overflow:hidden;flex-shrink:0}
#globe{flex:1;position:relative}
#cesiumContainer{width:100%;height:100%}

/* Sidebar */
.sec{border-bottom:1px solid #21262d;padding:10px 12px}
.st{font-size:9px;font-weight:700;color:#8b949e;text-transform:uppercase;letter-spacing:1px;margin-bottom:8px}
.dz{border:2px dashed #30363d;border-radius:8px;padding:14px 10px;text-align:center;cursor:pointer;background:#0d1117;transition:.2s;position:relative}
.dz:hover,.dz.on{border-color:#58a6ff;background:#0d1f3c}
.dz input{display:none}
.dz .ic{font-size:24px;margin-bottom:5px}
.dz p{font-size:10px;color:#8b949e;font-weight:600;margin-bottom:3px}
.dz small{font-size:9px;color:#484f58;line-height:1.5;display:block}
.fmt-badge{display:inline-block;background:#21262d;border:1px solid #30363d;border-radius:4px;padding:1px 5px;font-size:8px;color:#8b949e;margin:1px}

/* Layers */
#lys{flex:1;overflow-y:auto;padding:8px}
.ly{background:#0d1117;border:1px solid #21262d;border-radius:6px;padding:8px;margin-bottom:5px}
.lh{display:flex;align-items:center;gap:5px;margin-bottom:5px}
.ln{font-size:10px;font-weight:600;flex:1;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.tg{font-size:8px;padding:1px 5px;border-radius:8px;font-weight:700}
.tv{background:#14532d;color:#4ade80}.tr{background:#0c4a6e;color:#38bdf8}
.lc{display:flex;align-items:center;gap:5px;flex-wrap:wrap}
.lc label{font-size:9px;color:#8b949e}
input[type=range]{width:70px;accent-color:#58a6ff}
input[type=checkbox]{accent-color:#58a6ff;width:13px;height:13px}
.xb{padding:2px 7px;border-radius:4px;border:none;background:#da3633;color:#fff;font-size:9px;cursor:pointer}
.zb{padding:2px 7px;border-radius:4px;border:1px solid #30363d;background:#21262d;color:#c9d1d9;font-size:9px;cursor:pointer}
.zb:hover{border-color:#58a6ff}

/* Map overlays */
#coords{position:absolute;bottom:10px;right:10px;background:#161b22cc;border:1px solid #30363d;border-radius:5px;padding:4px 9px;font-size:10px;color:#8b949e;z-index:10;backdrop-filter:blur(4px)}
#toast{position:absolute;top:10px;left:50%;transform:translateX(-50%);background:#161b22ee;border:1px solid #30363d;border-radius:6px;padding:7px 16px;font-size:11px;color:#58a6ff;z-index:50;display:none;max-width:500px;text-align:center}
#info{position:absolute;bottom:10px;left:10px;background:#161b22ee;border:1px solid #30363d;border-radius:8px;padding:10px;max-width:300px;max-height:220px;overflow-y:auto;z-index:10;display:none;backdrop-filter:blur(4px)}
#info h3{font-size:11px;color:#58a6ff;margin-bottom:5px}
.iclose{float:right;cursor:pointer;color:#f85149}
.irow{display:flex;gap:6px;margin-bottom:2px;font-size:10px}
.ik{color:#58a6ff;min-width:80px;font-weight:600;flex-shrink:0}
#dropOverlay{position:absolute;inset:0;background:#0d1f3cdd;border:3px dashed #58a6ff;border-radius:8px;display:none;align-items:center;justify-content:center;z-index:100;font-size:18px;color:#58a6ff;font-weight:700;pointer-events:none}

.cesium-widget-credits,.cesium-viewer-toolbar,.cesium-viewer-animationContainer,
.cesium-viewer-timelineContainer,.cesium-viewer-fullscreenContainer{display:none!important}
::-webkit-scrollbar{width:4px}::-webkit-scrollbar-thumb{background:#30363d;border-radius:2px}
</style>
</head>
<body>

<div id="hdr">
  <span>🌍</span>
  <h1>GeoViewer 3D</h1>
  <span style="font-size:10px;color:#8b949e">Visualizador Geoespacial Universal</span>
  <div style="margin-left:auto;display:flex;gap:5px;flex-wrap:wrap">
    <button class="hb" onclick="V&&V.camera.flyHome(1.5)">🏠 Mundo</button>
    <button class="hb" onclick="flyToAll()">🔍 Ajustar</button>
    <button class="hb" onclick="toggleSat()" id="btnSat">🛰️ Satélite ON</button>
    <button class="hb" onclick="V&&V.scene.morphTo2D(1)">🗺️ 2D</button>
    <button class="hb" onclick="V&&V.scene.morphTo3D(1)">🌍 3D</button>
    <button class="hb" onclick="clearAll()" style="border-color:#da363366">🗑️ Limpar tudo</button>
  </div>
</div>

<div id="main">
  <div id="side">

    <!-- RASTER -->
    <div class="sec">
      <div class="st">🖼️ Dados Raster</div>
      <div class="dz" id="dzR"
           onclick="document.getElementById('rIn').click()"
           ondragover="dzDrag(event,'dzR',1)" ondragleave="dzDrag(event,'dzR',0)" ondrop="dzDrop(event,'r')">
        <input type="file" id="rIn" multiple
          accept=".tif,.tiff,.jpg,.jpeg,.jp2,.j2k,.j2c,.ecw,.img,.adf,.nc,.hdf,.h5,.png,.bmp,.gif,.webp"
          onchange="loadR(this)">
        <div class="ic">🖼️</div>
        <p>Clique ou arraste imagens</p>
        <small>
          <span class="fmt-badge">GeoTIFF</span>
          <span class="fmt-badge">TIF/TIFF</span>
          <span class="fmt-badge">JPG</span>
          <span class="fmt-badge">JPEG2000</span>
          <span class="fmt-badge">ECW*</span>
          <span class="fmt-badge">IMG</span>
          <span class="fmt-badge">PNG</span>
        </small>
        <small style="color:#58a6ff66;margin-top:4px">* ECW requer conversão prévia para GeoTIFF</small>
      </div>
    </div>

    <!-- VECTOR -->
    <div class="sec">
      <div class="st">📐 Dados Vetoriais</div>
      <div class="dz" id="dzV"
           onclick="document.getElementById('vIn').click()"
           ondragover="dzDrag(event,'dzV',1)" ondragleave="dzDrag(event,'dzV',0)" ondrop="dzDrop(event,'v')">
        <input type="file" id="vIn" multiple
          accept=".zip,.shp,.geojson,.json,.kml,.kmz,.dxf,.dgn,.dwg,.csv,.gpx,.gml,.tab,.mid,.mif"
          onchange="loadV(this)">
        <div class="ic">📍</div>
        <p>Clique ou arraste vetores</p>
        <small>
          <span class="fmt-badge">Shapefile</span>
          <span class="fmt-badge">DGN</span>
          <span class="fmt-badge">DXF</span>
          <span class="fmt-badge">DWG*</span>
          <span class="fmt-badge">KML/KMZ</span>
          <span class="fmt-badge">GeoJSON</span>
          <span class="fmt-badge">GPX</span>
          <span class="fmt-badge">CSV</span>
          <span class="fmt-badge">GML</span>
        </small>
        <small style="color:#58a6ff66;margin-top:4px">Shapefile: enviar .zip com .shp+.dbf+.prj<br>* DWG requer conversão para DXF</small>
      </div>
    </div>

    <!-- LAYERS -->
    <div class="sec" style="padding-bottom:4px">
      <div class="st" style="display:flex;justify-content:space-between;align-items:center">
        <span>🗂️ Camadas</span>
        <span id="layerCount" style="color:#484f58;font-size:9px">0 camadas</span>
      </div>
    </div>
    <div id="lys">
      <div id="emsg" style="text-align:center;padding:20px 10px;color:#484f58;font-size:10px;line-height:1.6">
        Nenhuma camada carregada.<br>Arraste ficheiros para as áreas acima<br>ou clique para seleccionar.
      </div>
    </div>

  </div>

  <div id="globe">
    <div id="cesiumContainer"></div>
    <div id="dropOverlay">⬇️ Largar para carregar</div>
    <div id="coords">Lat: — | Lon: — | Alt: —m</div>
    <div id="toast"></div>
    <div id="info">
      <span class="iclose" onclick="document.getElementById('info').style.display='none'">✕</span>
      <h3 id="iT">Atributos</h3>
      <div id="iB"></div>
    </div>
  </div>
</div>

<script>
// ═══════════════════════════════════════════════════════════
// CESIUM INIT
// ═══════════════════════════════════════════════════════════
Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJlYWE1OWUxNy1mMWZiLTQzYjYtYTQ0OS1kMWFjYmFkNjc4ZDkiLCJpZCI6NTc3MzMsImlhdCI6MTYyNzg0NTE4Mn0.XcKpgANiY19MC4bdFUXMVEBToBmqS8kuYpUlxJHYZxk';

let V = null;

try {
  V = new Cesium.Viewer('cesiumContainer', {
    terrainProvider: Cesium.createWorldTerrain({ requestVertexNormals:true, requestWaterMask:true }),
    imageryProvider: new Cesium.ArcGisMapServerImageryProvider({
      url: 'https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer'
    }),
    baseLayerPicker: false, navigationHelpButton: false, sceneModePicker: false,
    geocoder: false, homeButton: false, animation: false, timeline: false,
    fullscreenButton: false, shadows: true, orderIndependentTranslucency: true
  });

  V.scene.globe.enableLighting = true;
  V.scene.skyAtmosphere.show = true;
  V.scene.fog.enabled = true;
  V.scene.backgroundColor = Cesium.Color.fromCssColorString('#0d1117');
  V.scene.globe.depthTestAgainstTerrain = true;

  // Coords on hover
  const eH = new Cesium.ScreenSpaceEventHandler(V.scene.canvas);
  eH.setInputAction(e => {
    const ray = V.camera.getPickRay(e.endPosition);
    const pos = V.scene.globe.pick(ray, V.scene);
    if (pos) {
      const c = Cesium.Cartographic.fromCartesian(pos);
      document.getElementById('coords').textContent =
        `Lat: ${Cesium.Math.toDegrees(c.latitude).toFixed(5)}° | Lon: ${Cesium.Math.toDegrees(c.longitude).toFixed(5)}° | Alt: ${c.height.toFixed(0)}m`;
    }
  }, Cesium.ScreenSpaceEventType.MOUSE_MOVE);

  // Click → show attributes
  eH.setInputAction(e => {
    const p = V.scene.pick(e.position);
    if (p && p.id) showInfo(p.id);
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK);

  toast('✅ GeoViewer 3D pronto — carregue os seus dados!', 4000);

} catch(err) {
  console.error(err);
  toast('❌ Erro ao inicializar o globo: ' + err.message, 0);
}

// Globe drag & drop
const globe = document.getElementById('globe');
const dropOv = document.getElementById('dropOverlay');
globe.addEventListener('dragover', e => { e.preventDefault(); dropOv.style.display='flex'; });
globe.addEventListener('dragleave', e => { if(!globe.contains(e.relatedTarget)) dropOv.style.display='none'; });
globe.addEventListener('drop', e => {
  e.preventDefault(); dropOv.style.display='none';
  [...e.dataTransfer.files].forEach(f => isRaster(f) ? loadRasterFile(f) : loadVectorFile(f));
});

function isRaster(f) {
  const e = f.name.split('.').pop().toLowerCase();
  return ['tif','tiff','jpg','jpeg','jp2','j2k','j2c','ecw','img','png','bmp','gif','webp','nc','hdf','h5'].includes(e);
}

// ═══════════════════════════════════════════════════════════
// LAYER SYSTEM
// ═══════════════════════════════════════════════════════════
const layers = [];
const VCOLS = ['#4ade80','#f59e0b','#f87171','#a78bfa','#34d399','#60a5fa','#fb923c','#e879f9','#2dd4bf'];
let vci = 0;
function vc() { return VCOLS[vci++ % VCOLS.length]; }

function addLayer(name, type, obj) {
  const id = 'L' + Date.now() + Math.random().toString(36).slice(2);
  layers.push({ id, name, type, obj, vis: true, opacity: 1 });
  renderLayers();
  return id;
}

function renderLayers() {
  const el = document.getElementById('lys');
  document.getElementById('emsg').style.display = layers.length ? 'none' : 'block';
  document.getElementById('layerCount').textContent = layers.length + ' camada' + (layers.length !== 1 ? 's' : '');
  [...el.querySelectorAll('.ly')].forEach(n => n.remove());
  [...layers].reverse().forEach(l => {
    const d = document.createElement('div'); d.className = 'ly';
    const tc = l.type === 'raster' ? 'tr' : 'tv';
    const tl = l.type === 'raster' ? 'raster' : 'vetor';
    d.innerHTML = `
      <div class="lh">
        <span>${l.type === 'raster' ? '🖼️' : '📍'}</span>
        <span class="ln" title="${l.name}">${l.name}</span>
        <span class="tg ${tc}">${tl}</span>
      </div>
      <div class="lc">
        <input type="checkbox" ${l.vis ? 'checked' : ''} onchange="lyV('${l.id}',this.checked)">
        <label>Vis.</label>
        <input type="range" min="0" max="100" value="${Math.round(l.opacity*100)}" oninput="lyO('${l.id}',this.value)">
        <button class="zb" onclick="lyZ('${l.id}')">🔍</button>
        <button class="xb" onclick="lyD('${l.id}')">✕</button>
      </div>`;
    el.appendChild(d);
  });
}

function lyV(id, v) {
  const l = layers.find(x => x.id === id); if (!l) return; l.vis = v;
  try { if (l.type === 'raster') l.obj.show = v; else l.obj.show = v; } catch(e) {}
}
function lyO(id, v) {
  const l = layers.find(x => x.id === id); if (!l) return; l.opacity = v/100;
  try { if (l.type === 'raster') l.obj.alpha = l.opacity; } catch(e) {}
}
function lyZ(id) {
  const l = layers.find(x => x.id === id); if (!l || !V) return;
  try { V.flyTo(l.obj, { duration: 1.5 }); } catch(e) {}
}
function lyD(id) {
  const idx = layers.findIndex(x => x.id === id); if (idx < 0) return;
  const l = layers[idx];
  try {
    if (l.type === 'raster') V.imageryLayers.remove(l.obj, true);
    else V.dataSources.remove(l.obj, true);
  } catch(e) {}
  layers.splice(idx, 1); renderLayers();
}
function clearAll() {
  [...layers].forEach(l => lyD(l.id));
}
function flyToAll() {
  if (!V || !layers.length) return;
  try { V.flyTo(layers.map(l => l.obj), { duration: 1.5 }); } catch(e) {}
}

window.clearAll = clearAll;
window.flyToAll = flyToAll;

// ═══════════════════════════════════════════════════════════
// SAT TOGGLE
// ═══════════════════════════════════════════════════════════
let satOn = true;
function toggleSat() {
  satOn = !satOn;
  if (V) for (let i = 0; i < V.imageryLayers.length; i++) V.imageryLayers.get(i).show = satOn;
  document.getElementById('btnSat').textContent = satOn ? '🛰️ Satélite ON' : '🗺️ Satélite OFF';
}
window.toggleSat = toggleSat;

// ═══════════════════════════════════════════════════════════
// FEATURE INFO
// ═══════════════════════════════════════════════════════════
function showInfo(ent) {
  if (!ent) return;
  document.getElementById('iT').textContent = ent.name || 'Feição';
  const names = ent.properties?.propertyNames || [];
  let h = '';
  names.slice(0, 20).forEach(k => {
    const v = ent.properties[k]?.getValue?.();
    if (v != null && v !== '') h += `<div class="irow"><span class="ik">${k}</span><span style="word-break:break-all">${v}</span></div>`;
  });
  document.getElementById('iB').innerHTML = h || '<span style="color:#484f58;font-size:10px">Sem atributos</span>';
  document.getElementById('info').style.display = 'block';
}

// ═══════════════════════════════════════════════════════════
// TOAST
// ═══════════════════════════════════════════════════════════
let toastTimer = null;
function toast(msg, ms = 0) {
  const el = document.getElementById('toast'); el.textContent = msg; el.style.display = 'block';
  if (toastTimer) clearTimeout(toastTimer);
  if (ms) toastTimer = setTimeout(() => el.style.display = 'none', ms);
}

// ═══════════════════════════════════════════════════════════
// DRAG ZONES
// ═══════════════════════════════════════════════════════════
function dzDrag(e, id, on) { e.preventDefault(); document.getElementById(id).classList.toggle('on', !!on); }
function dzDrop(e, t) {
  e.preventDefault(); e.currentTarget.classList.remove('on');
  [...e.dataTransfer.files].forEach(f => t === 'r' ? loadRasterFile(f) : loadVectorFile(f));
}

// ═══════════════════════════════════════════════════════════
// RASTER LOADER
// ═══════════════════════════════════════════════════════════
window.loadR = inp => { [...inp.files].forEach(loadRasterFile); inp.value = ''; };

async function loadRasterFile(file) {
  if (!V) { toast('❌ Globo não inicializado', 4000); return; }
  toast(`⏳ A carregar raster: ${file.name}...`);
  const ext = file.name.split('.').pop().toLowerCase();
  try {
    if (['tif', 'tiff'].includes(ext)) {
      await loadGeoTIFF(file);
    } else if (['jpg','jpeg','png','bmp','gif','webp'].includes(ext)) {
      await loadImageOverlay(file);
    } else if (['jp2','j2k','j2c'].includes(ext)) {
      toast(`⚠️ JPEG2000 (${file.name}): suporte limitado no browser. Converta para GeoTIFF no QGIS para melhor resultado.`, 6000);
      await loadImageOverlay(file); // tenta carregar como imagem
    } else if (ext === 'ecw') {
      toast(`⚠️ ECW não suportado directamente no browser. Converta para GeoTIFF: QGIS → Guardar como → GeoTIFF`, 7000);
    } else if (['img','adf'].includes(ext)) {
      toast(`⚠️ ${ext.toUpperCase()} é formato de sistema de ficheiros raster. Converta para GeoTIFF no QGIS primeiro.`, 6000);
    } else {
      await loadImageOverlay(file);
    }
  } catch(err) { toast(`❌ Erro ao carregar ${file.name}: ${err.message}`, 6000); console.error(err); }
}

async function loadGeoTIFF(file) {
  if (!window.GeoTIFF) await ls('https://cdnjs.cloudflare.com/ajax/libs/geotiff.js/2.0.7/geotiff.bundle.min.js');
  const buf = await file.arrayBuffer();
  const tiff = await GeoTIFF.fromArrayBuffer(buf);
  const img = await tiff.getImage();
  const [w, s, e, n] = img.getBoundingBox();

  // Validate bounds
  if (w < -180 || e > 180 || s < -90 || n > 90 || w >= e || s >= n) {
    toast(`⚠️ "${file.name}": coordenadas fora do intervalo WGS84. Verifique a projecção (deve ser EPSG:4326).`, 7000);
    return;
  }

  const data = await img.readRasters({ interleave: true });
  const IW = img.getWidth(), IH = img.getHeight(), spp = img.getSamplesPerPixel();
  const canvas = document.createElement('canvas'); canvas.width = IW; canvas.height = IH;
  const c2 = canvas.getContext('2d'); const id2 = c2.createImageData(IW, IH);

  const mn = [], mx = [];
  for (let b = 0; b < Math.min(spp, 3); b++) {
    let a = Infinity, z = -Infinity;
    for (let i = b; i < data.length; i += spp) { if (data[i] !== undefined) { a = Math.min(a, data[i]); z = Math.max(z, data[i]); } }
    mn.push(a); mx.push(z);
  }
  for (let i = 0, p = 0; i < IW*IH; i++, p += 4) {
    if (spp >= 3) {
      id2.data[p]   = nv(data[i*spp],   mn[0], mx[0]);
      id2.data[p+1] = nv(data[i*spp+1], mn[1], mx[1]);
      id2.data[p+2] = nv(data[i*spp+2], mn[2], mx[2]);
      id2.data[p+3] = spp >= 4 ? data[i*spp+3] : 255;
    } else {
      const v = nv(data[i*spp], mn[0], mx[0]);
      id2.data[p] = id2.data[p+1] = id2.data[p+2] = v; id2.data[p+3] = 255;
    }
  }
  c2.putImageData(id2, 0, 0);
  const rect = Cesium.Rectangle.fromDegrees(w, s, e, n);
  const prov = new Cesium.SingleTileImageryProvider({ url: canvas.toDataURL('image/png'), rectangle: rect });
  const lyr = V.imageryLayers.addImageryProvider(prov);
  addLayer(file.name, 'raster', lyr);
  V.camera.flyTo({ destination: rect, duration: 1.5 });
  toast(`✅ GeoTIFF "${file.name}" carregado em 3D!`, 3000);
}

async function loadImageOverlay(file) {
  const url = URL.createObjectURL(file);
  // Without georef, ask user or try to detect from filename
  const rect = Cesium.Rectangle.fromDegrees(-180, -90, 180, 90);
  const prov = new Cesium.SingleTileImageryProvider({ url, rectangle: rect });
  const lyr = V.imageryLayers.addImageryProvider(prov);
  addLayer(file.name, 'raster', lyr);
  toast(`⚠️ "${file.name}" sem georreferenciação detectada — exibido no globo todo. Use GeoTIFF para posicionamento preciso.`, 6000);
}

function nv(v, mn, mx) { return mx === mn ? 128 : Math.round(((v-mn)/(mx-mn))*255); }

// ═══════════════════════════════════════════════════════════
// VECTOR LOADER
// ═══════════════════════════════════════════════════════════
window.loadV = inp => { [...inp.files].forEach(loadVectorFile); inp.value = ''; };

async function loadVectorFile(file) {
  if (!V) { toast('❌ Globo não inicializado', 4000); return; }
  toast(`⏳ A carregar: ${file.name}...`);
  const ext = file.name.split('.').pop().toLowerCase();
  try {
    switch(ext) {
      case 'zip': await loadShapefile(file); break;
      case 'shp': toast('⚠️ Envie o Shapefile como .zip contendo .shp + .dbf + .prj', 6000); break;
      case 'geojson': case 'json': await addGeoJSON(JSON.parse(await file.text()), file.name); break;
      case 'kml': await addKML(await file.text(), file.name); break;
      case 'kmz': await loadKMZ(file); break;
      case 'dxf': await addDXF(await file.text(), file.name); break;
      case 'dgn': await addDGN(await file.arrayBuffer(), file.name); break;
      case 'dwg': toast(`⚠️ DWG (${file.name}): formato binário proprietário da Autodesk. Converta para DXF no AutoCAD ou no QGIS com o plugin DWG Importer.`, 7000); break;
      case 'gpx': await addGeoJSON(gpx2gj(await file.text()), file.name); break;
      case 'csv': await addCSV(await file.text(), file.name); break;
      case 'gml': await addGeoJSON(gml2gj(await file.text()), file.name); break;
      case 'tab': case 'mid': case 'mif': toast(`⚠️ MapInfo ${ext.toUpperCase()}: converta para GeoJSON ou Shapefile no QGIS primeiro.`, 6000); break;
      default: toast(`⚠️ Formato .${ext} não reconhecido. Formatos suportados: ZIP(SHP), GeoJSON, KML, KMZ, DXF, DGN, GPX, CSV, GML`, 6000);
    }
  } catch(err) { toast(`❌ Erro em ${file.name}: ${err.message}`, 6000); console.error(err); }
}

// ── Shapefile (ZIP) ─────────────────────────────────────────
async function loadShapefile(file) {
  if (!window.shp) await ls('https://cdnjs.cloudflare.com/ajax/libs/shpjs/4.0.2/shp.js');
  const gj = await shp(await file.arrayBuffer());
  await addGeoJSON(gj, file.name.replace('.zip',''));
}

// ── GeoJSON ─────────────────────────────────────────────────
async function addGeoJSON(gj, name) {
  if (!gj || (!gj.features && gj.type !== 'FeatureCollection' && !gj.geometry)) {
    toast(`⚠️ "${name}": estrutura GeoJSON inválida`, 4000); return;
  }
  const col = vc();
  const ds = await Cesium.GeoJsonDataSource.load(gj, {
    stroke: Cesium.Color.fromCssColorString(col),
    fill: Cesium.Color.fromCssColorString(col).withAlpha(0.35),
    strokeWidth: 3,
    clampToGround: true
  });
  V.dataSources.add(ds);
  addLayer(name, 'vector', ds);
  V.flyTo(ds, { duration: 1.5 });
  const count = ds.entities.values.length;
  toast(`✅ "${name}" — ${count} feição(ões) em 3D!`, 3000);
}

// ── KML ─────────────────────────────────────────────────────
async function addKML(text, name) {
  const blob = new Blob([text], { type: 'application/vnd.google-earth.kml+xml' });
  const url = URL.createObjectURL(blob);
  const ds = await Cesium.KmlDataSource.load(url, {
    camera: V.scene.camera, canvas: V.scene.canvas, clampToGround: true
  });
  const col = vc();
  ds.entities.values.forEach(ent => {
    if (ent.polyline) {
      ent.polyline.material = new Cesium.PolylineOutlineMaterialProperty({
        color: Cesium.Color.fromCssColorString(col), outlineWidth: 1, outlineColor: Cesium.Color.WHITE
      });
      ent.polyline.width = 3; ent.polyline.clampToGround = true;
    }
    if (ent.point) {
      ent.point.pixelSize = 10; ent.point.color = Cesium.Color.fromCssColorString(col);
      ent.point.outlineColor = Cesium.Color.WHITE; ent.point.outlineWidth = 1.5;
      ent.point.heightReference = Cesium.HeightReference.CLAMP_TO_GROUND;
    }
    if (ent.polygon) {
      ent.polygon.material = Cesium.Color.fromCssColorString(col).withAlpha(0.4);
      ent.polygon.outline = true; ent.polygon.outlineColor = Cesium.Color.fromCssColorString(col);
    }
  });
  V.dataSources.add(ds); addLayer(name, 'vector', ds); V.flyTo(ds, { duration: 1.5 });
  toast(`✅ KML "${name}" — ${ds.entities.values.length} feição(ões) em 3D!`, 3000);
}

// ── KMZ ─────────────────────────────────────────────────────
async function loadKMZ(file) {
  const zip = await JSZip.loadAsync(await file.arrayBuffer());
  const kf = Object.values(zip.files).find(f => f.name.toLowerCase().endsWith('.kml'));
  if (!kf) { toast(`⚠️ KMZ sem ficheiro KML interno`, 4000); return; }
  await addKML(await kf.async('text'), file.name);
}

// ── DXF ─────────────────────────────────────────────────────
async function addDXF(text, name) {
  const lines = text.split(/\r?\n/).map(l => l.trim());
  const ds = new Cesium.CustomDataSource(name);
  const col = Cesium.Color.fromCssColorString(vc());
  let i = 0, count = 0;

  while (i < lines.length) {
    if (lines[i] === '0' && i+1 < lines.length) {
      const tp = lines[i+1].trim().toUpperCase();

      if (tp === 'LINE') {
        let x1,y1,z1=0,x2,y2,z2=0; i+=2;
        while(i<lines.length&&lines[i]!=='0'){const c=+lines[i],v=+lines[i+1];if(c===10)x1=v;if(c===20)y1=v;if(c===30)z1=v;if(c===11)x2=v;if(c===21)y2=v;if(c===31)z2=v;i+=2;}
        if(x1!==undefined){
          ds.entities.add({polyline:{positions:Cesium.Cartesian3.fromDegreesArrayHeights([x1,y1,z1,x2,y2,z2]),width:2,material:col,clampToGround:z1===0&&z2===0}});count++;
        }
      }
      else if (tp === 'POINT') {
        let x,y,z=0; i+=2;
        while(i<lines.length&&lines[i]!=='0'){const c=+lines[i],v=+lines[i+1];if(c===10)x=v;if(c===20)y=v;if(c===30)z=v;i+=2;}
        if(x!==undefined){
          ds.entities.add({position:Cesium.Cartesian3.fromDegrees(x,y,z),point:{pixelSize:8,color:col,outlineColor:Cesium.Color.WHITE,outlineWidth:1,heightReference:z===0?Cesium.HeightReference.CLAMP_TO_GROUND:Cesium.HeightReference.NONE}});count++;
        }
      }
      else if (tp === 'LWPOLYLINE' || tp === 'POLYLINE') {
        const verts=[];let cv_={},closed=false; i+=2;
        while(i<lines.length&&lines[i]!=='0'){
          const c=+lines[i],v=+lines[i+1];
          if(c===10)cv_.x=v;
          if(c===20){cv_.y=v;verts.push({...cv_});cv_={};}
          if(c===70&&(v&1))closed=true;
          i+=2;
        }
        if(verts.length>1){
          if(closed&&verts.length>2)verts.push(verts[0]);
          ds.entities.add({polyline:{positions:Cesium.Cartesian3.fromDegreesArray(verts.flatMap(v=>[v.x,v.y])),width:2,material:col,clampToGround:true}});count++;
        }
      }
      else if (tp === 'ARC' || tp === 'CIRCLE') {
        let cx,cy,r,sa=0,ea=360; i+=2;
        while(i<lines.length&&lines[i]!=='0'){const c=+lines[i],v=+lines[i+1];if(c===10)cx=v;if(c===20)cy=v;if(c===40)r=v;if(c===50)sa=v;if(c===51)ea=v;i+=2;}
        if(cx!==undefined&&r!==undefined){
          const pts=[],steps=32;
          const angRange=ea>sa?ea-sa:ea-sa+360;
          for(let s=0;s<=steps;s++){const a=(sa+angRange*s/steps)*Math.PI/180;pts.push(cx+r*Math.cos(a));pts.push(cy+r*Math.sin(a));}
          ds.entities.add({polyline:{positions:Cesium.Cartesian3.fromDegreesArray(pts),width:2,material:col,clampToGround:true}});count++;
        }
      }
      else if (tp === 'TEXT' || tp === 'MTEXT') {
        let tx,ty,txt=''; i+=2;
        while(i<lines.length&&lines[i]!=='0'){const c=+lines[i];if(c===10)tx=+lines[i+1];if(c===20)ty=+lines[i+1];if(c===1||c===3)txt+=lines[i+1];i+=2;}
        if(tx!==undefined&&txt){
          ds.entities.add({position:Cesium.Cartesian3.fromDegrees(tx,ty),label:{text:txt,font:'12px sans-serif',fillColor:Cesium.Color.WHITE,outlineColor:Cesium.Color.BLACK,outlineWidth:2,style:Cesium.LabelStyle.FILL_AND_OUTLINE,heightReference:Cesium.HeightReference.CLAMP_TO_GROUND,disableDepthTestDistance:Number.POSITIVE_INFINITY}});count++;
        }
      }
      else if (tp === 'INSERT') {
        let ix,iy; i+=2;
        while(i<lines.length&&lines[i]!=='0'){const c=+lines[i],v=+lines[i+1];if(c===10)ix=v;if(c===20)iy=v;i+=2;}
        if(ix!==undefined){ds.entities.add({position:Cesium.Cartesian3.fromDegrees(ix,iy),point:{pixelSize:6,color:col,heightReference:Cesium.HeightReference.CLAMP_TO_GROUND}});count++;}
      }
      else { i+=2; }
    } else { i++; }
  }

  V.dataSources.add(ds); addLayer(name, 'vector', ds);
  if (count) { V.flyTo(ds, { duration: 1.5 }); toast(`✅ DXF "${name}": ${count} entidade(s) em 3D!`, 3000); }
  else toast(`⚠️ DXF "${name}": nenhuma entidade reconhecida. Verifique se as coordenadas são geográficas (lon/lat).`, 6000);
}

// ── DGN ─────────────────────────────────────────────────────
async function addDGN(buffer, name) {
  // DGN V8 — tentativa de leitura básica de elementos
  // Nota: DGN é formato binário complexo — suporte básico apenas
  toast(`⚠️ DGN "${name}": formato binário MicroStation. Suporte básico.`, 0);
  try {
    // Tentativa de ler como texto (DGN V7 pode ter segmentos legíveis)
    const text = new TextDecoder('utf-8', { fatal: false }).decode(new Uint8Array(buffer));
    // Extrair coordenadas numéricas que possam estar no ficheiro
    const coordPairs = [];
    const nums = text.match(/-?[\d]+\.[\d]+/g);
    if (nums && nums.length >= 4) {
      for (let i = 0; i < Math.min(nums.length-1, 200); i += 2) {
        const lon = parseFloat(nums[i]), lat = parseFloat(nums[i+1]);
        if (lon >= -180 && lon <= 180 && lat >= -90 && lat <= 90) coordPairs.push([lon, lat]);
      }
    }
    if (coordPairs.length > 0) {
      const gj = { type:'FeatureCollection', features: coordPairs.map(([lon,lat]) => ({
        type:'Feature', geometry:{type:'Point',coordinates:[lon,lat]}, properties:{}
      }))};
      await addGeoJSON(gj, name);
      toast(`✅ DGN "${name}": ${coordPairs.length} pontos extraídos. Para melhor resultado, converta no MicroStation ou QGIS para DXF/GeoJSON.`, 6000);
    } else {
      toast(`⚠️ DGN "${name}": não foi possível extrair geometrias automaticamente. Recomenda-se converter para DXF ou GeoJSON no MicroStation ou QGIS.`, 8000);
    }
  } catch(e) {
    toast(`⚠️ DGN "${name}": converta para DXF/GeoJSON no MicroStation ou QGIS para carregar correctamente.`, 7000);
  }
}

// ── GPX ─────────────────────────────────────────────────────
function gpx2gj(text) {
  const xml = new DOMParser().parseFromString(text, 'text/xml');
  const features = [];
  // Tracks
  xml.querySelectorAll('trk').forEach(trk => {
    const name = trk.querySelector('name')?.textContent || 'Track';
    const coords = [...trk.querySelectorAll('trkpt')].map(p => [+p.getAttribute('lon'), +p.getAttribute('lat')]);
    if (coords.length > 1) features.push({type:'Feature',geometry:{type:'LineString',coordinates:coords},properties:{name}});
  });
  // Waypoints
  xml.querySelectorAll('wpt').forEach(wpt => {
    const name = wpt.querySelector('name')?.textContent || 'Waypoint';
    features.push({type:'Feature',geometry:{type:'Point',coordinates:[+wpt.getAttribute('lon'),+wpt.getAttribute('lat')]},properties:{name}});
  });
  // Routes
  xml.querySelectorAll('rte').forEach(rte => {
    const name = rte.querySelector('name')?.textContent || 'Route';
    const coords = [...rte.querySelectorAll('rtept')].map(p => [+p.getAttribute('lon'), +p.getAttribute('lat')]);
    if (coords.length > 1) features.push({type:'Feature',geometry:{type:'LineString',coordinates:coords},properties:{name}});
  });
  return { type:'FeatureCollection', features };
}

// ── CSV ─────────────────────────────────────────────────────
async function addCSV(text, name) {
  // Auto-detect separator
  const firstLine = text.split('\n')[0];
  const sep = firstLine.includes(';') ? ';' : firstLine.includes('\t') ? '\t' : ',';
  const rows = text.trim().split('\n').map(r => r.split(sep).map(c => c.trim().replace(/^"|"$/g,'')));
  const hdr = rows[0].map(h => h.toLowerCase().trim());
  const li = hdr.findIndex(h => ['lat','latitude','y','ylat','latitud'].includes(h));
  const oi = hdr.findIndex(h => ['lon','lng','longitude','x','xlon','longitud'].includes(h));
  if (li < 0 || oi < 0) { toast(`⚠️ CSV "${name}": colunas lat/lon não encontradas. Cabeçalhos detectados: ${hdr.join(', ')}`, 7000); return; }
  const col = Cesium.Color.fromCssColorString(vc());
  const ds = new Cesium.CustomDataSource(name);
  let count = 0;
  rows.slice(1).forEach((r, i) => {
    const lat = +r[li], lon = +r[oi];
    if (isNaN(lat) || isNaN(lon)) return;
    const ent = ds.entities.add({
      name: r[hdr.indexOf('name')] || r[hdr.indexOf('nome')] || `Ponto ${i+1}`,
      position: Cesium.Cartesian3.fromDegrees(lon, lat),
      point: { pixelSize: 9, color: col, outlineColor: Cesium.Color.WHITE, outlineWidth: 1.5, heightReference: Cesium.HeightReference.CLAMP_TO_GROUND, disableDepthTestDistance: Number.POSITIVE_INFINITY }
    });
    hdr.forEach((k, j) => { try { ent.properties.addProperty(k, r[j] || ''); } catch(e) {} });
    count++;
  });
  V.dataSources.add(ds); addLayer(name, 'vector', ds);
  if (count) { V.flyTo(ds, { duration: 1.5 }); toast(`✅ CSV "${name}": ${count} ponto(s) em 3D!`, 3000); }
  else toast(`⚠️ CSV "${name}": nenhum ponto válido encontrado`, 4000);
}

// ── GML ─────────────────────────────────────────────────────
function gml2gj(text) {
  const xml = new DOMParser().parseFromString(text, 'text/xml');
  const features = [];
  // GML 2/3 — Points, LineStrings, Polygons
  xml.querySelectorAll('Point, pos, coordinates').forEach(el => {
    const coords = el.textContent.trim().split(/[\s,]+/).map(Number).filter(n => !isNaN(n));
    if (coords.length >= 2) features.push({type:'Feature',geometry:{type:'Point',coordinates:[coords[0],coords[1]]},properties:{}});
  });
  xml.querySelectorAll('LineString posList, LineString coordinates').forEach(el => {
    const nums = el.textContent.trim().split(/[\s,]+/).map(Number);
    const coords = [];
    for (let i = 0; i < nums.length - 1; i += 2) coords.push([nums[i], nums[i+1]]);
    if (coords.length > 1) features.push({type:'Feature',geometry:{type:'LineString',coordinates:coords},properties:{}});
  });
  return { type:'FeatureCollection', features };
}

// ── Utils ────────────────────────────────────────────────────
function ls(src) {
  return new Promise((res, rej) => {
    if (document.querySelector(`script[src="${src}"]`)) { res(); return; }
    const s = document.createElement('script'); s.src = src;
    s.onload = res; s.onerror = () => rej(new Error('Falha ao carregar: ' + src));
    document.head.appendChild(s);
  });
}
</script>
</body>
</html>