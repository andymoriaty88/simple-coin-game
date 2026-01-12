# simple-coin-game
coin collector game
index.html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Coin Collector</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <main class="wrap">
    <header class="bar">
      <h1>Coin Collector</h1>
      <div class="hud">
        <span>Score: <b id="score">0</b></span>
        <span>Time: <b id="time">30</b>s</span>
      </div>
    </header>

    <canvas id="game" width="900" height="520"></canvas>

    <footer class="bar footer">
      <div class="help">Move: Arrow Keys or WASD · Collect coins before time runs out</div>
      <button id="startBtn">Start</button>
    </footer>
  </main>

  <script src="main.js"></script>
</body>
</html>
style.css
:root { color-scheme: dark; }
* { box-sizing: border-box; font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; }
body { margin: 0; background: #0b0d18; color: #eef0ff; }
.wrap { max-width: 980px; margin: 24px auto; padding: 0 16px; }
.bar { display: flex; justify-content: space-between; align-items: center; gap: 16px; }
h1 { margin: 0; font-size: 24px; letter-spacing: 0.4px; }
.hud { display: flex; gap: 16px; opacity: 0.9; }
#game {
  width: 100%;
  display: block;
  margin: 14px 0;
  border-radius: 16px;
  background: radial-gradient(900px 500px at 50% 30%, rgba(120,140,255,0.18), rgba(0,0,0,0));
  box-shadow: 0 20px 60px rgba(0,0,0,0.45);
}
.footer { flex-wrap: wrap; }
.help { opacity: 0.85; font-size: 14px; }
button {
  border: 0; border-radius: 12px; padding: 10px 14px;
  font-weight: 700; cursor: pointer;
  background: rgba(120,140,255,0.95); color: #090a12;
}
button:hover { filter: brightness(1.08); }

main.js
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const scoreEl = document.getElementById("score");
const timeEl = document.getElementById("time");
const startBtn = document.getElementById("startBtn");

const W = canvas.width;
const H = canvas.height;

const keys = new Set();
addEventListener("keydown", (e) => keys.add(e.key.toLowerCase()));
addEventListener("keyup", (e) => keys.delete(e.key.toLowerCase()));

const rand = (a, b) => a + Math.random() * (b - a);
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));

let running = false;
let score = 0;
let timeLeft = 30;
let lastTick = 0;

const player = { x: W / 2, y: H / 2, r: 14, speed: 260 };
let coin = { x: rand(40, W - 40), y: rand(40, H - 40), r: 10 };

function reset() {
  running = true;
  score = 0;
  timeLeft = 30;
  lastTick = performance.now();

  player.x = W / 2;
  player.y = H / 2;

  coin = { x: rand(40, W - 40), y: rand(40, H - 40), r: 10 };

  scoreEl.textContent = score;
  timeEl.textContent = timeLeft;
  startBtn.textContent = "Restart";
}

startBtn.addEventListener("click", () => reset());

function circlesHit(a, b) {
  const dx = a.x - b.x;
  const dy = a.y - b.y;
  const rr = a.r + b.r;
  return dx * dx + dy * dy <= rr * rr;
}

function update(dt) {
  if (!running) return;

  // timer counts down in whole seconds
  const now = performance.now();
  if (now - lastTick >= 1000) {
    timeLeft -= 1;
    lastTick += 1000;
    timeEl.textContent = timeLeft;
    if (timeLeft <= 0) {
      running = false;
      return;
    }
  }

  const up = keys.has("arrowup") || keys.has("w");
  const down = keys.has("arrowdown") || keys.has("s");
  const left = keys.has("arrowleft") || keys.has("a");
  const right = keys.has("arrowright") || keys.has("d");

  let vx = (right ? 1 : 0) - (left ? 1 : 0);
  let vy = (down ? 1 : 0) - (up ? 1 : 0);

  const m = Math.hypot(vx, vy) || 1;
  vx /= m; vy /= m;

  player.x = clamp(player.x + vx * player.speed * dt, player.r, W - player.r);
  player.y = clamp(player.y + vy * player.speed * dt, player.r, H - player.r);

  if (circlesHit(player, coin)) {
    score += 1;
    scoreEl.textContent = score;
    coin.x = rand(40, W - 40);
    coin.y = rand(40, H - 40);
  }
}

function draw() {
  ctx.clearRect(0, 0, W, H);

  // subtle grid
  ctx.globalAlpha = 0.14;
  ctx.beginPath();
  for (let x = 20; x < W; x += 35) { ctx.moveTo(x, 0); ctx.lineTo(x, H); }
  for (let y = 20; y < H; y += 35) { ctx.moveTo(0, y); ctx.lineTo(W, y); }
  ctx.strokeStyle = "#fff";
  ctx.stroke();
  ctx.globalAlpha = 1;

  // coin
  ctx.beginPath();
  ctx.arc(coin.x, coin.y, coin.r, 0, Math.PI * 2);
  ctx.fillStyle = "rgba(255, 215, 90, 0.95)";
  ctx.shadowBlur = 14;
  ctx.shadowColor = "rgba(255, 215, 90, 0.7)";
  ctx.fill();
  ctx.shadowBlur = 0;

  // player
  ctx.beginPath();
  ctx.arc(player.x, player.y, player.r, 0, Math.PI * 2);
  ctx.fillStyle = "rgba(120, 210, 255, 0.95)";
  ctx.shadowBlur = 14;
  ctx.shadowColor = "rgba(120, 210, 255, 0.7)";
  ctx.fill();
  ctx.shadowBlur = 0;

  // overlay text
  if (!running) {
    ctx.fillStyle = "rgba(0,0,0,0.45)";
    ctx.fillRect(0, 0, W, H);

    ctx.fillStyle = "#eef0ff";
    ctx.textAlign = "center";
    ctx.font = "700 34px system-ui";
    ctx.fillText(timeLeft <= 0 ? "Time!" : "Press Start", W / 2, H / 2 - 10);

    ctx.font = "500 16px system-ui";
    ctx.fillText("Collect as many coins as you can in 30 seconds.", W / 2, H / 2 + 20);
  }
}

let last = performance.now();
function loop(now) {
  const dt = Math.min(0.033, (now - last) / 1000);
  last = now;

  update(dt);
  draw();

  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
