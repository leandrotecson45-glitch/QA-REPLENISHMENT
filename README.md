<!DOCTYPE html>
<html>
<head>
  <title>QA Portal - Map + Fieldworker Check</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <style>
    body{font-family:Arial; background:#0f172a; color:white; padding:10px;}
    header{padding:10px; text-align:center; background:linear-gradient(45deg,#22c55e,#0ea5e9); font-size:22px;}
    input{padding:8px; margin:5px; border-radius:8px; border:none;}
    table{width:95%; margin:10px auto; border-collapse:collapse; font-size:12px;}
    th,td{border:1px solid #ccc; padding:6px; text-align:center;}
    th{background:#1e40af; cursor:pointer;}
    img{width:60px; border-radius:6px;}
    .day-row{font-weight:bold;}
    .normal{background:#1e293b;}
    .warning{background:#facc15; color:black;}
    .over{background:#dc2626;}
    #map{height:400px; width:95%; margin:10px auto; border-radius:8px;}
  </style>
</head>
<body>

<header>📝 QA Portal - Fieldworker Review with Map</header>

<div style="text-align:center;">
  Upload Fieldworker JSON: <input type="file" id="jsonUpload" accept=".json"><br>
  Filter Fieldworker: <input type="text" id="filterWorker" placeholder="Enter name">
  Filter Date: <input type="date" id="filterDate">
  Expected Daily KM: <input type="number" id="expectedKM" value="10">
</div>

<div id="map"></div>

<table id="reportTable">
<thead>
<tr>
<th onclick="sortTable(0)">Date</th>
<th>Photo</th>
<th>File</th>
<th>Lat</th>
<th>Lng</th>
<th onclick="sortTable(5)">KM</th>
<th onclick="sortTable(6)">Allowance</th>
<th>Fieldworker</th>
</tr>
</thead>
<tbody></tbody>
</table>

<script>
let trips=[];
let map = L.map('map').setView([14.6, 121.0], 10);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
  attribution: '&copy; OpenStreetMap contributors'
}).addTo(map);
let markers=[];

// Load JSON
document.getElementById("jsonUpload").addEventListener("change", e=>{
  const file=e.target.files[0];
  const reader=new FileReader();
  reader.onload=function(ev){
    try{
      trips=JSON.parse(ev.target.result);
      renderTable();
      renderMap();
    }catch(err){ alert("Invalid JSON file!"); }
  };
  reader.readAsText(file);
});

document.getElementById("filterWorker").addEventListener("input", renderTable);
document.getElementById("filterDate").addEventListener("input", renderTable);
document.getElementById("expectedKM").addEventListener("input", renderTable);

function renderTable(){
  const tbody=document.querySelector("#reportTable tbody");
  tbody.innerHTML="";
  let workerFilter=document.getElementById("filterWorker").value.toLowerCase();
  let dateFilter=document.getElementById("filterDate").value;
  let expectedKM=parseFloat(document.getElementById("expectedKM").value) || 0;

  let filtered=trips.filter(t=>{
    return (!workerFilter || t.fieldworker.toLowerCase().includes(workerFilter)) &&
           (!dateFilter || t.date===dateFilter);
  });

  let grouped={};
  filtered.forEach(t=>{ if(!grouped[t.date]) grouped[t.date]=[]; grouped[t.date].push(t); });

  for(let date in grouped){
    const dayTrips=grouped[date];
    const totalKM=dayTrips.reduce((s,t)=>s+(t.distanceKM||0),0);
    const totalAllowance=dayTrips.reduce((s,t)=>s+(t.allowance||0),0);

    let rowClass="normal";
    if(totalKM < expectedKM) rowClass="warning";
    else if(totalKM > expectedKM*1.5) rowClass="over";

    tbody.innerHTML+=`<tr class="day-row ${rowClass}"><td colspan="8">📅 ${date} | 📏 ${totalKM.toFixed(2)} KM | 💰 ₱ ${totalAllowance.toFixed(2)}</td></tr>`;

    dayTrips.forEach(t=>{
      tbody.innerHTML+=`<tr>
        <td>${t.date}</td>
        <td><img src="${t.image}"></td>
        <td>${t.file}</td>
        <td>${t.lat.toFixed(6)}</td>
        <td>${t.lng.toFixed(6)}</td>
        <td>${t.distanceKM.toFixed(2)}</td>
        <td>${t.allowance.toFixed(2)}</td>
        <td>${t.fieldworker}</td>
      </tr>`;
    });
  }
}

function renderMap(){
  markers.forEach(m=>map.removeLayer(m));
  markers=[];
  trips.forEach(t=>{
    const marker=L.marker([t.lat,t.lng]).addTo(map);
    marker.bindPopup(`<b>${t.fieldworker}</b><br>${t.date}<br>${t.file}<br>KM: ${t.distanceKM}<br>₱ ${t.allowance}`);
    markers.push(marker);
  });
  if(markers.length>0) map.setView([markers[0].getLatLng().lat, markers[0].getLatLng().lng], 12);
}

// Simple sort
function sortTable(col){
  trips.sort((a,b)=>{
    if(col===5 || col===6) return (b[col===5?'distanceKM':'allowance']||0) - (a[col===5?'distanceKM':'allowance']||0);
    if(col===0) return new Date(a.date) - new Date(b.date);
  });
  renderTable();
  renderMap();
}
</script>

</body>
</html>
