<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Space Defender - One File Game</title>
    <style>
        body {
            margin: 0;
            background: #050505;
            color: white;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        canvas {
            border: 2px solid #333;
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.2);
            background: black;
            cursor: crosshair;
        }
        .ui {
            position: absolute;
            top: 20px;
            pointer-events: none;
            text-align: center;
        }
        #score { font-size: 24px; font-weight: bold; color: #0ff; }
        .controls { color: #888; font-size: 14px; margin-top: 10px; }
    </style>
</head>
<body>

    <div class="ui">
        <div id="score">Счёт: 0</div>
        <div class="controls">Управление: Стрелки / WASD — движение | Пробел — огонь</div>
    </div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');

        canvas.width = 800;
        canvas.height = 600;

        let score = 0;
        let gameActive = true;

        const player = {
            x: canvas.width / 2,
            y: canvas.height - 50,
            w: 40,
            h: 40,
            speed: 5,
            color: '#00d2ff'
        };

        const bullets = [];
        const enemies = [];
        const keys = {};

        window.addEventListener('keydown', e => keys[e.code] = true);
        window.addEventListener('keyup', e => keys[e.code] = false);

        function spawnEnemy() {
            if (!gameActive) return;
            enemies.push({
                x: Math.random() * (canvas.width - 30),
                y: -30,
                size: 20 + Math.random() * 30,
                speed: 2 + Math.random() * 3,
                color: '#ff4b2b'
            });
            setTimeout(spawnEnemy, 1000 - Math.min(score * 5, 700));
        }

        function update() {
            if (!gameActive) return;

            // Движение игрока
            if ((keys['ArrowLeft'] || keys['KeyA']) && player.x > 0) player.x -= player.speed;
            if ((keys['ArrowRight'] || keys['KeyD']) && player.x < canvas.width - player.w) player.x += player.speed;
            if ((keys['ArrowUp'] || keys['KeyW']) && player.y > 0) player.y -= player.speed;
            if ((keys['ArrowDown'] || keys['KeyS']) && player.y < canvas.height - player.h) player.y += player.speed;

            // Стрельба
            if (keys['Space']) {
                if (bullets.length === 0 || bullets[bullets.length-1].y < player.y - 20) {
                    bullets.push({ x: player.x + player.w/2 - 2, y: player.y, w: 4, h: 10 });
                }
            }

            // Обновление пуль
            bullets.forEach((b, i) => {
                b.y -= 8;
                if (b.y < 0) bullets.splice(i, 1);
            });

            // Обновление врагов
            enemies.forEach((en, i) => {
                en.y += en.speed;
                
                // Столкновение с игроком
                if (en.x < player.x + player.w && en.x + en.size > player.x &&
                    en.y < player.y + player.h && en.y + en.size > player.y) {
                    gameOver();
                }

                // Проверка попадания пуль
                bullets.forEach((b, bi) => {
                    if (b.x < en.x + en.size && b.x + b.w > en.x &&
                        b.y < en.y + en.size && b.y + b.h > en.y) {
                        enemies.splice(i, 1);
                        bullets.splice(bi, 1);
                        score += 10;
                        scoreElement.innerText = `Счёт: ${score}`;
                    }
                });

                if (en.y > canvas.height) enemies.splice(i, 1);
            });
        }

        function draw() {
            ctx.fillStyle = 'black';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Игрок (корабль)
            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.moveTo(player.x + player.w / 2, player.y);
            ctx.lineTo(player.x, player.y + player.h);
            ctx.lineTo(player.x + player.w, player.y + player.h);
            ctx.fill();

            // Пули
            ctx.fillStyle = 'yellow';
            bullets.forEach(b => ctx.fillRect(b.x, b.y, b.w, b.h));

            // Враги
            ctx.fillStyle = '#ff4b2b';
            enemies.forEach(en => {
                ctx.beginPath();
                ctx.arc(en.x + en.size/2, en.y + en.size/2, en.size/2, 0, Math.PI*2);
                ctx.fill();
            });

            if (!gameActive) {
                ctx.fillStyle = 'rgba(0,0,0,0.7)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = 'white';
                ctx.font = '40px Arial';
                ctx.textAlign = 'center';
                ctx.fillText('ИГРА ОКОНЧЕНА', canvas.width/2, canvas.height/2);
                ctx.font = '20px Arial';
                ctx.fillText('Нажмите F5, чтобы начать заново', canvas.width/2, canvas.height/2 + 40);
            }

            requestAnimationFrame(() => {
                update();
                draw();
            });
        }

        function gameOver() {
            gameActive = false;
        }

        spawnEnemy();
        draw();
    </script>
</body>
</html>
