v<!DOCTYPE html>
<html lang="no">
<head>
<meta charset="UTF-8">
<title>Clicker Spill</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background: #111;
        color: #eee;
        text-align: center;
    }
    button {
        padding: 8px;
        margin: 4px;
        cursor: pointer;
    }
    .box {
        border: 1px solid #444;
        padding: 10px;
        margin: 10px;
        background: #1c1c1c;
    }
</style>
</head>
<body>

<h1>🔥 Clicker Spill 🔥</h1>

<h2>Poeng: <span id="points">0</span></h2>
<h3>Per klikk: <span id="perClick">1</span></h3>
<h3>Per sekund: <span id="perSecond">0</span></h3>

<button id="clickBtn">Klikk!</button>

<div class="box">
    <h2>⬆ Oppgraderinger (30)</h2>
    <div id="upgrades"></div>
</div>

<div class="box">
    <h2>🌌 Ascension</h2>
    <p>Ascensions: <span id="ascensions">0</span></p>
    <p>Kostnad: <span id="ascCost">100000</span></p>
    <button id="ascendBtn">Ascend</button>
</div>

<div class="box">
    <h2>🏆 Achievements (20)</h2>
    <ul id="achievements"></ul>
</div>

<script>
/* ===== SPILLDATA ===== */
let points = 0;
let perClick = 1;
let perSecond = 0;

let ascensions = 0;
let ascensionCost = 100000;

const upgrades = Array.from({ length: 30 }, (_, i) => ({
    name: `Oppgradering ${i + 1}`,
    level: 0,
    baseCost: 10 * (i + 1) ** 2,
    cost: 10 * (i + 1) ** 2,
    clickBonus: i < 15 ? i + 1 : 0,
    passiveBonus: i >= 15 ? i - 14 : 0
}));

const achievements = Array.from({ length: 20 }, (_, i) => ({
    name: `Achievement ${i + 1}`,
    unlocked: false,
    requirement: (i + 1) * 1000
}));

/* ===== FUNKSJONER ===== */
function clickPoint() {
    points += perClick;
    update();
}

function buyUpgrade(i) {
    const u = upgrades[i];
    if (points < u.cost) return;

    points -= u.cost;
    u.level++;
    u.cost = Math.floor(u.cost * 1.5);

    perClick += u.clickBonus;
    perSecond += u.passiveBonus;

    update();
}

function buyAscension() {
    if (points < ascensionCost) return;

    ascensions++;
    points = 0;
    perClick = 1 + ascensions;
    perSecond = 0;

    ascensionCost = Math.floor(ascensionCost * 2);

    upgrades.forEach((u, i) => {
        u.level = 0;
        u.cost = u.baseCost;
    });

    update();
}

function checkAchievements() {
    achievements.forEach(a => {
        if (!a.unlocked && points >= a.requirement) {
            a.unlocked = true;
        }
    });
}

function update() {
    document.getElementById("points").textContent = Math.floor(points);
    document.getElementById("perClick").textContent = perClick;
    document.getElementById("perSecond").textContent = perSecond;
    document.getElementById("ascensions").textContent = ascensions;
    document.getElementById("ascCost").textContent = ascensionCost;

    renderUpgrades();
    renderAchievements();
}

function renderUpgrades() {
    const container = document.getElementById("upgrades");
    container.innerHTML = upgrades.map((u, i) =>
        `<button onclick="buyUpgrade(${i})">
            ${u.name} (Lvl ${u.level}) – Kostnad: ${u.cost}
        </button>`
    ).join("");
}

function renderAchievements() {
    checkAchievements();
    const list = document.getElementById("achievements");
    list.innerHTML = achievements.map(a =>
        `<li>${a.unlocked ? "✅" : "❌"} ${a.name}</li>`
    ).join("");
}

/* ===== PASSIV INNTEKT ===== */
setInterval(() => {
    points += perSecond;
    update();
}, 1000);

/* ===== EVENT LISTENERS ===== */
document.getElementById("clickBtn").onclick = clickPoint;
document.getElementById("ascendBtn").onclick = buyAscension;

update();
</script>

</body>
</html>
