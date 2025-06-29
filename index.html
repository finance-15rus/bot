<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Настольный теннис</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: linear-gradient(135deg, #1e3c72, #2a5298);
            font-family: 'Arial', sans-serif;
            overflow: hidden;
            color: white;
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
            z-index: 100;
            background: rgba(0,0,0,0.8);
            padding: 15px 20px;
            border-radius: 10px;
            backdrop-filter: blur(10px);
        }
        
        .score {
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 10px;
        }
        
        .controls {
            font-size: 14px;
            line-height: 1.6;
        }
        
        #instructions {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            z-index: 200;
            background: rgba(0,0,0,0.9);
            padding: 30px;
            border-radius: 15px;
            backdrop-filter: blur(15px);
            max-width: 400px;
        }
        
        #startButton {
            background: linear-gradient(45deg, #4CAF50, #45a049);
            color: white;
            border: none;
            padding: 15px 30px;
            font-size: 18px;
            border-radius: 8px;
            cursor: pointer;
            margin-top: 20px;
            transition: all 0.3s ease;
        }
        
        #startButton:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(76, 175, 80, 0.4);
        }
        
        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div id="ui">
            <div class="score">Игрок: <span id="playerScore">0</span></div>
            <div class="score">Компьютер: <span id="aiScore">0</span></div>
            <div class="controls">
                <strong>Управление:</strong><br>
                Мышь - движение ракетки<br>
                Клик - удар по мячу
            </div>
        </div>
        
        <div id="instructions">
            <h2>3D Настольный теннис</h2>
            <p>Управляйте ракеткой мышью и кликайте для удара по мячу.</p>
            <p>Первый до 5 очков выигрывает!</p>
            <button id="startButton">Начать игру</button>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        let scene, camera, renderer, ball, playerPaddle, aiPaddle, table;
        let ballVelocity = { x: 0, y: 0, z: 0 };
        let gameStarted = false;
        let playerScore = 0, aiScore = 0;
        let mouse = { x: 0, y: 0 };
        let gameActive = false;

        function init() {
            // Создание сцены
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87CEEB);

            // Камера
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 8, 12);
            camera.lookAt(0, 0, 0);

            // Рендерер
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            document.getElementById('gameContainer').appendChild(renderer.domElement);

            // Освещение
            const ambientLight = new THREE.AmbientLight(0x404040, 0.6);
            scene.add(ambientLight);

            const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight.position.set(0, 20, 10);
            directionalLight.castShadow = true;
            directionalLight.shadow.mapSize.width = 2048;
            directionalLight.shadow.mapSize.height = 2048;
            scene.add(directionalLight);

            // Стол
            const tableGeometry = new THREE.BoxGeometry(12, 0.2, 6);
            const tableMaterial = new THREE.MeshLambertMaterial({ color: 0x0066cc });
            table = new THREE.Mesh(tableGeometry, tableMaterial);
            table.position.y = -0.1;
            table.receiveShadow = true;
            scene.add(table);

            // Сетка
            const netGeometry = new THREE.BoxGeometry(0.1, 1, 6);
            const netMaterial = new THREE.MeshLambertMaterial({ color: 0xffffff });
            const net = new THREE.Mesh(netGeometry, netMaterial);
            net.position.set(0, 0.5, 0);
            scene.add(net);

            // Линии стола
            const lineGeometry = new THREE.BoxGeometry(12, 0.01, 0.1);
            const lineMaterial = new THREE.MeshLambertMaterial({ color: 0xffffff });
            
            const centerLine = new THREE.Mesh(lineGeometry, lineMaterial);
            centerLine.position.y = 0.11;
            scene.add(centerLine);

            // Мяч
            const ballGeometry = new THREE.SphereGeometry(0.2, 16, 16);
            const ballMaterial = new THREE.MeshLambertMaterial({ color: 0xff6600 });
            ball = new THREE.Mesh(ballGeometry, ballMaterial);
            ball.position.set(0, 2, 0);
            ball.castShadow = true;
            scene.add(ball);

            // Ракетка игрока
            const paddleGeometry = new THREE.CylinderGeometry(0.8, 0.8, 0.1, 16);
            const paddleMaterial = new THREE.MeshLambertMaterial({ color: 0xcc0000 });
            playerPaddle = new THREE.Mesh(paddleGeometry, paddleMaterial);
            playerPaddle.position.set(0, 1, 4);
            playerPaddle.castShadow = true;
            scene.add(playerPaddle);

            // Ракетка ИИ
            const aiPaddleMaterial = new THREE.MeshLambertMaterial({ color: 0x00cc00 });
            aiPaddle = new THREE.Mesh(paddleGeometry, aiPaddleMaterial);
            aiPaddle.position.set(0, 1, -4);
            aiPaddle.castShadow = true;
            scene.add(aiPaddle);

            // Границы стола
            const boundaryGeometry = new THREE.BoxGeometry(0.2, 2, 6.5);
            const boundaryMaterial = new THREE.MeshLambertMaterial({ color: 0x444444 });
            
            const leftBoundary = new THREE.Mesh(boundaryGeometry, boundaryMaterial);
            leftBoundary.position.set(-6.1, 1, 0);
            scene.add(leftBoundary);
            
            const rightBoundary = new THREE.Mesh(boundaryGeometry, boundaryMaterial);
            rightBoundary.position.set(6.1, 1, 0);
            scene.add(rightBoundary);

            // События мыши
            document.addEventListener('mousemove', onMouseMove);
            document.addEventListener('click', onMouseClick);
            document.addEventListener('resize', onWindowResize);

            // Запуск игры
            document.getElementById('startButton').addEventListener('click', startGame);
        }

        function startGame() {
            gameStarted = true;
            gameActive = true;
            document.getElementById('instructions').classList.add('hidden');
            
            // Сброс мяча
            resetBall();
            animate();
        }

        function resetBall() {
            ball.position.set(0, 2, 0);
            ballVelocity.x = (Math.random() - 0.5) * 0.1;
            ballVelocity.y = 0;
            ballVelocity.z = 0.15;
        }

        function onMouseMove(event) {
            if (!gameActive) return;
            
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
            
            // Обновление позиции ракетки игрока
            playerPaddle.position.x = mouse.x * 5;
            playerPaddle.position.y = Math.max(0.5, 1 + mouse.y * 2);
        }

        function onMouseClick() {
            if (!gameActive) return;
            
            // Проверка близости мяча к ракетке для удара
            const distance = ball.position.distanceTo(playerPaddle.position);
            if (distance < 1.5) {
                // Удар по мячу
                ballVelocity.x = (ball.position.x - playerPaddle.position.x) * 0.3;
                ballVelocity.y = Math.abs(ballVelocity.y) + 0.1;
                ballVelocity.z = -Math.abs(ballVelocity.z) - 0.05;
            }
        }

        function updateBall() {
            if (!gameActive) return;
            
            // Применение гравитации
            ballVelocity.y -= 0.01;
            
            // Обновление позиции мяча
            ball.position.x += ballVelocity.x;
            ball.position.y += ballVelocity.y;
            ball.position.z += ballVelocity.z;

            // Отскок от стола
            if (ball.position.y <= 0.3 && ballVelocity.y < 0) {
                ballVelocity.y = -ballVelocity.y * 0.8;
                ballVelocity.x *= 0.95;
                ballVelocity.z *= 0.95;
            }

            // Отскок от боковых границ
            if (Math.abs(ball.position.x) > 6) {
                ballVelocity.x = -ballVelocity.x;
            }

            // Столкновение с ракеткой ИИ
            const aiDistance = ball.position.distanceTo(aiPaddle.position);
            if (aiDistance < 1 && ballVelocity.z < 0) {
                ballVelocity.x = (ball.position.x - aiPaddle.position.x) * 0.2;
                ballVelocity.y = Math.abs(ballVelocity.y) + 0.1;
                ballVelocity.z = Math.abs(ballVelocity.z) + 0.05;
            }

            // Проверка очков
            if (ball.position.z > 7) {
                aiScore++;
                updateScore();
                resetBall();
            } else if (ball.position.z < -7) {
                playerScore++;
                updateScore();
                resetBall();
            }

            // Проверка победы
            if (playerScore >= 5 || aiScore >= 5) {
                endGame();
            }
        }

        function updateAI() {
            if (!gameActive) return;
            
            // Простой ИИ для ракетки компьютера
            const targetX = ball.position.x * 0.8;
            aiPaddle.position.x += (targetX - aiPaddle.position.x) * 0.1;
            
            // Ограничение движения ИИ
            aiPaddle.position.x = Math.max(-5, Math.min(5, aiPaddle.position.x));
        }

        function updateScore() {
            document.getElementById('playerScore').textContent = playerScore;
            document.getElementById('aiScore').textContent = aiScore;
        }

        function endGame() {
            gameActive = false;
            const winner = playerScore >= 5 ? 'Игрок' : 'Компьютер';
            
            setTimeout(() => {
                alert(`${winner} выиграл! Итоговый счет: ${playerScore}:${aiScore}`);
                
                // Сброс игры
                playerScore = 0;
                aiScore = 0;
                updateScore();
                document.getElementById('instructions').classList.remove('hidden');
                gameStarted = false;
            }, 1000);
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        function animate() {
            if (!gameStarted) return;
            
            requestAnimationFrame(animate);
            
            updateBall();
            updateAI();
            
            renderer.render(scene, camera);
        }

        // Инициализация при загрузке страницы
        init();
    </script>
</body>
</html>
