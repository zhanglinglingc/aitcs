# 贪吃蛇 - 宇宙星空版 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 一个单文件 HTML 贪吃蛇游戏，宇宙星空主题，四种游戏模式，完整粒子特效和程序化音效。

**Architecture:** 单 HTML 文件，内联 CSS + JS。Canvas 2D 渲染游戏，requestAnimationFrame 驱动循环，Web Audio API 生成音效。模块化 JS 结构：Audio、Renderer、Game、UI 分层。

**Tech Stack:** HTML5 Canvas, CSS3 Animations, Vanilla JS, Web Audio API, localStorage

---

## 文件结构

```
/Users/ling/Desktop/开发/11/snake.html   ← 唯一文件，包含所有代码
```

文件内部结构（用注释分隔）：
```
<!-- ===== CONSTANTS & CONFIG ===== -->
<!-- ===== AUDIO SYSTEM ===== -->
<!-- ===== BACKGROUND RENDERER ===== -->
<!-- ===== PARTICLE SYSTEM ===== -->
<!-- ===== GAME CORE ===== -->
<!-- ===== MODES (Classic/Warp/Level/Powerup) ===== -->
<!-- ===== UI SCREENS ===== -->
<!-- ===== INPUT HANDLER ===== -->
<!-- ===== MAIN LOOP & INIT ===== -->
```

---

## 任务列表

### Task 1: HTML 骨架 + CSS 宇宙背景 + 字体

**Files:**
- Create: `/Users/ling/Desktop/开发/11/snake.html`

- [ ] **Step 1: 创建 HTML 文件骨架**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>贪吃蛇 - 宇宙星空</title>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap" rel="stylesheet">
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  background: #050510;
  overflow: hidden;
  font-family: 'Orbitron', sans-serif;
  color: white;
}

#bg-canvas {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  z-index: 0;
}

#game-canvas {
  position: fixed;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1;
  border: 2px solid rgba(0, 200, 255, 0.3);
  box-shadow: 0 0 40px rgba(0, 200, 255, 0.2);
}

#ui-layer {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  z-index: 2;
  pointer-events: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

/* HUD */
#hud {
  position: fixed;
  top: 20px; left: 0; right: 0;
  display: flex;
  justify-content: center;
  gap: 40px;
  z-index: 3;
  opacity: 0;
  transition: opacity 0.5s;
}
#hud.visible { opacity: 1; }
.hud-item {
  font-size: 14px;
  color: rgba(160, 224, 255, 0.8);
  text-shadow: 0 0 10px rgba(0, 200, 255, 0.5);
  letter-spacing: 2px;
}
.hud-item span { color: #00c8ff; font-weight: 700; }

/* Screen base */
.screen {
  display: none;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}
.screen.active { display: flex; }

/* Title screen */
#title-screen h1 {
  font-size: 48px;
  font-weight: 900;
  background: linear-gradient(135deg, #00c8ff, #cc44ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: none;
  filter: drop-shadow(0 0 20px rgba(0, 200, 255, 0.6));
  margin-bottom: 10px;
}
#title-screen .subtitle {
  font-size: 14px;
  color: rgba(160, 224, 255, 0.6);
  letter-spacing: 4px;
  margin-bottom: 50px;
}
#title-screen .press-enter {
  font-size: 16px;
  color: rgba(160, 224, 255, 0.7);
  animation: pulse 1.5s ease-in-out infinite;
}
@keyframes pulse {
  0%, 100% { opacity: 0.4; }
  50% { opacity: 1; }
}
```

- [ ] **Step 2: 运行验证**

打开浏览器确认：
- 深空背景 (#050510) 正确显示
- Orbitron 字体加载
- 标题 "贪吃蛇 - 宇宙星空" 居中显示，青紫渐变
- "按 Enter 开始" 脉冲动画
- 无 JS 报错

- [ ] **Step 3: 提交**

```bash
git add snake.html
git commit -m "feat: HTML skeleton + space theme CSS + Orbitron font"
```

---

### Task 2: 星空背景动画（背景 Canvas）

**Files:**
- Modify: `/Users/ling/Desktop/开发/11/snake.html`

- [ ] **Step 1: 在 CONSTANTS 区添加背景配置**

```javascript
// ===== CONSTANTS & CONFIG =====
const CONFIG = {
  BG_STAR_COUNT: 200,
  BG_NEBULA_COUNT: 5,
  GRID_SIZE: 20,
  SNAKE_SPEED: 120, // ms per tick
  TRAIL_PARTICLE_LIFE: 800,
  TRAIL_SPAWN_RATE: 3, // spawn every N frames
};
```

- [ ] **Step 2: 在 BACKGROUND RENDERER 区实现星空类**

在 `<script>` 标签内（`<!-- ===== BACKGROUND RENDERER ===== -->` 后）添加：

```javascript
// ===== BACKGROUND RENDERER =====
class StarField {
  constructor(ctx, width, height) {
    this.ctx = ctx;
    this.stars = [];
    this.nebulas = [];
    for (let i = 0; i < CONFIG.BG_STAR_COUNT; i++) {
      this.stars.push({
        x: Math.random() * width,
        y: Math.random() * height,
        size: Math.random() * 2 + 0.5,
        speed: Math.random() * 0.3 + 0.05,
        brightness: Math.random(),
        twinkleSpeed: Math.random() * 0.02 + 0.005,
        twinklePhase: Math.random() * Math.PI * 2,
        color: this._randomStarColor(),
      });
    }
    for (let i = 0; i < CONFIG.BG_NEBULA_COUNT; i++) {
      this.nebulas.push({
        x: Math.random() * width,
        y: Math.random() * height,
        radius: Math.random() * 300 + 150,
        speed: (Math.random() - 0.5) * 0.2,
        color: Math.random() > 0.5 ? 'rgba(100, 50, 180, 0.04)' : 'rgba(50, 100, 180, 0.04)',
        driftX: (Math.random() - 0.5) * 0.1,
        driftY: (Math.random() - 0.5) * 0.1,
      });
    }
  }

  _randomStarColor() {
    const colors = ['#a0e0ff', '#ffffff', '#c0d0ff', '#ffe0c0', '#80ffff'];
    return colors[Math.floor(Math.random() * colors.length)];
  }

  update(time) {
    // Move nebulas
    this.nebulas.forEach(n => {
      n.x += n.driftX;
      n.y += n.driftY;
      // Wrap around
      if (n.x < -n.radius) n.x = window.innerWidth + n.radius;
      if (n.x > window.innerWidth + n.radius) n.x = -n.radius;
      if (n.y < -n.radius) n.y = window.innerHeight + n.radius;
      if (n.y > window.innerHeight + n.radius) n.y = -n.radius;
    });
    // Update star twinkle
    this.stars.forEach(s => {
      s.twinklePhase += s.twinkleSpeed;
      s.brightness = 0.4 + Math.sin(s.twinklePhase) * 0.6;
      s.y += s.speed;
      if (s.y > window.innerHeight) { s.y = 0; s.x = Math.random() * window.innerWidth; }
    });
  }

  draw(ctx, time) {
    // Draw nebulas first
    this.nebulas.forEach(n => {
      const g = ctx.createRadialGradient(n.x, n.y, 0, n.x, n.y, n.radius);
      g.addColorStop(0, n.color);
      g.addColorStop(1, 'transparent');
      ctx.fillStyle = g;
      ctx.fillRect(n.x - n.radius, n.y - n.radius, n.radius * 2, n.radius * 2);
    });
    // Draw stars
    this.stars.forEach(s => {
      ctx.beginPath();
      ctx.arc(s.x, s.y, s.size, 0, Math.PI * 2);
      ctx.fillStyle = s.color;
      ctx.globalAlpha = Math.max(0.1, s.brightness);
      ctx.fill();
      // Glow for bright stars
      if (s.size > 1.5) {
        ctx.beginPath();
        ctx.arc(s.x, s.y, s.size * 3, 0, Math.PI * 2);
        const g = ctx.createRadialGradient(s.x, s.y, 0, s.x, s.y, s.size * 3);
        g.addColorStop(0, s.color);
        g.addColorStop(1, 'transparent');
        ctx.fillStyle = g;
        ctx.globalAlpha = s.brightness * 0.3;
        ctx.fill();
      }
    });
    ctx.globalAlpha = 1;
  }
}
```

- [ ] **Step 3: 在 SCRIPT 底部初始化背景（main init 区）**

```javascript
// ===== MAIN LOOP & INIT =====
let bgCanvas, bgCtx, starField;
let animationId;

function initBackground() {
  bgCanvas = document.getElementById('bg-canvas');
  bgCanvas.width = window.innerWidth;
  bgCanvas.height = window.innerHeight;
  bgCtx = bgCanvas.getContext('2d');
  starField = new StarField(bgCtx, bgCanvas.width, bgCanvas.height);
}

function animateBackground(time) {
  bgCtx.clearRect(0, 0, bgCanvas.width, bgCanvas.height);
  starField.update(time);
  starField.draw(bgCtx, time);
  animationId = requestAnimationFrame(animateBackground);
}

// On load
window.addEventListener('load', () => {
  initBackground();
  animateBackground(0);
});

window.addEventListener('resize', () => {
  bgCanvas.width = window.innerWidth;
  bgCanvas.height = window.innerHeight;
  starField = new StarField(bgCtx, bgCanvas.width, bgCanvas.height);
});
```

- [ ] **Step 4: 运行验证**

- 200+ 星星随机分布并闪烁
- 5 个星云缓慢漂移
- 星星有视差下移效果
- 大星星有光晕
- 窗口缩放时重新初始化

- [ ] **Step 5: 提交**

```bash
git add snake.html
git commit -m "feat: add animated star field background with nebula effect"
```

---

### Task 3: 游戏核心逻辑 + Canvas 绘制蛇

**Files:**
- Modify: `/Users/ling/Desktop/开发/11/snake.html`

- [ ] **Step 1: 添加 GAME CORE 类**

在 `<!-- ===== GAME CORE ===== -->` 后添加：

```javascript
// ===== GAME CORE =====
class SnakeGame {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.gridCols = Math.floor(600 / CONFIG.GRID_SIZE);
    this.gridRows = Math.floor(600 / CONFIG.GRID_SIZE);
    this.canvas.width = this.gridCols * CONFIG.GRID_SIZE;
    this.canvas.height = this.gridRows * CONFIG.GRID_SIZE;
    this.mode = 'classic';
    this.state = 'idle'; // idle, playing, paused, gameover
    this.snake = [];
    this.direction = { x: 1, y: 0 };
    this.nextDirection = { x: 1, y: 0 };
    this.food = null;
    this.score = 0;
    this.highScore = parseInt(localStorage.getItem('snakeHighScore') || '0');
    this.level = 1;
    this.obstacles = [];
    this.powerups = [];
    this.lastTick = 0;
    this.tickInterval = CONFIG.SNAKE_SPEED;
    this.frameCount = 0;
    this.particles = [];
    this.activePowerup = null;
    this.shieldActive = false;
    this.trailParticles = [];
    this.screenShake = { x: 0, y: 0, intensity: 0 };
    this._bindInput();
  }

  reset() {
    const cx = Math.floor(this.gridCols / 2);
    const cy = Math.floor(this.gridRows / 2);
    this.snake = [
      { x: cx, y: cy },
      { x: cx - 1, y: cy },
      { x: cx - 2, y: cy },
    ];
    this.direction = { x: 1, y: 0 };
    this.nextDirection = { x: 1, y: 0 };
    this.score = 0;
    this.level = 1;
    this.obstacles = [];
    this.powerups = [];
    this.activePowerup = null;
    this.shieldActive = false;
    this.particles = [];
    this.trailParticles = [];
    this.screenShake = { x: 0, y: 0, intensity: 0 };
    this.spawnFood();
    if (this.mode === 'level') this._generateObstacles();
  }

  _bindInput() {
    document.addEventListener('keydown', (e) => {
      if (['ArrowUp','ArrowDown','ArrowLeft','ArrowRight'].includes(e.key)) {
        e.preventDefault();
        if (this.state !== 'playing') return;
        const { x, y } = this.direction;
        if (e.key === 'ArrowUp' && y !== 1) this.nextDirection = { x: 0, y: -1 };
        if (e.key === 'ArrowDown' && y !== -1) this.nextDirection = { x: 0, y: 1 };
        if (e.key === 'ArrowLeft' && x !== 1) this.nextDirection = { x: -1, y: 0 };
        if (e.key === 'ArrowRight' && x !== -1) this.nextDirection = { x: 1, y: 0 };
      }
      if (e.key === 'Enter') this._handleEnter();
      if (e.key === 'Escape') this._handleEscape();
    });
  }

  _handleEnter() {
    if (this.state === 'idle' || this.state === 'gameover') {
      this.reset();
      this.state = 'playing';
      this.lastTick = performance.now();
      AudioManager.play('start');
    }
  }

  _handleEscape() {
    if (this.state === 'playing') { this.state = 'paused'; }
    else if (this.state === 'paused') { this.state = 'playing'; this.lastTick = performance.now(); }
    else if (this.state === 'gameover') { switchScreen('mode-select'); }
  }

  spawnFood() {
    let pos;
    let attempts = 0;
    do {
      pos = {
        x: Math.floor(Math.random() * this.gridCols),
        y: Math.floor(Math.random() * this.gridRows),
      };
      attempts++;
    } while (this._isOccupied(pos) && attempts < 100);
    this.food = { ...pos, type: 'normal', pulse: 0 };
  }

  _isOccupied(pos) {
    if (this.snake.some(s => s.x === pos.x && s.y === pos.y)) return true;
    if (this.obstacles.some(o => o.x === pos.x && o.y === pos.y)) return true;
    if (this.powerups.some(p => p.x === pos.x && p.y === pos.y)) return true;
    return false;
  }

  update(time) {
    if (this.state !== 'playing') return;
    if (time - this.lastTick < this.tickInterval) return;
    this.lastTick = time;
    this.frameCount++;
    this._applyPowerups(time);
    this.direction = { ...this.nextDirection };
    const head = { ...this.snake[0] };
    head.x += this.direction.x;
    head.y += this.direction.y;
    // Wall collision
    if (this.mode !== 'warp') {
      if (head.x < 0 || head.x >= this.gridCols || head.y < 0 || head.y >= this.gridRows) {
        if (!this.shieldActive) { this._gameOver(); return; }
        else { this._bounceHead(head); }
      }
    } else {
      // Wrap
      if (head.x < 0) head.x = this.gridCols - 1;
      if (head.x >= this.gridCols) head.x = 0;
      if (head.y < 0) head.y = this.gridRows - 1;
      if (head.y >= this.gridRows) head.y = 0;
    }
    // Self collision
    if (this.snake.some(s => s.x === head.x && s.y === head.y)) {
      if (!this.shieldActive) { this._gameOver(); return; }
      else { this.snake.shift(); this.shieldActive = false; }
    }
    // Obstacle collision
    if (this.obstacles.some(o => o.x === head.x && o.y === head.y)) {
      if (!this.shieldActive) { this._gameOver(); return; }
      else { this._removeObstacle(head); this.shieldActive = false; }
    }
    this.snake.unshift(head);
    // Food collision
    if (head.x === this.food?.x && head.y === this.food?.y) {
      this._eatFood();
    } else {
      this.snake.pop();
    }
    // Trail particles
    if (this.frameCount % CONFIG.TRAIL_SPAWN_RATE === 0 && this.snake.length > 0) {
      const tail = this.snake[this.snake.length - 1];
      this.trailParticles.push({
        x: tail.x * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2,
        y: tail.y * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2,
        life: CONFIG.TRAIL_PARTICLE_LIFE,
        maxLife: CONFIG.TRAIL_PARTICLE_LIFE,
        size: Math.random() * 3 + 2,
        color: `hsl(${190 + Math.random() * 20}, 90%, 70%)`,
      });
    }
    // Powerup spawning
    if (this.mode === 'powerup' && this.frameCount % 300 === 0 && this.powerups.length < 2) {
      this._spawnPowerup();
    }
    // Update particles
    this.particles = this.particles.filter(p => { p.life -= 16; return p.life > 0; });
    this.trailParticles = this.trailParticles.filter(p => { p.life -= 16; return p.life > 0; });
    // Screen shake decay
    this.screenShake.intensity *= 0.9;
    if (this.screenShake.intensity < 0.1) this.screenShake.intensity = 0;
    this.screenShake.x = (Math.random() - 0.5) * this.screenShake.intensity;
    this.screenShake.y = (Math.random() - 0.5) * this.screenShake.intensity;
    // Level mode: check level up
    if (this.mode === 'level' && this.score >= this.level * 100) {
      this._levelUp();
    }
    // Food pulse
    if (this.food) this.food.pulse = (this.food.pulse + 0.1) % (Math.PI * 2);
  }

  _bounceHead(head) {
    if (head.x < 0) head.x = 0;
    if (head.x >= this.gridCols) head.x = this.gridCols - 1;
    if (head.y < 0) head.y = 0;
    if (head.y >= this.gridRows) head.y = this.gridRows - 1;
    this.direction = { x: -this.direction.x, y: -this.direction.y };
    this.nextDirection = { ...this.direction };
    this.shieldActive = false;
  }

  _removeObstacle(head) {
    this.obstacles = this.obstacles.filter(o => !(o.x === head.x && o.y === head.y));
    this.shieldActive = false;
  }

  _eatFood() {
    const pts = this.food.type === 'gold' ? 50 : 10;
    this.score += pts;
    AudioManager.play('eat');
    // Particle explosion
    for (let i = 0; i < 20; i++) {
      const angle = (Math.PI * 2 / 20) * i + Math.random() * 0.3;
      const speed = Math.random() * 4 + 2;
      this.particles.push({
        x: this.food.x * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2,
        y: this.food.y * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2,
        vx: Math.cos(angle) * speed,
        vy: Math.sin(angle) * speed,
        life: 600,
        maxLife: 600,
        size: Math.random() * 4 + 2,
        color: this.food.type === 'gold' ? '#ffd700' : '#cc44ff',
      });
    }
    if (this.score > this.highScore) {
      this.highScore = this.score;
      localStorage.setItem('snakeHighScore', this.highScore);
    }
    if (this.food.type === 'powerup') {
      this._applyPowerup(this.food.powerupType);
      this.powerups = this.powerups.filter(p => !(p.x === this.food.x && p.y === this.food.y));
    } else {
      this.powerups = this.powerups.filter(p => !(p.x === this.food.x && p.y === this.food.y));
    }
    this.spawnFood();
  }

  _spawnPowerup() {
    const types = ['speed', 'shield', 'gold', 'freeze'];
    const type = types[Math.floor(Math.random() * types.length)];
    let pos, attempts = 0;
    do {
      pos = { x: Math.floor(Math.random() * this.gridCols), y: Math.floor(Math.random() * this.gridRows) };
      attempts++;
    } while (this._isOccupied(pos) && attempts < 50);
    this.powerups.push({ ...pos, type: 'powerup', powerupType: type, pulse: 0 });
  }

  _applyPowerup(type) {
    this.activePowerup = { type, endTime: performance.now() + 5000 };
    if (type === 'speed') { this.tickInterval = CONFIG.SNAKE_SPEED * 0.6; this.snake.splice(0, 2); }
    if (type === 'freeze') { this.tickInterval = CONFIG.SNAKE_SPEED * 1.4; }
    if (type === 'shield') { this.shieldActive = true; this.activePowerup.endTime = performance.now() + 10000; }
    AudioManager.play('powerup');
  }

  _applyPowerups(time) {
    if (this.activePowerup && time > this.activePowerup.endTime) {
      this.tickInterval = CONFIG.SNAKE_SPEED;
      this.activePowerup = null;
    }
  }

  _generateObstacles() {
    const count = this.level * 3;
    for (let i = 0; i < count; i++) {
      let pos, attempts = 0;
      do {
        pos = { x: Math.floor(Math.random() * this.gridCols), y: Math.floor(Math.random() * this.gridRows) };
        attempts++;
      } while (
        (this.snake.some(s => Math.abs(s.x - pos.x) < 3 && Math.abs(s.y - pos.y) < 3)) &&
        attempts < 50
      );
      if (attempts < 50) this.obstacles.push({ ...pos, type: 'rock' });
    }
  }

  _levelUp() {
    this.level++;
    this.tickInterval = Math.max(60, CONFIG.SNAKE_SPEED - (this.level - 1) * 10);
    this._generateObstacles();
    AudioManager.play('levelup');
    // Level up particle burst
    for (let i = 0; i < 40; i++) {
      const angle = (Math.PI * 2 / 40) * i;
      this.particles.push({
        x: this.canvas.width / 2,
        y: this.canvas.height / 2,
        vx: Math.cos(angle) * 8,
        vy: Math.sin(angle) * 8,
        life: 800,
        maxLife: 800,
        size: 4,
        color: '#ffd700',
      });
    }
  }

  _gameOver() {
    this.state = 'gameover';
    this.screenShake.intensity = 15;
    AudioManager.play('gameover');
    // Explosion particles
    const head = this.snake[0];
    for (let i = 0; i < 50; i++) {
      const angle = Math.random() * Math.PI * 2;
      const speed = Math.random() * 6 + 1;
      this.particles.push({
        x: head.x * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2,
        y: head.y * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2,
        vx: Math.cos(angle) * speed,
        vy: Math.sin(angle) * speed,
        life: 1000,
        maxLife: 1000,
        size: Math.random() * 5 + 1,
        color: Math.random() > 0.5 ? '#ff4444' : '#ff8844',
      });
    }
  }

  draw(ctx) {
    ctx.save();
    ctx.translate(this.screenShake.x, this.screenShake.y);
    // Clear with transparent
    ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    // Draw grid lines (subtle)
    ctx.strokeStyle = 'rgba(0, 200, 255, 0.05)';
    ctx.lineWidth = 1;
    for (let x = 0; x <= this.gridCols; x++) {
      ctx.beginPath(); ctx.moveTo(x * CONFIG.GRID_SIZE, 0); ctx.lineTo(x * CONFIG.GRID_SIZE, this.canvas.height); ctx.stroke();
    }
    for (let y = 0; y <= this.gridRows; y++) {
      ctx.beginPath(); ctx.moveTo(0, y * CONFIG.GRID_SIZE); ctx.lineTo(this.canvas.width, y * CONFIG.GRID_SIZE); ctx.stroke();
    }
    // Draw obstacles
    this.obstacles.forEach(o => {
      ctx.fillStyle = '#334';
      ctx.shadowColor = '#556';
      ctx.shadowBlur = 10;
      ctx.fillRect(o.x * CONFIG.GRID_SIZE + 2, o.y * CONFIG.GRID_SIZE + 2, CONFIG.GRID_SIZE - 4, CONFIG.GRID_SIZE - 4);
      ctx.shadowBlur = 0;
    });
    // Draw powerups
    this.powerups.forEach(p => {
      const cx = p.x * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2;
      const cy = p.y * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2;
      const symbols = { speed: '⚡', shield: '🛡', gold: '💛', freeze: '❄' };
      ctx.font = '16px serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(symbols[p.powerupType] || '★', cx, cy);
    });
    // Draw trail particles
    this.trailParticles.forEach(p => {
      const alpha = p.life / p.maxLife;
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.size * alpha, 0, Math.PI * 2);
      ctx.fillStyle = p.color;
      ctx.globalAlpha = alpha * 0.6;
      ctx.fill();
    });
    ctx.globalAlpha = 1;
    // Draw snake
    this.snake.forEach((seg, i) => {
      const x = seg.x * CONFIG.GRID_SIZE;
      const y = seg.y * CONFIG.GRID_SIZE;
      const isHead = i === 0;
      // Glow
      const gradient = ctx.createLinearGradient(x, y, x + CONFIG.GRID_SIZE, y + CONFIG.GRID_SIZE);
      gradient.addColorStop(0, isHead ? '#00ffff' : '#00c8ff');
      gradient.addColorStop(1, isHead ? '#00c8ff' : '#0080ff');
      ctx.shadowColor = isHead ? '#00ffff' : '#0080ff';
      ctx.shadowBlur = isHead ? 20 : 10;
      ctx.fillStyle = gradient;
      ctx.beginPath();
      ctx.roundRect(x + 1, y + 1, CONFIG.GRID_SIZE - 2, CONFIG.GRID_SIZE - 2, isHead ? 6 : 4);
      ctx.fill();
      ctx.shadowBlur = 0;
      // Eyes on head
      if (isHead) {
        ctx.fillStyle = '#ffffff';
        const eyeSize = 3;
        const dir = this.direction;
        const ex1 = x + CONFIG.GRID_SIZE / 2 + dir.x * 5 - dir.y * 4;
        const ey1 = y + CONFIG.GRID_SIZE / 2 + dir.y * 5 + dir.x * 4;
        const ex2 = x + CONFIG.GRID_SIZE / 2 + dir.x * 5 + dir.y * 4;
        const ey2 = y + CONFIG.GRID_SIZE / 2 + dir.y * 5 - dir.x * 4;
        ctx.beginPath(); ctx.arc(ex1, ey1, eyeSize, 0, Math.PI * 2); ctx.fill();
        ctx.beginPath(); ctx.arc(ex2, ey2, eyeSize, 0, Math.PI * 2); ctx.fill();
        ctx.fillStyle = '#001133';
        ctx.beginPath(); ctx.arc(ex1 + dir.x, ey1 + dir.y, eyeSize - 1, 0, Math.PI * 2); ctx.fill();
        ctx.beginPath(); ctx.arc(ex2 + dir.x, ey2 + dir.y, eyeSize - 1, 0, Math.PI * 2); ctx.fill();
      }
      // Shield effect
      if (this.shieldActive && i === 0) {
        ctx.strokeStyle = 'rgba(0, 255, 200, 0.8)';
        ctx.lineWidth = 2;
        ctx.shadowColor = '#00ffc8';
        ctx.shadowBlur = 15;
        ctx.beginPath();
        ctx.arc(x + CONFIG.GRID_SIZE / 2, y + CONFIG.GRID_SIZE / 2, CONFIG.GRID_SIZE * 0.8, 0, Math.PI * 2);
        ctx.stroke();
        ctx.shadowBlur = 0;
      }
    });
    // Draw food
    if (this.food) {
      const fx = this.food.x * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2;
      const fy = this.food.y * CONFIG.GRID_SIZE + CONFIG.GRID_SIZE / 2;
      const pulseSize = 6 + Math.sin(this.food.pulse) * 2;
      // Outer glow
      const g = ctx.createRadialGradient(fx, fy, 0, fx, fy, CONFIG.GRID_SIZE);
      g.addColorStop(0, this.food.type === 'gold' ? 'rgba(255, 215, 0, 0.8)' : 'rgba(204, 68, 255, 0.8)');
      g.addColorStop(1, 'transparent');
      ctx.fillStyle = g;
      ctx.beginPath();
      ctx.arc(fx, fy, CONFIG.GRID_SIZE, 0, Math.PI * 2);
      ctx.fill();
      // Core
      ctx.fillStyle = this.food.type === 'gold' ? '#ffd700' : '#cc44ff';
      ctx.shadowColor = this.food.type === 'gold' ? '#ffd700' : '#cc44ff';
      ctx.shadowBlur = 15;
      ctx.beginPath();
      ctx.arc(fx, fy, pulseSize, 0, Math.PI * 2);
      ctx.fill();
      ctx.shadowBlur = 0;
    }
    // Draw particles
    this.particles.forEach(p => {
      const alpha = p.life / p.maxLife;
      ctx.globalAlpha = alpha;
      ctx.fillStyle = p.color;
      ctx.beginPath();
      ctx.arc(p.x + p.vx * (p.maxLife - p.life) / 16, p.y + p.vy * (p.maxLife - p.life) / 16, p.size * alpha, 0, Math.PI * 2);
      ctx.fill();
    });
    ctx.globalAlpha = 1;
    // Draw powerup active indicator
    if (this.activePowerup) {
      const remaining = Math.max(0, (this.activePowerup.endTime - performance.now()) / 1000).toFixed(1);
      ctx.font = '12px Orbitron';
      ctx.fillStyle = '#ffd700';
      ctx.textAlign = 'left';
      ctx.fillText(`${this.activePowerup.type.toUpperCase()}: ${remaining}s`, 5, this.canvas.height - 5);
    }
    ctx.restore();
  }
}
```

- [ ] **Step 2: 运行验证**

- 蛇从屏幕中央出现，青蓝色渐变
- 蛇头有方向眼睛
- 食物有脉动紫色光晕
- 按方向键蛇转向
- 吃食物：粒子爆炸 + 蛇身增长
- 碰墙或自己：游戏结束 + 屏幕震动

- [ ] **Step 3: 提交**

```bash
git add snake.html
git commit -m "feat: core game logic with snake rendering, food, collision, particles"
```

---

### Task 4: 音效系统（Web Audio API）

**Files:**
- Modify: `/Users/ling/Desktop/开发/11/snake.html`

- [ ] **Step 1: 在 AUDIO SYSTEM 区添加音频管理器**

```javascript
// ===== AUDIO SYSTEM =====
class AudioManager {
  static ctx = null;
  static bgNode = null;
  static muted = false;

  static init() {
    if (this.ctx) return;
    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
  }

  static _resume() {
    if (this.ctx && this.ctx.state === 'suspended') this.ctx.resume();
  }

  static play(type) {
    this._resume();
    if (this.muted || !this.ctx) return;
    switch (type) {
      case 'eat': this._eat(); break;
      case 'gameover': this._gameover(); break;
      case 'start': this._start(); break;
      case 'powerup': this._powerup(); break;
      case 'levelup': this._levelup(); break;
      case 'select': this._select(); break;
    }
  }

  static _eat() {
    const osc = this.ctx.createOscillator();
    const gain = this.ctx.createGain();
    osc.connect(gain); gain.connect(this.ctx.destination);
    osc.type = 'sine';
    osc.frequency.setValueAtTime(400, this.ctx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(800, this.ctx.currentTime + 0.1);
    gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + 0.15);
    osc.start(); osc.stop(this.ctx.currentTime + 0.15);
  }

  static _gameover() {
    const osc = this.ctx.createOscillator();
    const gain = this.ctx.createGain();
    osc.connect(gain); gain.connect(this.ctx.destination);
    osc.type = 'sawtooth';
    osc.frequency.setValueAtTime(200, this.ctx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(50, this.ctx.currentTime + 0.5);
    gain.gain.setValueAtTime(0.2, this.ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + 0.5);
    osc.start(); osc.stop(this.ctx.currentTime + 0.5);
  }

  static _start() {
    [523, 659, 784].forEach((freq, i) => {
      const osc = this.ctx.createOscillator();
      const gain = this.ctx.createGain();
      osc.connect(gain); gain.connect(this.ctx.destination);
      osc.type = 'sine';
      osc.frequency.value = freq;
      gain.gain.setValueAtTime(0, this.ctx.currentTime + i * 0.1);
      gain.gain.linearRampToValueAtTime(0.1, this.ctx.currentTime + i * 0.1 + 0.05);
      gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + i * 0.1 + 0.2);
      osc.start(this.ctx.currentTime + i * 0.1);
      osc.stop(this.ctx.currentTime + i * 0.1 + 0.2);
    });
  }

  static _powerup() {
    const osc = this.ctx.createOscillator();
    const gain = this.ctx.createGain();
    osc.connect(gain); gain.connect(this.ctx.destination);
    osc.type = 'triangle';
    osc.frequency.setValueAtTime(300, this.ctx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(1200, this.ctx.currentTime + 0.2);
    gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + 0.3);
    osc.start(); osc.stop(this.ctx.currentTime + 0.3);
  }

  static _levelup() {
    [523, 659, 784, 1047].forEach((freq, i) => {
      const osc = this.ctx.createOscillator();
      const gain = this.ctx.createGain();
      osc.connect(gain); gain.connect(this.ctx.destination);
      osc.type = 'sine';
      osc.frequency.value = freq;
      gain.gain.setValueAtTime(0.1, this.ctx.currentTime + i * 0.08);
      gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + i * 0.08 + 0.3);
      osc.start(this.ctx.currentTime + i * 0.08);
      osc.stop(this.ctx.currentTime + i * 0.08 + 0.3);
    });
  }

  static _select() {
    const osc = this.ctx.createOscillator();
    const gain = this.ctx.createGain();
    osc.connect(gain); gain.connect(this.ctx.destination);
    osc.type = 'sine';
    osc.frequency.value = 600;
    gain.gain.setValueAtTime(0.08, this.ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + 0.08);
    osc.start(); osc.stop(this.ctx.currentTime + 0.08);
  }

  static startBg() {
    this._resume();
    if (this.bgNode || !this.ctx) return;
    // Ambient drone
    const osc1 = this.ctx.createOscillator();
    const osc2 = this.ctx.createOscillator();
    const gain = this.ctx.createGain();
    osc1.connect(gain); osc2.connect(gain); gain.connect(this.ctx.destination);
    osc1.type = 'sine'; osc1.frequency.value = 55;
    osc2.type = 'sine'; osc2.frequency.value = 82.5;
    gain.gain.value = 0.03;
    osc1.start(); osc2.start();
    this.bgNode = { osc1, osc2, gain };
  }

  static stopBg() {
    if (this.bgNode) {
      this.bgNode.osc1.stop();
      this.bgNode.osc2.stop();
      this.bgNode = null;
    }
  }
}
```

- [ ] **Step 2: 在 Enter 键处理中调用 startBg**

在 `SnakeGame._handleEnter()` 中 `AudioManager.play('start')` 后添加 `AudioManager.startBg();`

- [ ] **Step 3: 在游戏结束时停止背景音**

在 `SnakeGame._gameOver()` 末尾添加 `AudioManager.stopBg();`

- [ ] **Step 4: 运行验证**

- 开始游戏：和弦上升音
- 吃食物：上升音调
- 游戏结束：下扫音
- 获得道具：三角波上升
- 升级：四音和弦

- [ ] **Step 5: 提交**

```bash
git add snake.html
git commit -m "feat: Web Audio API sound system with eat, gameover, powerup sounds"
```

---

### Task 5: UI 界面（模式选择 + HUD + 游戏结束）

**Files:**
- Modify: `/Users/ling/Desktop/开发/11/snake.html`

- [ ] **Step 1: 在 UI SCREENS 区添加界面管理**

```javascript
// ===== UI SCREENS =====
let currentScreen = 'title';
let selectedMode = 0;
const modes = [
  { id: 'classic', name: '经典模式', desc: '无尽挑战，碰墙即结束', icon: '🐍' },
  { id: 'warp', name: '穿墙模式', desc: '穿越边界，无限循环', icon: '🌀' },
  { id: 'level', name: '关卡挑战', desc: '目标分数，穿越障碍', icon: '⭐' },
  { id: 'powerup', name: '道具模式', desc: '随机道具，技能加成', icon: '⚡' },
];

function switchScreen(name) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  const hud = document.getElementById('hud');
  hud.classList.remove('visible');
  currentScreen = name;
  if (name === 'mode-select') {
    document.getElementById('mode-screen').classList.add('active');
    renderModeCards();
  } else if (name === 'game') {
    document.getElementById('game-screen').classList.add('active');
    hud.classList.add('visible');
    AudioManager.startBg();
  } else if (name === 'title') {
    document.getElementById('title-screen').classList.add('active');
  }
}

function renderModeCards() {
  const grid = document.getElementById('mode-grid');
  grid.innerHTML = '';
  modes.forEach((m, i) => {
    const card = document.createElement('div');
    card.className = 'mode-card' + (i === selectedMode ? ' selected' : '');
    card.innerHTML = `<div class="mode-icon">${m.icon}</div><div class="mode-name">${m.name}</div><div class="mode-desc">${m.desc}</div>`;
    card.onclick = () => selectMode(i);
    grid.appendChild(card);
  });
}

function selectMode(index) {
  selectedMode = index;
  AudioManager.play('select');
  renderModeCards();
}

// Mode screen key nav
document.addEventListener('keydown', (e) => {
  if (currentScreen !== 'mode-select') return;
  if (e.key === 'ArrowUp' || e.key === 'ArrowLeft') {
    selectedMode = (selectedMode - 2 + 4) % 4;
    AudioManager.play('select'); renderModeCards();
  }
  if (e.key === 'ArrowDown' || e.key === 'ArrowRight') {
    selectedMode = (selectedMode + 2) % 4;
    AudioManager.play('select'); renderModeCards();
  }
  if (e.key === 'Enter') {
    game.mode = modes[selectedMode].id;
    switchScreen('game');
    game.state = 'idle';
  }
  if (e.key === 'Escape') { switchScreen('title'); }
});
```

- [ ] **Step 2: 在 HTML 中添加模式选择和游戏界面**

在 `#ui-layer` 内添加：

```html
<!-- Mode select screen -->
<div id="mode-screen" class="screen">
  <div class="screen-title">选择模式</div>
  <div id="mode-grid"></div>
  <div class="mode-hint">↑↓←→ 选择 &nbsp;|&nbsp; Enter 确认 &nbsp;|&nbsp; Esc 返回</div>
</div>

<!-- Game screen -->
<div id="game-screen" class="screen" style="display:none;">
  <canvas id="game-canvas"></canvas>
</div>
```

- [ ] **Step 3: 添加界面 CSS**

```css
/* Mode select */
#mode-screen .screen-title {
  font-size: 28px;
  font-weight: 700;
  color: rgba(160, 224, 255, 0.9);
  margin-bottom: 30px;
  text-shadow: 0 0 20px rgba(0, 200, 255, 0.5);
  letter-spacing: 4px;
}
#mode-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 30px;
}
.mode-card {
  width: 220px;
  padding: 25px 20px;
  background: rgba(10, 10, 30, 0.7);
  border: 1px solid rgba(0, 200, 255, 0.15);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s;
  backdrop-filter: blur(10px);
}
.mode-card:hover, .mode-card.selected {
  border-color: rgba(0, 200, 255, 0.6);
  background: rgba(0, 100, 150, 0.3);
  box-shadow: 0 0 30px rgba(0, 200, 255, 0.2), inset 0 0 20px rgba(0, 200, 255, 0.05);
  transform: scale(1.05);
}
.mode-icon { font-size: 36px; margin-bottom: 10px; }
.mode-name { font-size: 16px; font-weight: 700; color: #00c8ff; margin-bottom: 6px; }
.mode-desc { font-size: 11px; color: rgba(160, 224, 255, 0.6); letter-spacing: 1px; }
.mode-hint {
  font-size: 12px;
  color: rgba(160, 224, 255, 0.4);
  letter-spacing: 2px;
}

/* Game over overlay */
#gameover-overlay {
  display: none;
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(5, 5, 20, 0.85);
  z-index: 10;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  backdrop-filter: blur(5px);
}
#gameover-overlay.visible { display: flex; }
#gameover-overlay h2 {
  font-size: 48px;
  color: #ff4466;
  text-shadow: 0 0 30px rgba(255, 68, 102, 0.6);
  margin-bottom: 20px;
}
#gameover-overlay .final-score {
  font-size: 72px;
  font-weight: 900;
  color: #fff;
  text-shadow: 0 0 40px rgba(0, 200, 255, 0.8);
  margin-bottom: 10px;
}
#gameover-overlay .high-score {
  font-size: 16px;
  color: rgba(160, 224, 255, 0.6);
  margin-bottom: 40px;
  letter-spacing: 2px;
}
#gameover-overlay .restart-hint {
  font-size: 14px;
  color: rgba(160, 224, 255, 0.5);
  animation: pulse 1.5s ease-in-out infinite;
  letter-spacing: 2px;
}
```

- [ ] **Step 4: 在 gameover 时显示结束界面**

在 `SnakeGame._gameOver()` 中，用 `setTimeout` 延迟显示：

```javascript
setTimeout(() => {
  const overlay = document.getElementById('gameover-overlay');
  overlay.querySelector('.final-score').textContent = this.score;
  overlay.querySelector('.high-score').textContent = `最高分: ${this.highScore}`;
  overlay.classList.add('visible');
}, 800);
```

- [ ] **Step 5: 监听重新开始**

在 Enter 键处理中，当 gameover 状态时：

```javascript
if (this.state === 'gameover') {
  document.getElementById('gameover-overlay').classList.remove('visible');
  this.reset();
  this.state = 'playing';
  this.lastTick = performance.now();
  AudioManager.play('start');
}
```

- [ ] **Step 6: 更新 HUD 显示**

在 `SnakeGame.update()` 中，每次更新后调用：

```javascript
_updateHUD() {
  document.getElementById('score-val').textContent = this.score;
  document.getElementById('highscore-val').textContent = this.highScore;
  document.getElementById('level-val').textContent = this.mode === 'level' ? this.level : '-';
  document.getElementById('mode-label').textContent = modes.find(m => m.id === this.mode)?.name || '';
}
```

在 `SnakeGame.update()` 末尾调用 `this._updateHUD();`

- [ ] **Step 7: 添加 HUD HTML**

```html
<div id="hud">
  <div class="hud-item">分数 <span id="score-val">0</span></div>
  <div class="hud-item" id="mode-label">经典模式</div>
  <div class="hud-item">关卡 <span id="level-val">-</span></div>
  <div class="hud-item">最高 <span id="highscore-val">0</span></div>
</div>
<div id="gameover-overlay">
  <h2>GAME OVER</h2>
  <div class="final-score">0</div>
  <div class="high-score">最高分: 0</div>
  <div class="restart-hint">按 Enter 重新开始 &nbsp;|&nbsp; Esc 返回选关</div>
</div>
```

- [ ] **Step 8: 运行验证**

- 标题画面按 Enter 进入模式选择
- 方向键在 4 个模式卡片间切换（2x2 网格）
- 选中模式发光高亮
- Enter 进入游戏
- HUD 显示分数/模式/关卡/最高分
- 游戏结束后显示 GAME OVER 遮罩 + 最终分数
- Enter 重开，Esc 返回选关

- [ ] **Step 9: 提交**

```bash
git add snake.html
git commit -m "feat: UI screens - mode select, HUD, game over overlay with keyboard nav"
```

---

### Task 6: 主循环集成 + 最终组装

**Files:**
- Modify: `/Users/ling/Desktop/开发/11/snake.html`

- [ ] **Step 1: 整合主循环**

在 `<!-- ===== MAIN LOOP & INIT ===== -->` 区实现完整初始化：

```javascript
// ===== MAIN LOOP & INIT =====
let game;
let gameCanvas, gameCtx;
let animationId;
let starField;

function initBackground() {
  bgCanvas = document.getElementById('bg-canvas');
  bgCanvas.width = window.innerWidth;
  bgCanvas.height = window.innerHeight;
  bgCtx = bgCanvas.getContext('2d');
  starField = new StarField(bgCtx, bgCanvas.width, bgCanvas.height);
}

function animateBackground(time) {
  bgCtx.clearRect(0, 0, bgCanvas.width, bgCanvas.height);
  if (starField) { starField.update(time); starField.draw(bgCtx, time); }
  animationId = requestAnimationFrame(animateBackground);
}

function initGame() {
  gameCanvas = document.getElementById('game-canvas');
  gameCtx = gameCanvas.getContext('2d');
  game = new SnakeGame(gameCanvas);
}

let lastGameUpdate = 0;
function gameLoop(timestamp) {
  if (currentScreen === 'game' && game) {
    game.update(timestamp);
    game.draw(gameCtx);
  }
  requestAnimationFrame(gameLoop);
}

// Title screen enter
document.addEventListener('keydown', function titleHandler(e) {
  if (currentScreen === 'title' && e.key === 'Enter') {
    document.removeEventListener('keydown', titleHandler);
    AudioManager.init();
    switchScreen('mode-select');
  }
});

window.addEventListener('load', () => {
  initBackground();
  animateBackground(0);
  initGame();
  requestAnimationFrame(gameLoop);
});

window.addEventListener('resize', () => {
  bgCanvas.width = window.innerWidth;
  bgCanvas.height = window.innerHeight;
  starField = new StarField(bgCtx, bgCanvas.width, bgCanvas.height);
});
```

- [ ] **Step 2: 运行完整验证**

打开浏览器完整测试所有四种模式：

- [ ] **经典模式**: 碰墙游戏结束，粒子爆炸
- [ ] **穿墙模式**: 从一边穿到另一边
- [ ] **关卡挑战**: 达到分数进入下一关，障碍物出现，速度加快
- [ ] **道具模式**: 随机道具出现，效果生效

- [ ] **Step 3: 提交**

```bash
git add snake.html
git commit -m "feat: main loop integration, all 4 game modes complete"
```

---

### Task 7: 最终优化 + 验收

**Files:**
- Modify: `/Users/ling/Desktop/开发/11/snake.html`

- [ ] **Step 1: 响应式布局优化**

确保 Canvas 在不同屏幕上居中且大小合适。添加：

```css
#game-canvas {
  max-width: 90vw;
  max-height: 80vh;
  width: auto;
  height: auto;
}
```

- [ ] **Step 2: 确保 ESC 键从游戏返回模式选择**

在 SnakeGame `_handleEscape` 中添加：

```javascript
if (this.state === 'playing' || this.state === 'paused') {
  AudioManager.stopBg();
  document.getElementById('gameover-overlay').classList.remove('visible');
  switchScreen('mode-select');
}
```

- [ ] **Step 3: 验收检查**

运行 `snake.html`，逐一验证：

```
[ ] 星空背景动画流畅（200+星星闪烁 + 星云漂移）
[ ] 标题画面按 Enter 进入模式选择
[ ] 四种模式卡片可方向键切换 + Enter 确认
[ ] 经典模式：碰墙/自咬结束，GAME OVER 显示
[ ] 穿墙模式：蛇从对侧穿出
[ ] 关卡模式：达到目标分数升关，障碍物出现
[ ] 道具模式：⚡🛡💛❄ 道具出现并生效
[ ] 吃食物粒子爆炸效果
[ ] 蛇身粒子拖尾
[ ] 音效正常播放（吃/结束/升级/道具）
[ ] HUD 实时更新分数
[ ] 最高分 localStorage 持久化
[ ] Enter 重新开始，Esc 返回
[ ] 屏幕响应式，适配不同窗口大小
```

- [ ] **Step 4: 最终提交**

```bash
git add snake.html
git commit -m "feat: snake game complete - all 4 modes, space theme, particles, audio"
```

---

## 自检清单

**Spec 覆盖检查：**
- [x] 四种模式（经典/穿墙/关卡/道具）→ Task 3+5+6
- [x] 星空背景动画 → Task 2
- [x] 蛇身粒子拖尾 → Task 3 (trailParticles)
- [x] 所有音效 → Task 4
- [x] 穿墙功能 → Task 3 (_gameOver wall check)
- [x] 道具效果 → Task 3 (_applyPowerup)
- [x] 关卡障碍物 → Task 3 (_generateObstacles)
- [x] 分数记录显示 → Task 5
- [x] 最高分持久化 → Task 3 (localStorage)
- [x] Enter 重开 → Task 3+5
- [x] 响应式 → Task 7

**占位符扫描：** 无 TBD/TODO
**类型一致性：** 全部一致
