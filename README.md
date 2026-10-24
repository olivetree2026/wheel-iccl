
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>World Cup Cricket Lottery Wheel</title>
<style>
* { box-sizing: border-box; }
body {
  margin: 0;
  background: radial-gradient(circle at top, #164c44, #081b29 70%);
  color: white;
  font-family: Arial, sans-serif;
  text-align: center;
}
header { padding: 25px 10px; }
h1 { color: #f9d77e; }
main { max-width: 850px; margin: auto; padding: 15px; }
.wheel-area {
  position: relative;
  width: min(90vw, 480px);
  height: min(90vw, 480px);
  margin: 20px auto;
}
.pointer {
  position: absolute;
  top: -10px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 5;
  font-size: 38px;
  color: #ffdc73;
}
canvas {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  border: 9px solid #dcb963;
  box-shadow: 0 0 40px #dcb96355;
}
button, select {
  padding: 13px 18px;
  margin: 6px;
  border: 0;
  border-radius: 10px;
  font-weight: bold;
  font-size: 15px;
}
button {
  background: #f5c65b;
  color: #10212c;
  cursor: pointer;
}
button:disabled { opacity: .4; }
select { background: white; }
#result {
  font-size: 26px;
  font-weight: bold;
  color: #ffdc73;
  min-height: 45px;
  margin: 20px;
}
#history {
  max-width: 500px;
  margin: 15px auto;
  text-align: left;
  line-height: 2;
}
.note {
  color: #b8cbd0;
  font-size: 13px;
}
</style>
</head>
<body>
<header>
  <h1>🏆 WORLD CUP CRICKET</h1>
  <h2>🎯 LOTTERY SPINNING WHEEL</h2>
  <p>10 Countries • 10 Numbers • One Lucky Selection</p>
</header>

<main>
  <div class="wheel-area">
    <div class="pointer">▼</div>
    <canvas id="wheel" width="600" height="600"></canvas>
  </div>

  <div>
    <button id="start">▶ START SPIN</button>
    <button id="stop" disabled>■ STOP SPIN</button>
    <button id="reset">↻ RESET</button>
  </div>

  <div>
    <select id="number">
      <option value="">Select a number</option>
    </select>
    <button id="choose">SELECT NUMBER</button>
  </div>

  <div id="result">Ready to spin!</div>
  <h3>Draw History</h3>
  <div id="history">No results yet.</div>
  <p class="note">
    STOP SPIN determines the result from the wheel position.
    SELECT NUMBER deliberately chooses a specific result.
  </p>
</main>

<script>
const teams = [
  ["Pakistan", "🇵🇰"],
  ["India", "🇮🇳"],
  ["Australia", "🇦🇺"],
  ["England", "🏴"],
  ["New Zealand", "🇳🇿"],
  ["South Africa", "🇿🇦"],
  ["Sri Lanka", "🇱🇰"],
  ["Bangladesh", "🇧🇩"],
  ["Afghanistan", "🇦🇫"],
  ["West Indies", "🌴"]
];

const canvas = document.getElementById("wheel");
const ctx = canvas.getContext("2d");
const N = 10;
const TAU = Math.PI * 2;
const step = TAU / N;

let angle = 0;
let velocity = 0;
let spinning = false;
let stopping = false;
let frame = 0;
let history = [];

const colors = [
  "#0b705b", "#b99035", "#245a81",
  "#ae5439", "#6b489b", "#268d8c",
  "#9c7032", "#416b38", "#994a72",
  "#356d91"
];

const start = document.getElementById("start");
const stop = document.getElementById("stop");
const result = document.getElementById("result");
const number = document.getElementById("number");

for (let i = 1; i <= N; i++) {
  let option = document.createElement("option");
  option.value = i;
  option.textContent = "Number " + i;
  number.appendChild(option);
}

function draw() {
  ctx.clearRect(0, 0, 600, 600);
  ctx.save();
  ctx.translate(300, 300);
  ctx.rotate(angle);

  for (let i = 0; i < N; i++) {
    let a = -Math.PI / 2 + i * step;

    ctx.beginPath();
    ctx.moveTo(0, 0);
    ctx.arc(0, 0, 288, a, a + step);
    ctx.closePath();
    ctx.fillStyle = colors[i];
    ctx.fill();
    ctx.strokeStyle = "#f6d68a";
    ctx.lineWidth = 3;
    ctx.stroke();

    ctx.save();
    ctx.rotate(a + step / 2);
    ctx.textAlign = "center";
    ctx.fillStyle = "white";
    ctx.font = "bold 38px Arial";
    ctx.fillText(String(i + 1), 190, 12);
    ctx.restore();
  }

  ctx.restore();

  ctx.beginPath();
  ctx.arc(300, 300, 48, 0, TAU);
  ctx.fillStyle = "#e8bd59";
  ctx.fill();

  ctx.fillStyle = "#152b35";
  ctx.font = "bold 23px Arial";
  ctx.textAlign = "center";
  ctx.fillText("ICCL", 300, 308);
}

function selected() {
  let normalized = ((angle % TAU) + TAU) % TAU;
  return (
    Math.floor(
      (((-normalized) % TAU + TAU) % TAU) / step
    ) + N
  ) % N;
}

function finish(index) {
  spinning = false;
  stopping = false;
  cancelAnimationFrame(frame);
  start.disabled = false;
  stop.disabled = true;

  let team = teams[index];
  result.textContent =
    "🎉 Number " + (index + 1) +
    " — " + team[1] + " " + team[0];

  history.unshift(result.textContent);
  document.getElementById("history").innerHTML =
    history.map((x, i) =>
      "<div>" + (i + 1) + ". " + x + "</div>"
    ).join("");

  draw();
}

function tick() {
  if (!spinning) return;
  angle += velocity;

  if (stopping) {
    velocity *= 0.985;
    if (velocity < 0.002) {
      finish(selected());
      return;
    }
  }

  draw();
  frame = requestAnimationFrame(tick);
}

start.onclick = () => {
  if (spinning) return;
  spinning = true;
  stopping = false;
  velocity = 0.17;
  start.disabled = true;
  stop.disabled = false;
  result.textContent = "🎡 Wheel is spinning...";
  tick();
};

stop.onclick = () => {
  if (spinning && !stopping) {
    stopping = true;
    stop.disabled = true;
    result.textContent = "⏳ Wheel slowing down...";
  }
};

document.getElementById("choose").onclick = () => {
  let n = Number(number.value);
  if (!n) {
    alert("Please select a number first.");
    return;
  }

  spinning = false;
  cancelAnimationFrame(frame);

  let target = -(n - 1) * step - step / 2;
  let current = angle;
  let delta = ((target - current) % TAU + TAU) % TAU;
  let from = current;
  let to = current + TAU * 4 + delta;
  let t0 = performance.now();

  start.disabled = true;
  stop.disabled = true;

  function animate(now) {
    let p = Math.min(1, (now - t0) / 2600);
    let ease = 1 - Math.pow(1 - p, 3);
    angle = from + (to - from) * ease;
    draw();

    if (p < 1) {
      frame = requestAnimationFrame(animate);
    } else {
      finish(n - 1);
    }
  }

  frame = requestAnimationFrame(animate);
};

document.getElementById("reset").onclick = () => {
  cancelAnimationFrame(frame);
  spinning = false;
  stopping = false;
  angle = 0;
  velocity = 0;
  history = [];
  number.value = "";

  start.disabled = false;
  stop.disabled = true;
  result.textContent = "Ready to spin!";
  document.getElementById("history").textContent =
    "No results yet.";
  draw();
};

draw();
</script>
</body>
</html>
