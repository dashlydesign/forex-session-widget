<html>
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
  gap:36px;
}

.clocks{
  display:flex;
  gap:140px;
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
  opacity:.55;
}

.session.active{
  opacity:1;
}

.countdowns{
  display:flex;
  gap:64px;
  font-size:14px;
}
</style>
</head>

<body>

<div class="wrapper">

  <div class="clocks">
    <canvas id="mt5" width="160" height="160"></canvas>
    <canvas id="ist" width="160" height="160"></canvas>
  </div>

  <div class="sessions" id="sessions"></div>

  <div class="countdowns">
    <div id="end"></div>
    <div id="next"></div>
  </div>

</div>

<script>
// -------- TIME BASE --------
let utcOffsetMs = 0;
fetch("https://worldtimeapi.org/api/timezone/Etc/UTC")
  .then(r=>r.json())
  .then(d=>{
    utcOffsetMs = new Date(d.utc_datetime).getTime() - Date.now();
  })
  .catch(()=>{});

// -------- SESSIONS (MT5 UTC+2) --------
const sessions = [
  {name:"Sydney", start:0, end:9},
  {name:"Tokyo", start:2, end:11},
  {name:"London", start:9, end:18},
  {name:"New York", start:14, end:23}
];

// render session boxes
const sessionsEl = document.getElementById("sessions");
sessions.forEach(s=>{
  const d=document.createElement("div");
  d.className="session";
  d.textContent=`${s.name} ${String(s.start).padStart(2,"0")}–${String(s.end).padStart(2,"0")}`;
  sessionsEl.appendChild(d);
});

// -------- CLOCK DRAW --------
function drawClock(canvas, date){
  const ctx = canvas.getContext("2d");
  const r = canvas.width/2;

  ctx.setTransform(1,0,0,1,0,0);
  ctx.clearRect(0,0,canvas.width,canvas.height);
  ctx.translate(r,r);

  // ticks
  ctx.strokeStyle="rgba(255,255,255,.25)";
  ctx.lineWidth=1;
  for(let i=0;i<12;i++){
    ctx.beginPath();
    const a=i*Math.PI/6;
    ctx.moveTo(Math.cos(a)*r*0.82,Math.sin(a)*r*0.82);
    ctx.lineTo(Math.cos(a)*r*0.88,Math.sin(a)*r*0.88);
    ctx.stroke();
  }

  const sec=date.getSeconds()+date.getMilliseconds()/1000;
  const min=date.getMinutes()+sec/60;
  const hr=date.getHours()%12+min/60;

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

// -------- COUNTDOWNS --------
function updateSessionsAndCountdown(mt5){
  const t = mt5.getHours()*3600 + mt5.getMinutes()*60 + mt5.getSeconds();

  let current, next;
  for(let i=0;i<sessions.length;i++){
    if(t>=sessions[i].start*3600 && t<sessions[i].end*3600){
      current=sessions[i];
      next=sessions[i+1]||sessions[0];
      break;
    }
  }
  if(!current){current=sessions[0];next=sessions[1];}

  // highlight active
  [...sessionsEl.children].forEach(el=>{
    el.classList.toggle("active",el.textContent.startsWith(current.name));
  });

  let toEnd=current.end*3600-t;
  if(toEnd<0)toEnd+=86400;

  let toNext=next.start*3600-t;
  if(toNext<0)toNext+=86400;

  const f=v=>String(v).padStart(2,"0");

  document.getElementById("end").textContent=
    `${current.name} ends in ${f(toEnd/3600|0)}:${f((toEnd%3600)/60|0)}:${f(toEnd%60)}`;

  document.getElementById("next").textContent=
    `${next.name} starts in ${f(toNext/3600|0)}:${f((toNext%3600)/60|0)}:${f(toNext%60)}`;
}

// -------- LOOP --------
const mt5Canvas=document.getElementById("mt5");
const istCanvas=document.getElementById("ist");

function loop(){
  const now=new Date(Date.now()+utcOffsetMs);
  const mt5=new Date(now.getTime()+2*3600000);
  const ist=new Date(now.getTime()+5.5*3600000);

  drawClock(mt5Canvas,mt5);
  drawClock(istCanvas,ist);
  updateSessionsAndCountdown(mt5);

  requestAnimationFrame(loop);
}
loop();
</script>

</body>
</html>
