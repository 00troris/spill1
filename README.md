<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clicker Game</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
            padding: 40px;
            max-width: 600px;
            margin: 0 auto;
        }

        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 10px;
            font-size: 2.5em;
        }

        .stats {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            margin-bottom: 30px;
            text-align: center;
        }

        .stat-item {
            margin: 10px 0;
            font-size: 1.1em;
            color: #555;
        }

        .stat-label {
            font-weight: bold;
            color: #333;
        }

        .stat-value {
            color: #667eea;
            font-size: 1.3em;
            font-weight: bold;
        }

        .progress-bar {
            width: 100%;
            height: 20px;
            background: #ddd;
            border-radius: 10px;
            overflow: hidden;
            margin-top: 10px;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #667eea, #764ba2);
            width: 0%;
            transition: width 0.3s ease;
        }

        .click-button {
            position: fixed;
            bottom: 40px;
            right: 40px;
            width: 150px;
            height: 150px;
            font-size: 2em;
            border: none;
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border-radius: 50%;
            cursor: pointer;
            transition: all 0.1s ease;
            box-shadow: 0 10px 30px rgba(102, 126, 234, 0.4);
            font-weight: bold;
            z-index: 1000;
        }

        .click-button:hover {
            transform: scale(1.05);
            box-shadow: 0 15px 40px rgba(102, 126, 234, 0.6);
        }

        .click-button:active {
            transform: scale(0.95);
        }

        @media (max-width: 768px) {
            .click-button {
                width: 120px;
                height: 120px;
                font-size: 1.5em;
                bottom: 30px;
                right: 30px;
            }
        }

        .upgrades-section {
            margin-top: 40px;
        }

        .section-title {
            font-size: 1.3em;
            font-weight: bold;
            color: #333;
            margin-bottom: 15px;
            border-bottom: 2px solid #667eea;
            padding-bottom: 10px;
        }

        .upgrade-list {
            display: grid;
            gap: 10px;
        }

        .upgrade-item {
            padding: 15px;
            background: #f8f9fa;
            border-radius: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border: 2px solid #ddd;
            transition: all 0.3s ease;
        }

        .upgrade-item.available {
            border-color: #667eea;
            background: #f0f4ff;
        }

        .upgrade-item.available:hover {
            background: #e8ecff;
            transform: translateX(5px);
        }

        .upgrade-info {
            flex: 1;
        }

        .upgrade-name {
            font-weight: bold;
            color: #333;
            margin-bottom: 5px;
        }

        .upgrade-cost {
            font-size: 0.9em;
            color: #666;
        }

        .upgrade-button {
            padding: 10px 20px;
            border: none;
            background: #667eea;
            color: white;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s ease;
        }

        .upgrade-button:hover:not(:disabled) {
            background: #764ba2;
            transform: scale(1.05);
        }

        .upgrade-button:disabled {
            background: #ccc;
            cursor: not-allowed;
            opacity: 0.6;
        }

        .locked-message {
            text-align: center;
            padding: 20px;
            background: #fff3cd;
            border-radius: 10px;
            color: #856404;
            margin-top: 20px;
            font-weight: bold;
        }

        .bonus-notification {
            text-align: center;
            padding: 15px;
            background: #d4edda;
            border-radius: 10px;
            color: #155724;
            margin-top: 15px;
            font-weight: bold;
            animation: slideIn 0.5s ease;
        }

        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(-10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🖱️ Clicker Game</h1>
        
        <div class="stats">
            <div class="stat-item">
                <span class="stat-label">Total Clicks:</span>
                <span class="stat-value" id="clickCount">0</span>
            </div>
            <div class="progress-bar">
                <div class="progress-fill" id="progressFill"></div>
            </div>
            <div class="stat-item" style="margin-top: 15px;">
                <span class="stat-label">Money:</span>
                <span class="stat-value" id="money">0</span> 💰
            </div>
        </div>

        <div id="bonusNotification" class="bonus-notification hidden">
            ⭐ Every 100 clicks you earn +1 money!
        </div>

        <div id="upgradesSection" class="hidden">
            <div class="section-title">🚀 Upgrades</div>
            <div class="upgrade-list" id="upgradeList"></div>
        </div>

        <div id="lockedMessage" class="locked-message">
            🔒 Click 100,000 times to unlock upgrades!
        </div>
    </div>

    <button class="click-button" id="clickBtn">+1</button>

    <script>
        // Game state
        const gameState = {
            clicks: 0,
            money: 0,
            upgradesUnlocked: false,
            upgrades: [
                { id: 1, name: '2x Click Power', cost: 5000, multiplier: 2, owned: false },
                { id: 2, name: 'Auto Clicker (1/s)', cost: 10000, autoClicks: 1, owned: false },
                { id: 3, name: '5x Money Boost', cost: 15000, moneyMultiplier: 5, owned: false },
                { id: 4, name: 'Super Auto (5/s)', cost: 25000, autoClicks: 5, owned: false },
                { id: 5, name: '10x Click Power', cost: 40000, multiplier: 10, owned: false },
            ],
            currentClickValue: 1,
            autoClickRate: 0,
            lastBonusClick: 0
        };

        // DOM elements
        const clickBtn = document.getElementById('clickBtn');
        const clickCountDisplay = document.getElementById('clickCount');
        const moneyDisplay = document.getElementById('money');
        const progressFill = document.getElementById('progressFill');
        const upgradesSection = document.getElementById('upgradesSection');
        const lockedMessage = document.getElementById('lockedMessage');
        const upgradeList = document.getElementById('upgradeList');
        const bonusNotification = document.getElementById('bonusNotification');

        // Click handler
        clickBtn.addEventListener('click', () => {
            gameState.clicks++;
            gameState.money += gameState.currentClickValue;
            
            // Check if we've reached every 100 clicks for bonus
            if (gameState.clicks % 100 === 0 && gameState.clicks !== gameState.lastBonusClick) {
                gameState.money++;
                gameState.lastBonusClick = gameState.clicks;
                showBonusNotification();
            }
            
            updateDisplay();
            checkUpgradeUnlock();
        });

        // Show bonus notification
        function showBonusNotification() {
            bonusNotification.classList.remove('hidden');
            setTimeout(() => {
                bonusNotification.classList.add('hidden');
            }, 2000);
        }

        // Update display
        function updateDisplay() {
            clickCountDisplay.textContent = gameState.clicks.toLocaleString();
            moneyDisplay.textContent = gameState.money.toLocaleString();
            progressFill.style.width = Math.min((gameState.clicks / 100000) * 100, 100) + '%';
        }

        // Check if upgrades should be unlocked
        function checkUpgradeUnlock() {
            if (gameState.clicks >= 100000 && !gameState.upgradesUnlocked) {
                gameState.upgradesUnlocked = true;
                upgradesSection.classList.remove('hidden');
                lockedMessage.classList.add('hidden');
                renderUpgrades();
            }
        }

        // Render upgrades
        function renderUpgrades() {
            upgradeList.innerHTML = '';
            gameState.upgrades.forEach(upgrade => {
                const canAfford = gameState.money >= upgrade.cost;
                const upgradeEl = document.createElement('div');
                upgradeEl.className = `upgrade-item ${canAfford ? 'available' : ''}`;
                upgradeEl.innerHTML = `
                    <div class="upgrade-info">
                        <div class="upgrade-name">${upgrade.name}</div>
                        <div class="upgrade-cost">Cost: ${upgrade.cost.toLocaleString()} 💰</div>
                    </div>
                    <button class="upgrade-button" ${!canAfford ? 'disabled' : ''} onclick="buyUpgrade(${upgrade.id})">
                        ${upgrade.owned ? 'Owned' : 'Buy'}
                    </button>
                `;
                upgradeList.appendChild(upgradeEl);
            });
        }

        // Buy upgrade
        window.buyUpgrade = function(id) {
            const upgrade = gameState.upgrades.find(u => u.id === id);
            if (upgrade && gameState.money >= upgrade.cost && !upgrade.owned) {
                gameState.money -= upgrade.cost;
                upgrade.owned = true;

                if (upgrade.multiplier) {
                    gameState.currentClickValue *= upgrade.multiplier;
                }
                if (upgrade.autoClicks) {
                    gameState.autoClickRate += upgrade.autoClicks;
                }
                if (upgrade.moneyMultiplier) {
                    gameState.currentClickValue *= upgrade.moneyMultiplier;
                }

                updateDisplay();
                renderUpgrades();
            }
        };

        // Auto clicker
        setInterval(() => {
            if (gameState.autoClickRate > 0) {
                gameState.clicks += gameState.autoClickRate;
                gameState.money += gameState.autoClickRate * gameState.currentClickValue;
                
                // Check for bonus milestones during auto clicks
                if (gameState.clicks % 100 === 0 && gameState.clicks !== gameState.lastBonusClick) {
                    gameState.money++;
                    gameState.lastBonusClick = gameState.clicks;
                    showBonusNotification();
                }
                
                updateDisplay();
                checkUpgradeUnlock();
            }
        }, 1000);

        // Initial render
        updateDisplay();
    </script>
</body>
</html>
