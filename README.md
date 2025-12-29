<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">

<style>
body{
  margin:0;
  background:#121212;
  color:#fff;
  font-family:Segoe UI, system-ui, sans-serif;
  height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
}

.wrapper{
  display:flex;
  flex-direction:column;
  align-items:center;
  gap:32px;
}

.clocks{
  display:flex;
  gap:120px;
}

canvas{display:block}

.sessions{
  display:grid;
  grid-template-columns:repeat(4,1fr);
  gap:20px;
  width:680px;
}

.session{
  background:#1a1a1a;
  border-radius:10px;
  padding:14px 0;
  text-align:center;
  font-size:14px;
}

.countdowns{
  display:flex;
  gap:56px;
  font-size:14px;
  opacity:.9;
}
</style>
</head>

<body>

<div class="wrapper">

  <div class="clocks">
    <canvas id="mt5" width="160" height="160"></canvas>
    <canvas id="ist" width="160" height="160"></canvas>
  </div>

  <div class="sessions">
    <div class="session">Sydney 00–09</div>
    <div class="session">Tokyo 02–11</div>
    <div class="session">London 09–18</div>
    <div class="session">New York 14–23</div>
  </div>

  <div class="countdowns">
    <div id="end"></div>
    <div id="next"></div>
  </div>

</div>

<script>
// ---- TIME BASE ----
let utcOffsetMs = 0;

// ---- SAFE INTERNET SYNC (NON-BLOCKING) ----
fetch("https://worldtimeapi.org/api/timezone/Etc/UTC")
  .then(r => r.json())
  .then(d => {
    const serverUTC = new Date(d.utc_datetime).getTime();
    utcOffsetMs = serverUTC - Date.now();
  })
  .catch(()=>{}); // fallback silently

// ---- SESSIONS (MT5 UTC+2) ----
const sessions = [
  {name:"Sydney", start:0, end:9},
  {name:"Tokyo", start:2, end:11},
  {name:"London", start:9, end:18},
  {name:"New York", start:14, end:23}
];

// ---- DRAW CLOCK ----
function drawClock(canvas, date){
  const ctx = canvas.getContext("2d");
  const r = canvas.width/2;

  ctx.setTransform(1,0,0,1,0,0);
  ctx.clearRect(0,0,canvas.width,canvas.height);
  ctx.translate(r,r);

  const sec = date.getSeconds()+date.getMilliseconds()/1000;
  const min = date.getMinutes()+sec/60;
  const hr  = date.getHours()%12+min/60;

  function hand(a,l,w){
    ctx.beginPath();
    ctx.lineWidth=w;
    ctx.lineCap="round";
    ctx.strokeStyle="#fff";
    ctx.moveTo(0,0);
    ctx.lineTo(
      Math.cos(a-Math.PI/2)*l,
      Math.sin(a-Math.PI/2)*l
    );
    ctx.stroke();
  }

  hand(hr*Math.PI/6,  r*0.48, 4.2);
  hand(min*Math.PI/30, r*0.68, 2.7);
  hand(sec*Math.PI/30, r*0.83, 1.6);
}

// ---- COUNTDOWN ----
function updateCountdowns(mt5){
  const t = mt5.getHours()*3600+mt5.getMinutes()*60+mt5.getSeconds();
  let c,n;

  for(let i=0;i<sessions.length;i++){
    const s=sessions[i];
    if(t>=s.start*3600 && t<s.end*3600){
      c=s; n=sessions[i+1]||sessions[0]; break;
    }
  }

  if(!c){ c=sessions[0]; n=sessions[1]; }

  let r=c.end*3600-t; if(r<0) r+=86400;

  document.getElementById("end").textContent =
    String(Math.floor(r/3600)).padStart(2,"0")+":"+
    String(Math.floor((r%3600)/60)).padStart(2,"0")+":"+
    String(r%60).padStart(2,"0");

  document.getElementById("next").textContent =
    String(n.start).padStart(2,"0")+":00";
}

// ---- LOOP ----
function loop(){
  const now = new Date(Date.now()+utcOffsetMs);
  const mt5 = new Date(now.getTime()+2*3600000);
  const ist = new Date(now.getTime()+5.5*3600000);

  drawClock(mt5Canvas,mt5);
  drawClock(istCanvas,ist);
  updateCountdowns(mt5);

  requestAnimationFrame(loop);
}

const mt5Canvas=document.getElementById("mt5");
const istCanvas=document.getElementById("ist");
loop();
</script>

</body>
</html>
