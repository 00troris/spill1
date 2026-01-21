<!DOCTYPE html>
<html lang="no">
<head>
<meta charset="UTF-8">
<title>Cookie Clicker</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background: #222;
        color: #f5f5f5;
        text-align: center;
        margin: 0;
        padding: 0;
    }
    h1 {
        margin-top: 20px;
    }
    #cookie {
        width: 200px;
        height: 200px;
        border-radius: 50%;
        background: radial-gradient(circle at 30% 30%, #f7d7a8, #c58b4e);
        margin: 20px auto;
        box-shadow: 0 0 20px rgba(0,0,0,0.6);
        position: relative;
        cursor: pointer;
        transition: transform 0.05s;
    }
    #cookie:active {
        transform: scale(0.95);
    }
    .chip {
        width: 20px;
        height: 20px;
        background: #3b2a1a;
        border-radius: 50%;
        position: absolute;
    }
    .chip:nth-child(1) { top: 40px; left: 60px; }
    .chip:nth-child(2) { top: 80px; left: 120px; }
    .chip:nth-child(3) { top: 130px; left: 80px; }
    .chip:nth-child(4) { top: 60px; left: 130px; }
    .chip:nth-child(5) { top: 120px; left: 40px; }

    .stats {
        margin: 10px 0;
    }

    .container {
        display: flex;
        justify-content: center;
        gap: 20px;
        padding: 20px;
        flex-wrap: wrap;
    }

    .box {
        background: #333;
        border: 1px solid #555;
        padding: 15px;
        width: 280px;
        box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }

    button {
        padding: 8px;
        margin: 4px 0;
        width: 100%;
        cursor: pointer;
        border: none;
        background: #ffb347;
        color: #222;
        font-weight: bold;
        border-radius: 4px;
        transition: background 0.2s;
    }
    button:hover {
        background: #ff9800;
    }
    button:disabled {
        background: #777;
        cursor: not-allowed;
    }

    ul {
        list-style: none;
        padding: 0;
        text-align: left;
        font-size: 14px;
    }
</style>
</head>
<body>

<h1>🍪 Cookie Clicker</h1>

<div id="cookie">
    <div class="chip"></div>
    <div class="chip"></div>
    <div class="chip"></div>
    <div class="chip"></div>
    <div class="chip"></div>
</div>

<div class="stats">
    <h2>Cookies: <span id="cookies">0</span></h2>
    <p>Per klikk: <span id="perClick">1</span></p>
    <p>Per sekund: <span id="perSecond">0</span></p>
</div>

<div class="container">
    <div class="box">
        <h2>🛒 Butikk</h2>
        <div id="shop"></div>
    </div>

    <div class="box">
        <h2>🏆 Achievements</h2>
        <ul id="achievements"></ul>
    </div>
</div>

<script>
let cookies = 0;
let perClick = 1;
let perSecond = 0;

const upgrades = [
    { name: "Bedre klikk", type: "click", level: 0, baseCost: 10, cost: 10, bonus: 1 },
    { name: "Super klikk", type: "click", level: 0, baseCost: 100, cost: 100, bonus: 5 },
    { name: "Bestemor", type: "passive", level: 0, baseCost: 50, cost: 50, bonus: 1 },
    { name: "Fabrikk", type: "passive", level: 0, baseCost: 500, cost: 500, bonus: 5 }
];

const achievements = [
    { name: "Første cookie", requirement: 1, unlocked: false },
    { name: "100 cookies", requirement: 100, unlocked: false },
    { name: "1 000 cookies", requirement: 1000, unlocked: false },
    { name: "10 000 cookies", requirement: 10000, unlocked: false }
];

const cookieEl = document.getElementById("cookie");
const cookiesEl = document.getElementById("cookies");
const perClickEl = document.getElementById("perClick");
const perSecondEl = document.getElementById("perSecond");
const shopEl = document.getElementById("shop");
const achievementsEl = document.getElementById("achievements");

cookieEl.addEventListener("click", () => {
    cookies += perClick;
    update();
});

function buyUpgrade(index) {
    const u = upgrades[index];
    if (cookies < u.cost) return;

    cookies -= u.cost;
    u.level++;
    u.cost = Math.floor(u.baseCost * Math.pow(1.5, u.level));

    if (u.type === "click") {
        perClick += u.bonus;
    } else if (u.type === "passive") {
        perSecond += u.bonus;
    }

    update();
}

function renderShop() {
    shopEl.innerHTML = "";
    upgrades.forEach((u, i) => {
        const btn = document.createElement("button");
        btn.textContent = `${u.name} (Lvl ${u.level}) – Kostnad: ${u.cost}`;
        btn.disabled = cookies < u.cost;
        btn.onclick = () => buyUpgrade(i);
        shopEl.appendChild(btn);
    });
}

function checkAchievements() {
    achievements.forEach(a => {
        if (!a.unlocked && cookies >= a.requirement) {
            a.unlocked = true;
        }
    });
}

function renderAchievements() {
    achievementsEl.innerHTML = "";
    achievements.forEach(a => {
        const li = document.createElement("li");
        li.textContent = `${a.unlocked ? "✅" : "❌"} ${a.name}`;
        achievementsEl.appendChild(li);
    });
}

function update() {
    cookiesEl.textContent = Math.floor(cookies);
    perClickEl.textContent = perClick;
    perSecondEl.textContent = perSecond;

    checkAchievements();
    renderShop();
    renderAchievements();
}

setInterval(() => {
    cookies += perSecond;
    update();
}, 1000);

update();
</script>

</body>
</html>
