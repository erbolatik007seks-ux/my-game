# my-game
HTML game».
<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
<title>Jungle Run — мульт, мобильный, звук</title>
<style>
  :root{
    --ui-bg: rgba(0,0,0,0.55);
    --accent: #ffd94a;
    --muted: rgba(255,255,255,0.9);
    --btn-bg: rgba(255,255,255,0.08);
  }
  html,body{height:100%;margin:0;background:#87e08a;font-family:Inter,Segoe UI,Roboto,Arial,sans-serif;-webkit-tap-highlight-color:transparent;}
  #gameContainer{position:relative;height:100vh;width:100vw;overflow:hidden;background:linear-gradient(#9ef0a0,#27a14e);display:flex;flex-direction:column;}
  canvas{display:block;width:100%;height:calc(100vh - 96px);background:transparent;touch-action:none;}
  #topUI{position:absolute;left:12px;top:12px;color:var(--muted);z-index:40;text-shadow:0 2px 6px rgba(0,0,0,0.35);}
  #topUI .row{margin-bottom:6px;font-weight:600}
  #shop{position:absolute;right:12px;top:12px;width:320px;max-width:86vw;background:var(--ui-bg);color:var(--muted);padding:12px;border-radius:12px;z-index:60;display:none;}
  .shop-item{display:flex;justify-content:space-between;align-items:center;padding:8px 4px;border-top:1px solid rgba(255,255,255,0.04);}
  .btn{background:var(--btn-bg);border:1px solid rgba(255,255,255,0.06);padding:8px 10px;border-radius:10px;color:var(--muted);font-weight:700;cursor:pointer}
  .btn:active{transform:translateY(1px)}
  #overlayMsg{position:absolute;left:50%;top:44%;transform:translate(-50%,-50%);z-index:50;color:#fff;font-size:20px;padding:8px 14px;border-radius:10px;text-shadow:0 3px 10px rgba(0,0,0,0.6);display:none}
  #controlsBar{height:96px;background:linear-gradient(180deg, rgba(0,0,0,0.06), rgba(0,0,0,0.12));display:flex;align-items:center;justify-content:space-between;padding:10px 16px;box-sizing:border-box;z-index:30}
  .virt-controls{display:flex;gap:12px;align-items:center}
  .vbtn{width:72px;height:64px;border-radius:12px;background:rgba(255,255,255,0.06);display:flex;align-items:center;justify-content:center;font-size:28px;color:var(--muted);user-select:none;touch-action:manipulation}
  .vbtn.hold{background:rgba(255,255,255,0.12);box-shadow:0 6px 18px rgba(0,0,0,0.18) inset}
  .small-ui{display:flex;gap:8px;align-items:center;color:var(--muted);font-weight:600}
  footer{position:absolute;left:50%;bottom:4px;transform:translateX(-50%);font-size:12px;color:rgba(255,255,255,0.8);z-index:10}
  @media (min-width:900px){
    canvas{height:calc(100vh - 88px)}
    .vbtn{width:64px;height:56px}
  }
</style>
</head>
<body>
<div id="gameContainer">
  <canvas id="gameCanvas" width="1200" height="700"></canvas>

  <div id="topUI">
    <div class="row">🍌 Монет: <span id="coins">0</span></div>
    <div class="row">Уровень: <span id="level">1</span> &nbsp; Жизни: <span id="lives">3</span></div>
  </div>

  <div id="shop">
    <h3 style="margin:0 0 8px 0;color:var(--accent)">МАГАЗИН</h3>
    <div class="shop-item">
      <div><strong>Скорость +10%</strong><div style="font-size:12px;opacity:0.8">Увелич. бег</div></div>
      <div><button class="btn" id="buySpeed">50</button></div>
    </div>
    <div class="shop-item">
      <div><strong>Прыжок +10%</strong><div style="font-size:12px;opacity:0.8">Выше прыжок</div></div>
      <div><button class="btn" id="buyJump">50</button></div>
    </div>
    <h4 style="margin:10px 0 6px 0;color:var(--muted)">Персонажи</h4>
    <div class="shop-item">
      <div><div style="display:flex;align-items:center"><div style="width:44px;height:44px;border-radius:8px;background:gold;margin-right:8px"></div>Джунгл</div></div>
      <div><button class="btn buyChar" data-id="default" data-cost="0">Выбран</button></div>
    </div>
    <div class="shop-item">
      <div><div style="display:flex;align-items:center"><div style="width:44px;height:44px;border-radius:8px;background:#6cf;margin-right:8px"></div>Спринтер</div></div>
      <div><button class="btn buyChar" data-id="sprinter" data-cost="150">150</button></div>
    </div>
    <div style="font-size:12px;margin-top:8px;color:rgba(255,255,255,0.85)">Нажми M чтобы открыть/закрыть магазин</div>
  </div>

  <div id="overlayMsg"></div>

  <div id="controlsBar">
    <div class="virt-controls" id="leftControls" aria-hidden="true">
      <div class="vbtn" id="btnLeft">◀</div>
      <div class="vbtn" id="btnRight">▶</div>
      <div class="vbtn" id="btnJump">⬆</div>
    </div>
    <div class="small-ui">
      <button class="btn" id="btnShop">Магазин</button>
      <button class="btn" id="btnPause">P</button>
    </div>
  </div>

  <footer>Мультяшный режим • Встроенные звуки • Опт. под мобильный</footer>
</div>

<script>
/* ========== ИНИЦИАЛИЗАЦИЯ И СОХРАНЕНИЕ ========== */
const STORAGE_KEY = 'jungle-run-v2';
let save = { coins: 0, upgrades: {speed:0, jump:0}, unlockedChars: {default:true}, currentChar:'default', level:1, lives:3 };
try { const raw = localStorage.getItem(STORAGE_KEY); if(raw) save = Object.assign(save, JSON.parse(raw)); } catch(e){ console.warn(e) }
function saveGame(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(save)); updateUI(); }

/* ========== CANVAS И СКЕЙЛ ========== */
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
function fitCanvas(){
  // адаптив: устанавливаем внутренний размер для чёткости на устройствах с DPR
  const rect = canvas.getBoundingClientRect();
  const dpr = Math.min(window.devicePixelRatio || 1, 2);
  canvas.width = Math.round(rect.width * dpr);
  canvas.height = Math.round(rect.height * dpr);
  ctx.setTransform(dpr,0,0,dpr,0,0);
}
window.addEventListener('resize', fitCanvas);
fitCanvas();

/* ========== АУДИО (WebAudio простые эффекты) ========== */
let audioCtx = null;
function ensureAudio(){
  if(audioCtx) return;
  try{
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }catch(e){ audioCtx = null; console.warn('Audio not available', e); }
}
function playTone(frequency, type='sine', duration=0.12, gain=0.12){
  if(!audioCtx) return;
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = type; o.frequency.value = frequency;
  g.gain.value = gain;
  o.connect(g); g.connect(audioCtx.destination);
  o.start();
  g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
  o.stop(audioCtx.currentTime + duration + 0.02);
}
function playCollect(){ playTone(820,'sine',0.12,0.14); playTone(1200,'sine',0.06,0.08); }
function playJump(){ playTone(300,'triangle',0.15,0.10); }
function playHit(){ playTone(120,'sawtooth',0.22,0.18); }

/* audio must be resumed on user gesture in many browsers */
function unlockAudioOnGesture(){
  if(!audioCtx) return;
  if(audioCtx.state === 'suspended') {
    const resume = ()=>{ audioCtx.resume(); window.removeEventListener('pointerdown', resume); window.removeEventListener('touchstart', resume); }
    window.addEventListener('pointerdown', resume, {once:true});
    window.addEventListener('touchstart', resume, {once:true});
  }
}
ensureAudio();
unlockAudioOnGesture();

/* ========== ИГРОВАЯ ЛОГИКА (упрощённая, мультяшная) ========== */
const W = canvas.width, H = canvas.height; // use dynamic in draw
let lastTime = performance.now();
let keys = {}, touchState = {left:false,right:false,jump:false};
window.addEventListener('keydown', e=>{ keys[e.key.toLowerCase()]=true; if(e.key.toLowerCase()==='m') toggleShop(); if(e.key.toLowerCase()==='p') togglePause(); });
window.addEventListener('keyup', e=>{ keys[e.key.toLowerCase()]=false; });

const player = {
  x:120, y:0, w:48, h:64, vx:0, vy:0, onGround:false,
  baseSpeed:5, baseJump:14,
  colorMap:{default:'#ffd94a', sprinter:'#6cf'}
};

let groundY, bananas=[], enemies=[], spawnTimer=0, level = save.level || 1, lives = save.lives || 3;

/* уровни и параметры */
function enemySpawnRate(lvl){ return Math.max(70 - lvl*3, 20); }
function enemySpeed(lvl){ return 2 + lvl*0.6; }
function bananaChance(lvl){ return 0.01 + lvl*0.002; }

/* ========== Магазин / UI ========== */
const shop = document.getElementById('shop');
document.getElementById('btnShop').addEventListener('click', toggleShop);
document.getElementById('btnPause').addEventListener('click', togglePause);
document.getElementById('buySpeed').addEventListener('click', ()=>buyUpgrade('speed',50));
document.getElementById('buyJump').addEventListener('click', ()=>buyUpgrade('jump',50));
document.querySelectorAll('.buyChar').forEach(btn=>{
  btn.addEventListener('click', ()=>{
    const id = btn.dataset.id; const cost = +btn.dataset.cost;
    if(save.unlockedChars[id]) { save.currentChar = id; saveGame(); showOverlay('Персонаж выбран'); return; }
    if(save.coins >= cost){ save.coins -= cost; save.unlockedChars[id]=true; save.currentChar=id; saveGame(); showOverlay('Открыт!'); } else showOverlay('Не хватает монет');
  });
});
function toggleShop(){ shop.style.display = shop.style.display === 'block' ? 'none' : 'block'; updateShop(); }
function updateShop(){
  document.getElementById('coins').innerText = save.coins;
  document.querySelectorAll('.buyChar').forEach(btn=>{
    const id = btn.dataset.id, cost = +btn.dataset.cost;
    if(save.unlockedChars[id]) { btn.innerText = (save.currentChar === id ? 'Выбран' : 'Выбрать'); btn.disabled=false; }
    else { btn.innerText = cost; btn.disabled = save.coins < cost; }
  });
  document.getElementById('buySpeed').disabled = save.coins < 50;
  document.getElementById('buyJump').disabled = save.coins < 50;
  document.getElementById('level').innerText = level;
  document.getElementById('lives').innerText = lives;
}
function buyUpgrade(name,cost){
  if(save.coins < cost){ showOverlay('Недостаточно'); return; }
  save.coins -= cost; save.upgrades[name] = (save.upgrades[name]||0)+1; saveGame(); showOverlay('Куплено');
}

/* ========== ВИРТУАЛЬНЫЕ КНОПКИ (тач) ========== */
const btnLeft = document.getElementById('btnLeft'), btnRight = document.getElementById('btnRight'), btnJump = document.getElementById('btnJump');
function bindHold(el, onStart, onEnd){
  el.addEventListener('pointerdown', e=>{ e.preventDefault(); onStart(); el.classList.add('hold'); ensureAudio(); if(audioCtx) audioCtx.resume(); });
  window.addEventListener('pointerup', e=>{ onEnd(); el.classList.remove('hold'); });
  el.addEventListener('pointercancel', ()=>{ onEnd(); el.classList.remove('hold'); });
  // for touch move outside
  el.addEventListener('touchstart', e=>{ e.preventDefault(); onStart(); el.classList.add('hold'); }, {passive:false});
  window.addEventListener('touchend', e=>{ onEnd(); el.classList.remove('hold'); });
}
bindHold(btnLeft, ()=>{ touchState.left=true; }, ()=>{ touchState.left=false; });
bindHold(btnRight, ()=>{ touchState.right=true; }, ()=>{ touchState.right=false; });
bindHold(btnJump, ()=>{ touchState.jump=true; playJump(); setTimeout(()=> touchState.jump=false, 170); }, ()=>{ touchState.jump=false; });

/* ========== SPAWN & OBJECTS ========== */
function spawnBanana(){
  const x = Math.random() * (canvas.width/ (window.devicePixelRatio||1) - 300) + 250;
  const y = groundY - 30 - Math.random()*140;
  bananas.push({x,y,r:12 + Math.random()*6, t:Math.random()*Math.PI*2});
}
function spawnEnemy(){
  const y = groundY - 44;
  const x = (canvas.width/(window.devicePixelRatio||1)) + 80;
  const speed = enemySpeed(level) + Math.random()*0.6;
  enemies.push({x,y,w:56,h:40,vx:-speed,alive:true, bob:Math.random()*100});
}

/* ========== ФИЗИКА ========== */
function applyInput(dt){
  const upg = save.upgrades || {};
  const speedBoost = 1 + (upg.speed||0)*0.1;
  const jumpBoost = 1 + (upg.jump||0)*0.08;
  const baseSpeed = player.baseSpeed * speedBoost;

  // keyboard or touch
  if(keys['arrowleft'] || keys['a'] || touchState.left) player.vx = -baseSpeed;
  else if(keys['arrowright'] || keys['d'] || touchState.right) player.vx = baseSpeed;
  else player.vx = 0;

  if((keys[' '] || keys['arrowup'] || keys['w'] || touchState.jump) && player.onGround){
    player.vy = -player.baseJump * jumpBoost;
    player.onGround = false;
    playJump();
  }
}

function physics(dt){
  player.vy += 0.8; // gravity
  player.x += player.vx;
  player.y += player.vy;

  // bounds
  if(player.x < 12) player.x = 12;
  const maxX = (canvas.width/(window.devicePixelRatio||1)) - player.w - 12;
  if(player.x > maxX) player.x = maxX;

  if(player.y + player.h >= groundY){
    player.y = groundY - player.h;
    player.vy = 0;
    player.onGround = true;
  }
}

/* ========== КОЛЛИЗИИ ========== */
function checkCollisions(){
  for(let b of bananas){
    if(b.collected) continue;
    const dx = (player.x + player.w/2) - b.x;
    const dy = (player.y + player.h/2) - b.y;
    const dist = Math.sqrt(dx*dx + dy*dy);
    if(dist < b.r + Math.max(player.w, player.h)/3){
      b.collected = true;
      save.coins += 10;
      playCollect();
      saveGame();
    }
  }

  for(let e of enemies){
    if(!e.alive) continue;
    if(player.x < e.x + e.w && player.x + player.w > e.x && player.y < e.y + e.h && player.y + player.h > e.y){
      e.alive = false;
      lives -= 1;
      save.lives = lives;
      saveGame();
      playHit();
      showOverlay('Удар! -1 жизнь');
      if(lives <= 0) gameOver();
    }
  }
}

/* ========== УРОВНИ ========== */
function checkLevelProgress(){
  const needed = 200 + (level-1)*150;
  if(save.coins >= needed){
    level += 1; save.level = level; save.coins = Math.max(0, save.coins - needed); saveGame();
    showOverlay('Новый уровень: ' + level);
  }
}

/* ========== РЕНДЕР (мультяшный) ========== */
function drawBackground(t){
  const w = canvas.width/(window.devicePixelRatio||1), h = canvas.height/(window.devicePixelRatio||1);
  // мягкий градиент
  const g = ctx.createLinearGradient(0,0,0,h);
  g.addColorStop(0,'#bfffb1'); g.addColorStop(0.6,'#5fcf63'); g.addColorStop(1,'#2a8a3a');
  ctx.fillStyle = g; ctx.fillRect(0,0,w,h);

  // параллакс силуэты (мультяшные)
  ctx.save();
  ctx.globalAlpha = 0.45;
  ctx.fillStyle = '#1a5e2b';
  for(let i=0;i<6;i++){
    const sx = (i*260 - (t/30)%520);
    ctx.beginPath();
    ctx.moveTo(sx, h*0.62);
    ctx.quadraticCurveTo(sx+80, h*0.4, sx+220, h*0.62);
    ctx.lineTo(sx+220, h); ctx.lineTo(sx, h); ctx.fill();
  }
  ctx.restore();

  // деревья ближе
  ctx.fillStyle = '#0f6b2a';
  for(let i=0;i<12;i++){
    const bx = (i*140 + (t/20)) % (w+160) - 80;
    ctx.fillRect(bx, h*0.57, 28, h*0.43);
    ctx.beginPath();
    ctx.ellipse(bx+14, h*0.57 - 26, 64, 32, 0, 0, Math.PI*2);
    ctx.fill();
  }

  // земля
  ctx.fillStyle = '#2f6b2a';
  ctx.fillRect(0, h - 72, w, 72);
  groundY = h - 72;
}

function drawPlayer(t){
  const color = player.colorMap[save.currentChar] || player.colorMap.default;
  // shadow
  ctx.fillStyle = 'rgba(0,0,0,0.18)';
  ctx.beginPath(); ctx.ellipse(player.x + player.w/2, player.y + player.h + 8, player.w*0.6, 10, 0, 0, Math.PI*2); ctx.fill();
  // body (multistyle)
  ctx.fillStyle = color; ctx.fillRect(player.x, player.y, player.w, player.h);
  // eyes and smile
  ctx.fillStyle = '#fff'; ctx.fillRect(player.x + player.w - 22, player.y + 14, 6,6);
  ctx.fillStyle = '#000'; ctx.fillRect(player.x + player.w - 20, player.y + 16, 3,3);
  ctx.strokeStyle = '#000'; ctx.lineWidth = 2;
  ctx.beginPath(); ctx.arc(player.x + player.w/2 - 2, player.y + player.h - 18, 10, 0, Math.PI); ctx.stroke();
}

function drawBananas(t){
  for(let b of bananas){
    if(b.collected) continue;
    ctx.save();
    ctx.translate(b.x, b.y);
    // simple bobbing
    const bob = Math.sin(t/220 + b.t) * 6;
    ctx.translate(0, bob);
    ctx.beginPath();
    ctx.ellipse(0,0, b.r*0.85, b.r*1.25, Math.PI/6, 0, Math.PI*2);
    ctx.fillStyle = '#ffd94a'; ctx.fill();
    ctx.fillStyle = 'rgba(255,255,255,0.6)'; ctx.beginPath(); ctx.ellipse(-2,-4,3,5,0,0,Math.PI*2); ctx.fill();
    ctx.restore();
  }
}

function drawEnemies(t){
  for(let e of enemies){
    if(!e.alive) continue;
    // bob animation
    const bob = Math.sin((t+e.bob)/180)*6;
    ctx.fillStyle = '#6a3f1d';
    ctx.fillRect(e.x, e.y + bob, e.w, e.h);
    // ears
    ctx.fillStyle = '#4a2b10';
    ctx.beginPath(); ctx.moveTo(e.x+8, e.y + bob); ctx.lineTo(e.x+18, e.y - 14 + bob); ctx.lineTo(e.x+28, e.y + bob); ctx.fill();
    // eye
    ctx.fillStyle = '#fff'; ctx.fillRect(e.x + e.w - 20, e.y + 10 + bob, 6,6);
  }
}

/* ========== ЦИКЛ ИГРЫ ========== */
let gamePaused = false, overlayTimer = null;
function showOverlay(text, time=1200){
  const o = document.getElementById('overlayMsg');
  o.innerText = text; o.style.display='block';
  if(overlayTimer) clearTimeout(overlayTimer);
  overlayTimer = setTimeout(()=> o.style.display='none', time);
}

function togglePause(){ gamePaused = !gamePaused; showOverlay(gamePaused ? 'Пауза' : '', 800); }

function gameOver(){
  showOverlay('Игра окончена', 2000);
  setTimeout(()=>{
    if(confirm('Игра окончена. Начать заново?')){
      save.coins = 0; save.level = 1; save.lives = 3; save.upgrades = {speed:0,jump:0}; save.unlockedChars = {default:true}; save.currentChar='default';
      level = 1; lives = 3; bananas = []; enemies = []; saveGame(); showOverlay('Заново!'); 
    }
  }, 300);
}

function loop(now){
  if(gamePaused){ lastTime = now; requestAnimationFrame(loop); return; }
  const dt = Math.min(40, now - lastTime);
  lastTime = now;

  // input
  applyInput(dt);
  // physics
  physics(dt);
  // update enemies
  for(let e of enemies) e.x += e.vx;
  // spawn
  spawnTimer -= 1;
  if(spawnTimer <= 0){ spawnTimer = enemySpawnRate(level) + Math.random()*30; spawnEnemy(); }
  if(Math.random() < bananaChance(level)) spawnBanana();

  // collisions & cleanup
  checkCollisions();
  enemies = enemies.filter(e => e.x > -200 && e.alive);
  bananas = bananas.filter(b => !b.collected && b.y < (canvas.height/(window.devicePixelRatio||1)));

  // level up
  checkLevelProgress();

  // rendering
  drawBackground(now);
  drawBananas(now);
  drawEnemies(now);
  drawPlayer(now);

  // HUD
  document.getElementById('coins').innerText = save.coins;
  document.getElementById('level').innerText = level;
  document.getElementById('lives').innerText = lives;

  requestAnimationFrame(loop);
}

/* ========== ИНИЦИАЛИЗАЦИЯ ========== */
function init(){
  // позиционировать игрок
  const h = canvas.height/(window.devicePixelRatio||1);
  groundY = h - 72;
  player.x = 120; player.h = 64; player.w = 48; player.y = groundY - player.h;
  level = save.level || 1; lives = save.lives || 3;
  updateShop();
  saveGame();
  ensureAudio(); unlockAudioOnGesture();
  requestAnimationFrame(loop);
}
init();

/* автосохранение */
setInterval(saveGame, 5000);

/* ========== ОСОБЕННОСТИ: первая касание включает аудио ========== */
function unlockAudioOnGesture(){
  if(!audioCtx) return;
  if(audioCtx.state === 'suspended'){
    const resume = ()=>{ audioCtx.resume(); window.removeEventListener('pointerdown', resume); window.removeEventListener('touchstart', resume); }
    window.addEventListener('pointerdown', resume, {once:true});
    window.addEventListener('touchstart', resume, {once:true});
  }
}

/* улучшение: таймер спауна стартует */
spawnTimer = enemySpawnRate(level) + 20;

</script>
</body>
</html>
