
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Coin Collector</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0;
    min-height: 100vh;
    display: grid;
    place-items: center;
    background: #101827;
    color: white;
    font-family: system-ui, sans-serif;
    padding: 16px;
  }
  .game { width: min(100%, 600px); text-align: center; }
  h1 { color: #facc15; margin-bottom: 8px; }
  .stats {
    display: flex;
    justify-content: space-around;
    background: #202d42;
    border-radius: 12px;
    padding: 12px;
    margin: 12px 0;
  }
  canvas {
    display: block;
    width: 100%;
    border: 3px solid #334155;
    border-radius: 12px;
    background: #86c95b;
    touch-action: none;
  }
  button {
    border: 0;
    border-radius: 10px;
    padding: 12px 20px;
    margin: 5px;
    font-size: 16px;
    font-weight: bold;
    background: #facc15;
    color: #172033;
    cursor: pointer;
  }
  .controls {
    display: grid;
    grid-template-columns: repeat(3, 65px);
    justify-content: center;
    gap: 6px;
    margin-top: 12px;
  }
  .controls button { margin: 0; padding: 12px; }
  .empty { visibility: hidden; }
  p { color: #cbd5e1; }
</style>
</head>
<body>
<div class="game">
  <h1>🪙 Coin Collector</h1>
  <p>Collect coins. Don't touch the monsters!</p>

  <div class="stats">
    <span>🪙 Score: <b id="score">0</b></span>
    <span>❤️ Lives: <b id="lives">3</b></span>
    <span>🏆 Best: <b id="best">0</b></span>
  </div>

  <canvas id="game" width="600" height="400"></canvas>
  <div>
    <button id="start">▶ Start Game</button>
    <button id="pause">⏸ Pause</button>
  </div>

  <div class="controls">
    <span class="empty"></span>
    <button data-dir="up">⬆️</button>
    <span class="empty"></span>
    <button data-dir="left">⬅️</button>
    <button data-dir="down">⬇️</button>
    <button data-dir="right">➡️</button>
  </div>
  <p>Keyboard: Arrow keys or WASD</p>
  <p id="message">Press Start Game to play!</p>
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const $ = id => document.getElementById(id);

const W = canvas.width, H = canvas.height;
let player, coins, enemies, score, lives, running;
let paused = false, gameOver = false;
let keys = {};
let best = 0;

try {
  best = Number(localStorage.getItem("coinCollectorBest")) || 0;
} catch (_) {}
$("best").textContent = best;

function randomPos(radius = 12) {
  return {
    x: radius + Math.random() * (W - radius * 2),
    y: radius + Math.random() * (H - radius * 2)
  };
}

function spawnCoin() {
  let p;
  do { p = randomPos(12); }
  while (player && Math.hypot(p.x-player.x, p.y-player.y) < 60);
  return { ...p, r: 10 };
}

function startGame() {
  player = { x: W/2, y: H/2, r: 13, speed: 3.5 };
  coins = Array.from({length: 5}, spawnCoin);
  enemies = [
    {x: 80, y: 80, r: 15, vx: 2, vy: 1.7},
    {x: 510, y: 300, r: 15, vx: -1.8, vy: 2}
  ];
  score = 0;
  lives = 3;
  running = true;
  paused = false;
  gameOver = false;
  keys = {};
  $("score").textContent = score;
  $("lives").textContent = lives;
  $("message").textContent = "Go collect those coins!";
  $("start").textContent = "↻ Restart";
  draw();
}

function draw() {
  ctx.clearRect(0, 0, W, H);

  // Grass pattern
  ctx.fillStyle = "#86c95b";
  ctx.fillRect(0, 0, W, H);
  ctx.fillStyle = "rgba(255,255,255,0.10)";
  for (let x=15; x<W; x+=40)
    for (let y=15; y<H; y+=40)
      ctx.fillRect(x, y, 3, 3);

  // Coins
  for (const c of coins) {
    ctx.beginPath();
    ctx.arc(c.x, c.y, c.r, 0, Math.PI*2);
    ctx.fillStyle = "#facc15";
    ctx.fill();
    ctx.lineWidth = 3;
    ctx.strokeStyle = "#ca8a04";
    ctx.stroke();
    ctx.fillStyle = "#fff2a6";
    ctx.font = "bold 13px sans-serif";
    ctx.textAlign = "center";
    ctx.fillText("$", c.x, c.y+5);
  }

  // Enemies
  for (const e of enemies) {
    ctx.fillStyle = "#ef4444";
    ctx.beginPath();
    ctx.arc(e.x, e.y, e.r, 0, Math.PI*2);
    ctx.fill();
    ctx.fillStyle = "white";
    ctx.beginPath();
    ctx.arc(e.x-5, e.y-3, 4, 0, Math.PI*2);
    ctx.arc(e.x+5, e.y-3, 4, 0, Math.PI*2);
    ctx.fill();
    ctx.fillStyle = "#1f2937";
    ctx.beginPath();
    ctx.arc(e.x-5, e.y-3, 2, 0, Math.PI*2);
    ctx.arc(e.x+5, e.y-3, 2, 0, Math.PI*2);
    ctx.fill();
  }

  // Player
  if (player) {
    ctx.fillStyle = "#2563eb";
    ctx.beginPath();
    ctx.arc(player.x, player.y, player.r, 0, Math.PI*2);
    ctx.fill();
    ctx.strokeStyle = "#bfdbfe";
    ctx.lineWidth = 3;
    ctx.stroke();
    ctx.fillStyle = "white";
    ctx.font = "bold 18px sans-serif";
    ctx.textAlign = "center";
    ctx.fillText("☺", player.x, player.y+6);
  }

  if (paused || gameOver || !running) {
    ctx.fillStyle = "rgba(15,23,42,0.72)";
    ctx.fillRect(0, 0, W, H);
    ctx.textAlign = "center";
    ctx.fillStyle = "#facc15";
    ctx.font = "bold 32px sans-serif";
    ctx.fillText(gameOver ? "GAME OVER" :
      paused ? "PAUSED" : "READY?", W/2, H/2-8);
    ctx.fillStyle = "white";
    ctx.font = "18px sans-serif";
    ctx.fillText(gameOver ? `Final score: ${score}` :
      paused ? "Press Pause to continue" : "Press Start Game",
      W/2, H/2+28);
  }
}

function update() {
  if (!running || paused || gameOver) return;

  let dx = 0, dy = 0;
  if (keys.ArrowLeft || keys.a || keys.left) dx--;
  if (keys.ArrowRight || keys.d || keys.right) dx++;
  if (keys.ArrowUp || keys.w || keys.up) dy--;
  if (keys.ArrowDown || keys.s || keys.down) dy++;

  if (dx && dy) {
    dx *= Math.SQRT1_2;
    dy *= Math.SQRT1_2;
  }

  player.x = Math.max(player.r, Math.min(W-player.r,
    player.x + dx*player.speed));
  player.y = Math.max(player.r, Math.min(H-player.r,
    player.y + dy*player.speed));

  for (let i=0; i<coins.length; i++) {
    const c = coins[i];
    if (Math.hypot(player.x-c.x, player.y-c.y)
        < player.r+c.r) {
      score++;
      $("score").textContent = score;
      coins[i] = spawnCoin();

      if (score > best) {
        best = score;
        $("best").textContent = best;
        try {
          localStorage.setItem("coinCollectorBest", best);
        } catch (_) {}
      }

      // More coins and faster enemies as you improve
      if (score % 5 === 0) {
        enemies.push({
          ...randomPos(15), r: 15,
          vx: Math.random() < .5 ? 2 : -2,
          vy: Math.random() < .5 ? 2 : -2
        });
        $("message").textContent = "Level up! A new monster appeared!";
      }
    }
  }

  for (const e of enemies) {
    e.x += e.vx;
    e.y += e.vy;
    if (e.x < e.r || e.x > W-e.r) e.vx *= -1;
    if (e.y < e.r || e.y > H-e.r) e.vy *= -1;

    if (Math.hypot(player.x-e.x, player.y-e.y)
        < player.r+e.r) {
      lives--;
      $("lives").textContent = lives;

      // Move the player away after a hit
      player.x = W/2;
      player.y = H/2;

      if (lives <= 0) {
        gameOver = true;
        running = false;
        $("message").textContent =
          `Game over! You collected ${score} coins.`;
      } else {
        $("message").textContent = "Ouch! Watch out for monsters!";
      }
      break;
    }
  }
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

$("start").addEventListener("click", startGame);
$("pause").addEventListener("click", () => {
  if (!running || gameOver) return;
  paused = !paused;
  $("pause").textContent = paused ? "▶ Resume" : "⏸ Pause";
  draw();
});

document.addEventListener("keydown", e => {
  const k = e.key.length === 1 ? e.key.toLowerCase() : e.key;
  if (["ArrowUp","ArrowDown","ArrowLeft","ArrowRight"," "]
      .includes(e.key)) e.preventDefault();
  keys[k] = true;
});
document.addEventListener("keyup", e => {
  const k = e.key.length === 1 ? e.key.toLowerCase() : e.key;
  keys[k] = false;
});

document.querySelectorAll("[data-dir]").forEach(btn => {
  const dir = btn.dataset.dir;
  const down = e => {
    e.preventDefault();
    keys[dir] = true;
  };
  const up = e => {
    e.preventDefault();
    keys[dir] = false;
  };
  btn.addEventListener("pointerdown", down);
  btn.addEventListener("pointerup", up);
  btn.addEventListener("pointercancel", up);
  btn.addEventListener("pointerleave", up);
});

draw();
loop();
</script>
</body>
</html>
