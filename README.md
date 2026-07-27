# marco-tools
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kalorien-Counter (laufend)</title>
<style>
  /* ============================================================
     ANPASSEN FÜR NEUE VIDEOS:
     - Zahlen: siehe CONFIG im <script> unten
     - Akzentfarbe: --accent unten ändern (Sushi = Lachs-Orange)
       Challenge = z.B. #3B82F6 (Blau), Psycho = #8B5CF6 (Violett)
     - Hintergrundbild: body background-image (im echten Einsatz
       nimmst du das raus und legst den Counter in CapCut über dein Footage)
     ============================================================ */
  :root {
    --accent: rgba(232,115,90,0.55);   /* Lachs-Orange fuer Food/Sushi */
    --success: #7DE0A6;                 /* Nori-Gruen fuer Pop-Effekt */
  }
  * { box-sizing:border-box; margin:0; padding:0; }
  body {
    min-height:100vh; display:flex; align-items:center; justify-content:center;
    font-family:-apple-system,'SF Pro Display',system-ui,sans-serif;
    background-image:url('https://images.unsplash.com/photo-1579871494447-9811cf80d66c?w=1200&q=80');
    background-size:cover; background-position:center;
  }
  .stage { padding:32px; width:100%; max-width:520px; }
  .modes { display:flex; gap:8px; margin-bottom:18px; flex-wrap:wrap; }
  .modes button {
    font-family:inherit; font-size:13px; font-weight:500; color:#fff;
    background:rgba(255,255,255,0.12); backdrop-filter:blur(12px);
    -webkit-backdrop-filter:blur(12px); border:1px solid rgba(255,255,255,0.35);
    border-radius:12px; padding:8px 16px; cursor:pointer;
  }
  .modes button.on { background:rgba(255,255,255,0.22); border-color:rgba(255,255,255,0.5); }
  .card {
    max-width:360px; backdrop-filter:blur(22px) saturate(140%);
    -webkit-backdrop-filter:blur(22px) saturate(140%);
    background:rgba(255,255,255,0.14); border:1px solid rgba(255,255,255,0.45);
    border-radius:24px; padding:22px 26px; box-shadow:0 8px 36px rgba(0,0,0,0.30);
    transition:transform 0.18s cubic-bezier(0.34,1.56,0.64,1), background 0.4s ease, border-color 0.4s ease;
  }
  .lab { font-size:12px; font-weight:500; letter-spacing:1.5px; color:rgba(255,255,255,0.88); text-transform:uppercase; }
  .kcal { font-size:52px; font-weight:600; color:#fff; line-height:1; letter-spacing:-1.5px; text-shadow:0 2px 14px rgba(0,0,0,0.35); }
  .foot { display:flex; justify-content:space-between; align-items:flex-end; margin-top:14px; padding-top:14px; border-top:1px solid rgba(255,255,255,0.28); }
  .sublab { font-size:10px; font-weight:500; letter-spacing:1.5px; color:rgba(255,255,255,0.7); text-transform:uppercase; margin-bottom:3px; }
  .eur { font-size:26px; font-weight:600; color:#fff; line-height:1; text-shadow:0 2px 10px rgba(0,0,0,0.3); }
  .paid { font-size:18px; font-weight:500; color:rgba(255,255,255,0.9); }
  .controls { display:flex; gap:10px; margin-top:20px; align-items:center; flex-wrap:wrap; }
  .controls button { font-family:inherit; font-size:14px; font-weight:500; border-radius:12px; padding:11px 22px; cursor:pointer; }
  .play { color:#1a1a1a; background:rgba(255,255,255,0.92); border:none; box-shadow:0 4px 16px rgba(0,0,0,0.2); }
  .step { color:#fff; background:var(--accent); backdrop-filter:blur(12px); -webkit-backdrop-filter:blur(12px); border:1px solid rgba(255,255,255,0.4); }
  .reset { color:#fff; background:rgba(255,255,255,0.15); backdrop-filter:blur(12px); -webkit-backdrop-filter:blur(12px); border:1px solid rgba(255,255,255,0.35); }
  .info { font-size:12px; color:rgba(255,255,255,0.85); background:rgba(0,0,0,0.25); padding:6px 12px; border-radius:10px; }
</style>
</head>
<body>
<div class="stage">
  <div class="modes">
    <button id="btnR1" class="on" onclick="setMode('R1')">Restaurant 1</button>
    <button id="btnR2" onclick="setMode('R2')">Restaurant 2</button>
    <button id="btnTotal" onclick="setMode('TOTAL')">Gesamt</button>
  </div>
  <div class="card" id="glassCard">
    <div style="display:flex; align-items:center; gap:10px; margin-bottom:8px;">
      <span style="font-size:24px;">🔥</span>
      <span class="lab" id="cardLabel">Restaurant 1 · Kalorien</span>
    </div>
    <div class="kcal" id="kcalVal">0</div>
    <div class="foot">
      <div><div class="sublab">Wert im Restaurant</div><div class="eur" id="eurVal">0 €</div></div>
      <div style="text-align:right;"><div class="sublab">Gezahlt</div><div class="paid" id="paidVal">0 €</div></div>
    </div>
  </div>
  <div class="controls">
    <button class="play" id="playBtn" onclick="togglePlay()">▶ Auto-Zählen</button>
    <button class="step" onclick="stepUp()">+ Ein Sprung</button>
    <button class="reset" onclick="resetCounter()">↺ Reset</button>
    <span class="info" id="stepInfo">Sprung 0 / 8</span>
  </div>
</div>
<script>
/* ============================================================
   HIER DIE ZAHLEN ANPASSEN — pro Video / pro Restaurant.
   kcal = Endzahl Kalorien, eur = Essenswert, paid = gezahlt,
   steps = in wie vielen Spruengen hochgezaehlt wird.
   Willst du ECHTE Zwischenstaende statt gleichmaessiger Spruenge?
   -> stops:[...] eintragen (Beispiel bei R1). Wenn stops fehlt,
      wird gleichmaessig in 'steps' Schritten gezaehlt.
   ============================================================ */
const CONFIG = {
  R1:    { label:'Restaurant 1 · Kalorien', kcal:26362, eur:799, paid:175, steps:8 /*, stops:[6000,12000,18000,22000,26362] */ },
  R2:    { label:'Restaurant 2 · Kalorien', kcal:11917, eur:401, paid:70,  steps:6 },
  TOTAL: { label:'Beide zusammen · Kalorien', kcal:38279, eur:1200, paid:245, steps:10 }
};
let mode='R1', curStep=0, playing=false, timer=null;
function fmt(n){ return Math.round(n).toLocaleString('de-DE'); }
function cfg(){ return CONFIG[mode]; }
function totalSteps(){ const c=cfg(); return c.stops ? c.stops.length : c.steps; }
function valsAt(step){
  const c=cfg();
  if(c.stops){
    if(step<=0) return {kcal:0,eur:0};
    const k=c.stops[Math.min(step,c.stops.length)-1];
    return {kcal:k, eur:Math.round(c.eur*(k/c.kcal))};
  }
  const frac=step/c.steps;
  return {kcal:Math.round(c.kcal*frac), eur:Math.round(c.eur*frac)};
}
function render(){
  const c=cfg(); const v=valsAt(curStep);
  document.getElementById('kcalVal').textContent=fmt(v.kcal);
  document.getElementById('eurVal').textContent=fmt(v.eur)+' €';
  document.getElementById('paidVal').textContent=fmt(c.paid)+' €';
  document.getElementById('cardLabel').textContent=c.label;
  document.getElementById('stepInfo').textContent='Sprung '+curStep+' / '+totalSteps();
}
function popFlash(){
  const card=document.getElementById('glassCard');
  const acc=getComputedStyle(document.documentElement).getPropertyValue('--success').trim();
  card.style.transform='scale(1.045)';
  card.style.background='rgba(125,224,166,0.30)';
  card.style.borderColor='rgba(125,224,166,0.85)';
  const k=document.getElementById('kcalVal');
  k.style.transition='color 0.15s ease'; k.style.color='#a8f5c8';
  setTimeout(()=>{ card.style.transform='scale(1)'; card.style.background='rgba(255,255,255,0.14)'; card.style.borderColor='rgba(255,255,255,0.45)'; k.style.color='#fff'; },320);
}
function stepUp(){ if(curStep>=totalSteps())return; curStep++; render(); popFlash(); if(curStep>=totalSteps()&&playing)togglePlay(); }
function togglePlay(){
  playing=!playing; const btn=document.getElementById('playBtn');
  if(playing){ btn.textContent='⏸ Pause'; if(curStep>=totalSteps())resetCounter(true);
    timer=setInterval(()=>{ stepUp(); if(curStep>=totalSteps()){clearInterval(timer); playing=false; btn.textContent='▶ Auto-Zählen';} },650);
  } else { btn.textContent='▶ Auto-Zählen'; clearInterval(timer); }
}
function resetCounter(keepPlay){ curStep=0; render(); if(!keepPlay&&playing){playing=false; clearInterval(timer); document.getElementById('playBtn').textContent='▶ Auto-Zählen';} }
function setMode(m){ mode=m; resetCounter();
  ['R1','R2','Total'].forEach(id=>{ const on=('btn'+(m==='TOTAL'?'Total':m))===('btn'+id); document.getElementById('btn'+id).classList.toggle('on',on); });
}
render();
</script>
</body>
</html>
