<!DOCTYPE html>
<html>
<head>
  <title>QA Portal - Approval System</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <style>
    body { font-family: Arial; background:#0f172a; color:BLACK; margin:0; }
    header { padding:15px; text-align:center; background:linear-gradient(45deg,#3b82f6,#9333ea); font-size:22px; }
    table { width:95%; margin:20px auto; border-collapse: collapse; }
    th, td { border:1px solid #ccc; padding:8px; text-align:center; font-size:12px; }
    th { background:#1e40af; color:white; }
    img { width:80px; border-radius:8px; cursor:pointer; }
    tr:nth-child(even){background:#111827;}
    #controls { text-align:center; margin:15px; }
    input, button { padding:8px; border-radius:8px; margin:3px; border:none; }
    button { cursor:pointer; }
    .approve { background:#22c55e; color:white; }
    .reject { background:#ef4444; color:white; }
    .export { background:#f59e0b; color:white; }
    .status { font-weight:bold; }
    .approved { color:#22c55e; }
    .rejected { color:#ef4444; }
    .pending { color:#facc15; }
  </style>
</head>
<body>

<header>QA Portal - Approval System</header>

<div id="controls">
  <input type="file" id="importJSON" accept=".json">
  <button class="export" onclick="exportQA()">Export QA Result</button>
  <button onclick="refreshTable()">Refresh</button>
</div>

<table id="qaTable">
  <thead>
    <tr>
      <th>Fieldworker</th>
      <th>Date</th>
      <th>Distance (KM)</th>
      <th>Allowance (₱)</th>
      <th>Photo</th>
      <th>Coordinates</th>
      <th>File</th>
      <th>Status</th>
      <th>Action</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
let trips=[];

// IMPORT JSON
document.getElementById("importJSON").addEventListener("change", function(e){
  const file = e.target.files[0];
  if(!file) return;

  const reader = new FileReader();
  reader.onload = function(evt){
    try{
      trips = JSON.parse(evt.target.result);

      // add default status
      trips = trips.map(t => ({
        ...t,
        status: t.status || "Pending"
      }));

      localStorage.setItem('qaTrips', JSON.stringify(trips));
      populateTable();

      alert("✅ Loaded successfully!");
    }catch(err){
      alert("❌ Invalid JSON file!");
    }
  }
  reader.readAsText(file);
});

// DISPLAY TABLE
function populateTable(){
  const stored = JSON.parse(localStorage.getItem('qaTrips')) || [];
  const tbody = document.querySelector("#qaTable tbody");
  tbody.innerHTML="";

  stored.forEach((t,index)=>{
    let statusClass = t.status.toLowerCase();

    const tr=document.createElement("tr");
    tr.innerHTML=`
      <td>${t.fieldworker || "Unknown"}</td>
      <td>${t.date}</td>
      <td>${t.distanceKM ? t.distanceKM.toFixed(2) : "-"}</td>
      <td>${t.allowance ? t.allowance.toFixed(2) : "-"}</td>
      <td><img src="${t.image}" onclick="viewImage('${t.image}')"></td>
      <td>Lat:${t.lat.toFixed(6)}<br>Lng:${t.lng.toFixed(6)}</td>
      <td>${t.file}</td>
      <td class="status ${statusClass}">${t.status}</td>
      <td>
        <button class="approve" onclick="setStatus(${index},'Approved')">✔</button>
        <button class="reject" onclick="setStatus(${index},'Rejected')">✖</button>
      </td>
    `;
    tbody.appendChild(tr);
  });
}

// SET STATUS
function setStatus(index,status){
  let stored = JSON.parse(localStorage.getItem('qaTrips')) || [];
  stored[index].status = status;
  localStorage.setItem('qaTrips', JSON.stringify(stored));
  populateTable();
}

// IMAGE VIEW (ZOOM)
function viewImage(src){
  let w = window.open("");
  w.document.write(`<img src="${src}" style="width:100%">`);
}

// EXPORT QA RESULT
function exportQA(){
  const data = localStorage.getItem('qaTrips');
  if(!data){ alert("⚠️ No data"); return; }

  const blob = new Blob([data], {type:"application/json"});
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = "QA_Approved_Data.json";
  link.click();
}

// REFRESH
function refreshTable(){ populateTable(); }

// AUTO LOAD
window.onload = populateTable;
</script>

</body>
</html>
