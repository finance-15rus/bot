<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Маджонг</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r149/three.min.js"></script>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            overflow: hidden;
            touch-action: manipulation;
        }
        
        #gameContainer {
            position: relative;
            width: 100vw;
            height: 100vh;
        }
        
        #ui {
            position: absolute;
            top: 20px;
            left: 20px;
            right: 20px;
            z-index: 100;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 15px;
            padding: 15px;
            color: white;
            font-weight: 500;
        }
        
        #score {
            font-size: 18px;
        }
        
        #timer {
            font-size: 16px;
            color: #ffeb3b;
        }
        
        #controls {
            position: absolute;
            bottom: 20px;
            left: 20px;
            right: 20px;
            z-index: 100;
            display: flex;
            gap: 10px;
            justify-content: center;
        }
        
        .btn {
            background: rgba(255, 255, 255, 0.2);
            backdrop-filter: blur(10px);
            border: none;
            color: white;
            padding: 12px 20px;
            border-radius: 10px;
            font-size: 14px;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.3s ease;
            touch-action: manipulation;
        }
        
        .btn:hover, .btn:active {
            background: rgba(255, 255, 255, 0.3);
            transform: scale(1.05);
        }
        
        .btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
        }
        
        #gameOverModal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        
        #gameOverContent {
            background: white;
            padding: 30px;
            border-radius: 20px;
            text-align: center;
            max-width: 300px;
            margin: 20px;
        }
        
        #gameOverContent h2 {
            color: #333;
            margin-bottom: 15px;
            font-size: 24px;
        }
        
        #gameOverContent p {
            color: #666;
            margin-bottom: 20px;
            font-size: 16px;
        }
        
        #restartBtn {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 12px 30px;
            border-radius: 10px;
            font-size: 16px;
            cursor: pointer;
            transition: background 0.3s ease;
        }
        
        #restartBtn:hover {
            background: #45a049;
        }
        
        #loading {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.9);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 2000;
            color: white;
            font-size: 18px;
        }
        
        .spinner {
            border: 3px solid rgba(255, 255, 255, 0.3);
            border-top: 3px solid white;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin-right: 15px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        @media (max-width: 768px) {
            #ui {
                top: 10px;
                left: 10px;
                right: 10px;
                padding: 10px;
            }
            
            #controls {
                bottom: 10px;
                left: 10px;
                right: 10px;
            }
            
            .btn {
                padding: 10px 15px;
                font-size: 12px;
            }
        }
    </style>
</head>
<body>
    <div id="loading">
        <div class="spinner"></div>
        Загрузка игры...
    </div>
    
    <div id="gameContainer">
        <div id="ui">
            <div id="score">Очки: 0</div>
            <div id="timer">Время: 00:00</div>
        </div>
        
        <div id="controls">
            <button class="btn" id="rotateBtn">Повернуть</button>
            <button class="btn" id="hintBtn">Подсказка</button>
            <button class="btn" id="shuffleBtn">Перемешать</button>
            <button class="btn" id="newGameBtn">Новая игра</button>
        </div>
    </div>
    
    <div id="gameOverModal">
        <div id="gameOverContent">
            <h2 id="gameOverTitle">Игра окончена!</h2>
            <p id="gameOverMessage">Поздравляем! Вы набрали <span id="finalScore">0</span> очков за <span id="finalTime">00:00</span>!</p>
            <button id="restartBtn">Играть снова</button>
        </div>
    </div>

    <script>
        // Инициализация Telegram WebApp
        if (window.Telegram && window.Telegram.WebApp) {
            const tg = window.Telegram.WebApp;
            tg.ready();
            tg.expand();
            document.body.style.backgroundColor = tg.themeParams.bg_color || '#667eea';
        } else {
            console.warn("Telegram WebApp not detected, proceeding without it.");
        }

        // Глобальные переменные
        let scene, camera, renderer, raycaster, mouse;
        let tiles = [];
        let selectedTiles = [];
        let gameState = 'playing';
        let score = 0;
        let startTime = Date.now();
        let gameTimer;
        let hints = 3;
        let shuffles = 2;
        
        // Типы плиток маджонга
        const tileTypes = [
            '🀄', '🀅', '🀆', // Драконы
            '🀇', '🀈', '🀉', '🀊', '🀋', '🀌', '🀍', '🀎', '🀏', // Бамбук
            '🀐', '🀑', '🀒', '🀓', '🀔', '🀕', '🀖', '🀗', '🀘', // Точки
            '🀙', '🀚', '🀛', '🀜', '🀝', '🀞', '🀟', '🀠', '🀡'  // Иероглифы
        ];
        
        // Макет пирамиды
        const pyramidLayout = [
            [
                [1,1,1,1,1,1,1,1,1,1,1,1],
                [0,1,1,1,1,1,1,1,1,1,1,0],
                [1,1,1,1,1,1,1,1,1,1,1,1],
                [0,1,1,1,1,1,1,1,1,1,1,0],
                [1,1,1,1,1,1,1,1,1,1,1,1],
                [0,1,1,1,1,1,1,1,1,1,1,0],
                [1,1,1,1,1,1,1,1,1,1,1,1],
                [0,1,1,1,1,1,1,1,1,1,1,0]
            ],
            [
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,1,1,1,1,1,1,1,1,0,0],
                [0,0,1,1,1,1,1,1,1,1,0,0],
                [0,0,1,1,1,1,1,1,1,1,0,0],
                [0,0,1,1,1,1,1,1,1,1,0,0],
                [0,0,1,1,1,1,1,1,1,1,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0]
            ],
            [
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,1,1,1,1,1,1,0,0,0],
                [0,0,0,1,1,1,1,1,1,0,0,0],
                [0,0,0,1,1,1,1,1,1,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0]
            ],
            [
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,1,1,1,1,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0]
            ],
            [
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,1,1,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0]
            ]
        ];

        // Инициализация Three.js
        function initThree() {
            console.log("Initializing Three.js...");
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x001122);
            
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 15, 15);
            camera.lookAt(0, 0, 0);
            
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            
            document.getElementById('gameContainer').appendChild(renderer.domElement);
            
            // Освещение
            const ambientLight = new THREE.AmbientLight(0x404040, 0.4);
            scene.add(ambientLight);
            
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
            directionalLight.position.set(10, 10, 5);
            directionalLight.castShadow = true;
            directionalLight.shadow.mapSize.width = 2048;
            directionalLight.shadow.mapSize.height = 2048;
            scene.add(directionalLight);
            
            const pointLight = new THREE.PointLight(0xffffff, 0.5, 100);
            pointLight.position.set(0, 10, 10);
            scene.add(pointLight);
            
            // Raycaster для взаимодействия с мышью
            raycaster = new THREE.Raycaster();
            mouse = new THREE.Vector2();
            
            // Обработчики событий
            renderer.domElement.addEventListener('click', onTileClick);
            renderer.domElement.addEventListener('touchstart', onTileClick);
            
            window.addEventListener('resize', onWindowResize);
        }

        // Создание плитки
        function createTile(x, y, z, type) {
            const geometry = new THREE.BoxGeometry(1.8, 0.3, 1.2);
            const material = new THREE.MeshLambertMaterial({ 
                color: 0xffffff,
                transparent: true,
                opacity: 0.9
            });
            
            const tile = new THREE.Mesh(geometry, material);
            tile.position.set(x * 2 - 11, z * 0.4, y * 1.5 - 6);
            tile.castShadow = true;
            tile.receiveShadow = true;
            
            // Текст на плитке
            const canvas = document.createElement('canvas');
            canvas.width = 128;
            canvas.height = 128;
            const context = canvas.getContext('2d');
            context.font = '60px sans-serif';
            context.textAlign = 'center';
            context.textBaseline = 'middle';
            context.fillStyle = '#000';
            context.fillText(type || 'X', 64, 64); // Fallback на 'X'
            
            const texture = new THREE.CanvasTexture(canvas);
            const textMaterial = new THREE.MeshBasicMaterial({ 
                map: texture,
                transparent: true
            });
            
            const textGeometry = new THREE.PlaneGeometry(1.5, 1.5);
            const textMesh = new THREE.Mesh(textGeometry, textMaterial);
            textMesh.position.set(0, 0.16, 0);
            textMesh.rotation.x = -Math.PI / 2;
            
            tile.add(textMesh);
            
            tile.userData = {
                type: type,
                gridX: x,
                gridY: y,
                gridZ: z,
                blocked: false,
                selected: false
            };
            
            return tile;
        }

        // Создание игрового поля
        function createGameBoard() {
            console.log("Creating game board...");
            tiles = [];
            scene.children = scene.children.filter(child => !child.isMesh); // Очистка только плиток
            
            const ambientLight = new THREE.AmbientLight(0x404040, 0.4);
            scene.add(ambientLight);
            
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
            directionalLight.position.set(10, 10, 5);
            directionalLight.castShadow = true;
            directionalLight.shadow.mapSize.width = 2048;
            directionalLight.shadow.mapSize.height = 2048;
            scene.add(directionalLight);
            
            const pointLight = new THREE.PointLight(0xffffff, 0.5, 100);
            pointLight.position.set(0, 10, 10);
            scene.add(pointLight);
            
            const tileTypeArray = [];
            let totalTiles = 0;
            
            for (let z = 0; z < pyramidLayout.length; z++) {
                for (let y = 0; y < pyramidLayout[z].length; y++) {
                    for (let x = 0; x < pyramidLayout[z][y].length; x++) {
                        if (pyramidLayout[z][y][x] === 1) totalTiles++;
                    }
                }
            }
            console.log("Total tiles:", totalTiles);
            
            const typesNeeded = Math.ceil(totalTiles / 4);
            for (let i = 0; i < typesNeeded; i++) {
                const typeIndex = i % tileTypes.length;
                for (let j = 0; j < 4; j++) {
                    if (tileTypeArray.length < totalTiles) {
                        tileTypeArray.push(tileTypes[typeIndex]);
                    }
                }
            }
            
            for (let i = tileTypeArray.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [tileTypeArray[i], tileTypeArray[j]] = [tileTypeArray[j], tileTypeArray[i]];
            }
            
            let tileIndex = 0;
            for (let z = 0; z < pyramidLayout.length; z++) {
                for (let y = 0; y < pyramidLayout[z].length; y++) {
                    for (let x = 0; x < pyramidLayout[z][y].length; x++) {
                        if (pyramidLayout[z][y][x] === 1) {
                            const tile = createTile(x, y, z, tileTypeArray[tileIndex]);
                            scene.add(tile);
                            tiles.push(tile);
                            tileIndex++;
                        }
                    }
                }
            }
            
            updateBlockedTiles();
        }

        // Обновление заблокированных плиток
        function updateBlockedTiles() {
            console.log("Updating blocked tiles...");
            let blockedCount = 0;
            tiles.forEach(tile => {
                tile.userData.blocked = false;
                tile.material.opacity = 0.9;
            });
            
            tiles.forEach(tile => {
                const { gridX, gridY, gridZ } = tile.userData;
                const tileAbove = tiles.find(t => 
                    t.userData.gridX === gridX && 
                    t.userData.gridY === gridY && 
                    t.userData.gridZ === gridZ + 1
                );
                
                if (tileAbove) {
                    tile.userData.blocked = true;
                    tile.material.opacity = 0.5;
                    blockedCount++;
                    return;
                }
                
                let leftBlocked = false;
                let rightBlocked = false;
                
                for (let dx = -1; dx >= -2; dx--) {
                    const leftTile = tiles.find(t => 
                        t.userData.gridX === gridX + dx && 
                        t.userData.gridY === gridY && 
                        t.userData.gridZ === gridZ
                    );
                    if (leftTile) {
                        leftBlocked = true;
                        break;
                    }
                }
                
                for (let dx = 1; dx <= 2; dx++) {
                    const rightTile = tiles.find(t => 
                        t.userData.gridX === gridX + dx && 
                        t.userData.gridY === gridY && 
                        t.userData.gridZ === gridZ
                    );
                    if (rightTile) {
                        rightBlocked = true;
                        break;
                    }
                }
                
                if (leftBlocked && rightBlocked) {
                    tile.userData.blocked = true;
                    tile.material.opacity = 0.5;
                    blockedCount++;
                }
            });
            console.log("Blocked tiles:", blockedCount, "Total tiles:", tiles.length);
        }

        // Обработчик клика по плитке
        function onTileClick(event) {
            if (gameState !== 'playing') return;
            
            const rect = renderer.domElement.getBoundingClientRect();
            let clientX, clientY;
            
            if (event.touches) {
                clientX = event.touches[0].clientX;
                clientY = event.touches[0].clientY;
            } else {
                clientX = event.clientX;
                clientY = event.clientY;
            }
            
            mouse.x = ((clientX - rect.left) / rect.width) * 2 - 1;
            mouse.y = -((clientY - rect.top) / rect.height) * 2 + 1;
            
            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(tiles);
            
            if (intersects.length > 0) {
                const clickedTile = intersects[0].object;
                selectTile(clickedTile);
            }
        }

        // Выбор плитки
        function selectTile(tile) {
            if (tile.userData.blocked) return;
            
            if (tile.userData.selected) {
                tile.userData.selected = false;
                tile.material.color.setHex(0xffffff);
                selectedTiles = selectedTiles.filter(t => t !== tile);
                return;
            }
            
            if (selectedTiles.length < 2) {
                tile.userData.selected = true;
                tile.material.color.setHex(0x00ff00);
                selectedTiles.push(tile);
                
                if (selectedTiles.length === 2) {
                    setTimeout(checkMatch, 300);
                }
            }
        }

        // Проверка совпадения
        function checkMatch() {
            if (selectedTiles.length !== 2) return;
            
            const [tile1, tile2] = selectedTiles;
            
            if (tile1.userData.type === tile2.userData.type) {
                score += 10;
                updateScore();
                
                const tween1 = { scale: 1, opacity: 1 };
                const tween2 = { scale: 1, opacity: 1 };
                
                const animate = () => {
                    tween1.scale -= 0.05;
                    tween1.opacity -= 0.05;
                    tween2.scale -= 0.05;
                    tween2.opacity -= 0.05;
                    
                    tile1.scale.set(tween1.scale, tween1.scale, tween1.scale);
                    tile1.material.opacity = tween1.opacity;
                    tile2.scale.set(tween2.scale, tween2.scale, tween2.scale);
                    tile2.material.opacity = tween2.opacity;
                    
                    if (tween1.scale > 0) {
                        requestAnimationFrame(animate);
                    } else {
                        scene.remove(tile1);
                        scene.remove(tile2);
                        tiles = tiles.filter(t => t !== tile1 && t !== tile2);
                        updateBlockedTiles();
                        checkGameEnd();
                    }
                };
                animate();
            } else {
                tile1.material.color.setHex(0xff0000);
                tile2.material.color.setHex(0xff0000);
                
                setTimeout(() => {
                    tile1.material.color.setHex(0xffffff);
                    tile2.material.color.setHex(0xffffff);
                }, 500);
            }
            
            selectedTiles.forEach(tile => tile.userData.selected = false);
            selectedTiles = [];
        }

        // Проверка окончания игры
        function checkGameEnd() {
            if (tiles.length === 0) {
                gameState = 'won';
                showGameOver(true);
                return;
            }
            
            const availableTiles = tiles.filter(tile => !tile.userData.blocked);
            const tilesByType = {};
            
            availableTiles.forEach(tile => {
                const type = tile.userData.type;
                if (!tilesByType[type]) tilesByType[type] = [];
                tilesByType[type].push(tile);
            });
            
            let hasValidMoves = false;
            for (const type in tilesByType) {
                if (tilesByType[type].length >= 2) {
                    hasValidMoves = true;
                    break;
                }
            }
            
            if (!hasValidMoves) {
                gameState = 'lost';
                showGameOver(false);
            }
        }

        // Показать окно окончания игры
        function showGameOver(won) {
            clearInterval(gameTimer);
            const modal = document.getElementById('gameOverModal');
            const title = document.getElementById('gameOverTitle');
            const message = document.getElementById('gameOverMessage');
            const finalScore = document.getElementById('finalScore');
            const finalTime = document.getElementById('finalTime');
            
            title.textContent = won ? 'Поздравляем!' : 'Игра окончена';
            finalScore.textContent = score;
            finalTime.textContent = formatTime(Math.floor((Date.now() - startTime) / 1000));
            
            if (won) {
                message.innerHTML = `Вы успешно разобрали пирамиду!<br>Очки: <span id="finalScore">${score}</span><br>Время: <span id="finalTime">${formatTime(Math.floor((Date.now() - startTime) / 1000))}</span>`;
            } else {
                message.innerHTML = `Нет доступных ходов.<br>Очки: <span id="finalScore">${score}</span><br>Время: <span id="finalTime">${formatTime(Math.floor((Date.now() - startTime) / 1000))}</span>`;
            }
            
            modal.style.display = 'flex';
            
            if (window.Telegram && window.Telegram.WebApp) {
                window.Telegram.WebApp.sendData(JSON.stringify({
                    score: score,
                    time: Math.floor((Date.now() - startTime) / 1000),
                    won: won
                }));
            }
        }

        // Новая игра
        function newGame() {
            gameState = 'playing';
            score = 0;
            startTime = Date.now();
            selectedTiles = [];
            hints = 3;
            shuffles = 2;
            
            updateScore();
            document.getElementById('gameOverModal').style.display = 'none';
            
            createGameBoard();
            startTimer();
            
            document.getElementById('hintBtn').textContent = `Подсказка (${hints})`;
            document.getElementById('shuffleBtn').textContent = `Перемешать (${shuffles})`;
        }

        // Обновление счета
        function updateScore() {
            document.getElementById('score').textContent = `Очки: ${score}`;
        }

        // Форматирование времени
        function formatTime(seconds) {
            const mins = Math.floor(seconds / 60);
            const secs = seconds % 60;
            return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
        }

        // Таймер
        function startTimer() {
            gameTimer = setInterval(() => {
                if (gameState === 'playing') {
                    const elapsed = Math.floor((Date.now() - startTime) / 1000);
                    document.getElementById('timer').textContent = `Время: ${formatTime(elapsed)}`;
                }
            }, 1000);
        }

        // Поворот камеры
        function rotateCamera() {
            const radius = 20;
            const angle = Date.now() * 0.001;
            camera.position.x = Math.cos(angle) * radius;
            camera.position.z = Math.sin(angle) * radius;
            camera.lookAt(0, 0, 0);
        }

        // Подсказка
        function showHint() {
            if (hints <= 0 || gameState !== 'playing') return;
            
            hints--;
            document.getElementById('hintBtn').textContent = `Подсказка (${hints})`;
            
            const availableTiles = tiles.filter(tile => !tile.userData.blocked);
            const tilesByType = {};
            
            availableTiles.forEach(tile => {
                const type = tile.userData.type;
                if (!tilesByType[type]) tilesByType[type] = [];
                tilesByType[type].push(tile);
            });
            
            for (const type in tilesByType) {
                if (tilesByType[type].length >= 2) {
                    const [tile1, tile2] = tilesByType[type];
                    tile1.material.color.setHex(0xffff00);
                    tile2.material.color.setHex(0xffff00);
                    
                    setTimeout(() => {
                        tile1.material.color.setHex(0xffffff);
                        tile2.material.color.setHex(0xffffff);
                    }, 2000);
                    break;
                }
            }
            
            if (hints === 0) document.getElementById('hintBtn').disabled = true;
        }

        // Перемешивание
        function shuffleTiles() {
            if (shuffles <= 0 || gameState !== 'playing') return;
            
            shuffles--;
            document.getElementById('shuffleBtn').textContent = `Перемешать (${shuffles})`;
            
            selectedTiles.forEach(tile => {
                tile.userData.selected = false;
                tile.material.color.setHex(0xffffff);
            });
            selectedTiles = [];
            
            const availableTiles = tiles.filter(tile => !tile.userData.blocked);
            const types = availableTiles.map(tile => tile.userData.type);
            
            for (let i = types.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [types[i], types[j]] = [types[j], types[i]];
            }
            
            availableTiles.forEach((tile, index) => {
                tile.userData.type = types[index];
                const canvas = document.createElement('canvas');
                canvas.width = 128;
                canvas.height = 128;
                const context = canvas.getContext('2d');
                context.font = '60px sans-serif';
                context.textAlign = 'center';
                context.textBaseline = 'middle';
                context.fillStyle = '#000';
                context.fillText(types[index] || 'X', 64, 64);
                const texture = new THREE.CanvasTexture(canvas);
                tile.children[0].material.map = texture;
                tile.children[0].material.needsUpdate = true;
            });
            
            if (shuffles === 0) document.getElementById('shuffleBtn').disabled = true;
        }

        // Обработчик изменения размера окна
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // Анимационный цикл
        function animate() {
            requestAnimationFrame(animate);
            if (gameState === 'playing') scene.rotation.y += 0.002;
            renderer.render(scene, camera);
        }

        // Обработчики кнопок
        document.getElementById('rotateBtn').addEventListener('click', () => {
            const currentRotation = camera.position.clone();
            const radius = currentRotation.length();
            const angle = Math.atan2(currentRotation.z, currentRotation.x) + Math.PI / 4;
            camera.position.x = Math.cos(angle) * radius;
            camera.position.z = Math.sin(angle) * radius;
            camera.lookAt(0, 0, 0);
        });

        document.getElementById('hintBtn').addEventListener('click', showHint);
        document.getElementById('shuffleBtn').addEventListener('click', shuffleTiles);
        document.getElementById('newGameBtn').addEventListener('click', newGame);
        document.getElementById('restartBtn').addEventListener('click', newGame);

        // Управление жестами для мобильных устройств
        let touchStartX, touchStartY, touchStartTime;
        let cameraRotation = 0;

        renderer.domElement.addEventListener('touchstart', (e) => {
            if (e.touches.length === 1) {
                touchStartX = e.touches[0].clientX;
                touchStartY = e.touches[0].clientY;
                touchStartTime = Date.now();
            }
        });

        renderer.domElement.addEventListener('touchmove', (e) => {
            if (e.touches.length === 1) {
                e.preventDefault();
                const deltaX = e.touches[0].clientX - touchStartX;
                const deltaY = e.touches[0].clientY - touchStartY;
                
                if (Math.abs(deltaX) > 50 && Math.abs(deltaY) < 100) {
                    cameraRotation += deltaX * 0.01;
                    const radius = 20;
                    camera.position.x = Math.cos(cameraRotation) * radius;
                    camera.position.z = Math.sin(cameraRotation) * radius;
                    camera.lookAt(0, 0, 0);
                    touchStartX = e.touches[0].clientX;
                }
            }
        });

        // Инициализация игры
        function initGame() {
            console.log("Initializing game...");
            document.getElementById('loading').style.display = 'none';
            initThree();
            newGame();
            animate();
        }

        // Запуск игры после загрузки
        window.addEventListener('load', () => {
            if (typeof THREE === 'undefined') {
                console.error("Three.js failed to load");
                document.getElementById('loading').innerText = "Ошибка загрузки Three.js";
                return;
            }
            initGame();
        });

        // Дополнительные настройки для Telegram WebApp
        if (window.Telegram && window.Telegram.WebApp) {
            const tg = window.Telegram.WebApp;
            tg.MainButton.text = "Поделиться результатом";
            tg.MainButton.onClick(() => {
                const gameData = {
                    score: score,
                    time: Math.floor((Date.now() - startTime) / 1000),
                    tiles_remaining: tiles.length
                };
                tg.sendData(JSON.stringify(gameData));
            });
            
            tg.onEvent('mainButtonClicked', () => {
                if (gameState !== 'playing') tg.MainButton.show();
            });
            
            if (tg.colorScheme === 'dark') {
                document.body.style.background = 'linear-gradient(135deg, #2c3e50 0%, #34495e 100%)';
            }
            
            tg.onEvent('viewportChanged', () => {
                if (tg.isExpanded) {
                    renderer.setSize(window.innerWidth, window.innerHeight);
                    camera.aspect = window.innerWidth / window.innerHeight;
                    camera.updateProjectionMatrix();
                }
            });
        }

        // Сохранение лучшего результата
        let bestScore = 0;
        let bestTime = Infinity;

        function updateBestScore() {
            if (score > bestScore) bestScore = score;
            if (tiles.length === 0) {
                const currentTime = Math.floor((Date.now() - startTime) / 1000);
                if (currentTime < bestTime) bestTime = currentTime;
            }
        }

        function getGameStats() {
            return {
                score: score,
                bestScore: bestScore,
                currentTime: Math.floor((Date.now() - startTime) / 1000),
                bestTime: bestTime === Infinity ? 0 : bestTime,
                tilesRemaining: tiles.length,
                hintsUsed: 3 - hints,
                shufflesUsed: 2 - shuffles
            };
        }
    </script>
</body>
</html>
