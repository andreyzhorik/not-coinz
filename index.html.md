<!DOCTYPE html>  
<html lang="en">  
<html lang="en">  
<head>  
<head>  
<meta charset="utf-8" />  
<meta charset="utf-8" />  
<title>Spin the Wheel — Fixed</title>  
<title>Spin the Wheel — Fixed</title>  
<style>  
<style>  
  body{  
    font-family: 'Segoe UI', Arial, sans-serif;  
    font-family: 'Segoe UI', Arial, sans-serif;  
    background: #0f0f18;  
    background: #0f0f18;  
    color: #fff;  
    color: #fff;  
    margin:0;  
    margin:0;  
    min-height:100vh;  
    min-height:100vh;  
    display:flex;  
    align-items:center;  
    justify-content:center;  
    justify-content:center;  
    flex-direction:column;  
    flex-direction:column;  
    gap:18px;  
  }  
  }  
  .wheel-container{ position: relative; width:500px; height:500px; }  
  #arrow{  
    position:absolute; left:50%; top:-5px; transform:translateX(-50%);  
    width:0;height:0;  
    border-left:18px solid transparent;  
    border-right:18px solid transparent;  
    border-top:28px solid #37106e;  
    z-index:10;  
  }  
  canvas{ border-radius:50%; border:8px solid #fff; display:block; }  
  canvas{ border-radius:50%; border:8px solid #fff; display:block; }  
  .controls{ display:flex; gap:10px; align-items:center; flex-direction:column; }  
  .controls{ display:flex; gap:10px; align-items:center; flex-direction:column; }  
  #spin { padding:10px 20px; font-size:16px; border-radius:8px; cursor:pointer; background:#28a745; color:white; border:none;}  
  input[type=number]{ padding:8px; width:110px; text-align:center; border-radius:6px; border:none; }  
  input[type=number]{ padding:8px; width:110px; text-align:center; border-radius:6px; border:none; }  
  #coins{ font-weight:600; margin-top:6px; }  
  button#reset{ background:#e74c3c; color:white; border:none; padding:6px 10px; border-radius:6px; cursor:pointer;}  
  button#reset{ background:#e74c3c; color:white; border:none; padding:6px 10px; border-radius:6px; cursor:pointer;}  
</style>  
</style>  
</head>  
</head>  
<body>  
<body>  
  <h1>🎡 Spin the Wheel</h1>  
  
  <div class="wheel-container">  
    <div id="arrow"></div>  
    <canvas id="wheel" width="500" height="500"></canvas>  
  </div>  
  </div>  
  
  <div class="controls">  
  <div class="controls">  
    <div>  
      <label>Bet: </label>  
      <input id="bet" type="number" min="1" value="10" />  
      <input id="bet" type="number" min="1" value="10" />  
      <button id="spin">SPIN</button>  
    </div>  
    <div id="coins">💰 Coins: <span id="coinCount">10</span></div>  
  </div>  
  
<script>  
<script>  
/* ---------------- CONFIG ---------------- */  
const outcomes = [  
const outcomes = [  
  "1.5x","0.5x","0.5x","1.1x","1.8x","10x","1.9x","Lose","Lose","1.7x",  
  "Lose","1x","Lose","1x","Lose","Lose","1.4x","Lose","Lose","1.2x",  
  "Lose","Lose","2.0x","5.0x","2.5x","1.3x","Lose","1.6x","0.5x","Lose",  
  "2.5x","Lose","0.75x","Lose","2.0x","Lose"  
];  
];  
  
const colors = [  
const colors = [  
  "#2ecc71","#3498db","#9b59b6","#f1c40f","#1abc9c","#84ff00","#16a085","#e74c3c","#e74c3c","#d35400",  
  "#2ecc71","#3498db","#9b59b6","#f1c40f","#1abc9c","#84ff00","#16a085","#e74c3c","#e74c3c","#d35400",  
  "#e74c3c","#f39c12","#e74c3c","#27ae60","#e74c3c","#e74c3c","#9b59b6","#e74c3c","#e74c3c","#f1c40f",  
  "#e74c3c","#f39c12","#e74c3c","#27ae60","#e74c3c","#e74c3c","#9b59b6","#e74c3c","#e74c3c","#f1c40f",  
  "#e74c3c","#e74c3c","#2980b9","#84ff00","#8e44ad","#2ecc71","#e74c3c","#1abc9c","#f39c12","#e74c3c",  
  "#27ae60","#e74c3c","#f1c40f","#e74c3c","#16a085","#e74c3c"  
  "#27ae60","#e74c3c","#f1c40f","#e74c3c","#16a085","#e74c3c"  
];  
];  
  
  
const canvas = document.getElementById('wheel');  
const ctx = canvas.getContext('2d');  
const spinBtn = document.getElementById('spin');  
const resetBtn = document.getElementById('reset');  
const resetBtn = document.getElementById('reset');  
const coinText = document.getElementById('coinCount');  
const betInput = document.getElementById('bet');  
const betInput = document.getElementById('bet');  
  
const cx = canvas.width/2, cy = canvas.height/2;  
const radius = canvas.width/2 - 8;  
const N = outcomes.length;  
const N = outcomes.length;  
const sliceDeg = 360 / N;  
const RAD = Math.PI/180;  
const RAD = Math.PI/180;  
  
/* ---------------- STORAGE ---------------- */  
/* ---------------- STORAGE ---------------- */  
let coins = parseInt(localStorage.getItem('coins')) || 100;  
let coins = parseInt(localStorage.getItem('coins')) || 100;  
coinText.textContent = coins;  
  
/* rotation in degrees (not normalized) */  
let rotationDeg = 0;  
let rotationDeg = 0;  
  
/* ---------------- DRAW ---------------- */  
/* ---------------- DRAW ---------------- */  
function drawWheel(rotDeg){  
function drawWheel(rotDeg){  
  ctx.clearRect(0,0,canvas.width,canvas.height);  
  ctx.save();  
  ctx.translate(cx,cy);  
  ctx.translate(cx,cy);  
  // IMPORTANT: rotate by (rotDeg - 90) so a rotationDeg that equals 0 places slice 0 to the top.  
  // IMPORTANT: rotate by (rotDeg - 90) so a rotationDeg that equals 0 places slice 0 to the top.  
  ctx.rotate((rotDeg - 90) * RAD);  
  ctx.rotate((rotDeg - 90) * RAD);  
  
  const slice = (2 * Math.PI) / N;  
  const slice = (2 * Math.PI) / N;  
  for(let i=0;i<N;i++){  
    const angle = i * slice;  
    // slice  
    // slice  
    ctx.beginPath();  
    ctx.moveTo(0,0);  
    ctx.arc(0,0,radius, angle, angle + slice);  
    ctx.arc(0,0,radius, angle, angle + slice);  
    ctx.closePath();  
    ctx.closePath();  
    ctx.fillStyle = colors[i % colors.length];  
    ctx.fill();  
    ctx.fill();  
    ctx.strokeStyle = "#111";  
    ctx.strokeStyle = "#111";  
    ctx.lineWidth = 2;  
    ctx.lineWidth = 2;  
    ctx.stroke();  
  
    // label: place at slice center, a bit out from center  
    // label: place at slice center, a bit out from center  
    ctx.save();  
    ctx.save();  
    ctx.rotate(angle + slice/2);  
    ctx.rotate(angle + slice/2);  
    ctx.translate(radius * 0.90, 0);  
    // rotate so text is upright relative to screen (this keeps top labels readable)  
    // rotate so text is upright relative to screen (this keeps top labels readable)  
    ctx.rotate(Math.PI/2);  
    ctx.rotate(Math.PI/2);  
    ctx.fillStyle = "white";  
    ctx.font = "bold 14px Arial";  
    ctx.textAlign = "center";  
    ctx.textAlign = "center";  
    ctx.textBaseline = "middle";  
    ctx.fillText(outcomes[i], 0, 0);  
    ctx.fillText(outcomes[i], 0, 0);  
    ctx.restore();  
  }  
  
  ctx.restore();  
  ctx.restore();  
}  
}  
  
// initial draw  
// initial draw  
drawWheel(rotationDeg);  
drawWheel(rotationDeg);  
  
/* ---------------- SPIN LOGIC (FIXED) ----------------  
/* ---------------- SPIN LOGIC (FIXED) ----------------  
   Key idea:  
   Key idea:  
   - slice center (measured from RIGHT, clockwise) = idx*sliceDeg + sliceDeg/2  
   - to place that center UNDER THE TOP ARROW after drawWheel (which rotates by rotDeg-90),  
   - to place that center UNDER THE TOP ARROW after drawWheel (which rotates by rotDeg-90),  
     compute baseRotationDeg = (360 - sliceCenterDeg) % 360  
     compute baseRotationDeg = (360 - sliceCenterDeg) % 360  
   - normalize current rotation, compute the minimal positive delta needed to reach baseRotation,  
     then add full spins so the wheel rotates many times.  
*/  
*/  
let spinning = false;  
let spinning = false;  
  
spinBtn.addEventListener('click', () => {  
spinBtn.addEventListener('click', () => {  
  if (spinning) return;  
  if (spinning) return;  
  
  const bet = Math.floor(Number(betInput.value));  
  if (!bet || bet <= 0) return alert("Enter a valid bet");  
  if (bet > coins) return alert("Not enough coins");  
  if (bet > coins) return alert("Not enough coins");  
  
  // consume bet immediately  
  // consume bet immediately  
  coins -= bet;  
  localStorage.setItem('coins', coins);  
  coinText.textContent = coins;  
  
  spinning = true;  
  
  // choose a random index (winning slice)  
  // choose a random index (winning slice)  
  const idx = Math.floor(Math.random() * N);  
  const idx = Math.floor(Math.random() * N);  
  const result = outcomes[idx];  
  
  // normalize current rotation to [0,360)  
  // normalize current rotation to [0,360)  
  const currentNorm = ((rotationDeg % 360) + 360) % 360;  
  const currentNorm = ((rotationDeg % 360) + 360) % 360;  
  
  // slice center measured from RIGHT, clockwise:  
  // slice center measured from RIGHT, clockwise:  
  const sliceCenterDeg = idx * sliceDeg + sliceDeg / 2;  
  
  // baseRotationDeg is rotationDeg value (0..360) that would place that slice center at top.  
  // derived from geometry (see explanation in chat): baseRotationDeg = 360 - sliceCenterDeg  
  const baseRotationDeg = (360 - sliceCenterDeg) % 360;  
  const baseRotationDeg = (360 - sliceCenterDeg) % 360;  
  
  // compute minimal positive delta from currentNorm to baseRotationDeg  
  // compute minimal positive delta from currentNorm to baseRotationDeg  
  const delta = (baseRotationDeg - currentNorm + 360) % 360;  
  const delta = (baseRotationDeg - currentNorm + 360) % 360;  
  
  // add several full spins so it rotates many times before stopping  
  // add several full spins so it rotates many times before stopping  
  const fullSpins = 6 + Math.floor(Math.random() * 5); // 6..10 spins  
  const targetRotation = rotationDeg + fullSpins * 360 + delta;  
  
  // animate from rotationDeg to targetRotation  
  const start = rotationDeg;  
  const start = rotationDeg;  
  const duration = 4200; // ms  
  const t0 = performance.now();  
  const t0 = performance.now();  
  
  function step(t){  
    const elapsed = t - t0;  
    const p = Math.min(elapsed / duration, 1);  
    // smooth ease out cubic  
    // smooth ease out cubic  
    const ease = 1 - Math.pow(1 - p, 3);  
    rotationDeg = start + (targetRotation - start) * ease;  
    drawWheel(rotationDeg);  
    if (p < 1) {  
      requestAnimationFrame(step);  
      requestAnimationFrame(step);  
    } else {  
      // snap and normalize to avoid precision buildup  
      rotationDeg = (baseRotationDeg + 360 * Math.floor(rotationDeg / 360));  
      rotationDeg = (baseRotationDeg + 360 * Math.floor(rotationDeg / 360));  
      // ensure rotationDeg normalized for future spins (keeps integer-ish)  
      // ensure rotationDeg normalized for future spins (keeps integer-ish)  
      rotationDeg = Math.round(rotationDeg * 1000) / 1000;  
      rotationDeg = Math.round(rotationDeg * 1000) / 1000;  
  
      // payout  
      let win = 0;  
      if (result !== "Lose") {  
      if (result !== "Lose") {  
        const mult = parseFloat(result.replace("x",""));  
        const mult = parseFloat(result.replace("x",""));  
        win = Math.floor(bet * mult);  
        win = Math.floor(bet * mult);  
        alert(`🎉 You won ${win} coins! (${result})`);  
        alert(`🎉 You won ${win} coins! (${result})`);  
      } else {  
        alert("😢 You lost your bet!");  
        alert("😢 You lost your bet!");  
      }  
      }  
      coins += win;  
      localStorage.setItem('coins', coins);  
      localStorage.setItem('coins', coins);  
      coinText.textContent = coins;  
      spinning = false;  
      spinning = false;  
    }  
    }  
  }  
  }  
  requestAnimationFrame(step);  
});  
  
/* Reset button */  
resetBtn.addEventListener('click', () => {  
resetBtn.addEventListener('click', () => {  
  coins = 100;  
  localStorage.setItem('coins', coins);  
  localStorage.setItem('coins', coins);  
  coinText.textContent = coins;  
  alert("Coins reset to 100");  
  alert("Coins reset to 100");  
});  
</script>  
</body>  
</body>  
</html>  
  
  
