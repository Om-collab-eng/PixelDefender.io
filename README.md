<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tower Defense Game - Advanced</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Press Start 2P', cursive;
            background-color: #0f172a; /* Tailwind slate-900 */
            color: #e2e8f0; /* Tailwind slate-200 */
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start; /* Align to top for scroll */
            min-height: 100vh;
            margin: 0;
            padding-top: 20px; /* Add some padding at the top */
            overflow-x: hidden;
            overflow-y: auto;
        }
        #gameCanvas {
            background-color: #334155; /* Tailwind slate-700 */
            border: 3px solid #64748b; /* Tailwind slate-500 */
            border-radius: 8px;
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
            cursor: default;
        }
        #gameCanvas.sell-mode-active {
            cursor: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='32' height='32' viewBox='0 0 24 24' fill='none' stroke='%23ef4444' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'><path d='M12 8c-2 0-4 2-4 4s2 4 4 4 4-2 4-4-2-4-4-4zm0 7c-1.66 0-3-1.34-3-3s1.34-3 3-3 3 1.34 3 3-1.34 3-3 3zM12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8zm-5-8h10v2H7z'/></svg>") 16 16, auto;
        }
        .game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
        }
        .game-controls {
            background-color: #1e293b; /* Tailwind slate-800 */
            padding: 10px; /* Slightly reduced padding */
            border-radius: 8px;
            margin-top: 15px;
            margin-bottom: 20px; /* Added margin at bottom */
            display: flex;
            flex-wrap: wrap;
            justify-content: space-around;
            align-items: flex-start;
            width: 100%;
            max-width: 950px; /* Increased width for new buttons */
            box-shadow: 0 0 15px rgba(0,0,0,0.3);
        }
        .control-panel-section {
            margin: 6px; /* Reduced margin */
            padding: 10px; /* Reduced padding */
            background-color: #334155; /* Tailwind slate-700 */
            border-radius: 6px;
            text-align: center;
            flex-grow: 1;
            min-width: 160px; /* Adjusted min-width */
        }
        .control-panel-section h3 {
            font-size: 0.9em; /* Adjusted font size */
            margin-bottom: 6px;
            color: #94a3b8; /* Tailwind slate-400 */
        }
        .control-panel-section p, .control-panel-section button {
            font-size: 0.75em; /* Adjusted font size */
        }
        button {
            background-color: #4f46e5; /* Tailwind indigo-600 */
            color: white;
            border: none;
            padding: 7px 10px; /* Adjusted padding */
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease, box-shadow 0.3s ease;
            font-family: 'Press Start 2P', cursive;
            margin-top: 4px; /* Adjusted margin */
            margin-right: 4px; /* Add some right margin for spacing */
        }
        button:last-child {
            margin-right: 0;
        }
        button:hover {
            background-color: #3730a3; /* Tailwind indigo-800 */
        }
        button:disabled {
            background-color: #475569; /* Tailwind slate-600 */
            cursor: not-allowed;
        }
        button.active-mode, button.toggle-active { /* .toggle-active might be unused now */
            background-color: #d97706; /* Tailwind amber-600 */
            box-shadow: 0 0 10px #f59e0b; /* Tailwind amber-500 */
        }
        button.active-mode:hover, button.toggle-active:hover {
             background-color: #b45309; /* Tailwind amber-700 */
        }
        .tower-button-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
        }
        .tower-button {
            width: 70px; /* Adjusted size */
            height: 85px; /* Adjusted height */
            margin: 3px; /* Adjusted margin */
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            border: 2px solid #64748b;
            font-size: 0.65em; /* Adjusted font size */
        }
        .tower-button.selected {
            border-color: #f59e0b;
            box-shadow: 0 0 10px #f59e0b;
        }
        .tower-icon {
            width: 26px; /* Adjusted size */
            height: 26px; /* Adjusted size */
            border-radius: 4px;
            margin-bottom: 3px;
        }
        .tower-icon.archer { background-color: #22c55e; }
        .tower-icon.cannon { background-color: #ef4444; }
        .tower-icon.frost { background-color: #7dd3fc; }
        .tower-icon.bomb { background-color: #4b5563; }
        .tower-icon.sniper { background-color: #a855f7; }

        #messageBox, #toastNotification {
            position: fixed;
            left: 50%;
            transform: translateX(-50%);
            background-color: rgba(30, 41, 59, 0.95); color: #e2e8f0;
            padding: 20px; border-radius: 10px; border: 2px solid #64748b;
            text-align: center; z-index: 1000; box-shadow: 0 0 25px rgba(0,0,0,0.7);
        }
        #messageBox { top: 50%; transform: translate(-50%, -50%); font-size: 1.3em; display: none; }
        #messageBox h2 { font-size: 1.6em; margin-bottom: 12px; color: #f59e0b; }
        #messageBox button { margin-top: 15px; padding: 10px 20px; font-size: 0.9em; }

        #toastNotification {
            bottom: 30px;
            font-size: 1em;
            padding: 15px 25px;
            display: none;
            opacity: 0; /* Start hidden for transition */
            transition: opacity 0.5s ease-out;
        }
        #toastNotification.show {
            display: block; /* Needs to be block to be visible */
            opacity: 1;
        }
        #toastNotification.hide {
            opacity: 0;
        }


        @media (max-width: 768px) {
            .game-controls { flex-direction: column; max-width: 95%; }
            .control-panel-section { width: 90%; margin-bottom: 10px; }
            #gameCanvas { width: 95vw; height: calc(95vw * 0.75); }
            #messageBox { width: 80%; font-size: 1.1em; }
            #messageBox h2 { font-size: 1.4em; }
        }
         @media (max-width: 480px) {
            body { padding-top: 10px; }
            h1 { font-size: 1.5rem; } /* Smaller H1 */
            .control-panel-section { min-width: 100%; margin-left:0; margin-right:0;}
            .control-panel-section h3 { font-size: 0.85em; }
            .control-panel-section p, .control-panel-section button { font-size: 0.7em; }
            button { padding: 6px 8px; }
            .tower-button { width: 60px; height: 75px; }
            .tower-icon { width: 20px; height: 20px; }
            #gameCanvas { height: calc(95vw * 1); } /* More square for small portrait */
            #messageBox { font-size: 1em; padding: 15px; }
            #messageBox h2 { font-size: 1.2em; }
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1 class="text-xl sm:text-3xl font-bold my-2 sm:my-4 text-center">Pixel Defender: Advanced</h1>
        <canvas id="gameCanvas"></canvas>

        <div class="game-controls">
            <div class="control-panel-section">
                <h3>Game Stats</h3>
                <p>Wave: <span id="waveNumber">0</span></p>
                <p>Money: $<span id="money">100</span></p>
                <p>Lives: <span id="lives">20</span></p>
                <p>Score: <span id="score">0</span></p>
                <p class="mt-1">UID: <span id="userIdDisplay" class="text-xs"></span></p>
            </div>

            <div class="control-panel-section">
                <h3>Towers</h3>
                <div class="tower-button-container">
                    <button id="buyArcherTower" class="tower-button">
                        <div class="tower-icon archer"></div><span>Archer</span><span>$50</span>
                    </button>
                    <button id="buyCannonTower" class="tower-button">
                        <div class="tower-icon cannon"></div><span>Cannon</span><span>$100</span>
                    </button>
                    <button id="buyFrostTower" class="tower-button">
                        <div class="tower-icon frost"></div><span>Frost</span><span>$75</span>
                    </button>
                    <button id="buyBombTower" class="tower-button">
                        <div class="tower-icon bomb"></div><span>Bomb</span><span>$125</span>
                    </button>
                    <button id="buySniperTower" class="tower-button">
                        <div class="tower-icon sniper"></div><span>Sniper</span><span>$150</span>
                    </button>
                </div>
            </div>

            <div class="control-panel-section">
                <h3>Game Actions</h3>
                <button id="startWaveButton">Start Wave</button>
                <button id="sellTowerButton" class="ml-1">Sell Tower (70%)</button>
                <button id="pauseButton" class="ml-1">Pause</button>
                <button id="saveGameButton" class="mt-2">Save Game</button>
                <button id="loadGameButton" class="ml-1 mt-2">Load Game</button>
            </div>
        </div>
    </div>

    <div id="messageBox">
        <h2 id="messageTitle">Game Over!</h2>
        <p id="messageText">Your final score is 0.</p>
        <button id="messageButton">Play Again</button>
    </div>
    <div id="toastNotification">Toast message here</div>

    <script type="module">
        // Firebase Imports
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        // import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js"; // For debugging

        // --- Game Setup ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // DOM Elements
        const waveNumberEl = document.getElementById('waveNumber');
        const moneyEl = document.getElementById('money');
        const livesEl = document.getElementById('lives');
        const scoreEl = document.getElementById('score');
        const startWaveButton = document.getElementById('startWaveButton');
        const pauseButton = document.getElementById('pauseButton');
        const sellTowerButton = document.getElementById('sellTowerButton');
        // const autoWaveButton = document.getElementById('autoWaveButton'); // Removed
        const saveGameButton = document.getElementById('saveGameButton');
        const loadGameButton = document.getElementById('loadGameButton');
        const userIdDisplay = document.getElementById('userIdDisplay');
        
        const towerButtons = {
            archer: document.getElementById('buyArcherTower'),
            cannon: document.getElementById('buyCannonTower'),
            frost: document.getElementById('buyFrostTower'),
            bomb: document.getElementById('buyBombTower'),
            sniper: document.getElementById('buySniperTower'),
        };

        const messageBox = document.getElementById('messageBox');
        const messageTitleEl = document.getElementById('messageTitle');
        const messageTextEl = document.getElementById('messageText');
        const messageButton = document.getElementById('messageButton');
        const toastNotification = document.getElementById('toastNotification');

        let canvasWidth = 800;
        let canvasHeight = 600;
        const tileSize = 40; 
        const SELL_REFUND_PERCENTAGE = 0.7;

        // --- Game State Variables ---
        let money = 150; 
        let lives = 20;
        let score = 0;
        let waveNumber = 0;
        let enemies = [];
        let towers = [];
        let projectiles = [];
        let selectedTowerType = null;
        let placingTower = false;
        let sellingTowerMode = false; 
        let gamePaused = false;
        let waveInProgress = false;
        let gameOver = false;
        // let autoWaveActive = false; // Removed
        let gameInitialized = false; 

        // Firebase State
        let db, auth, userId, app;
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id'; 
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null; 

        // --- Path Definition ---
        const pathCoordinates = [ 
            { x: 0, y: 5 }, { x: 3, y: 5 }, { x: 3, y: 2 }, { x: 7, y: 2 },
            { x: 7, y: 8 }, { x: 12, y: 8 }, { x: 12, y: 4 }, { x: 16, y: 4 },
            { x: 16, y: 10 }, { x: 19, y: 10 } 
        ];
        let path = []; 

        function calculatePath() {
            path = pathCoordinates.map(p => ({ x: p.x * tileSize + tileSize / 2, y: p.y * tileSize + tileSize / 2 }));
        }

        const towerTypes = {
            archer: { name: 'Archer', cost: 50, damage: 15, range: 100, fireRate: 1, projectileSpeed: 5, color: '#22c55e', projectileColor: '#a3e635', size: tileSize * 0.8 },
            cannon: { name: 'Cannon', cost: 100, damage: 40, range: 120, fireRate: 0.5, projectileSpeed: 3, color: '#ef4444', projectileColor: '#f97316', size: tileSize * 0.9 },
            frost:  { name: 'Frost', cost: 75, damage: 5, range: 90, fireRate: 1.5, projectileSpeed: 4, color: '#7dd3fc', projectileColor: '#e0f2fe', size: tileSize * 0.75, slowAmount: 0.4, slowDuration: 2000 },
            bomb:   { name: 'Bomb', cost: 125, damage: 30, range: 110, fireRate: 0.3, projectileSpeed: 2, color: '#4b5563', projectileColor: '#fb923c', size: tileSize * 0.85, splashDamage: 15, splashRadius: tileSize * 1.2 },
            sniper: { name: 'Sniper', cost: 150, damage: 100, range: 200, fireRate: 0.2, projectileSpeed: 10, color: '#a855f7', projectileColor: '#fde047', size: tileSize * 0.7 }
        };

        const enemyTypes = {
            basic: { name: 'Basic', health: 50, speed: 1, color: '#facc15', size: tileSize * 0.6, reward: 10 }, 
            strong: { name: 'Strong', health: 120, speed: 0.7, color: '#60a5fa', size: tileSize * 0.7, reward: 30 }, 
            fast: { name: 'Fast', health: 70, speed: 1.8, color: '#f472b6', size: tileSize * 0.5, reward: 70 },
            boss: { name: 'Mega Tank', health: 500, speed: 0.5, color: '#7c3aed', size: tileSize * 1.5, reward: 500}
        };
        
        const initialArcherPositions = [ 
            { x: 1, y: 1 }, { x: 1, y: 13 }, { x: 5, y: 0 }, 
            { x: 9, y: 6 }, { x: 14, y: 1 }
        ];

        // --- Utility Functions ---
        function showToast(message, duration = 3000) {
            toastNotification.textContent = message;
            toastNotification.style.display = 'block';
            toastNotification.classList.remove('hide');
            toastNotification.classList.add('show');

            if (toastNotification.hideTimeout) clearTimeout(toastNotification.hideTimeout);
            if (toastNotification.displayNoneTimeout) clearTimeout(toastNotification.displayNoneTimeout);

            toastNotification.hideTimeout = setTimeout(() => {
                toastNotification.classList.remove('show');
                toastNotification.classList.add('hide');
                toastNotification.displayNoneTimeout = setTimeout(() => {
                    if (toastNotification.classList.contains('hide')) { 
                        toastNotification.style.display = 'none';
                    }
                }, 500); 
            }, duration);
        }

        function drawGrid() { 
             ctx.strokeStyle = '#475569'; 
            for (let i = 0; i <= canvasWidth / tileSize; i++) {
                ctx.beginPath(); ctx.moveTo(i * tileSize, 0); ctx.lineTo(i * tileSize, canvasHeight); ctx.stroke();
            }
            for (let j = 0; j <= canvasHeight / tileSize; j++) {
                ctx.beginPath(); ctx.moveTo(0, j * tileSize); ctx.lineTo(canvasWidth, j * tileSize); ctx.stroke();
            }
        }
        function drawPath() { 
            if (path.length === 0) calculatePath(); 
            ctx.strokeStyle = '#64748b'; 
            ctx.lineWidth = tileSize * 0.8;
            ctx.beginPath();
            ctx.moveTo(path[0].x, path[0].y);
            for (let i = 1; i < path.length; i++) {
                ctx.lineTo(path[i].x, path[i].y);
            }
            ctx.stroke();
            ctx.lineWidth = 1;
        }

        function updateUI() {
            waveNumberEl.textContent = waveNumber;
            moneyEl.textContent = money;
            livesEl.textContent = lives;
            scoreEl.textContent = score;
            startWaveButton.disabled = waveInProgress || gameOver || placingTower || sellingTowerMode;
            // startWaveButton.textContent = "Start Wave"; // No longer changes for auto wave

            for (const type in towerButtons) {
                towerButtons[type].disabled = money < towerTypes[type].cost || placingTower || gameOver || sellingTowerMode;
                towerButtons[type].classList.toggle('selected', selectedTowerType === type && placingTower);
            }
            sellTowerButton.classList.toggle('active-mode', sellingTowerMode);
            sellTowerButton.disabled = gameOver || placingTower;
            // autoWaveButton related lines removed
            canvas.classList.toggle('sell-mode-active', sellingTowerMode);
            userIdDisplay.textContent = userId ? userId.substring(0, 8) + "..." : "N/A";
            saveGameButton.disabled = !userId || gameOver; 
            loadGameButton.disabled = !userId || waveInProgress; 
        }

        function showMessage(title, text, buttonText = "Play Again", onButtonClick = ()=>{resetGame(true);}) {
            gamePaused = true; 
            if (animationFrameId) cancelAnimationFrame(animationFrameId); 
            messageTitleEl.textContent = title;
            messageTextEl.textContent = text;
            messageButton.textContent = buttonText;
            messageBox.style.display = 'block';
            messageButton.onclick = () => {
                messageBox.style.display = 'none';
                gamePaused = false; 
                if (onButtonClick) onButtonClick();
                if (!gameOver && !animationFrameId && onButtonClick !== resetGame) { 
                    lastTime = performance.now();
                    animationFrameId = requestAnimationFrame(gameLoop);
                }
            };
        }
        
        function resizeCanvas() { 
            const controlsHeight = document.querySelector('.game-controls').offsetHeight;
            const headerHeight = document.querySelector('h1').offsetHeight;
            const verticalPadding = 80; 
            const availableHeight = window.innerHeight - controlsHeight - headerHeight - verticalPadding;
            const availableWidth = window.innerWidth * 0.95; 

            let newWidth = Math.min(availableWidth, 900); 
            let newHeight = newWidth * (3/4); 

            if (newHeight > availableHeight) {
                newHeight = availableHeight;
                newWidth = newHeight * (4/3);
            }
            
            newWidth = Math.max(newWidth, 320); 
            newHeight = Math.max(newHeight, 240);

            canvas.width = Math.floor(newWidth / tileSize) * tileSize; 
            canvas.height = Math.floor(newHeight / tileSize) * tileSize; 
            canvasWidth = canvas.width;
            canvasHeight = canvas.height;

            calculatePath(); 

            if (gameInitialized && !gamePaused && !gameOver) { 
                 drawGame();
            } else if (gameInitialized && (gameOver || gamePaused)) { 
                 drawGame();
            }
        }

        // --- Enemy Logic ---
        class Enemy { 
            constructor(type) {
                this.x = path[0].x; this.y = path[0].y; this.pathIndex = 0; this.type = type;
                this.baseHealth = enemyTypes[type].health * (1 + (waveNumber * 0.12)); 
                this.health = this.baseHealth; this.maxHealth = this.baseHealth;
                this.baseSpeed = enemyTypes[type].speed * (1 + (waveNumber * 0.025)); 
                this.currentSpeed = this.baseSpeed;
                this.slowTimer = 0; this.slowAmount = 0;
                this.color = enemyTypes[type].color; this.size = enemyTypes[type].size; this.reward = enemyTypes[type].reward;
            }
            move(deltaTime) {
                if (this.pathIndex >= path.length - 1) {
                    lives--;
                    if (lives <= 0) { lives = 0; gameOver = true; showMessage("Game Over!", `Your final score is ${score}.`);}
                    return true; 
                }
                if (this.slowTimer > 0) {
                    this.slowTimer -= deltaTime;
                    if (this.slowTimer <= 0) { this.currentSpeed = this.baseSpeed; this.slowAmount = 0; }
                    else { this.currentSpeed = this.baseSpeed * (1 - this.slowAmount); }
                }
                const targetNode = path[this.pathIndex + 1];
                const dx = targetNode.x - this.x; const dy = targetNode.y - this.y;
                const distance = Math.sqrt(dx * dx + dy * dy);
                const effectiveSpeed = this.currentSpeed * (deltaTime / (1000/60));
                if (distance < effectiveSpeed) { this.x = targetNode.x; this.y = targetNode.y; this.pathIndex++;}
                else { this.x += (dx / distance) * effectiveSpeed; this.y += (dy / distance) * effectiveSpeed; }
                return false;
            }
            draw() {
                ctx.fillStyle = this.color; ctx.beginPath(); ctx.arc(this.x, this.y, this.size / 2, 0, Math.PI * 2); ctx.fill();
                if (this.slowTimer > 0) {
                    ctx.strokeStyle = towerTypes.frost.projectileColor; ctx.lineWidth = 2;
                    ctx.beginPath(); ctx.arc(this.x, this.y, this.size / 2 + 2, 0, Math.PI * 2); ctx.stroke(); ctx.lineWidth = 1;
                }
                const healthBarWidth = this.size; const healthBarHeight = (this.type === 'boss') ? 8 : 5;
                const healthPercentage = this.health / this.maxHealth;
                ctx.fillStyle = '#dc2626'; ctx.fillRect(this.x - healthBarWidth / 2, this.y - this.size / 2 - healthBarHeight - 3, healthBarWidth, healthBarHeight);
                ctx.fillStyle = '#16a34a'; ctx.fillRect(this.x - healthBarWidth / 2, this.y - this.size / 2 - healthBarHeight - 3, healthBarWidth * healthPercentage, healthBarHeight);
            }
            takeDamage(amount, slowInfo = null) {
                this.health -= amount;
                if (slowInfo && slowInfo.amount > 0 && slowInfo.duration > 0) {
                    this.slowTimer = slowInfo.duration; this.slowAmount = slowInfo.amount; 
                    this.currentSpeed = this.baseSpeed * (1 - this.slowAmount);
                }
                if (this.health <= 0) { money += this.reward; score += this.reward * 2; return true; }
                return false;
            }
        }
        
        function spawnEnemies() { 
            waveInProgress = true; 
            startWaveButton.disabled = true;

            if (waveNumber > 0 && waveNumber % 10 === 0) { 
                setTimeout(() => { 
                    if (gameOver) return;
                    enemies.push(new Enemy('boss')); 
                    console.log(`Wave ${waveNumber}: BOSS INCOMING - ${enemyTypes.boss.name}!`);
                    showToast(`Wave ${waveNumber}: BOSS INCOMING!`, 3000);
                }, 500);
            } else { 
                const numEnemies = 3 + Math.floor(waveNumber * 2.2) + Math.floor(waveNumber / 4) * 2;
                let enemyTypeToSpawn = 'basic';
                
                for (let i = 0; i < numEnemies; i++) {
                    setTimeout(() => {
                        if (gameOver) return;
                        if (waveNumber > 2 && i % Math.max(1, (7 - Math.floor(waveNumber/3))) === 0) { 
                            enemyTypeToSpawn = 'strong'; 
                        } else if (waveNumber > 4 && i % Math.max(1, (9 - Math.floor(waveNumber/3))) === 0) {
                            enemyTypeToSpawn = 'fast';
                        } else {
                            enemyTypeToSpawn = 'basic';
                        }
                        enemies.push(new Enemy(enemyTypeToSpawn));
                    }, i * (900 - Math.min(waveNumber * 35, 650))); 
                }
            }
        }


        // --- Tower Logic ---
        class Tower { 
            constructor(x, y, type) {
                this.x = x; this.y = y; this.type = type;
                this.stats = { ...towerTypes[type] }; 
                this.originalCost = towerTypes[type].cost; 
                this.size = this.stats.size;
                this.lastShotTime = 0; this.target = null;
            }
            draw() {
                ctx.fillStyle = this.stats.color;
                ctx.beginPath();
                ctx.fillRect(this.x - this.size / 2, this.y - this.size / 2, this.size, this.size);
                
                const mouseOverThisTower = Math.floor(mousePos.x / tileSize) === Math.floor(this.x / tileSize) &&
                                          Math.floor(mousePos.y / tileSize) === Math.floor(this.y / tileSize);

                if ((placingTower && selectedTowerType === this.type && mouseOverThisTower) ||
                    (sellingTowerMode && mouseOverThisTower)) {
                    ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
                    ctx.beginPath(); ctx.arc(this.x, this.y, this.stats.range, 0, Math.PI * 2); ctx.stroke();
                    if (sellingTowerMode && mouseOverThisTower) { 
                        ctx.fillStyle = 'rgba(255, 0, 0, 0.2)'; 
                        ctx.fillRect(Math.floor(this.x/tileSize)*tileSize, Math.floor(this.y/tileSize)*tileSize, tileSize, tileSize);
                    }
                }
            }
            findTarget() { 
                this.target = null; let closestDistance = this.stats.range + 1;
                for (const enemy of enemies) {
                    const dx = enemy.x - this.x; const dy = enemy.y - this.y;
                    const distance = Math.sqrt(dx * dx + dy * dy);
                    if (distance <= this.stats.range && distance < closestDistance) { 
                        closestDistance = distance; this.target = enemy;
                    }
                }
            }
            shoot() { 
                if (!this.target || this.target.health <= 0 || 
                    Math.sqrt((this.target.x-this.x)**2 + (this.target.y-this.y)**2) > this.stats.range) { 
                    this.findTarget(); if (!this.target) return; 
                }
                const now = performance.now(); 
                if (now - this.lastShotTime > 1000 / this.stats.fireRate) {
                    projectiles.push(new Projectile(this.x, this.y, this.target, this.stats, this.type)); 
                    this.lastShotTime = now;
                }
            }
        }

        // --- Projectile Logic ---
        class Projectile { 
            constructor(x, y, target, towerStats, towerType) { 
                this.x = x; this.y = y; this.target = target;
                this.speed = towerStats.projectileSpeed;
                this.damage = towerStats.damage;
                this.color = towerStats.projectileColor;
                this.size = towerType === 'sniper' ? 7 : 5; 
                this.towerStats = towerStats; 
            }
            move(deltaTime) { 
                if (!this.target || this.target.health <= 0) return true; 
                const dx = this.target.x - this.x; const dy = this.target.y - this.y;
                const distance = Math.sqrt(dx * dx + dy * dy);
                const effectiveSpeed = this.speed * (deltaTime / (1000/60));

                if (distance < effectiveSpeed) { 
                    let slowInfo = null;
                    if (this.towerStats.slowAmount && this.towerStats.slowDuration) {
                        slowInfo = { amount: this.towerStats.slowAmount, duration: this.towerStats.slowDuration };
                    }
                    const enemyDied = this.target.takeDamage(this.damage, slowInfo);

                    if (this.towerStats.splashRadius > 0 && this.towerStats.splashDamage > 0) {
                        enemies.forEach(enemy => {
                            if (enemy !== this.target && enemy.health > 0) {
                                const dxSplash = enemy.x - this.target.x; 
                                const dySplash = enemy.y - this.target.y;
                                const distSplash = Math.sqrt(dxSplash*dxSplash + dySplash*dySplash);
                                if (distSplash <= this.towerStats.splashRadius) {
                                    if(enemy.takeDamage(this.towerStats.splashDamage)) { 
                                        const splashEnemyIndex = enemies.indexOf(enemy);
                                        if (splashEnemyIndex > -1) enemies.splice(splashEnemyIndex, 1);
                                    }
                                }
                            }
                        });
                    }
                    if (enemyDied) {
                        const mainEnemyIndex = enemies.indexOf(this.target);
                        if (mainEnemyIndex > -1) enemies.splice(mainEnemyIndex, 1);
                    }
                    return true; 
                } else {
                    this.x += (dx / distance) * effectiveSpeed;
                    this.y += (dy / distance) * effectiveSpeed;
                }
                return false;
            }
            draw() { 
                ctx.fillStyle = this.color;
                ctx.beginPath(); ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2); ctx.fill();
            }
        }

        // --- Game Loop ---
        let lastTime = 0;
        function gameLoop(timestamp) { 
            if (gameOver) return; 
            if (!lastTime) lastTime = timestamp; 
            const deltaTime = timestamp - lastTime;
            lastTime = timestamp;

            if (gamePaused) {
                requestAnimationFrame(gameLoop); 
                return;
            }

            ctx.clearRect(0, 0, canvasWidth, canvasHeight);
            drawGrid(); drawPath();

            towers.forEach(tower => { tower.draw(); tower.shoot(); });
            
            for (let i = projectiles.length - 1; i >= 0; i--) {
                projectiles[i].draw();
                if (projectiles[i].move(deltaTime)) projectiles.splice(i, 1);
            }
            
            for (let i = enemies.length - 1; i >= 0; i--) {
                enemies[i].draw();
                if (enemies[i].move(deltaTime)) enemies.splice(i, 1);
            }
            
            if (waveInProgress && enemies.length === 0 && waveNumber > 0) {
                waveInProgress = false; 
                // Auto wave logic removed
            }

            if (placingTower && selectedTowerType) drawPlacementPreview();
            updateUI();
            animationFrameId = requestAnimationFrame(gameLoop); 
        }

        function drawGame() { 
            ctx.clearRect(0, 0, canvasWidth, canvasHeight);
            drawGrid(); drawPath();
            towers.forEach(t => t.draw());
            projectiles.forEach(p => p.draw());
            enemies.forEach(e => e.draw());
            if (placingTower && selectedTowerType) drawPlacementPreview();
            updateUI();
        }

        // --- Event Handlers ---
        let mousePos = { x: 0, y: 0, tileX: 0, tileY: 0 };
        canvas.addEventListener('mousemove', (e) => { 
            const rect = canvas.getBoundingClientRect();
            mousePos.x = e.clientX - rect.left; mousePos.y = e.clientY - rect.top;
            mousePos.tileX = Math.floor(mousePos.x / tileSize); mousePos.tileY = Math.floor(mousePos.y / tileSize);
        });

        canvas.addEventListener('click', () => { 
            if (gameOver || gamePaused) return;

            if (sellingTowerMode) {
                const tileX = mousePos.tileX;
                const tileY = mousePos.tileY;
                for (let i = towers.length - 1; i >= 0; i--) {
                    const tower = towers[i];
                    if (Math.floor(tower.x / tileSize) === tileX && Math.floor(tower.y / tileSize) === tileY) {
                        money += Math.floor(tower.originalCost * SELL_REFUND_PERCENTAGE);
                        towers.splice(i, 1);
                        break; 
                    }
                }
            } else if (placingTower && selectedTowerType) {
                const towerCost = towerTypes[selectedTowerType].cost;
                if (money >= towerCost) {
                    const tileX = mousePos.tileX; const tileY = mousePos.tileY;
                    const onPath = pathCoordinates.some(p => p.x === tileX && p.y === tileY);
                    const occupied = towers.some(t => Math.floor(t.x / tileSize) === tileX && Math.floor(t.y / tileSize) === tileY);
                    const inBounds = tileX >= 0 && tileX < canvasWidth/tileSize && tileY >=0 && tileY < canvasHeight/tileSize;

                    if (!onPath && !occupied && inBounds) {
                        towers.push(new Tower(tileX * tileSize + tileSize / 2, tileY * tileSize + tileSize / 2, selectedTowerType));
                        money -= towerCost;
                    } else { console.log("Cannot place tower: Invalid location."); }
                } else { 
                    console.log("Not enough money."); 
                    placingTower = false; selectedTowerType = null; 
                }
            } else { 
                selectedTowerType = null; placingTower = false; 
            }
            updateUI();
        });
        
        document.addEventListener('click', (e) => { 
            if (placingTower && e.target !== canvas && !e.target.closest('.tower-button') && !e.target.closest('#sellTowerButton')) {
                selectedTowerType = null; placingTower = false; updateUI();
            }
            if (sellingTowerMode && e.target !== canvas && e.target !== sellTowerButton && !e.target.closest('.tower-button')) {
                sellingTowerMode = false; updateUI();
            }
        });
        document.addEventListener('keydown', (e) => { 
            if (e.key === "Escape") {
                if (placingTower) { selectedTowerType = null; placingTower = false; }
                if (sellingTowerMode) { sellingTowerMode = false; }
                updateUI();
            }
        });

        function drawPlacementPreview() { 
            if (!selectedTowerType) return;
            const tileX = mousePos.tileX; const tileY = mousePos.tileY;
            const previewX = tileX * tileSize + tileSize / 2; const previewY = tileY * tileSize + tileSize / 2;
            const towerStats = towerTypes[selectedTowerType];

            const onPath = pathCoordinates.some(p => p.x === tileX && p.y === tileY);
            const occupied = towers.some(t => Math.floor(t.x / tileSize) === tileX && Math.floor(t.y / tileSize) === tileY);
            const canAfford = money >= towerStats.cost;
            const inBounds = tileX >= 0 && tileX < canvasWidth/tileSize && tileY >=0 && tileY < canvasHeight/tileSize;

            let previewColor = 'rgba(0, 255, 0, 0.3)'; 
            if (onPath || occupied || !canAfford || !inBounds) previewColor = 'rgba(255, 0, 0, 0.3)'; 
            
            ctx.fillStyle = previewColor;
            ctx.fillRect(tileX * tileSize, tileY * tileSize, tileSize, tileSize);
            ctx.strokeStyle = 'rgba(255, 255, 255, 0.5)';
            ctx.beginPath(); ctx.arc(previewX, previewY, towerStats.range, 0, Math.PI * 2); ctx.stroke();
        }

        for (const type in towerButtons) { 
            towerButtons[type].addEventListener('click', () => {
                if (sellingTowerMode) sellingTowerMode = false; 
                if (money >= towerTypes[type].cost && !gameOver) {
                    if (selectedTowerType === type && placingTower) { 
                        selectedTowerType = null; placingTower = false;
                    } else {
                        selectedTowerType = type; placingTower = true;
                    }
                } else if (money < towerTypes[type].cost) {
                    console.log(`Not enough money for ${towerTypes[type].name} tower.`);
                    selectedTowerType = null; placingTower = false;
                }
                updateUI();
            });
        }

        sellTowerButton.addEventListener('click', () => { 
            sellingTowerMode = !sellingTowerMode;
            if (sellingTowerMode) { 
                placingTower = false;
                selectedTowerType = null;
            }
            updateUI();
        });
        
        function handleStartWave() {
            if (!waveInProgress && !gameOver && !placingTower && !sellingTowerMode) {
                waveNumber++; 
                if (waveNumber > 0 && waveNumber % 10 === 0) {
                    showToast(`BOSS WAVE ${waveNumber} INCOMING!`, 3000);
                } else {
                    showToast(`Wave ${waveNumber} starting!`, 1500);
                }
                spawnEnemies(); 
                updateUI();
            }
        }
        startWaveButton.addEventListener('click', handleStartWave);

        // Auto Wave button listener removed
        
        pauseButton.addEventListener('click', () => { 
            gamePaused = !gamePaused;
            pauseButton.textContent = gamePaused ? "Resume" : "Pause";
            if (!gamePaused && !gameOver) { 
                lastTime = performance.now(); 
                if (animationFrameId) cancelAnimationFrame(animationFrameId); 
                animationFrameId = requestAnimationFrame(gameLoop);
                messageBox.style.display = 'none'; 
            } else if (gamePaused) {
                if (animationFrameId) cancelAnimationFrame(animationFrameId); 
                showMessage("Paused", "Game is paused. Press Resume to continue.", "Resume", () => {
                    gamePaused = false; pauseButton.textContent = "Pause";
                    lastTime = performance.now(); 
                    if (animationFrameId) cancelAnimationFrame(animationFrameId);
                    animationFrameId = requestAnimationFrame(gameLoop);
                });
            }
        });

        // --- Firebase Save/Load ---
        async function saveGame() {
            if (!userId || !db) {
                showToast("Error: Not signed in or DB not ready.", 3000);
                console.error("Save Error: User ID or DB not available.");
                return;
            }
            if (gameOver) {
                showToast("Cannot save: Game Over.", 3000);
                return;
            }

            const towersData = towers.map(tower => ({ x: tower.x, y: tower.y, type: tower.type }));
            const gameState = {
                money, lives, score, waveNumber,
                towers: towersData,
                // autoWaveActive removed from save data
                timestamp: serverTimestamp() 
            };

            try {
                const saveDocRef = doc(db, `artifacts/${appId}/users/${userId}/towerDefenseSave/gameState`);
                await setDoc(saveDocRef, gameState);
                showToast("Game Saved!", 2000);
                console.log("Game saved successfully to Firestore.");
            } catch (error) {
                console.error("Error saving game to Firestore:", error);
                showToast("Error saving game!", 3000);
            }
        }

        async function loadGame() {
            if (!userId || !db) {
                showToast("Error: Not signed in or DB not ready.", 3000);
                console.error("Load Error: User ID or DB not available.");
                return;
            }
            if (waveInProgress) {
                showToast("Cannot load during a wave.", 3000);
                return;
            }

            try {
                const saveDocRef = doc(db, `artifacts/${appId}/users/${userId}/towerDefenseSave/gameState`);
                const docSnap = await getDoc(saveDocRef);

                if (docSnap.exists()) {
                    const loadedState = docSnap.data();
                    
                    resetGame(false); 

                    money = loadedState.money;
                    lives = loadedState.lives;
                    score = loadedState.score;
                    waveNumber = loadedState.waveNumber;
                    // autoWaveActive removed from load data

                    towers = []; 
                    loadedState.towers.forEach(towerData => {
                        towers.push(new Tower(towerData.x, towerData.y, towerData.type));
                    });
                    
                    enemies = []; 
                    projectiles = []; 
                    waveInProgress = false; 
                    gameOver = false; 
                    gamePaused = false; 

                    showToast("Game Loaded!", 2000);
                    console.log("Game loaded successfully from Firestore.");
                } else {
                    showToast("No saved game found.", 3000);
                    console.log("No saved game document found.");
                }
            } catch (error) {
                console.error("Error loading game from Firestore:", error);
                showToast("Error loading game!", 3000);
            } finally {
                updateUI(); 
                drawGame(); 
            }
        }

        saveGameButton.addEventListener('click', saveGame);
        loadGameButton.addEventListener('click', loadGame);


        // --- Game Initialization ---
        function placeInitialTowers() { 
            initialArcherPositions.forEach(pos => {
                if (pos.x < canvasWidth / tileSize && pos.y < canvasHeight / tileSize) {
                    const onPath = pathCoordinates.some(p => p.x === pos.x && p.y === pos.y);
                    const occupied = towers.some(t => Math.floor(t.x / tileSize) === pos.x && Math.floor(t.y / tileSize) === pos.y);
                    if (!onPath && !occupied) {
                         towers.push(new Tower(pos.x * tileSize + tileSize / 2, pos.y * tileSize + tileSize / 2, 'archer'));
                    } else { console.warn(`Could not place initial archer at tile (${pos.x}, ${pos.y}) - invalid.`); }
                } else { console.warn(`Could not place initial archer at tile (${pos.x}, ${pos.y}) - out of bounds.`); }
            });
        }
        
        function resetGame(placeStarters = true) { 
            console.log(`Resetting game. Place starters: ${placeStarters}`);
            money = 150; lives = 20; score = 0; waveNumber = 0;
            enemies = []; 
            towers = []; 
            projectiles = [];
            selectedTowerType = null; placingTower = false; sellingTowerMode = false;
            gamePaused = false; waveInProgress = false; gameOver = false;
            pauseButton.textContent = "Pause";
            
            if(placeStarters) {
                placeInitialTowers(); 
            }

            updateUI();
            messageBox.style.display = 'none';
            lastTime = performance.now(); 
            if (animationFrameId) cancelAnimationFrame(animationFrameId); 
            animationFrameId = requestAnimationFrame(gameLoop);
            console.log("Game Initialized and Loop Started after reset.");
        }
        
        let animationFrameId; 

        async function initFirebaseAndGame() {
            if (!firebaseConfig) {
                console.error("Firebase config is not available. Save/Load will not work.");
                showToast("Firebase not configured. Save/Load disabled.", 5000);
                initGame(); 
                return;
            }
            console.log("Initializing Firebase...");
            try {
                app = initializeApp(firebaseConfig);
                auth = getAuth(app);
                db = getFirestore(app);
                console.log("Firebase Initialized. Setting up Auth state listener.");
                // setLogLevel('debug'); 

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        console.log("User signed in with UID:", userId);
                    } else {
                        userId = null;
                        console.log("User signed out or not signed in.");
                    }
                    if (!gameInitialized) {
                        initGame();
                    }
                    updateUI(); 
                });
                
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    console.log("Attempting sign in with custom token.");
                    try {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } catch (error) {
                        console.error("Error signing in with custom token, trying anonymous:", error);
                        await signInAnonymously(auth);
                    }
                } else {
                    console.log("No custom token, attempting anonymous sign in.");
                    await signInAnonymously(auth);
                }

            } catch (error) {
                console.error("Firebase initialization error:", error);
                showToast("Error initializing Firebase. Save/Load disabled.", 5000);
                initGame(); 
            }
        }

        function initGame() {
            if (gameInitialized) return; 
            console.log("Initializing Game (initGame function)...");
            resizeCanvas(); 
            resetGame(true); 
            gameInitialized = true;
            updateUI();
        }

        window.addEventListener('resize', resizeCanvas);
        document.addEventListener('DOMContentLoaded', initFirebaseAndGame);

    </script>
</body>
</html>
