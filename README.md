<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Deadlock Detection & Recovery Toolkit</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<script src="https://cdn.tailwindcss.com"></script>

<style>
  body {
    font-family: 'Inter', system-ui;
    background:
      linear-gradient(rgba(0,255,255,0.05) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,255,255,0.05) 1px, transparent 1px),
      #020617;
    background-size: 40px 40px;
  }
  .neon {
    box-shadow: 0 0 15px rgba(56,189,248,.35);
  }
</style>
</head>

<body class="text-slate-100 p-6 min-h-screen">

<div class="max-w-7xl mx-auto space-y-8">

  <!-- HEADER -->
  <header class="flex justify-between items-center">
    <div>
      <h1 class="text-4xl font-extrabold text-cyan-400">
        Deadlock Control Toolkit
      </h1>
      <p class="text-slate-400 mt-1">
        Real-Time Detection · Prevention · Recovery
      </p>
    </div>
    <div class="text-right">
      <p class="text-xs text-slate-500">Operating Systems Project</p>
      <p class="text-xs text-slate-500">Developed by</p>
      <p class="font-semibold">Birendra Singh</p>
    </div>
  </header>

  <!-- SYSTEM STATUS -->
  <div class="grid md:grid-cols-3 gap-4">
    <div class="bg-slate-900 border border-cyan-400 rounded-xl p-4 neon">
      <div class="text-xs text-slate-400">SYSTEM STATE</div>
      <div id="systemState" class="text-2xl font-bold text-emerald-400">
        STABLE
      </div>
    </div>
    <div class="bg-slate-900 border border-slate-700 rounded-xl p-4">
      <div class="text-xs text-slate-400">TOTAL PROCESSES</div>
      <div id="procCount" class="text-2xl font-bold">0</div>
    </div>
    <div class="bg-slate-900 border border-slate-700 rounded-xl p-4">
      <div class="text-xs text-slate-400">RESOURCES TYPES</div>
      <div id="resCount" class="text-2xl font-bold">0</div>
    </div>
  </div>

  <!-- MAIN GRID -->
  <main class="grid lg:grid-cols-3 gap-6">

    <!-- INPUT ZONE -->
    <section class="space-y-6">

      <div class="bg-slate-900 rounded-xl border border-slate-700 p-5">
        <h2 class="font-semibold text-cyan-300 mb-3">Process Configuration</h2>

        <input id="pid" placeholder="Process ID"
          class="w-full mb-2 px-3 py-2 rounded bg-transparent border border-slate-600">

        <input id="alloc" placeholder="Allocated Resources (e.g. 1,0,2)"
          class="w-full mb-2 px-3 py-2 rounded bg-transparent border border-slate-600">

        <input id="max" placeholder="Maximum Demand (e.g. 3,2,2)"
          class="w-full px-3 py-2 rounded bg-transparent border border-slate-600">

        <button id="addProcess"
          class="mt-4 w-full bg-cyan-400 text-slate-900 py-2 rounded-lg font-semibold hover:brightness-110">
          Register Process
        </button>
      </div>

      <div class="bg-slate-900 rounded-xl border border-slate-700 p-5">
        <h2 class="font-semibold text-cyan-300 mb-2">Available Resources</h2>
        <input id="available" placeholder="e.g. 3,3,2"
          class="w-full px-3 py-2 rounded bg-transparent border border-slate-600">
      </div>

      <div class="space-y-3">
        <button id="runBanker"
          class="w-full bg-emerald-400 text-slate-900 py-2 rounded-lg font-semibold">
          Execute Banker's Algorithm
        </button>
        <button id="detect"
          class="w-full bg-amber-400 text-slate-900 py-2 rounded-lg font-semibold">
          Detect Deadlock
        </button>
        <button onclick="location.reload()"
          class="w-full border border-slate-600 py-2 rounded-lg">
          Reset Toolkit
        </button>
      </div>

    </section>

    <!-- PROCESS TABLE -->
    <section class="lg:col-span-2 bg-slate-900 rounded-xl border border-slate-700 p-5">
      <h2 class="font-semibold text-cyan-300 mb-4">Active Process Table</h2>

      <table class="w-full text-sm">
        <thead class="text-slate-400">
          <tr>
            <th class="text-left">PID</th>
            <th>Allocation</th>
            <th>Max</th>
            <th>Need</th>
          </tr>
        </thead>
        <tbody id="table" class="divide-y divide-slate-700"></tbody>
      </table>
    </section>
  </main>

  <!-- OUTPUT ZONE -->
  <section class="grid md:grid-cols-2 gap-6">

    <div class="bg-slate-900 rounded-xl border border-slate-700 p-5">
      <h2 class="font-semibold text-cyan-300 mb-2">System Console</h2>
      <div id="output" class="text-sm text-slate-300">
        Waiting for simulation...
      </div>
    </div>

    <div class="bg-slate-900 rounded-xl border border-slate-700 p-5">
      <h2 class="font-semibold text-cyan-300 mb-2">Resource Allocation Graph</h2>
      <svg id="graph" width="100%" height="240"></svg>
    </div>

  </section>

</div>

<script>
const processes = [];

function updateStats() {
  document.getElementById("procCount").textContent = processes.length;
}

function drawGraph() {
  const svg = document.getElementById("graph");
  svg.innerHTML = "";

  processes.forEach((p,i)=>{
    svg.innerHTML += `
      <circle cx="70" cy="${50+i*45}" r="18" fill="#22d3ee"/>
      <text x="70" y="${55+i*45}" text-anchor="middle">${p.pid}</text>
    `;
  });
}

function renderTable() {
  const tbody = document.getElementById("table");
  tbody.innerHTML = "";
  processes.forEach(p=>{
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${p.pid}</td>
      <td class="text-center">${p.alloc.join(",")}</td>
      <td class="text-center">${p.max.join(",")}</td>
      <td class="text-center">${p.need.join(",")}</td>
    `;
    tbody.appendChild(tr);
  });
}

document.getElementById("addProcess").onclick = () => {
  const pid = document.getElementById("pid").value || "P"+(processes.length+1);
  const alloc = document.getElementById("alloc").value.split(",").map(Number);
  const max = document.getElementById("max").value.split(",").map(Number);

  processes.push({
    pid,
    alloc,
    max,
    need: max.map((m,i)=>m-alloc[i])
  });

  renderTable();
  drawGraph();
  updateStats();
};

document.getElementById("runBanker").onclick = () => {
  const available = document.getElementById("available").value.split(",").map(Number);
  let work=[...available], finish=Array(processes.length).fill(false), safe=[];

  while (safe.length<processes.length) {
    let found=false;
    processes.forEach((p,i)=>{
      if(!finish[i] && p.need.every((n,j)=>n<=work[j])){
        work=work.map((w,j)=>w+p.alloc[j]);
        finish[i]=true; safe.push(p.pid); found=true;
      }
    });
    if(!found) break;
  }

  if(safe.length===processes.length){
    document.getElementById("systemState").textContent="SAFE";
    document.getElementById("systemState").className="text-2xl font-bold text-emerald-400";
    document.getElementById("output").innerHTML=`✅ Safe Sequence:<br>${safe.join(" → ")}`;
  } else {
    document.getElementById("systemState").textContent="UNSAFE";
    document.getElementById("systemState").className="text-2xl font-bold text-red-400";
    document.getElementById("output").innerHTML="❌ System in Unsafe State";
  }
};

document.getElementById("detect").onclick = () => {
  const state=document.getElementById("systemState").textContent;
  document.getElementById("output").innerHTML=
    state==="UNSAFE" ? "🚨 Deadlock Detected" : "✅ No Deadlock Detected";
};
</script>

</body>
</html>
