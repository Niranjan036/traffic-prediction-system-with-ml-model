<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Coimbatore Traffic Command — 3D</title>
<link href="https://fonts.googleapis.com/css2?family=Rajdhani:wght@300;400;500;600;700&family=Orbitron:wght@400;500;700;900&family=Share+Tech+Mono&display=swap" rel="stylesheet">
<style>
:root {
  --neon: #00f5d4;
  --neon2: #f72585;
  --neon3: #ffd60a;
  --blue: #4cc9f0;
  --dim: rgba(0,245,212,0.12);
  --panel: rgba(4,10,24,0.88);
  --border: rgba(0,245,212,0.25);
  --text: #cbe8ff;
  --low: #00f5d4;
  --mod: #ffd60a;
  --high: #ff6b35;
  --sev: #f72585;
}
*{margin:0;padding:0;box-sizing:border-box}
html,body{width:100%;height:100%;overflow:hidden;background:#020812;font-family:'Rajdhani',sans-serif;color:var(--text)}
#canvas-wrap{position:fixed;inset:0;z-index:0}
canvas{display:block}

/* HUD overlay */
#hud{position:fixed;inset:0;z-index:10;pointer-events:none;display:flex;flex-direction:column}
.hud-top{display:flex;align-items:center;justify-content:space-between;padding:18px 24px 0;pointer-events:none}
.logo{font-family:'Orbitron',monospace;font-size:13px;font-weight:700;color:var(--neon);letter-spacing:.2em;text-transform:uppercase;text-shadow:0 0 20px var(--neon)}
.logo span{color:var(--text);opacity:.5}
.hud-time{font-family:'Share Tech Mono',monospace;font-size:12px;color:var(--neon);opacity:.7}

/* Scan line effect */
#hud::after{content:'';position:fixed;inset:0;background:repeating-linear-gradient(0deg,transparent,transparent 2px,rgba(0,245,212,0.015) 2px,rgba(0,245,212,0.015) 4px);pointer-events:none;z-index:99}

/* Corner brackets */
.corner{position:fixed;width:40px;height:40px;z-index:20;pointer-events:none}
.corner::before,.corner::after{content:'';position:absolute;background:var(--neon);box-shadow:0 0 8px var(--neon)}
.corner::before{width:100%;height:2px;top:0;left:0}
.corner::after{width:2px;height:100%;top:0;left:0}
.c-tl{top:12px;left:12px}
.c-tr{top:12px;right:12px;transform:scaleX(-1)}
.c-bl{bottom:12px;left:12px;transform:scaleY(-1)}
.c-br{bottom:12px;right:12px;transform:scale(-1)}

/* Left panel */
#left-panel{position:fixed;left:18px;top:70px;width:290px;z-index:20;pointer-events:all;display:flex;flex-direction:column;gap:10px}
.panel{background:var(--panel);border:1px solid var(--border);border-radius:4px;padding:16px;backdrop-filter:blur(12px);position:relative;overflow:hidden}
.panel::before{content:'';position:absolute;top:0;left:0;right:0;height:1px;background:linear-gradient(90deg,transparent,var(--neon),transparent)}
.panel-title{font-family:'Orbitron',monospace;font-size:9px;letter-spacing:.2em;color:var(--neon);opacity:.7;margin-bottom:12px;text-transform:uppercase}
.inp-group{margin-bottom:10px}
.inp-label{font-size:10px;letter-spacing:.12em;color:var(--text);opacity:.5;margin-bottom:5px;text-transform:uppercase}
select,input[type=time]{width:100%;background:rgba(0,245,212,0.06);border:1px solid var(--border);border-radius:3px;padding:8px 10px;font-family:'Rajdhani',sans-serif;font-size:13px;font-weight:500;color:var(--text);outline:none;cursor:pointer;-webkit-appearance:none;appearance:none}
select:focus,input[type=time]:focus{border-color:var(--neon);box-shadow:0 0 12px rgba(0,245,212,0.2)}
select option{background:#040a18;color:var(--text)}

/* Vehicle selector */
.veh-row{display:grid;grid-template-columns:1fr 1fr;gap:6px;margin-bottom:4px}
.veh-btn{background:rgba(0,245,212,0.05);border:1px solid var(--border);border-radius:3px;padding:8px 6px;font-family:'Rajdhani',sans-serif;font-size:11px;font-weight:600;color:var(--text);cursor:pointer;display:flex;align-items:center;justify-content:center;gap:5px;letter-spacing:.05em;transition:all .2s;text-transform:uppercase}
.veh-btn svg{width:16px;height:16px;fill:currentColor;flex-shrink:0}
.veh-btn:hover{border-color:var(--neon);background:rgba(0,245,212,0.1)}
.veh-btn.active{border-color:var(--neon);background:rgba(0,245,212,0.15);color:var(--neon);box-shadow:0 0 12px rgba(0,245,212,0.2) inset}

/* Predict button */
.predict-btn{width:100%;padding:11px;background:transparent;border:1px solid var(--neon);border-radius:3px;font-family:'Orbitron',monospace;font-size:10px;font-weight:700;letter-spacing:.2em;color:var(--neon);cursor:pointer;text-transform:uppercase;position:relative;overflow:hidden;transition:all .3s;text-shadow:0 0 10px var(--neon)}
.predict-btn::before{content:'';position:absolute;inset:0;background:linear-gradient(90deg,transparent,rgba(0,245,212,0.15),transparent);transform:translateX(-100%);transition:transform .4s}
.predict-btn:hover::before{transform:translateX(100%)}
.predict-btn:hover{background:rgba(0,245,212,0.08);box-shadow:0 0 20px rgba(0,245,212,0.3)}
.predict-btn:disabled{opacity:.35;cursor:not-allowed}

/* Right panel — results */
#right-panel{position:fixed;right:18px;top:70px;width:270px;z-index:20;pointer-events:all;display:flex;flex-direction:column;gap:10px}

/* Congestion gauge */
.gauge-wrap{text-align:center;padding:6px 0 4px}
.gauge-label{font-family:'Orbitron',monospace;font-size:9px;letter-spacing:.15em;color:var(--text);opacity:.5;margin-bottom:8px}
.gauge-value{font-family:'Orbitron',monospace;font-size:36px;font-weight:900;line-height:1;transition:color .5s}
.gauge-unit{font-size:11px;opacity:.5;letter-spacing:.1em}
.gauge-bar-track{height:4px;background:rgba(255,255,255,0.06);border-radius:2px;margin:10px 0 4px;overflow:hidden}
.gauge-bar{height:100%;border-radius:2px;width:0%;transition:width .8s cubic-bezier(.4,0,.2,1)}
.gauge-status{font-family:'Orbitron',monospace;font-size:10px;letter-spacing:.15em;text-align:center;padding:5px 10px;border-radius:2px;margin:0 auto;display:inline-block}

/* Stats grid */
.stats-grid{display:grid;grid-template-columns:1fr 1fr;gap:6px}
.stat-box{background:rgba(0,245,212,0.04);border:1px solid var(--border);border-radius:3px;padding:10px 10px 8px}
.stat-box-lbl{font-size:9px;letter-spacing:.1em;color:var(--text);opacity:.4;text-transform:uppercase;margin-bottom:3px}
.stat-box-val{font-family:'Orbitron',monospace;font-size:18px;font-weight:700;color:var(--neon)}
.stat-box-unit{font-size:10px;color:var(--text);opacity:.4}

/* Hourly chart */
.hourly-chart{display:flex;align-items:flex-end;gap:2px;height:42px;margin-top:4px}
.h-col{flex:1;display:flex;flex-direction:column;align-items:center;gap:2px}
.h-bar{width:100%;border-radius:1px 1px 0 0;min-height:3px;transition:height .5s ease}
.h-tick{font-family:'Share Tech Mono',monospace;font-size:8px;color:var(--text);opacity:.3}

/* AI insight */
.insight-text{font-size:12px;line-height:1.7;color:var(--text);opacity:.75;font-weight:400}
.insight-tag{font-family:'Orbitron',monospace;font-size:8px;letter-spacing:.15em;color:var(--neon);opacity:.6;margin-bottom:6px}

/* Loading */
.loading-row{display:none;align-items:center;gap:8px;padding:4px 0}
.loading-row.on{display:flex}
@keyframes spin{to{transform:rotate(360deg)}}
.spin-ring{width:14px;height:14px;border:2px solid var(--border);border-top-color:var(--neon);border-radius:50%;animation:spin .7s linear infinite;flex-shrink:0}
.loading-txt{font-family:'Share Tech Mono',monospace;font-size:10px;color:var(--neon);opacity:.6}

.err-row{display:none;font-size:11px;color:var(--sev);padding:6px 8px;background:rgba(247,37,133,0.08);border:1px solid rgba(247,37,133,0.2);border-radius:3px}
.err-row.on{display:block}

/* Bottom — mini map labels */
#bottom-hud{position:fixed;bottom:20px;left:50%;transform:translateX(-50%);z-index:20;pointer-events:none;display:flex;gap:20px;align-items:center}
.legend-item{display:flex;align-items:center;gap:5px;font-family:'Share Tech Mono',monospace;font-size:9px;letter-spacing:.1em;color:var(--text);opacity:.5}
.legend-dot{width:7px;height:7px;border-radius:50%}

/* Pulse animation for active route */
@keyframes pulse{0%,100%{opacity:.6}50%{opacity:1}}
.pulse{animation:pulse 2s ease-in-out infinite}

/* Node labels on 3D (overlaid via CSS absolute) */
#city-labels{position:fixed;inset:0;z-index:15;pointer-events:none}
.city-label{position:absolute;transform:translate(-50%,-50%);font-family:'Share Tech Mono',monospace;font-size:9px;color:var(--neon);opacity:.55;letter-spacing:.08em;white-space:nowrap;text-shadow:0 0 6px var(--neon)}

/* result panel hidden state */
#right-panel{opacity:0;transform:translateX(20px);transition:opacity .5s,transform .5s}
#right-panel.visible{opacity:1;transform:translateX(0)}

/* scrollable left panel */
#left-panel{max-height:calc(100vh - 90px);overflow-y:auto;scrollbar-width:none}
#left-panel::-webkit-scrollbar{display:none}
</style>
</head>
<body>

<!-- Corner brackets -->
<div class="corner c-tl"></div>
<div class="corner c-tr"></div>
<div class="corner c-bl"></div>
<div class="corner c-br"></div>

<!-- City label overlay -->
<div id="city-labels"></div>

<!-- 3D canvas -->
<div id="canvas-wrap"><canvas id="c"></canvas></div>

<!-- HUD -->
<div id="hud">
  <div class="hud-top">
    <div class="logo">CBE <span>//</span> Traffic Command</div>
    <div class="hud-time" id="hud-clock">--:--:--</div>
  </div>
</div>

<!-- Left panel -->
<div id="left-panel">
  <div class="panel">
    <div class="panel-title">▸ Route Input</div>
    <div class="inp-group">
      <div class="inp-label">Origin</div>
      <select id="from">
        <option value="">Select area…</option>
        <optgroup label="— Central —">
          <option>Gandhipuram</option><option>RS Puram</option><option>Town Hall</option><option>Ukkadam</option><option>Selvapuram</option>
        </optgroup>
        <optgroup label="— North / East —">
          <option>Peelamedu</option><option>Singanallur</option><option>Ramanathapuram</option><option>Ondipudur</option><option>Saravanampatti</option><option>Kalapatti</option><option>Hopes College</option><option>Tidel Park</option>
        </optgroup>
        <optgroup label="— West / South —">
          <option>Saibaba Colony</option><option>Ganapathy</option><option>Vadavalli</option><option>Kovaipudur</option><option>Podanur</option><option>Kurichi</option><option>Sulur</option>
        </optgroup>
        <optgroup label="— Corridors —">
          <option>Avinashi Road</option><option>Trichy Road</option><option>Mettupalayam Road</option><option>Airport Road</option><option>Sathy Road</option>
        </optgroup>
        <optgroup label="— Landmarks —">
          <option>Coimbatore Airport</option><option>KMCH Hospital</option><option>PSG Hospital</option><option>Brookefields Mall</option>
        </optgroup>
      </select>
    </div>
    <div class="inp-group">
      <div class="inp-label">Destination</div>
      <select id="to">
        <option value="">Select area…</option>
        <optgroup label="— Central —">
          <option>Gandhipuram</option><option>RS Puram</option><option>Town Hall</option><option>Ukkadam</option><option>Selvapuram</option>
        </optgroup>
        <optgroup label="— North / East —">
          <option>Peelamedu</option><option>Singanallur</option><option>Ramanathapuram</option><option>Ondipudur</option><option>Saravanampatti</option><option>Kalapatti</option><option>Hopes College</option><option>Tidel Park</option>
        </optgroup>
        <optgroup label="— West / South —">
          <option>Saibaba Colony</option><option>Ganapathy</option><option>Vadavalli</option><option>Kovaipudur</option><option>Podanur</option><option>Kurichi</option><option>Sulur</option>
        </optgroup>
        <optgroup label="— Corridors —">
          <option>Avinashi Road</option><option>Trichy Road</option><option>Mettupalayam Road</option><option>Airport Road</option><option>Sathy Road</option>
        </optgroup>
        <optgroup label="— Landmarks —">
          <option>Coimbatore Airport</option><option>KMCH Hospital</option><option>PSG Hospital</option><option>Brookefields Mall</option>
        </optgroup>
      </select>
    </div>
  </div>

  <div class="panel">
    <div class="panel-title">▸ Vehicle Type</div>
    <div class="veh-row">
      <button class="veh-btn active" data-v="two-wheeler (bike/scooter)" onclick="selVeh(this)">
        <svg viewBox="0 0 24 24"><path d="M5 16a3 3 0 1 0 6 0 3 3 0 0 0-6 0zm11 0a3 3 0 1 0 6 0 3 3 0 0 0-6 0zm-6.5-2l1-4h4l2 2-4 2h-3zM9 8l2-2h4l1 2"/></svg>
        Bike
      </button>
      <button class="veh-btn" data-v="car or SUV" onclick="selVeh(this)">
        <svg viewBox="0 0 24 24"><path d="M3 17h1a2 2 0 1 0 4 0h8a2 2 0 1 0 4 0h1v-5l-2-5H5L3 12v5zM7 7h10l1.5 4H5.5L7 7z"/></svg>
        Car / SUV
      </button>
      <button class="veh-btn" data-v="city bus" onclick="selVeh(this)">
        <svg viewBox="0 0 24 24"><path d="M4 16a2 2 0 1 0 4 0 2 2 0 0 0-4 0zm12 0a2 2 0 1 0 4 0 2 2 0 0 0-4 0zM3 8h18v8H3V8zm3-4h12l1 4H5L6 4z"/></svg>
        Bus
      </button>
      <button class="veh-btn" data-v="truck or heavy goods vehicle" onclick="selVeh(this)">
        <svg viewBox="0 0 24 24"><path d="M1 17h1a2 2 0 1 0 4 0h8a2 2 0 1 0 4 0h3v-5l-3-5H1V17zm4-8h10v4H1V9h4zm12 0h3l2 4h-5V9z"/></svg>
        Truck
      </button>
    </div>
  </div>

  <div class="panel">
    <div class="panel-title">▸ Time Parameters</div>
    <div class="inp-group">
      <div class="inp-label">Day Type</div>
      <select id="day">
        <option value="weekday">Weekday</option>
        <option value="weekend">Weekend</option>
        <option value="public holiday">Public Holiday</option>
        <option value="rainy day">Rainy Day</option>
      </select>
    </div>
    <div class="inp-group" style="margin-bottom:0">
      <div class="inp-label">Departure Time</div>
      <input type="time" id="ttime" value="08:30">
    </div>
  </div>

  <div class="panel">
    <button class="predict-btn" id="pbtn" onclick="predict()">◈ Analyse Traffic</button>
    <div class="loading-row" id="load" style="margin-top:10px">
      <div class="spin-ring"></div>
      <div class="loading-txt">AI scanning route…</div>
    </div>
    <div class="err-row" id="err"></div>
  </div>
</div>

<!-- Right panel — results -->
<div id="right-panel">
  <div class="panel">
    <div class="panel-title">▸ Congestion Level</div>
    <div class="gauge-wrap">
      <div class="gauge-label">TRAFFIC DENSITY</div>
      <div class="gauge-value" id="g-val">--</div>
      <div class="gauge-unit">PERCENT</div>
      <div class="gauge-bar-track"><div class="gauge-bar" id="g-bar"></div></div>
      <span class="gauge-status" id="g-status">—</span>
    </div>
  </div>
  <div class="panel">
    <div class="panel-title">▸ Route Metrics</div>
    <div class="stats-grid">
      <div class="stat-box"><div class="stat-box-lbl">Travel Time</div><div class="stat-box-val" id="s-time">--</div><div class="stat-box-unit">minutes</div></div>
      <div class="stat-box"><div class="stat-box-lbl">Avg Speed</div><div class="stat-box-val" id="s-spd">--</div><div class="stat-box-unit">km/h</div></div>
      <div class="stat-box"><div class="stat-box-lbl">Fuel Impact</div><div class="stat-box-val" id="s-fuel">--</div><div class="stat-box-unit">vs free-flow</div></div>
      <div class="stat-box"><div class="stat-box-lbl">Best Time</div><div class="stat-box-val" id="s-alt" style="font-size:12px;padding-top:4px">--</div><div class="stat-box-unit">to depart</div></div>
    </div>
  </div>
  <div class="panel">
    <div class="panel-title">▸ Hourly Forecast</div>
    <div class="hourly-chart" id="hchart"></div>
  </div>
  <div class="panel">
    <div class="panel-title">▸ AI Tactical Insight</div>
    <div class="insight-tag">// SYSTEM ANALYSIS</div>
    <div class="insight-text" id="insight-txt">Select a route and run analysis to receive tactical traffic intelligence.</div>
  </div>
</div>

<!-- Bottom legend -->
<div id="bottom-hud">
  <div class="legend-item"><div class="legend-dot" style="background:#00f5d4"></div>LOW</div>
  <div class="legend-item"><div class="legend-dot" style="background:#ffd60a"></div>MODERATE</div>
  <div class="legend-item"><div class="legend-dot" style="background:#ff6b35"></div>HIGH</div>
  <div class="legend-item"><div class="legend-dot" style="background:#f72585"></div>SEVERE</div>
  <div class="legend-item" style="margin-left:10px;opacity:.3">DRAG TO ROTATE · SCROLL TO ZOOM</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// ─── CLOCK ───────────────────────────────────────────────
function tick(){
  const n=new Date();
  document.getElementById('hud-clock').textContent=n.toLocaleTimeString('en-IN',{hour12:false});
}
setInterval(tick,1000);tick();

// ─── THREE.JS SCENE ──────────────────────────────────────
const W=window.innerWidth,H=window.innerHeight;
const renderer=new THREE.WebGLRenderer({canvas:document.getElementById('c'),antialias:true,alpha:true});
renderer.setSize(W,H);renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.setClearColor(0x020812,1);

const scene=new THREE.Scene();
const camera=new THREE.PerspectiveCamera(50,W/H,0.1,1000);
camera.position.set(0,55,65);camera.lookAt(0,0,0);

// Fog
scene.fog=new THREE.FogExp2(0x020812,0.018);

// Lights
const ambL=new THREE.AmbientLight(0x0a1628,2);scene.add(ambL);
const dirL=new THREE.DirectionalLight(0x4cc9f0,1.5);dirL.position.set(20,40,20);scene.add(dirL);
const ptL=new THREE.PointLight(0x00f5d4,2,80);ptL.position.set(0,20,0);scene.add(ptL);

// Ground grid
const gridH=new THREE.GridHelper(120,40,0x0a2040,0x050f1f);
gridH.position.y=-0.05;scene.add(gridH);

// Ground plane
const gpGeo=new THREE.PlaneGeometry(200,200);
const gpMat=new THREE.MeshLambertMaterial({color:0x020a14,transparent:true,opacity:.95});
const gp=new THREE.Mesh(gpGeo,gpMat);gp.rotation.x=-Math.PI/2;gp.position.y=-0.1;scene.add(gp);

// ─── CITY NODES (Coimbatore areas) ───────────────────────
const NODES={
  'Gandhipuram':     {x:0,   z:0,   h:6},
  'RS Puram':        {x:-8,  z:5,   h:5},
  'Town Hall':       {x:3,   z:-3,  h:4},
  'Ukkadam':         {x:5,   z:8,   h:3.5},
  'Selvapuram':      {x:-4,  z:10,  h:3},
  'Peelamedu':       {x:16,  z:-8,  h:5},
  'Singanallur':     {x:18,  z:4,   h:4},
  'Ramanathapuram':  {x:14,  z:12,  h:3},
  'Ondipudur':       {x:20,  z:14,  h:2.5},
  'Saravanampatti':  {x:12,  z:-18, h:3},
  'Kalapatti':       {x:18,  z:-16, h:2.5},
  'Hopes College':   {x:8,   z:-10, h:3},
  'Tidel Park':      {x:14,  z:-2,  h:4.5},
  'Saibaba Colony':  {x:-12, z:-4,  h:4},
  'Ganapathy':       {x:-16, z:0,   h:3.5},
  'Vadavalli':       {x:-20, z:-6,  h:3},
  'Kovaipudur':      {x:-18, z:8,   h:2.5},
  'Podanur':         {x:2,   z:18,  h:3},
  'Kurichi':         {x:-6,  z:16,  h:2.5},
  'Sulur':           {x:24,  z:-10, h:2},
  'Avinashi Road':   {x:10,  z:-12, h:2},
  'Trichy Road':     {x:8,   z:14,  h:2},
  'Mettupalayam Road':{x:-14,z:-10, h:2},
  'Airport Road':    {x:18,  z:-20, h:2},
  'Sathy Road':      {x:-8,  z:-16, h:2},
  'Coimbatore Airport':{x:22,z:-22, h:4},
  'KMCH Hospital':   {x:-6,  z:-6,  h:4},
  'PSG Hospital':    {x:4,   z:-14, h:3.5},
  'Brookefields Mall':{x:-10,z:2,   h:5},
};

const nodeObjs={};
const nodeMeshes=[];

function buildingColor(h){
  if(h>=6) return 0x0a3060;
  if(h>=4) return 0x082040;
  return 0x051428;
}
function buildingEmissive(h){
  if(h>=6) return 0x002244;
  if(h>=4) return 0x001530;
  return 0x000e20;
}

Object.entries(NODES).forEach(([name,{x,z,h}])=>{
  const w=Math.random()*1.5+1.5,d=Math.random()*1.5+1.5;
  const geo=new THREE.BoxGeometry(w,h,d);
  const mat=new THREE.MeshLambertMaterial({
    color:buildingColor(h),
    emissive:buildingEmissive(h),
    emissiveIntensity:1
  });
  const mesh=new THREE.Mesh(geo,mat);
  mesh.position.set(x,h/2,z);
  scene.add(mesh);

  // Top glow cap
  const capGeo=new THREE.BoxGeometry(w,0.12,d);
  const capMat=new THREE.MeshBasicMaterial({color:0x00f5d4,transparent:true,opacity:.35});
  const cap=new THREE.Mesh(capGeo,capMat);
  cap.position.set(x,h+0.06,z);
  scene.add(cap);

  nodeObjs[name]={mesh,cap,x,z,h,w,d};
  nodeMeshes.push(mesh);
});

// ─── ROAD CONNECTIONS ────────────────────────────────────
const ROADS=[
  ['Gandhipuram','RS Puram'],['Gandhipuram','Town Hall'],['Gandhipuram','Peelamedu'],
  ['Gandhipuram','Saibaba Colony'],['Gandhipuram','Hopes College'],
  ['RS Puram','Saibaba Colony'],['RS Puram','Brookefields Mall'],
  ['Peelamedu','Tidel Park'],['Peelamedu','Singanallur'],['Peelamedu','Avinashi Road'],
  ['Singanallur','Ramanathapuram'],['Ramanathapuram','Ondipudur'],
  ['Avinashi Road','Sulur'],['Avinashi Road','Coimbatore Airport'],
  ['Town Hall','Ukkadam'],['Ukkadam','Podanur'],['Ukkadam','Selvapuram'],
  ['Trichy Road','Podanur'],['Trichy Road','Singanallur'],
  ['Saibaba Colony','Ganapathy'],['Ganapathy','Vadavalli'],
  ['Mettupalayam Road','Saibaba Colony'],['Mettupalayam Road','Vadavalli'],
  ['Hopes College','PSG Hospital'],['Hopes College','Avinashi Road'],
  ['KMCH Hospital','Gandhipuram'],['PSG Hospital','Gandhipuram'],
  ['Brookefields Mall','Ganapathy'],['Kovaipudur','Vadavalli'],
  ['Airport Road','Coimbatore Airport'],['Sathy Road','Vadavalli'],
];

const roadLines=[];
ROADS.forEach(([a,b])=>{
  const na=NODES[a],nb=NODES[b];
  if(!na||!nb) return;
  const pts=[new THREE.Vector3(na.x,0.1,na.z),new THREE.Vector3(nb.x,0.1,nb.z)];
  const geo=new THREE.BufferGeometry().setFromPoints(pts);
  const mat=new THREE.LineBasicMaterial({color:0x0a3050,transparent:true,opacity:.6});
  const line=new THREE.Line(geo,mat);
  scene.add(line);
  roadLines.push({line,a,b,mat});
});

// ─── ACTIVE ROUTE LINE ───────────────────────────────────
let routeLine=null;
let routeParticles=[];

function clearRoute(){
  if(routeLine){scene.remove(routeLine);routeLine=null;}
  routeParticles.forEach(p=>scene.remove(p));
  routeParticles=[];
  roadLines.forEach(r=>r.mat.color.setHex(0x0a3050));
}

function showRoute(from,to,color=0x00f5d4){
  clearRoute();
  const na=NODES[from],nb=NODES[to];
  if(!na||!nb) return;

  const mid={x:(na.x+nb.x)/2+((Math.random()-.5)*6),z:(na.z+nb.z)/2+((Math.random()-.5)*6)};
  const pts=[];
  for(let i=0;i<=30;i++){
    const t=i/30;
    const bx=(1-t)*(1-t)*na.x+2*(1-t)*t*mid.x+t*t*nb.x;
    const bz=(1-t)*(1-t)*na.z+2*(1-t)*t*mid.z+t*t*nb.z;
    pts.push(new THREE.Vector3(bx,0.35,bz));
  }
  const geo=new THREE.BufferGeometry().setFromPoints(pts);
  const mat=new THREE.LineBasicMaterial({color,transparent:true,opacity:.9,linewidth:2});
  routeLine=new THREE.Line(geo,mat);
  scene.add(routeLine);

  // Highlight origin & dest nodes
  [from,to].forEach((name,i)=>{
    const n=NODES[name];if(!n) return;
    const geo2=new THREE.SphereGeometry(.8,8,8);
    const mat2=new THREE.MeshBasicMaterial({color:i===0?0x00f5d4:0xf72585,transparent:true,opacity:.7});
    const sphere=new THREE.Mesh(geo2,mat2);
    sphere.position.set(n.x,n.h+0.9,n.z);
    scene.add(sphere);
    routeParticles.push(sphere);
  });
}

// ─── FLOATING PARTICLES ──────────────────────────────────
const pGeo=new THREE.BufferGeometry();
const pCount=300;
const pPos=new Float32Array(pCount*3);
for(let i=0;i<pCount;i++){
  pPos[i*3]=(Math.random()-.5)*100;
  pPos[i*3+1]=Math.random()*30;
  pPos[i*3+2]=(Math.random()-.5)*100;
}
pGeo.setAttribute('position',new THREE.BufferAttribute(pPos,3));
const pMat=new THREE.PointsMaterial({color:0x0a3060,size:.18,transparent:true,opacity:.5});
const particles=new THREE.Points(pGeo,pMat);
scene.add(particles);

// ─── MOUSE ORBIT ─────────────────────────────────────────
let isDragging=false,prevMX=0,prevMY=0;
let camTheta=0.2,camPhi=0.9,camR=80;
const canvas=document.getElementById('c');
canvas.addEventListener('mousedown',e=>{isDragging=true;prevMX=e.clientX;prevMY=e.clientY});
window.addEventListener('mouseup',()=>isDragging=false);
window.addEventListener('mousemove',e=>{
  if(!isDragging) return;
  camTheta-=(e.clientX-prevMX)*0.005;
  camPhi=Math.max(.3,Math.min(1.3,camPhi-(e.clientY-prevMY)*0.004));
  prevMX=e.clientX;prevMY=e.clientY;
});
canvas.addEventListener('wheel',e=>{camR=Math.max(30,Math.min(130,camR+e.deltaY*.08))});
canvas.addEventListener('touchstart',e=>{isDragging=true;prevMX=e.touches[0].clientX;prevMY=e.touches[0].clientY},{passive:true});
canvas.addEventListener('touchend',()=>isDragging=false);
canvas.addEventListener('touchmove',e=>{
  camTheta-=(e.touches[0].clientX-prevMX)*0.005;
  camPhi=Math.max(.3,Math.min(1.3,camPhi-(e.touches[0].clientY-prevMY)*0.004));
  prevMX=e.touches[0].clientX;prevMY=e.touches[0].clientY;
},{passive:true});

// ─── CITY LABELS (2D projected) ───────────────────────────
const labelsDiv=document.getElementById('city-labels');
const labelEls={};
const SHOW_LABELS=['Gandhipuram','RS Puram','Peelamedu','Singanallur','Saibaba Colony','Ganapathy','Tidel Park','Coimbatore Airport','KMCH Hospital','Brookefields Mall'];
SHOW_LABELS.forEach(name=>{
  const el=document.createElement('div');
  el.className='city-label';
  el.textContent=name;
  labelsDiv.appendChild(el);
  labelEls[name]=el;
});

const tmpV=new THREE.Vector3();
function updateLabels(){
  SHOW_LABELS.forEach(name=>{
    const n=NODES[name];if(!n) return;
    tmpV.set(n.x,n.h+1.5,n.z);
    tmpV.project(camera);
    const sx=(tmpV.x+1)/2*window.innerWidth;
    const sy=(-tmpV.y+1)/2*window.innerHeight;
    const el=labelEls[name];
    if(tmpV.z>1){el.style.opacity='0';}
    else{el.style.opacity='';el.style.left=sx+'px';el.style.top=sy+'px';}
  });
}

// ─── ANIMATE ─────────────────────────────────────────────
let t=0;
function animate(){
  requestAnimationFrame(animate);
  t+=0.005;
  // Orbit camera
  if(!isDragging){camTheta+=0.0008;}
  camera.position.x=Math.sin(camTheta)*Math.cos(camPhi)*camR;
  camera.position.y=Math.sin(camPhi)*camR;
  camera.position.z=Math.cos(camTheta)*Math.cos(camPhi)*camR;
  camera.lookAt(0,0,0);

  // Pulse particles
  particles.rotation.y+=0.0003;
  const pa=particles.geometry.attributes.position;
  for(let i=0;i<pCount;i++){
    pa.array[i*3+1]+=0.01;
    if(pa.array[i*3+1]>30) pa.array[i*3+1]=0;
  }
  pa.needsUpdate=true;

  // Pulse top caps
  Object.values(nodeObjs).forEach(({cap},i)=>{
    cap.material.opacity=0.2+0.2*Math.sin(t+i*.3);
  });

  // Pulse route spheres
  routeParticles.forEach((p,i)=>{
    p.scale.setScalar(0.8+0.4*Math.sin(t*2+i*Math.PI));
  });

  updateLabels();
  renderer.render(scene,camera);
}
animate();

window.addEventListener('resize',()=>{
  renderer.setSize(window.innerWidth,window.innerHeight);
  camera.aspect=window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
});

// ─── VEHICLE ─────────────────────────────────────────────
let selV='two-wheeler (bike/scooter)';
function selVeh(btn){
  document.querySelectorAll('.veh-btn').forEach(b=>b.classList.remove('active'));
  btn.classList.add('active');
  selV=btn.dataset.v;
}

// ─── PREDICT ─────────────────────────────────────────────
const SYS=`You are a Coimbatore traffic expert for Tamil Nadu, India. Key facts:
- Morning peak 7:30–9:30am, evening peak 5–8pm
- Gandhipuram always congested, Avinashi Road has heavy vehicles
- Two-wheelers faster in congestion; buses and trucks slowest
- Rainy days +30-40% travel time; trucks restricted inner roads peak hours
- Weekends: lower AM, heavier near malls evenings

Return ONLY valid JSON (no markdown):
{"congestion_pct":<0-100>,"travel_minutes":<int>,"avg_speed_kmh":<int>,"fuel_impact":"<e.g.+20%>","best_time":"<e.g. after 10am>","insight":"<2 specific tactical sentences>","hourly":[<24 ints 0-100 for hours 0-23>]}`;

async function predict(){
  const from=document.getElementById('from').value;
  const to=document.getElementById('to').value;
  const day=document.getElementById('day').value;
  const ttime=document.getElementById('ttime').value;
  const err=document.getElementById('err');

  err.className='err-row';
  if(!from||!to){err.textContent='Select both origin and destination.';err.className='err-row on';return;}
  if(from===to){err.textContent='Origin and destination must differ.';err.className='err-row on';return;}

  document.getElementById('load').className='loading-row on';
  document.getElementById('pbtn').disabled=true;

  // Show route on 3D (pending color)
  showRoute(from,to,0x4cc9f0);

  const prompt=`Route: ${selV} from ${from} to ${to} at ${ttime} on a ${day} in Coimbatore. JSON only.`;
  try{
    const r=await fetch('https://api.anthropic.com/v1/messages',{
      method:'POST',headers:{'Content-Type':'application/json'},
      body:JSON.stringify({model:'claude-sonnet-4-20250514',max_tokens:1000,system:SYS,messages:[{role:'user',content:prompt}]})
    });
    const d=await r.json();
    const raw=d.content.map(b=>b.text||'').join('').replace(/```json|```/g,'').trim();
    renderResult(JSON.parse(raw),from,to,ttime,day);
  }catch(e){
    err.textContent='Analysis failed. Retry.';err.className='err-row on';
    clearRoute();
  }finally{
    document.getElementById('load').className='loading-row';
    document.getElementById('pbtn').disabled=false;
  }
}

function renderResult(x,from,to,ttime,day){
  const pct=Math.min(100,Math.max(0,x.congestion_pct));
  let col,hex,statusClass;
  if(pct<30){col='var(--low)';hex=0x00f5d4;statusClass='LOW';}
  else if(pct<55){col='var(--mod)';hex=0xffd60a;statusClass='MODERATE';}
  else if(pct<75){col='var(--high)';hex=0xff6b35;statusClass='HIGH';}
  else{col='var(--sev)';hex=0xf72585;statusClass='SEVERE';}

  showRoute(from,to,hex);

  document.getElementById('g-val').textContent=pct;
  document.getElementById('g-val').style.color=col;
  const bar=document.getElementById('g-bar');
  bar.style.width='0%';bar.style.background=col;bar.style.boxShadow='0 0 10px '+col;
  setTimeout(()=>bar.style.width=pct+'%',100);

  const st=document.getElementById('g-status');
  st.textContent='◈ '+statusClass;st.style.color=col;st.style.background='rgba(0,0,0,.4)';st.style.border='1px solid '+col;st.style.boxShadow='0 0 10px '+col;

  document.getElementById('s-time').textContent=Math.round(x.travel_minutes)||'--';
  document.getElementById('s-spd').textContent=Math.round(x.avg_speed_kmh)||'--';
  document.getElementById('s-fuel').textContent=x.fuel_impact||'--';
  document.getElementById('s-alt').textContent=x.best_time||'--';
  document.getElementById('insight-txt').textContent=x.insight||'';

  buildHourly(x.hourly||[],parseInt(ttime.split(':')[0]));
  document.getElementById('right-panel').classList.add('visible');
}

function buildHourly(h,cur){
  const c=document.getElementById('hchart');c.innerHTML='';
  [6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22].forEach(hr=>{
    const p=h[hr]??20;
    const bh=Math.max(3,Math.round((p/100)*36));
    const clr=p<30?'#00f5d4':p<55?'#ffd60a':p<75?'#ff6b35':'#f72585';
    const div=document.createElement('div');
    div.className='h-col';
    div.innerHTML=`<div class="h-bar" style="height:${bh}px;background:${clr};opacity:${hr===cur?1:.45};box-shadow:${hr===cur?'0 0 6px '+clr:'none'}"></div><div class="h-tick">${hr<12?hr+'a':hr===12?'12':hr-12+'p'}</div>`;
    c.appendChild(div);
  });
}
</script>
</body>
</html>
