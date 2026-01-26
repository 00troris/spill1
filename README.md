<!DOCTYPE html>
<html lang="no">
<head>
<meta charset="UTF-8">
<title>Enkelt Tower Defence</title>
<style>
    body {
        background: #222;
        color: #eee;
        font-family: Arial, sans-serif;
        text-align: center;
        margin: 0;
        padding: 10px;
    }
    #gameContainer {
        margin: 10px auto;
        position: relative;
        width: 640px;
    }
    canvas {
        background: #3b6b2a;
        border: 3px solid #555;
        display: block;
        margin: 0 auto;
    }
    #ui {
        margin-top: 10px;
    }
    #ui span {
        margin: 0 10px;
    }
    #info {
        font-size: 0.9rem;
        margin-top: 5px;
        color: #ccc;
    }
</style>
</head>
<body>
<h1>Mini Tower Defence</h1>

<div id="gameContainer">
    <canvas id="game" width="640" height="480"></canvas>
</div>

<div id="ui">
    <span>Liv: <strong id="lives">20</strong></span>
    <span>Penger: <strong id="money">100</strong></span>
    <span>Bølge: <strong id="wave">1</strong></span>
    <span>Tårnpris: <strong>50</strong></span>
</div>
<div id="info">
    Klikk på gresset for å plassere tårn. Ikke på stien. Fiender følger den lyse stien.
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const TILE = 40;
const ROWS = canvas.height / TILE;
const COLS = canvas.width / TILE;

// Enkel sti (liste med grid-koordinater)
const path = [
    {x:0,y:5},{x:1,y:5},{x:2,y:5},{x:3,y:5},{x:4,y:5},
    {x:5,y:5},{x:6,y:5},{x:7,y:5},{x:8,y:5},{x:9,y:5},
    {x:10,y:5},{x:11,y:5},{x:12,y:5},{x:13,y:5},{x:14,y:5},
    {x:15,y:5}
];

let towers = [];
let enemies = [];
let bullets = [];

let lives = 20;
let money = 100;
let wave = 1;
let spawnTimer = 0;
let enemiesToSpawn = 10;

const livesEl = document.getElementById("lives");
const moneyEl = document.getElementById("money");
const waveEl = document.getElementById("wave");

function gridToPixel(g) {
    return { x: g.x * TILE + TILE/2, y: g.y * TILE + TILE/2 };
}

class Enemy {
    constructor() {
        this.pathIndex = 0;
        const start = gridToPixel(path[0]);
        this.x = start.x;
        this.y = start.y;
        this.speed = 1 + wave * 0.1;
        this.radius = 12;
        this.hp = 20 + wave * 5;
        this.maxHp = this.hp;
    }
    update() {
        const targetGrid = path[this.pathIndex + 1];
        if (!targetGrid) {
            lives--;
            this.hp = 0;
            return;
        }
        const target = gridToPixel(targetGrid);
        const dx = target.x - this.x;
        const dy = target.y - this.y;
        const dist = Math.hypot(dx, dy);
        if (dist < this.speed) {
            this.x = target.x;
            this.y = target.y;
            this.pathIndex++;
        } else {
            this.x += this.speed * dx / dist;
            this.y += this.speed * dy / dist;
        }
    }
    draw() {
        // kropp
        ctx.fillStyle = "#b22222";
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI*2);
        ctx.fill();
        // HP-bar
        const barWidth = 26;
        const hpRatio = this.hp / this.maxHp;
        ctx.fillStyle = "#000";
        ctx.fillRect(this.x - barWidth/2, this.y - this.radius - 8, barWidth, 4);
        ctx.fillStyle = "#0f0";
        ctx.fillRect(this.x - barWidth/2, this.y - this.radius - 8, barWidth * hpRatio, 4);
    }
}

class Tower {
    constructor(x, y) {
        this.x = x;
        this.y = y;
        this.range = 100;
        this.fireRate = 40; // frames
        this.cooldown = 0;
    }
    update() {
        if (this.cooldown > 0) {
            this.cooldown--;
            return;
        }
        // finn nærmeste fiende i rekkevidde
        let target = null;
        let closest = Infinity;
        for (const e of enemies) {
            const d = Math.hypot(e.x - this.x, e.y - this.y);
            if (d < this.range && d < closest) {
                closest = d;
                target = e;
            }
        }
        if (target) {
            bullets.push(new Bullet(this.x, this.y, target));
            this.cooldown = this.fireRate;
        }
    }
    draw() {
        ctx.fillStyle = "#333";
        ctx.beginPath();
        ctx.arc(this.x, this.y, 14, 0, Math.PI*2);
        ctx.fill();
        ctx.fillStyle = "#999";
        ctx.beginPath();
        ctx.arc(this.x, this.y, 8, 0, Math.PI*2);
        ctx.fill();
    }
}

class Bullet {
    constructor(x, y, target) {
        this.x = x;
        this.y = y;
        this.target = target;
        this.speed = 5;
        this.radius = 4;
        this.damage = 10;
    }
    update() {
        if (!this.target || this.target.hp <= 0) {
            this.dead = true;
            return;
        }
        const dx = this.target.x - this.x;
        const dy = this.target.y - this.y;
        const dist = Math.hypot(dx, dy);
        if (dist < this.speed) {
            this.target.hp -= this.damage;
            if (this.target.hp <= 0) {
                money += 5;
            }
            this.dead = true;
        } else {
            this.x += this.speed * dx / dist;
            this.y += this.speed * dy / dist;
        }
    }
    draw() {
        ctx.fillStyle = "#ff0";
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI*2);
        ctx.fill();
    }
}

function isOnPath(gridX, gridY) {
    return path.some(p => p.x === gridX && p.y === gridY);
}

canvas.addEventListener("click", (e) => {
    const rect = canvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    const gridX = Math.floor(x / TILE);
    const gridY = Math.floor(y / TILE);

    if (isOnPath(gridX, gridY)) return;
    const cost = 50;
    if (money < cost) return;

    const pos = gridToPixel({x:gridX,y:gridY});
    towers.push(new Tower(pos.x, pos.y));
    money -= cost;
});

function spawnEnemy() {
    enemies.push(new Enemy());
}

function nextWave() {
    wave++;
    enemiesToSpawn = 10 + wave * 2;
    spawnTimer = 0;
}

function updateGame() {
    // spawn logikk
    if (enemiesToSpawn > 0) {
        spawnTimer--;
        if (spawnTimer <= 0) {
            spawnEnemy();
            enemiesToSpawn--;
            spawnTimer = 60; // tid mellom fiender
        }
    } else if (enemies.length === 0) {
        nextWave();
    }

    for (const e of enemies) e.update();
    for (const t of towers) t.update();
    for (const b of bullets) b.update();

    enemies = enemies.filter(e => e.hp > 0);
    bullets = bullets.filter(b => !b.dead);

    if (lives <= 0) {
        alert("Game Over! Du nådde bølge " + wave);
        resetGame();
    }

    livesEl.textContent = lives;
    moneyEl.textContent = money;
    waveEl.textContent = wave;
}

function drawGrid() {
    for (let y = 0; y < ROWS; y++) {
        for (let x = 0; x < COLS; x++) {
            const isPath = isOnPath(x,y);
            ctx.fillStyle = isPath ? "#c2b280" : "#3b6b2a";
            ctx.fillRect(x*TILE, y*TILE, TILE, TILE);
            ctx.strokeStyle = "rgba(0,0,0,0.2)";
            ctx.strokeRect(x*TILE, y*TILE, TILE, TILE);
        }
    }
}

function drawGame() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    drawGrid();
    for (const t of towers) t.draw();
    for (const e of enemies) e.draw();
    for (const b of bullets) b.draw();
}

function loop() {
    updateGame();
    drawGame();
    requestAnimationFrame(loop);
}

function resetGame() {
    lives = 20;
    money = 100;
    wave = 1;
    enemies = [];
    towers = [];
    bullets = [];
    enemiesToSpawn = 10;
    spawnTimer = 0;
}

resetGame();
loop();
</script>
</body>
</html>
