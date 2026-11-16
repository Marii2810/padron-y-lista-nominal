<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Mapa Electoral - Distritos 11 y 15</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    body { margin: 0; font-family: Arial, sans-serif; display: flex; flex-direction: column; height: 100vh; }
    #map { flex: 1; }
    #reporte { background: #f8f9fa; padding: 15px 25px; border-top: 2px solid #ccc; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; background: white; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
    th { background-color: #2c3e50; color: white; }
  </style>
</head>
<body>

<div id="map"></div>

<div id="reporte">
  <h3>Reporte Consolidado por Distrito Electoral</h3>
  <table>
    <thead>
      <tr>
        <th>Distrito Local</th>
        <th>Total Secciones</th>
        <th>Padrón Electoral Total</th>
        <th>Lista Nominal Total</th>
        <th>% Cobertura (LN / Padrón)</th>
      </tr>
    </thead>
    <tbody id="tabla-body"></tbody>
  </table>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
  const map = L.map('map').setView([19.4326, -99.1332], 12); // Ajusta las coordenadas iniciales si es necesario
  L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
    attribution: '&copy; OpenStreetMap'
  }).addTo(map);

  // Asegúrate de que los nombres coincidan exactamente con tus archivos en el repo
  const archivos = [
    'Distrito Local 11_Lista_Nominal_Padron.geojson',
    'Distrito Local 15_Lista_Nominal_Padron.geojson'
  ];

  const resumenDistritos = {};
  const grupoCapas = L.featureGroup().addTo(map);

  Promise.all(archivos.map(url => fetch(encodeURI(url)).then(res => res.json())))
    .then(datasets => {
      datasets.forEach(data => {
        const capa = L.geoJSON(data, {
          style: feature => ({
            color: feature.properties.DISTRITO_L == 11 ? '#1f78b4' : '#33a02c',
            weight: 1.5,
            fillOpacity: 0.45
          }),
          onEachFeature: (feature, layer) => {
            const p = feature.properties;
            const dist = p['DISTRITO_L'];
            const sec = p['SECCION'];
            const padron = Number(p['PADRÓN ELECTORAL TOTAL']) || 0;
            const nominal = Number(p['LISTA NOMINAL TOTAL']) || 0;

            // Acumular datos para el reporte por distrito
            if (!resumenDistritos[dist]) {
              resumenDistritos[dist] = { secciones: 0, padron: 0, nominal: 0 };
            }
            resumenDistritos[dist].secciones += 1;
            resumenDistritos[dist].padron += padron;
            resumenDistritos[dist].nominal += nominal;

            // Popup al hacer clic en cada sección
            layer.bindPopup(`
              <strong>Distrito Local:</strong> ${dist}<br>
              <strong>Sección Electoral:</strong> ${sec}<br>
              <strong>Padrón Electoral:</strong> ${padron.toLocaleString()}<br>
              <strong>Lista Nominal:</strong> ${nominal.toLocaleString()}
            `);
          }
        });
        grupoCapas.addLayer(capa);
      });

      // Centrar el mapa automáticamente en tus polígonos
      map.fitBounds(grupoCapas.getBounds());
      renderizarTabla();
    });

  function renderizarTabla() {
    const tbody = document.getElementById('tabla-body');
    Object.keys(resumenDistritos).sort((a, b) => a - b).forEach(dist => {
      const d = resumenDistritos[dist];
      const pct = d.padron > 0 ? ((d.nominal / d.padron) * 100).toFixed(2) : 0;
      tbody.innerHTML += `
        <tr>
          <td><strong>Distrito ${dist}</strong></td>
          <td>${d.secciones}</td>
          <td>${d.padron.toLocaleString()}</td>
          <td>${d.nominal.toLocaleString()}</td>
          <td>${pct}%</td>
        </tr>
      `;
    });
  }
</script>
</body>
</html>