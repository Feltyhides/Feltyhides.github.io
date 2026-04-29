<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Space Survivor RPG</title>
    <style>
        body { margin: 0; background: #0a0a0c; color: white; font-family: 'Courier New', monospace; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        canvas { background: radial-gradient(circle, #1b1b2f 0%, #000000 100%); border: 3px solid #444; box-shadow: 0 0 30px rgba(0,0,0,0.5); }
        #ui { position: absolute; top: 10px; left: 10px; pointer-events: none; width: 780px; }
        .bar-container { width: 200px; height: 15px; background: #333; border: 1px solid #fff; margin-bottom: 5px; }
        #hp-bar { width: 100%; height: 100%; background: #ff4b2b; transition: width 0.3s; }
        #exp-bar { width: 0%; height: 100%; background: #2ecc71; transition: width 0.3s; }
        #level-up-screen { position: absolute; display: none; flex-direction: column; align-items: center; background: rgba(0,0,0,0.9); padding: 30px; border: 2px solid #2ecc71; z-index: 10; }
        .upgrade-btn { background: #222; color: #2ecc71; border: 1px solid #2ecc71; padding: 10px 20px; margin: 5px; cursor: pointer; font-family: inherit; width: 250px; }
        .upgrade-btn:hover { background: #2ecc71; color: #000; }
        #stats { font-size: 14px; color: #aaa; }
    </style>
</head>
<body>

    <div id="ui">
        HP <div class="bar-container"><div id="hp-bar"></div></div>
        EXP <div class="bar-container"><div id="exp-bar"></div></div>
        <div id="stats">Уровень: 1 | Счет: 0</div>
    </div>

    <div id="level-up-screen">
        <h2>УРОВЕНЬ ПОВЫШЕН!</h2>
        <p>Выберите улучшение:</p>
        <div id="upgrade-list"></div>
    </div>

    <canvas id="game"></canvas>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
canvas.width = 800; canvas.height = 600;

// Состояние игры
let state = {
    score: 0,
    level: 1,
    exp: 0,
    expToNext: 50,
    paused: false,
    gameOver: false,
    shake: 0
};

// Игрок
const player = {
    x: 400, y: 500, w: 30, h: 30,
    hp: 100, maxHp: 100,
    speed: 5,
    fireRate: 400, // мс
    lastShot: 0,
    bulletCount: 1,
    damage: 1
};

let entities = { bullets: [], enemies: [], particles: [], loot: [] };
const keys = {};

window.onkeydown = e => keys[e.code] = true;
window.onkeyup = e => keys[e.code] = false;

function createParticle(x, y, color) {
    for(let i=0; i<8; i++) {
        entities.particles.push({
            x, y, 
            vx: (Math.random()-0.5)*5, vy: (Math.random()-0.5)*5,
            life: 1, color, size: Math.random()*3
        });
    }
}

function spawnEnemy() {
    if(state.paused || state.gameOver) return;
    const isFast = Math.random() > 0.8;
    entities.enemies.push({
        x: Math.random() * (canvas.width - 30),
        y: -50,
        hp: isFast ? 1 : 2 + Math.floor(state.level/2),
        speed: isFast ? 4 : 1.5 + Math.random(),
        size: isFast ? 15 : 25 + Math.random()*20,
        type: isFast ? 'fast' : 'normal',
        color: isFast ? '#f1c40f' : '#e74c3c'
    });
    setTimeout(spawnEnemy, Math.max(200, 1200 - (state.level * 50)));
}

function showLevelUp() {
    state.paused = true;
    const screen = document.getElementById('level-up-screen');
    const list = document.getElementById('upgrade-list');
    screen.style.display = 'flex';
    list.innerHTML = '';

    const upgrades = [
        { t: 'Пулемет (Скорострельность)', f: () => player.fireRate *= 0.8 },
        { t: 'Двойной ствол (+1 пуля)', f: () => player.bulletCount++ },
        { t: 'Тяжелые снаряды (Урон)', f: () => player.damage += 1 },
        { t: 'Ремкомплект (+40 HP)', f: () => player.hp = Math.min(player.maxHp, player.hp + 40) }
    ];

    upgrades.sort(() => 0.5 - Math.random()).slice(0, 3).forEach(upg => {
        const btn = document.createElement('button');
        btn.className = 'upgrade-btn';
        btn.textContent = upg.t;
        btn.onclick = () => {
            upg.f();
            state.paused = false;
            screen.style.display = 'none';
        };
        list.appendChild(btn);
    });
}

function update() {
    if(state.paused || state.gameOver) return;

    // Движение
    if((keys.ArrowLeft || keys.KeyA) && player.x > 0) player.x -= player.speed;
    if((keys.ArrowRight || keys.KeyD) && player.x < canvas.width - player.w) player.x += player.speed;
    if((keys.ArrowUp || keys.KeyW) && player.y > 0) player.y -= player.speed;
    if((keys.ArrowDown || keys.KeyS) && player.y < canvas.height - player.h) player.y += player.speed;

    // Стрельба
    const now = Date.now();
    if(keys.Space && now - player.lastShot > player.fireRate) {
        for(let i=0; i<player.bulletCount; i++) {
            entities.bullets.push({
                x: player.x + player.w/2,
                y: player.y,
                vx: (i - (player.bulletCount-1)/2) * 2,
                vy: -7
            });
        }
        player.lastShot = now;
    }

    // Логика объектов
    entities.bullets.forEach((b, i) => {
        b.x += b.vx; b.y += b.vy;
        if(b.y < 0) entities.bullets.splice(i, 1);
    });

    entities.enemies.forEach((en, i) => {
        en.y += en.speed;
        if(en.y > canvas.height) entities.enemies.splice(i, 1);

        // Столкновение с игроком
        if(en.x < player.x+player.w && en.x+en.size > player.x && en.y < player.y+player.h && en.y+en.size > player.y) {
            player.hp -= 10;
            state.shake = 10;
            entities.enemies.splice(i, 1);
            createParticle(en.x, en.y, 'red');
            if(player.hp <= 0) state.gameOver = true;
        }

        // Попадание пуль
        entities.bullets.forEach((b, bi) => {
            if(b.x > en.x && b.x < en.x+en.size && b.y > en.y && b.y < en.y+en.size) {
                en.hp -= player.damage;
                entities.bullets.splice(bi, 1);
                if(en.hp <= 0) {
                    entities.loot.push({x: en.x, y: en.y, v: 10});
                    entities.enemies.splice(i, 1);
                    state.score += 10;
                    createParticle(en.x, en.y, en.color);
                }
            }
        });
    });

    entities.loot.forEach((l, i) => {
        l.y += 2;
        if(l.x < player.x+player.w && l.x+10 > player.x && l.y < player.y+player.h && l.y+10 > player.y) {
            state.exp += l.v;
            entities.loot.splice(i, 1);
            if(state.exp >= state.expToNext) {
                state.level++;
                state.exp = 0;
                state.expToNext *= 1.5;
                showLevelUp();
            }
        }
    });

    entities.particles.forEach((p, i) => {
        p.x += p.vx; p.y += p.vy; p.life -= 0.02;
        if(p.life <= 0) entities.particles.splice(i, 1);
    });

    // UI update
    document.getElementById('hp-bar').style.width = (player.hp/player.maxHp)*100 + '%';
    document.getElementById('exp-bar').style.width = (state.exp/state.expToNext)*100 + '%';
    document.getElementById('stats').innerText = `Уровень: ${state.level} | Счет: ${state.score} | Урон: ${player.damage}`;
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    ctx.save();
    if(state.shake > 0) {
        ctx.translate(Math.random()*state.shake, Math.random()*state.shake);
        state.shake *= 0.9;
    }

    // Игрок
    ctx.fillStyle = '#00d2ff';
    ctx.shadowBlur = 15; ctx.shadowColor = '#00d2ff';
    ctx.fillRect(player.x, player.y, player.w, player.h);
    ctx.shadowBlur = 0;

    // Пули
    ctx.fillStyle = '#fff';
    entities.bullets.forEach(b => ctx.fillRect(b.x-2, b.y, 4, 10));

    // Враги
    entities.enemies.forEach(en => {
        ctx.fillStyle = en.color;
        ctx.fillRect(en.x, en.y, en.size, en.size);
    });

    // Опыт (лут)
    ctx.fillStyle = '#2ecc71';
    entities.loot.forEach(l => ctx.beginPath() || ctx.arc(l.x, l.y, 5, 0, Math.PI*2) || ctx.fill());

    // Частицы
    entities.particles.forEach(p => {
        ctx.globalAlpha = p.life;
        ctx.fillStyle = p.color;
        ctx.fillRect(p.x, p.y, p.size, p.size);
    });
    ctx.globalAlpha = 1;

    ctx.restore();

    if(state.gameOver) {
        ctx.fillStyle = 'white';
        ctx.font = '40px monospace';
        ctx.textAlign = 'center';
        ctx.fillText('МИССИЯ ПРОВАЛЕНА', 400, 300);
        ctx.font = '20px monospace';
        ctx.fillText(`Ваш счет: ${state.score}. Нажмите F5`, 400, 340);
    }

    requestAnimationFrame(() => { update(); draw(); });
}

spawnEnemy();
draw();
</script>
</body>
</html>
