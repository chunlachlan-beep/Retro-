<!DOCTYPE html>
<html>
<head>
    <title>Simple Space Shooter</title>
    <style>
        body { margin: 0; background: #000; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; color: white; font-family: sans-serif; }
        canvas { border: 2px solid #333; background: #050505; }
        #ui { position: absolute; top: 20px; left: 20px; pointer-events: none; }
    </style>
</head>
<body>
    <div id="ui">Score: <span id="score">0</span></div>
    <canvas id="gameCanvas"></canvas>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreElement = document.getElementById('score');

    canvas.width = 400;
    canvas.height = 600;

    // Game State
    let score = 0;
    let gameOver = false;
    const player = { x: 180, y: 500, w: 40, h: 40, speed: 5, color: '#00ffcc' };
    const bullets = [];
    const enemies = [];
    const keys = {};

    // Input Listeners
    window.addEventListener('keydown', e => keys[e.code] = true);
    window.addEventListener('keyup', e => keys[e.code] = false);

    function spawnEnemy() {
        if (Math.random() < 0.03) {
            enemies.push({
                x: Math.random() * (canvas.width - 30),
                y: -30,
                w: 30,
                h: 30,
                speed: 2 + Math.random() * 2,
                color: '#ff4444'
            });
        }
    }

    function update() {
        if (gameOver) return;

        // Move Player
        if (keys['ArrowLeft'] && player.x > 0) player.x -= player.speed;
        if (keys['ArrowRight'] && player.x < canvas.width - player.w) player.x += player.speed;
        
        // Shooting (Spacebar)
        if (keys['Space'] && bullets.length < 5) {
            bullets.push({ x: player.x + player.w/2 - 2, y: player.y, w: 4, h: 10, speed: 7 });
            keys['Space'] = false; // Prevents rapid-fire holding
        }

        // Update Bullets
        bullets.forEach((b, i) => {
            b.y -= b.speed;
            if (b.y < 0) bullets.splice(i, 1);
        });

        // Update Enemies
        enemies.forEach((en, ei) => {
            en.y += en.speed;
            
            // Collision: Bullet vs Enemy
            bullets.forEach((b, bi) => {
                if (b.x < en.x + en.w && b.x + b.w > en.x && b.y < en.y + en.h && b.y + b.h > en.y) {
                    enemies.splice(ei, 1);
                    bullets.splice(bi, 1);
                    score += 10;
                    scoreElement.innerText = score;
                }
            });

            // Collision: Player vs Enemy
            if (player.x < en.x + en.w && player.x + player.w > en.x && player.y < en.y + en.h && player.y + player.h > en.y) {
                gameOver = true;
                alert("Game Over! Score: " + score);
                location.reload();
            }

            if (en.y > canvas.height) enemies.splice(ei, 1);
        });

        spawnEnemy();
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Draw Player (Triangle ship)
        ctx.fillStyle = player.color;
        ctx.beginPath();
        ctx.moveTo(player.x + player.w/2, player.y);
        ctx.lineTo(player.x, player.y + player.h);
        ctx.lineTo(player.x + player.w, player.y + player.h);
        ctx.fill();

        // Draw Bullets
        ctx.fillStyle = 'yellow';
        bullets.forEach(b => ctx.fillRect(b.x, b.y, b.w, b.h));

        // Draw Enemies
        ctx.fillStyle = '#ff4444';
        enemies.forEach(en => ctx.fillRect(en.x, en.y, en.w, en.h));

        requestAnimationFrame(() => {
            update();
            draw();
        });
    }

    draw();
</script>
</body>
</html>
