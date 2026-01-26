<!DOCTYPE html>
<html lang="no">
<head>
<meta charset="UTF-8">
<title>Forsterket Cookie Clicker</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background: #f3e5ab;
        text-align: center;
        margin: 0;
        padding: 20px;
    }

    h1 { margin-bottom: 5px; }

    #cookie {
        width: 220px;
        height: 220px;
        border-radius: 50%;
        background: radial-gradient(circle at 30% 30%, #f7d9a5, #c58b3b);
        box-shadow: 0 0 25px rgba(0,0,0,0.3);
        margin: 20px auto;
        position: relative;
        cursor: pointer;
        transition: transform 0.1s ease;
    }

    #cookie:active {
        transform: scale(0.92);
    }

    .chip {
        width: 22px;
        height: 22px;
        background: #4b2e19;
        border-radius: 50%;
        position: absolute;
    }

    /* Flyvende +1 */
    .floatText {
        position: absolute;
        color: #5a3b1a;
        font-weight: bold;
        animation: floatUp 1s ease-out forwards;
        pointer-events: none;
    }

    @keyframes floatUp {
        from { opacity: 1; transform: translateY(0); }
        to { opacity: 0; transform: translateY(-40px); }
    }

    #shop {
        background: #fff7d6;
        padding: 15px;
        border-radius: 10px;
        width: 330px;
        margin: 20px auto;
        box-shadow: 0 0 10px rgba(0,0,0,0.15);
    }

    .upgrade {
        display: flex;
        justify-content: space-between;
        background: #ffeeb5;
        padding: 10px;
        border-radius: 6px;
        margin: 8px 0;
    }

    .upgrade button {
        background: #d48b39;
        border: none;
        padding: 6px 12px;
        color: white;
        border-radius: 4px;
        cursor: pointer;
    }

    .upgrade button:disabled {
        background: #b3b3b3;
        cursor: not-allowed;
    }

</style>
</head>
<body>

<h1>Cookie Clicker+</h1>

<div id="stats">
    <p>Cookies: <strong id="cookieCount">0</strong></p>
    <p>Per klikk: <strong id="perClick">1</strong></p>
    <p>Per sekund: <strong id="perSecond">0</strong></p>
</div>

<div id="cookie">
    <div class="chip" style="top:40px; left:60px;"></div>
    <div class="chip" style="top:80px; right:50px;"></div>
    <div class="chip" style="bottom:50px; left:80px;"></div>
    <div class="chip" style="top:110px; right:90px;"></div>
    <div class="chip" style="bottom:70px; right:70px;"></div>
</div>

<div id="shop">
    <h2>Butikk</h2>

    <div class="upgrade">
        <div><strong>Bedre klikk</strong><br>+1 per klikk<br>Kostnad: <span id="clickCost">20</span></div>
        <button id="buyClick">Kjøp</button>
    </div>

    <div class="upgrade">
        <div><strong>Bestemor</strong><br>+1 per sekund<br>Kostnad: <span id="grandmaCost">50</span></div>
        <button id="buyGrandma">Kjøp</button>
    </div>

    <div class="upgrade">
        <div><strong>Fabrikk</strong><br>+5 per sekund<br>Kostnad: <span id="factoryCost">200</span></div>
        <button id="buyFactory">Kjøp</button>
    </div>

    <div class="upgrade">
        <div><strong>Superklikk</strong><br>10% sjanse for ×10 kritisk klikk<br>Kostnad: <span id="critCost">500</span></div>
        <button id="buyCrit">Kjøp</button>
    </div>

</div>

<script>
    let cookies = 0;
    let perClick = 1;
    let perSecond = 0;

    let clickCost = 20;
    let grandmaCost = 50;
    let factoryCost = 200;
    let critCost = 500;

    let critUnlocked = false;

    const cookieEl = document.getElementById("cookie");
    const cookieCountEl = document.getElementById("cookieCount");
    const perClickEl = document.getElementById("perClick");
    const perSecondEl = document.getElementById("perSecond");

    function update() {
        cookieCountEl.textContent = Math.floor(cookies);
        perClickEl.textContent = perClick;
        perSecondEl.textContent = perSecond;

        document.getElementById("clickCost").textContent = clickCost;
        document.getElementById("grandmaCost").textContent = grandmaCost;
        document.getElementById("factoryCost").textContent = factoryCost;
        document.getElementById("critCost").textContent = critCost;

        document.getElementById("buyClick").disabled = cookies < clickCost;
        document.getElementById("buyGrandma").disabled = cookies < grandmaCost;
        document.getElementById("buyFactory").disabled = cookies < factoryCost;
        document.getElementById("buyCrit").disabled = cookies < critCost;
    }

    function floatText(x, y, text) {
        const el = document.createElement("div");
        el.className = "floatText";
        el.style.left = x + "px";
        el.style.top = y + "px";
        el.textContent = text;
        document.body.appendChild(el);
        setTimeout(() => el.remove(), 1000);
    }

    cookieEl.addEventListener("click", (e) => {
        let gain = perClick;

        if (critUnlocked && Math.random() < 0.10) {
            gain *= 10;
            floatText(e.pageX, e.pageY, "KRITISK! +" + gain);
        } else {
            floatText(e.pageX, e.pageY, "+" + gain);
        }

        cookies += gain;
        update();
        save();
    });

    document.getElementById("buyClick").onclick = () => {
        if (cookies >= clickCost) {
            cookies -= clickCost;
            perClick++;
            clickCost = Math.floor(clickCost * 1.5);
            update();
            save();
        }
    };

    document.getElementById("buyGrandma").onclick = () => {
        if (cookies >= grandmaCost) {
            cookies -= grandmaCost;
            perSecond++;
            grandmaCost = Math.floor(grandmaCost * 1.6);
            update();
            save();
        }
    };

    document.getElementById("buyFactory").onclick = () => {
        if (cookies >= factoryCost) {
            cookies -= factoryCost;
            perSecond += 5;
            factoryCost = Math.floor(factoryCost * 1.7);
            update();
            save();
        }
    };

    document.getElementById("buyCrit").onclick = () => {
        if (cookies >= critCost) {
            cookies -= critCost;
            critUnlocked = true;
            critCost = "KJØPT";
            document.getElementById("buyCrit").disabled = true;
            update();
            save();
        }
    };

    setInterval(() => {
        cookies += perSecond / 10;
        update();
        save();
    }, 100);

    function save() {
        localStorage.setItem("cookieSave", JSON.stringify({
            cookies, perClick, perSecond,
            clickCost, grandmaCost, factoryCost, critCost,
            critUnlocked
        }));
    }

    function load() {
        const data = JSON.parse(localStorage.getItem("cookieSave"));
        if (!data) return;

        cookies = data.cookies;
        perClick = data.perClick;
        perSecond = data.perSecond;
        clickCost = data.clickCost;
        grandmaCost = data.grandmaCost;
        factoryCost = data.factoryCost;
        critCost = data.critCost;
        critUnlocked = data.critUnlocked;
    }

    load();
    update();
</script>

</body>
</html>
