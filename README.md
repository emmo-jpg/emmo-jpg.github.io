# emmo-jpg.github.io

<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<title>KippenKarte</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css">
<style>
  :root{
    --bg:#15120E; --bg-2:#1E1913; --surface:#241E17; --surface-2:#2C251C;
    --line:#3A3025; --ink:#F4EDE1; --muted:#A99A85;
    --ember:#FBA92C; --ember-hot:#F97316; --ember-soft:rgba(251,169,44,.14);
    --ok:#6BD98A; --bad:#F0685A; --unknown:#8B7E6B;
    --blue:#7FB2E8; --green:#8BD3A0; --pink:#D98BCE; --gold:#E9B84A;
    --mono: ui-monospace,"SF Mono","Menlo","Cascadia Code",monospace;
    --sans: -apple-system,BlinkMacSystemFont,"Segoe UI",system-ui,sans-serif;
  }
  *{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
  html,body{margin:0;height:100%;background:var(--bg);color:var(--ink);font-family:var(--sans);overscroll-behavior:none}
  #app{position:fixed;inset:0;display:flex;flex-direction:column}
  #map{flex:1;background:var(--bg-2);z-index:1}
  .leaflet-container{background:var(--bg-2);font-family:var(--sans)}
  .leaflet-control-attribution{background:rgba(21,18,14,.7)!important;color:#8a7d69!important;font-size:9px}
  .leaflet-control-attribution a{color:#b09d80!important}
  .leaflet-bar a{background:var(--surface);color:var(--ink);border-color:var(--line)}
  .leaflet-bar a:hover{background:var(--surface-2)}

  /* ---- Top bar ---- */
  .topbar{position:absolute;top:0;left:0;right:0;z-index:500;padding:calc(env(safe-area-inset-top) + 10px) 12px 10px;
    background:linear-gradient(180deg,rgba(21,18,14,.96) 55%,rgba(21,18,14,0));pointer-events:none}
  .brand{display:flex;align-items:center;gap:9px;margin:0 2px 9px;pointer-events:auto}
  .brand .ember{width:12px;height:12px;border-radius:50%;background:var(--ember);
    box-shadow:0 0 10px 2px var(--ember),0 0 20px 6px rgba(251,169,44,.35);animation:glow 2.4s ease-in-out infinite}
  .brand h1{font-size:17px;letter-spacing:.5px;margin:0;font-weight:800}
  .brand h1 span{color:var(--ember)}
  .brand small{margin-left:auto;font-family:var(--mono);font-size:10px;color:var(--muted);letter-spacing:.5px;text-transform:uppercase}
  .searchrow{display:flex;gap:8px;pointer-events:auto}
  .search{flex:1;display:flex;align-items:center;gap:8px;background:var(--surface);border:1px solid var(--line);
    border-radius:13px;padding:0 12px;height:44px}
  .search input{flex:1;background:none;border:none;outline:none;color:var(--ink);font-size:15px;font-family:var(--sans)}
  .search input::placeholder{color:var(--muted)}
  .search svg{opacity:.6}
  .iconbtn{width:44px;height:44px;flex:none;display:grid;place-items:center;background:var(--surface);
    border:1px solid var(--line);border-radius:13px;color:var(--ink);cursor:pointer}
  .iconbtn:active{background:var(--surface-2)}
  .iconbtn.live{border-color:var(--ember);color:var(--ember)}

  /* ---- City autocomplete ---- */
  .suggest{margin-top:8px;background:var(--surface);border:1px solid var(--line);border-radius:13px;overflow:hidden;
    pointer-events:auto;display:none;box-shadow:0 14px 34px rgba(0,0,0,.55)}
  .suggest.show{display:block}
  .suggest .row{padding:11px 13px;border-bottom:1px solid var(--line);cursor:pointer;display:flex;gap:10px;align-items:center}
  .suggest .row:last-child{border-bottom:none}
  .suggest .row:active{background:var(--surface-2)}
  .suggest .row svg{opacity:.5;flex:none;color:var(--ember)}
  .suggest .row .txt{min-width:0}
  .suggest .row b{font-size:14px;font-weight:700;display:block;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
  .suggest .row small{font-size:11px;color:var(--muted);display:block;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}

  /* ---- Radius chips ---- */
  .chips{display:flex;gap:6px;margin-top:9px;pointer-events:auto;overflow-x:auto;scrollbar-width:none}
  .chips::-webkit-scrollbar{display:none}
  .chip{flex:none;font-family:var(--mono);font-size:11px;letter-spacing:.4px;padding:6px 11px;border-radius:20px;
    background:var(--surface);border:1px solid var(--line);color:var(--muted);cursor:pointer}
  .chip.on{background:var(--ember-soft);border-color:var(--ember);color:var(--ember)}

  /* ---- Pins ---- */
  .pin{width:30px;height:30px;border-radius:50% 50% 50% 2px;transform:rotate(45deg);
    display:grid;place-items:center;border:1.5px solid rgba(0,0,0,.35);
    background:var(--pc);box-shadow:0 3px 6px rgba(0,0,0,.5)}
  .pin > *{transform:rotate(-45deg);font-size:14px;line-height:1}
  .pin-ember{background:radial-gradient(circle at 40% 35%,#FFD27A,var(--ember) 55%,var(--ember-hot));
    box-shadow:0 0 0 3px rgba(251,169,44,.16),0 3px 8px rgba(0,0,0,.55);animation:pinGlow 2.6s ease-in-out infinite}
  .pin-bad{filter:grayscale(.55) brightness(.8)}
  .pin-bad::after{content:"";position:absolute;inset:-3px;border-radius:inherit;box-shadow:0 0 0 2px var(--bad)}
  .me{width:18px;height:18px;border-radius:50%;background:var(--ember);border:3px solid #fff;
    box-shadow:0 0 0 6px rgba(251,169,44,.25),0 0 14px 3px var(--ember)}
  @keyframes glow{0%,100%{opacity:1}50%{opacity:.45}}
  @keyframes pinGlow{0%,100%{box-shadow:0 0 0 3px rgba(251,169,44,.16),0 3px 8px rgba(0,0,0,.55)}
    50%{box-shadow:0 0 0 7px rgba(251,169,44,.05),0 0 16px 3px rgba(251,169,44,.5),0 3px 8px rgba(0,0,0,.55)}}

  /* ---- Status / toast ---- */
  .toast{position:absolute;left:50%;top:calc(env(safe-area-inset-top) + 132px);transform:translateX(-50%);
    z-index:600;background:var(--surface-2);border:1px solid var(--line);border-radius:11px;padding:9px 14px;
    font-size:13px;color:var(--ink);box-shadow:0 8px 24px rgba(0,0,0,.5);max-width:88%;text-align:center;
    opacity:0;transition:opacity .25s,transform .25s;pointer-events:none}
  .toast.show{opacity:1}
  .toast .ld{display:inline-block;width:12px;height:12px;margin-right:8px;border:2px solid var(--muted);
    border-top-color:var(--ember);border-radius:50%;vertical-align:-2px;animation:spin .8s linear infinite}
  @keyframes spin{to{transform:rotate(360deg)}}

  /* ---- List toggle FAB ---- */
  .fab{position:absolute;right:14px;bottom:calc(env(safe-area-inset-bottom) + 20px);z-index:500;
    height:48px;padding:0 16px;display:flex;align-items:center;gap:8px;border-radius:24px;
    background:var(--ember);color:#241400;border:none;font-weight:800;font-size:14px;cursor:pointer;
    box-shadow:0 6px 18px rgba(251,169,44,.4)}
  .fab:active{transform:scale(.96)}

  /* ---- Diagnose ---- */
  .dbgbadge{position:absolute;left:12px;bottom:calc(env(safe-area-inset-bottom) + 20px);z-index:500;
    font-family:var(--mono);font-size:11px;letter-spacing:.4px;padding:9px 13px;border-radius:22px;
    background:var(--surface);border:1px solid var(--line);color:var(--muted);cursor:pointer}
  .dbgbadge.err{border-color:var(--bad);color:var(--bad)}
  .dbgbadge.ok{border-color:var(--ok);color:var(--ok)}
  .dbgpanel{position:absolute;left:8px;right:8px;bottom:calc(env(safe-area-inset-bottom) + 72px);z-index:800;
    background:var(--bg-2);border:1px solid var(--line);border-radius:15px;display:none;flex-direction:column;
    max-height:54vh;box-shadow:0 14px 44px rgba(0,0,0,.65)}
  .dbgpanel.open{display:flex}
  .dbg-head{display:flex;align-items:center;gap:8px;padding:11px 12px;border-bottom:1px solid var(--line)}
  .dbg-head b{font-size:13px;flex:1;font-weight:700}
  .dbg-head b span{font-family:var(--mono);font-size:11px;color:var(--muted);font-weight:400}
  .dbg-head button{font-family:var(--mono);font-size:12px;background:var(--surface);border:1px solid var(--line);
    color:var(--ember);border-radius:9px;padding:8px 11px;cursor:pointer}
  .dbg-head #dbgClose{color:var(--muted);padding:8px 11px}
  .dbg-log{overflow-y:auto;padding:9px 12px;font-family:var(--mono);font-size:11px;line-height:1.6;color:var(--muted);word-break:break-word}
  .dbg-log .ok{color:var(--ok)} .dbg-log .bad{color:var(--bad)} .dbg-log .in{color:var(--ink)}

  /* ---- Bottom sheet ---- */
  .sheet{position:absolute;left:0;right:0;bottom:0;z-index:700;background:var(--bg-2);
    border-top:1px solid var(--line);border-radius:20px 20px 0 0;transform:translateY(102%);
    transition:transform .32s cubic-bezier(.4,0,.2,1);box-shadow:0 -12px 40px rgba(0,0,0,.55);
    max-height:82vh;display:flex;flex-direction:column;padding-bottom:env(safe-area-inset-bottom)}
  .sheet.open{transform:translateY(0)}
  .grip{width:38px;height:4px;border-radius:2px;background:var(--line);margin:9px auto 4px}
  .sheet-scroll{overflow-y:auto;padding:6px 18px 22px}
  .ph{display:flex;align-items:flex-start;gap:12px;margin-bottom:14px}
  .ph .badge{width:44px;height:44px;flex:none;border-radius:12px;display:grid;place-items:center;font-size:22px;
    background:var(--surface);border:1px solid var(--line)}
  .ph h2{margin:0;font-size:19px;font-weight:800;line-height:1.15}
  .ph .type{font-family:var(--mono);font-size:11px;text-transform:uppercase;letter-spacing:.6px;color:var(--muted);margin-top:3px}
  .ph .close{margin-left:auto;background:none;border:none;color:var(--muted);font-size:26px;line-height:1;cursor:pointer;padding:0 4px}

  .readout{display:flex;align-items:center;gap:14px;background:var(--surface);border:1px solid var(--line);
    border-radius:14px;padding:12px 14px;margin-bottom:14px}
  .readout .dist{font-family:var(--mono);font-size:26px;font-weight:700;color:var(--ember);line-height:1}
  .readout .dist small{font-size:12px;color:var(--muted);font-weight:400;margin-left:2px}
  .readout .lbl{font-family:var(--mono);font-size:10px;letter-spacing:.5px;text-transform:uppercase;color:var(--muted);margin-top:3px}
  .compass{width:44px;height:44px;flex:none;border-radius:50%;border:1px solid var(--line);display:grid;place-items:center;background:var(--bg)}
  .compass svg{transition:transform .4s}
  .route{margin-left:auto;height:40px;padding:0 15px;display:flex;align-items:center;gap:7px;border-radius:11px;
    background:var(--ember);color:#241400;font-weight:800;font-size:14px;text-decoration:none}

  .sec-t{font-family:var(--mono);font-size:10px;letter-spacing:.7px;text-transform:uppercase;color:var(--muted);margin:16px 2px 9px}
  .seg{display:flex;gap:8px}
  .seg button{flex:1;height:46px;border-radius:12px;border:1px solid var(--line);background:var(--surface);
    color:var(--muted);font-weight:700;font-size:14px;cursor:pointer;display:flex;align-items:center;justify-content:center;gap:6px}
  .seg button.on[data-v=ok]{background:rgba(107,217,138,.16);border-color:var(--ok);color:var(--ok)}
  .seg button.on[data-v=bad]{background:rgba(240,104,90,.16);border-color:var(--bad);color:var(--bad)}
  .seg button.on[data-v=unknown]{background:var(--surface-2);border-color:var(--muted);color:var(--ink)}

  .flags{display:flex;flex-wrap:wrap;gap:8px}
  .flag{padding:9px 13px;border-radius:11px;border:1px solid var(--line);background:var(--surface);
    color:var(--muted);font-size:13px;font-weight:600;cursor:pointer;display:flex;align-items:center;gap:7px}
  .flag.on{border-color:var(--ember);background:var(--ember-soft);color:var(--ember)}
  .flag.on.warn{border-color:var(--bad);background:rgba(240,104,90,.14);color:var(--bad)}

  .stars{display:flex;gap:6px}
  .stars span{font-size:30px;color:var(--line);cursor:pointer;transition:color .12s,transform .12s}
  .stars span.on{color:var(--ember)}
  .stars span:active{transform:scale(1.2)}

  .cmt-in{display:flex;gap:8px;margin-top:4px}
  .cmt-in input{flex:1;height:46px;background:var(--surface);border:1px solid var(--line);border-radius:12px;
    padding:0 14px;color:var(--ink);font-size:14px;outline:none;font-family:var(--sans)}
  .cmt-in button{width:46px;height:46px;border-radius:12px;border:none;background:var(--ember);color:#241400;font-size:20px;cursor:pointer}
  .cmt{background:var(--surface);border:1px solid var(--line);border-radius:12px;padding:11px 13px;margin-top:9px}
  .cmt .meta{font-family:var(--mono);font-size:10px;color:var(--muted);margin-bottom:4px;text-transform:uppercase;letter-spacing:.4px}
  .cmt p{margin:0;font-size:14px;line-height:1.45}
  .empty-note{color:var(--muted);font-size:13px;text-align:center;padding:6px 0 2px}
  .local-note{font-size:11px;color:var(--muted);text-align:center;margin-top:16px;line-height:1.5}

  /* ---- List view ---- */
  .listview{position:absolute;inset:0;z-index:650;background:var(--bg);transform:translateY(100%);
    transition:transform .3s;display:flex;flex-direction:column}
  .listview.open{transform:translateY(0)}
  .lv-head{padding:calc(env(safe-area-inset-top) + 12px) 16px 12px;display:flex;align-items:center;gap:12px;border-bottom:1px solid var(--line)}
  .lv-head h2{margin:0;font-size:18px;font-weight:800}
  .lv-head .count{font-family:var(--mono);font-size:11px;color:var(--muted);margin-left:auto}
  .lv-scroll{flex:1;overflow-y:auto;padding:8px 12px calc(env(safe-area-inset-bottom) + 20px)}
  .lv-item{display:flex;align-items:center;gap:13px;padding:13px 12px;border-radius:14px;cursor:pointer}
  .lv-item:active{background:var(--surface)}
  .lv-item .badge{width:40px;height:40px;flex:none;border-radius:11px;display:grid;place-items:center;font-size:19px;
    background:var(--surface);border:1px solid var(--line)}
  .lv-item .info{flex:1;min-width:0}
  .lv-item .info b{font-size:15px;font-weight:700;display:block;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
  .lv-item .info small{font-family:var(--mono);font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:.4px}
  .lv-item .d{font-family:var(--mono);font-size:15px;font-weight:700;color:var(--ember)}
  .lv-item .dot{width:7px;height:7px;border-radius:50%;flex:none}
</style>
</head>
<body>
<div id="app">
  <div id="map"></div>

  <div class="topbar">
    <div class="brand">
      <div class="ember"></div>
      <h1>Kippen<span>Karte</span></h1>
      <small id="sub">Automaten &amp; Kioske</small>
    </div>
    <div class="searchrow">
      <div class="search">
        <svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4-4"/></svg>
        <input id="cityInput" placeholder="Stadt oder Adresse suchen…" enterkeyhint="search">
      </div>
      <button class="iconbtn" id="locBtn" title="Mein Standort">
        <svg width="21" height="21" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="3.2"/><path d="M12 2v3M12 19v3M2 12h3M19 12h3"/></svg>
      </button>
    </div>
    <div class="suggest" id="suggestBox"></div>
    <div class="chips" id="radiusChips">
      <div class="chip" data-r="500">500 m</div>
      <div class="chip on" data-r="1500">1,5 km</div>
      <div class="chip" data-r="3000">3 km</div>
      <div class="chip" data-r="6000">6 km</div>
    </div>
  </div>

  <div class="toast" id="toast"></div>

  <button class="dbgbadge" id="dbgBadge">● Log</button>
  <div class="dbgpanel" id="dbgPanel">
    <div class="dbg-head">
      <b>Diagnose <span id="dbgVer"></span></b>
      <button id="dbgTest">Test: Frankfurt ⌖</button>
      <button id="dbgClose">×</button>
    </div>
    <div class="dbg-log" id="dbgLog"></div>
  </div>

  <button class="fab" id="listFab">
    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2"><path d="M8 6h13M8 12h13M8 18h13M3 6h.01M3 12h.01M3 18h.01"/></svg>
    <span id="fabCount">Liste</span>
  </button>

  <!-- List view -->
  <div class="listview" id="listView">
    <div class="lv-head">
      <button class="iconbtn" id="lvBack" style="width:40px;height:40px">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2"><path d="M15 18l-6-6 6-6"/></svg>
      </button>
      <h2>In der Nähe</h2>
      <span class="count" id="lvCount"></span>
    </div>
    <div class="lv-scroll" id="lvScroll"></div>
  </div>

  <!-- Bottom sheet -->
  <div class="sheet" id="sheet">
    <div class="grip"></div>
    <div class="sheet-scroll" id="sheetBody"></div>
  </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js"></script>
<script>
/* ================= Storage (Artifact window.storage → localStorage → memory) ============ */
const Store = (() => {
  const hasWS = !!(window.storage && window.storage.get);
  const mem = {};
  return {
    async get(k){
      if(hasWS){ try{ const r=await window.storage.get(k); return r?JSON.parse(r.value):null; }catch(e){ return null; } }
      try{ const v=localStorage.getItem(k); return v?JSON.parse(v):null; }catch(e){ return mem[k]??null; }
    },
    async set(k,v){
      const s=JSON.stringify(v);
      if(hasWS){ try{ await window.storage.set(k,s); return; }catch(e){} }
      try{ localStorage.setItem(k,s); return; }catch(e){ mem[k]=v; }
    }
  };
})();

/* ================= Categories ============ */
const CAT = {
  automat:    {label:"Zigarettenautomat", glyph:"🚬", color:"var(--ember)",  ember:true,  cig:"sicher"},
  tabak:      {label:"Tabakladen",        glyph:"🚬", color:"var(--gold)",   cig:"sicher"},
  spaeti:     {label:"Späti / Kiosk",     glyph:"🏪", color:"var(--blue)",   cig:"meistens"},
  kiosk:      {label:"Kiosk",             glyph:"🏪", color:"var(--blue)",   cig:"meistens"},
  supermarkt: {label:"Supermarkt",        glyph:"🛒", color:"var(--green)",  cig:"an der Kasse"},
  getraenke:  {label:"Getränkemarkt",     glyph:"🍺", color:"#6FC7B6",       cig:"oft"},
  tankstelle: {label:"Tankstelle",        glyph:"⛽", color:"var(--pink)",   cig:"sicher"},
};
function categorize(t){
  if(!t) return null;
  if(t.vending==="cigarettes") return "automat";
  if(t.shop==="tobacco") return "tabak";
  if(t.shop==="convenience") return "spaeti";
  if(t.shop==="kiosk") return "kiosk";
  if(t.shop==="newsagent") return "kiosk";
  if(t.shop==="beverages") return "getraenke";
  if(t.shop==="supermarket") return "supermarkt";
  if(t.amenity==="fuel") return "tankstelle";
  return null;
}

/* ================= Map ============ */
let userLat = 50.8676, userLon = 9.7040; // Fallback: Raum Bad Hersfeld
const map = L.map('map',{zoomControl:false,attributionControl:true}).setView([userLat,userLon],15);
L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png',{
  attribution:'&copy; OpenStreetMap &copy; CARTO', maxZoom:20, subdomains:'abcd'
}).addTo(map);

const markerLayer = L.layerGroup().addTo(map);
let meMarker = null;
let places = [];      // aktuelle POIs
let radius = 1500;
let markerIndex = {};  // key -> marker

/* ================= Helpers ============ */
function haversine(la1,lo1,la2,lo2){
  const R=6371000,r=Math.PI/180;
  const dLa=(la2-la1)*r,dLo=(lo2-lo1)*r;
  const a=Math.sin(dLa/2)**2+Math.cos(la1*r)*Math.cos(la2*r)*Math.sin(dLo/2)**2;
  return 2*R*Math.asin(Math.sqrt(a));
}
function bearing(la1,lo1,la2,lo2){
  const r=Math.PI/180;
  const y=Math.sin((lo2-lo1)*r)*Math.cos(la2*r);
  const x=Math.cos(la1*r)*Math.sin(la2*r)-Math.sin(la1*r)*Math.cos(la2*r)*Math.cos((lo2-lo1)*r);
  return (Math.atan2(y,x)/r+360)%360;
}
function fmtDist(m){ return m<1000 ? Math.round(m)+" m" : (m/1000).toFixed(1).replace(".",",")+" km"; }
function keyOf(el){ return el.type[0]+el.id; }

let toastT;
function toast(msg,loading,sticky){
  const el=document.getElementById('toast');
  el.innerHTML=(loading?'<span class="ld"></span>':'')+msg;
  el.classList.add('show'); clearTimeout(toastT);
  el.onclick=sticky?()=>el.classList.remove('show'):null;
  if(!loading && !sticky) toastT=setTimeout(()=>el.classList.remove('show'),3200);
}
function hideToast(){ document.getElementById('toast').classList.remove('show'); }

/* ================= Overpass ============ */
const OVERPASS = [
  "https://overpass-api.de/api/interpreter",
  "https://overpass.openstreetmap.fr/api/interpreter",
  "https://overpass.private.coffee/api/interpreter",
  "https://overpass.kumi.systems/api/interpreter",
  "https://overpass.osm.ch/api/interpreter"
];
function buildQuery(lat,lon,r){
  const f=k=>`nwr[${k}](around:${r},${lat},${lon});`;
  return `[out:json][timeout:25];(`+
    f('"vending"="cigarettes"')+f('"shop"="tobacco"')+f('"shop"="kiosk"')+
    f('"shop"="convenience"')+f('"shop"="supermarket"')+f('"amenity"="fuel"')+
    f('"shop"="beverages"')+f('"shop"="newsagent"')+
    `);out center;`;
}
function host(u){ try{ return new URL(u).hostname; }catch(e){ return u; } }
function fetchJSON(url,ms){
  const c=new AbortController(), t=setTimeout(()=>c.abort(),ms||13000);
  return fetch(url,{signal:c.signal}).finally(()=>clearTimeout(t));
}
async function tryOverpass(q){
  let best=null,bestUsed=null,anyOk=false,err=null;
  for(const url of OVERPASS){
    try{
      dbg("→ frage "+host(url)+" …");
      const res=await fetchJSON(url+"?data="+encodeURIComponent(q));
      if(!res.ok){ err="HTTP "+res.status+" ("+host(url)+")"; dbg("✗ "+err,'bad'); continue; }
      const j=await res.json(); anyOk=true;
      const n=(j.elements||[]).length;
      dbg("← 200 von "+host(url)+" · "+n+" Objekte",n?'ok':'');
      if(n>0) return {data:j,used:host(url),anyOk:true,err:null};
      if(!best){ best=j; bestUsed=host(url); }
    }catch(e){
      err=(e.name==='AbortError'?"Timeout nach 13s":(e.message||"Netzwerkfehler"))+" ("+host(url)+")";
      dbg("✗ "+err,'bad');
    }
  }
  return {data:best,used:bestUsed,anyOk,err};
}
async function fetchPlaces(lat,lon,r){
  toast("Kippenquellen werden gesucht…",true);
  dbg("Overpass-Abruf @ "+(+lat).toFixed(4)+","+(+lon).toFixed(4)+" · r="+r+"m",'in');
  const q=buildQuery(lat,lon,r);
  let R=await tryOverpass(q);
  if(!R.data || (R.data.elements||[]).length===0){
    dbg("Kein echter Treffer — 2. Versuch …",'in');
    await new Promise(s=>setTimeout(s,800));
    const R2=await tryOverpass(q);
    if((R2.data&&(R2.data.elements||[]).length>0) || (!R.data&&R2.data)) R=R2;
  }
  if(!R.data){
    dbg("Kein Server lieferte verwertbare Daten.",'bad');
    toast("⚠️ Kein Overpass-Server erreichbar ("+(R.err||"Netzwerk")+"). Log unten links.",false,true);
    return;
  }
  const used=R.used, raw=(R.data.elements||[]).length;
  places=(R.data.elements||[]).map(el=>{
    const la=el.lat??el.center?.lat, lo=el.lon??el.center?.lon;
    const cat=categorize(el.tags);
    if(la==null||lo==null||!cat) return null;
    return {key:keyOf(el),lat:la,lon:lo,cat,tags:el.tags||{},
            name:(el.tags?.name)||CAT[cat].label,
            dist:haversine(userLat,userLon,la,lo)};
  }).filter(Boolean).sort((a,b)=>a.dist-b.dist);
  await renderMarkers();
  hideToast();
  const autos=places.filter(p=>p.cat==="automat").length;
  dbg(raw+" Objekte → "+places.length+" relevant · "+autos+" Automaten",places.length?'ok':'bad');
  if(places.length===0){
    toast(raw>0
      ? (raw+" Objekte geladen, aber keine passende Kategorie dabei ("+used+").")
      : ("0 Treffer über "+used+" — dieser Server hat evtl. keine Deutschland-Daten. Log unten links prüfen."),false,true);
  } else {
    toast("✓ "+places.length+" Stellen · "+autos+" Automaten · via "+used);
  }
  document.getElementById('fabCount').textContent="Liste · "+places.length;
}

async function renderMarkers(){
  markerLayer.clearLayers(); markerIndex={};
  for(const p of places){
    const rep=await Store.get("place:"+p.key);
    const bad=rep?.status==="bad";
    const c=CAT[p.cat];
    const html=`<div class="pin ${c.ember?'pin-ember':''} ${bad?'pin-bad':''}" style="--pc:${c.color};position:relative"><span>${c.glyph}</span></div>`;
    const m=L.marker([p.lat,p.lon],{icon:L.divIcon({className:'',html,iconSize:[30,30],iconAnchor:[15,28]})});
    m.on('click',()=>openSheet(p));
    m.addTo(markerLayer); markerIndex[p.key]=m;
  }
}

/* ================= Locate / search ============ */
function setUser(lat,lon,zoom){
  userLat=lat; userLon=lon;
  if(meMarker) map.removeLayer(meMarker);
  meMarker=L.marker([lat,lon],{icon:L.divIcon({className:'',html:'<div class="me"></div>',iconSize:[18,18],iconAnchor:[9,9]}),zIndexOffset:1000}).addTo(map);
  map.setView([lat,lon],zoom||15);
}
function locate(){
  const btn=document.getElementById('locBtn'); btn.classList.add('live');
  toast("Standort wird ermittelt…",true);
  if(!navigator.geolocation){ toast("Standort nicht verfügbar — Stadt suchen."); btn.classList.remove('live'); return; }
  navigator.geolocation.getCurrentPosition(
    p=>{ setUser(p.coords.latitude,p.coords.longitude,16); btn.classList.remove('live'); fetchPlaces(userLat,userLon,radius); },
    e=>{ btn.classList.remove('live'); toast("Kein Standort ("+e.message+") — such deine Stadt oben."); },
    {enableHighAccuracy:true,timeout:8000,maximumAge:30000}
  );
}
async function searchCity(q){
  if(!q.trim()) return;
  toast("Suche „"+q+"“…",true);
  try{
    const r=await geocode(q,1);
    if(!r.length){ toast("Nichts gefunden zu „"+q+"“"); return; }
    setUser(r[0].lat,r[0].lon,15);
    fetchPlaces(r[0].lat,r[0].lon,radius);
  }catch(e){ toast("Suche fehlgeschlagen ("+e.message+")",false,true); }
}

/* ================= Bottom sheet ============ */
const sheet=document.getElementById('sheet'), sheetBody=document.getElementById('sheetBody');
let current=null, currentRep=null;

async function openSheet(p){
  current=p;
  currentRep=await Store.get("place:"+p.key) || {status:null,flags:{},stars:0,comments:[]};
  currentRep.flags=currentRep.flags||{}; currentRep.comments=currentRep.comments||[];
  renderSheet();
  sheet.classList.add('open');
  map.panTo([p.lat,p.lon],{animate:true});
}
function closeSheet(){ sheet.classList.remove('open'); current=null; }

function renderSheet(){
  const p=current, c=CAT[p.cat], rep=currentRep;
  const oh=p.tags.opening_hours;
  const allday = /24\/7/.test(oh||"") || p.cat==="automat";
  const br=bearing(userLat,userLon,p.lat,p.lon);
  const flag=(k,label,warn)=>`<div class="flag ${rep.flags[k]?'on':''} ${warn?'warn':''}" data-flag="${k}">${label}</div>`;
  const stars=[1,2,3,4,5].map(n=>`<span class="${n<=rep.stars?'on':''}" data-star="${n}">★</span>`).join('');
  const comments = rep.comments.length
    ? rep.comments.slice().reverse().map(cm=>`<div class="cmt"><div class="meta">${new Date(cm.t).toLocaleDateString('de-DE',{day:'2-digit',month:'short',hour:'2-digit',minute:'2-digit'})}${cm.stars?' · '+'★'.repeat(cm.stars):''}</div><p>${escapeHtml(cm.text)}</p></div>`).join('')
    : `<div class="empty-note">Noch kein Kommentar. Warst du hier? Sag den anderen Bescheid.</div>`;

  sheetBody.innerHTML=`
    <div class="ph">
      <div class="badge">${c.glyph}</div>
      <div>
        <h2>${escapeHtml(p.name)}</h2>
        <div class="type">${c.label} · Kippen ${c.cig}${allday?' · 24/7':''}</div>
      </div>
      <button class="close" id="sheetClose">×</button>
    </div>

    <div class="readout">
      <div class="compass"><svg width="20" height="20" viewBox="0 0 24 24" style="transform:rotate(${br}deg)"><path d="M12 2 L16 20 L12 15 L8 20 Z" fill="var(--ember)"/></svg></div>
      <div>
        <div class="dist">${fmtDist(p.dist).split(" ")[0]}<small> ${fmtDist(p.dist).split(" ")[1]}</small></div>
        <div class="lbl">Luftlinie</div>
      </div>
      <a class="route" target="_blank" rel="noopener" href="https://www.google.com/maps/dir/?api=1&destination=${p.lat},${p.lon}">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4"><path d="M3 11l19-9-9 19-2-8-8-2z"/></svg>Route</a>
    </div>

    <div class="sec-t">Läuft der Laden?</div>
    <div class="seg" id="statusSeg">
      <button data-v="ok" class="${rep.status==='ok'?'on':''}">✓ Läuft</button>
      <button data-v="bad" class="${rep.status==='bad'?'on':''}">✗ Defekt</button>
      <button data-v="unknown" class="${rep.status==='unknown'?'on':''}">? Weiß nicht</button>
    </div>

    <div class="sec-t">Was geht?</div>
    <div class="flags" id="flagWrap">
      ${flag('karte','💳 Kartenzahlung')}
      ${flag('bar','💶 Bargeld')}
      ${flag('allday','🌙 24/7')}
      ${flag('schluckt','⚠️ Schluckt Geld',true)}
    </div>

    <div class="sec-t">Deine Bewertung</div>
    <div class="stars" id="starWrap">${stars}</div>

    <div class="sec-t">Kommentare &amp; Status</div>
    <div class="cmt-in">
      <input id="cmtInput" placeholder="z. B. „Nimmt nur Karte, Fach 3 leer“" enterkeyhint="send">
      <button id="cmtSend">↑</button>
    </div>
    ${comments}

    <div class="local-note">Bewertungen werden vorerst nur auf <b>deinem Gerät</b> gespeichert.<br>Die native App bekommt später ein gemeinsames Backend, damit alle dieselben Infos sehen.</div>
  `;
  wireSheet();
}

async function saveRep(){ await Store.set("place:"+current.key,currentRep); }

function wireSheet(){
  document.getElementById('sheetClose').onclick=closeSheet;
  document.getElementById('statusSeg').querySelectorAll('button').forEach(b=>{
    b.onclick=async()=>{ currentRep.status=(currentRep.status===b.dataset.v?null:b.dataset.v); await saveRep(); renderSheet(); if(markerIndex[current.key]) renderMarkers(); };
  });
  document.getElementById('flagWrap').querySelectorAll('.flag').forEach(f=>{
    f.onclick=async()=>{ const k=f.dataset.flag; currentRep.flags[k]=!currentRep.flags[k]; await saveRep(); renderSheet(); };
  });
  document.getElementById('starWrap').querySelectorAll('span').forEach(s=>{
    s.onclick=async()=>{ currentRep.stars=+s.dataset.star; await saveRep(); renderSheet(); };
  });
  const send=async()=>{
    const inp=document.getElementById('cmtInput'); const v=inp.value.trim(); if(!v) return;
    currentRep.comments.push({t:Date.now(),text:v,stars:currentRep.stars,status:currentRep.status});
    await saveRep(); renderSheet();
  };
  document.getElementById('cmtSend').onclick=send;
  document.getElementById('cmtInput').addEventListener('keydown',e=>{ if(e.key==='Enter') send(); });
}
function escapeHtml(s){ return s.replace(/[&<>"']/g,m=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m])); }

/* ================= List view ============ */
const listView=document.getElementById('listView');
async function openList(){
  const scroll=document.getElementById('lvScroll');
  document.getElementById('lvCount').textContent=places.length+" Stellen";
  if(!places.length){ scroll.innerHTML='<div class="empty-note" style="padding:40px 0">Noch nichts geladen. Standort oder Stadt wählen.</div>'; }
  else{
    const rows=await Promise.all(places.map(async p=>{
      const rep=await Store.get("place:"+p.key);
      const dc = rep?.status==='ok'?'var(--ok)':rep?.status==='bad'?'var(--bad)':'transparent';
      return `<div class="lv-item" data-key="${p.key}">
        <div class="badge">${CAT[p.cat].glyph}</div>
        <div class="info"><b>${escapeHtml(p.name)}</b><small>${CAT[p.cat].label} · Kippen ${CAT[p.cat].cig}</small></div>
        <div class="dot" style="background:${dc}"></div>
        <div class="d">${fmtDist(p.dist)}</div>
      </div>`;
    }));
    scroll.innerHTML=rows.join('');
    scroll.querySelectorAll('.lv-item').forEach(it=>it.onclick=()=>{
      const p=places.find(x=>x.key===it.dataset.key);
      listView.classList.remove('open'); openSheet(p);
    });
  }
  listView.classList.add('open');
}

/* ================= Wire up ============ */
document.getElementById('locBtn').onclick=locate;
document.getElementById('listFab').onclick=openList;
document.getElementById('lvBack').onclick=()=>listView.classList.remove('open');
/* ---- City autocomplete (Tippen → Vorschläge) ---- */
const cityInput=document.getElementById('cityInput');
const suggestBox=document.getElementById('suggestBox');
let suggestT, suggestAbort;

cityInput.addEventListener('input',()=>{
  const q=cityInput.value.trim();
  clearTimeout(suggestT);
  if(q.length<2){ hideSuggest(); return; }
  suggestT=setTimeout(()=>loadSuggest(q),280);
});
cityInput.addEventListener('keydown',e=>{
  if(e.key==='Enter'){
    const first=suggestBox.querySelector('.row');
    if(first){ first.click(); } else { e.target.blur(); searchCity(e.target.value); }
    hideSuggest();
  }
});
/* Photon (komoot) — für Tipp-Vorschläge gebaut, blockt Website-Anfragen nicht */
function fmtFeat(f){
  const pr=f.properties||{}, c=f.geometry.coordinates;
  const main=pr.name||pr.city||pr.street||pr.district||pr.county||"?";
  const parts=[];
  if(pr.street&&pr.street!==main) parts.push(pr.street+(pr.housenumber?" "+pr.housenumber:""));
  if(pr.city&&pr.city!==main) parts.push(pr.city);
  if(pr.state) parts.push(pr.state);
  return {lat:c[1], lon:c[0], main, sub:parts.join(", ")||pr.country||""};
}
async function geocode(q,limit,signal){
  dbg("Suche „"+q+"“ → photon.komoot.io",'in');
  const res=await fetch("https://photon.komoot.io/api/?lang=de&limit="+(limit||6)+"&q="+encodeURIComponent(q), signal?{signal}:{});
  if(!res.ok){ dbg("✗ Photon HTTP "+res.status,'bad'); throw new Error("HTTP "+res.status); }
  const d=await res.json();
  const out=(d.features||[]).map(fmtFeat);
  dbg("← Photon: "+out.length+" Treffer",out.length?'ok':'');
  return out;
}
async function loadSuggest(q){
  try{
    if(suggestAbort) suggestAbort.abort();
    suggestAbort=new AbortController();
    renderSuggest(await geocode(q,6,suggestAbort.signal));
  }catch(e){ if(e.name!=='AbortError'){ hideSuggest(); } }
}
function renderSuggest(list){
  if(!list||!list.length){ hideSuggest(); return; }
  suggestBox.innerHTML=list.map((p,i)=>`<div class="row" data-i="${i}">
    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 21s-7-5.5-7-11a7 7 0 1114 0c0 5.5-7 11-7 11z"/><circle cx="12" cy="10" r="2.5"/></svg>
    <div class="txt"><b>${escapeHtml(p.main)}</b><small>${escapeHtml(p.sub)}</small></div></div>`).join('');
  suggestBox.querySelectorAll('.row').forEach(r=>r.onclick=()=>{
    const p=list[+r.dataset.i]; cityInput.value=p.main; hideSuggest(); cityInput.blur();
    setUser(p.lat,p.lon,14); fetchPlaces(p.lat,p.lon,radius);
  });
  suggestBox.classList.add('show');
}
function hideSuggest(){ suggestBox.classList.remove('show'); suggestBox.innerHTML=''; }
map.on('click',hideSuggest);
document.getElementById('radiusChips').querySelectorAll('.chip').forEach(ch=>{
  ch.onclick=()=>{ document.querySelectorAll('.chip').forEach(x=>x.classList.remove('on')); ch.classList.add('on');
    radius=+ch.dataset.r; fetchPlaces(userLat,userLon,radius); };
});

/* ================= Diagnose ============ */
const VERSION="v5";
const dbgLog=document.getElementById('dbgLog'), dbgBadge=document.getElementById('dbgBadge');
function dbg(msg,cls){
  const t=new Date().toLocaleTimeString('de-DE',{hour:'2-digit',minute:'2-digit',second:'2-digit'});
  const line=document.createElement('div'); if(cls) line.className=cls;
  line.textContent="["+t+"] "+msg; dbgLog.appendChild(line); dbgLog.scrollTop=dbgLog.scrollHeight;
  if(cls==='bad'){ dbgBadge.classList.add('err'); dbgBadge.classList.remove('ok'); }
  else if(cls==='ok' && !dbgBadge.classList.contains('err')){ dbgBadge.classList.add('ok'); }
  try{ console.log("[KK] "+msg); }catch(e){}
}
document.getElementById('dbgVer').textContent=VERSION;
dbgBadge.textContent="● Log "+VERSION;
dbgBadge.onclick=()=>document.getElementById('dbgPanel').classList.toggle('open');
document.getElementById('dbgClose').onclick=()=>document.getElementById('dbgPanel').classList.remove('open');
document.getElementById('dbgTest').onclick=()=>{
  document.getElementById('dbgPanel').classList.add('open');
  dbg("── Test-Abruf: Frankfurt Zentrum ──",'in');
  setUser(50.1109,8.6821,15); fetchPlaces(50.1109,8.6821,3000);
};
window.addEventListener('error',e=>dbg("JS-Fehler: "+e.message+(e.filename?" @ "+e.lineno:""),'bad'));
window.addEventListener('unhandledrejection',e=>dbg("Promise-Fehler: "+(e.reason&&e.reason.message||e.reason),'bad'));
dbg("Build "+VERSION+" geladen. Leaflet: "+(window.L?"OK":"FEHLT!"),window.L?'ok':'bad');

/* Start */
setUser(userLat,userLon,15);
locate();
setTimeout(()=>{ if(!places.length) fetchPlaces(userLat,userLon,radius); },3500);
</script>
</body>
</html>
