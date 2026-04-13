# 🌍 GeoViewer 3D

Visualizador geoespacial 3D universal no browser, powered by [CesiumJS](https://cesium.com/).

![GeoViewer 3D](https://img.shields.io/badge/CesiumJS-1.114-blue) ![License](https://img.shields.io/badge/license-MIT-green) ![Node](https://img.shields.io/badge/Node.js-v18%2B-brightgreen)

---

## ✨ Funcionalidades

- 🌍 **Globo 3D interactivo** com giro, zoom e inclinação
- 🛰️ **Camadas base** — Google Satélite, Google Híbrido, Google Terreno, OpenStreetMap
- ⛰️ **Exagero de terreno** ajustável (1x a 50x)
- 📂 **Carregar dados** por clique ou drag & drop
- 🗂️ **Painel de camadas** com visibilidade e opacidade

---

## 🗂️ Formatos suportados

### Raster
| Formato | Suporte |
|---|---|
| GeoTIFF (.tif/.tiff) | ✅ Total — georreferenciado |
| JPG / PNG | ✅ Overlay |
| JPEG2000 (.jp2) | ⚠️ Parcial |
| ECW | ❌ Converter para GeoTIFF |

### Vetorial
| Formato | Suporte |
|---|---|
| Shapefile (.zip) | ✅ Total |
| GeoJSON | ✅ Total |
| KML / KMZ | ✅ Total |
| DXF | ✅ LINE, POINT, LWPOLYLINE, ARC, TEXT |
| DGN | ⚠️ Básico |
| DWG | ❌ Converter para DXF |
| GPX | ✅ Total |
| CSV (lat/lon) | ✅ Total |
| GML | ✅ Básico |

---

## 🚀 Instalação e arranque

### Pré-requisitos
- [Node.js](https://nodejs.org) v18+

### Passos

```bash
# 1. Clona o repositório
git clone https://github.com/flaviocaos/geoviewer3d.git
cd geoviewer3d

# 2. Arranca o servidor
node server.js

# 3. Abre no browser
# http://localhost:3000
```

---

## 🎮 Navegação 3D

| Acção | Resultado |
|---|---|
| Clique esquerdo + arrastar | Move o mapa |
| Clique direito + arrastar | Gira e inclina em 3D |
| Scroll | Zoom in/out |
| Duplo clique | Zoom para esse ponto |
| Clique numa feição | Mostra os atributos |

---

## 📁 Estrutura do projecto

```
geoviewer3d/
├── index.html    # App principal (CesiumJS + UI)
├── server.js     # Servidor Node.js local
└── README.md
```

---

## 💡 Notas

- **Shapefile**: enviar como `.zip` com `.shp` + `.dbf` + `.prj`
- **GeoTIFF**: deve estar em **EPSG:4326 (WGS84)**
- **DXF**: coordenadas devem ser geográficas (lon/lat)
- Requer ligação à internet para carregar os tiles de satélite e terreno

---

## 🛠️ Tecnologias

- [CesiumJS](https://cesium.com/) — motor 3D geoespacial
- [shpjs](https://github.com/calvinmetcalf/shapefile-js) — leitura de Shapefiles
- [geotiff.js](https://geotiffjs.github.io/) — leitura de GeoTIFF
- [JSZip](https://stuk.github.io/jszip/) — leitura de KMZ/ZIP
- Node.js — servidor local

---

## 📄 Licença

MIT © [flaviocaos](https://github.com/flaviocaos)