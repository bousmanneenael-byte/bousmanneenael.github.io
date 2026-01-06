<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jeu de Plateforme Vertical</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: #87ceeb;
    font-family: Arial, sans-serif;
  }
  canvas {
    display: block;
    background: #87ceeb;
    display: none; /* caché au début */
  }
  .menu {
    position: absolute;
    width: 100%;
    height: 100%;
    background: #87ceeb;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .menu h1 {
    font-size: 50px;
    margin-bottom: 50px;
    color: black;
  }
  .menu button {
    padding: 15px 30px;
    font-size: 25px;
    margin: 10px;
    border: none;
    border-radius: 10px;
    background-color: #4CAF50;
    color: white;
  }
  #replayBtn, #menuBtn {
    position: absolute;
    left: 50%;
    top: 60%;
    transform: translate(-50%, -50%);
    padding: 15px 30px;
    font-size: 20px;
    display: none;
    border: none;
    border-radius: 10px;
  }
  #replayBtn { background-color: #ff4d4d; color: white; }
  #menuBtn { background-color: #4CAF50; color: white; top: 70%; }
  .button {
    position: absolute;
    bottom: 20px;
    width: 80px;
    height: 80px;
    background-color: rgba(0,0,0,0.5);
    color: white;
    font-size: 40px;
    text-align: center;
    line-height: 80px;
    border-radius: 50%;
    user-select: none;
  }
  #leftBtn { left: 20px; }
  #rightBtn { right: 20px; }
</style>
</head>
<body>

<div class="menu" id="menu">
    <h1>Plateforme Vertical</h1>
    <div>Meilleur Score : <span id="bestScore">0</span></div>
    <button id="startBtn">Jouer</button>
</div>

<canvas id="game"></canvas>
<button id="replayBtn">Rejouer</button>
<button id="menuBtn">Menu</button>
<div id="leftBtn" class="button">&#8592;</div>
<div id="rightBtn" class="button">&#8594;</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const replayBtn = document.getElementById('replayBtn');
const menuBtn = document.getElementById('menuBtn');
const leftBtn = document.getElementById('leftBtn');
const rightBtn = document.getElementById('rightBtn');
const menu = document.getElementById('menu');
const startBtn = document.getElementById('startBtn');
const bestScoreDisplay = document.getElementById('bestScore');

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let player, platforms, score, left, right, gameOver;
let bestScore = localStorage.getItem('bestScore') || 0;
bestScoreDisplay.textContent = bestScore;

// Initialisation du jeu
function initGame() {
    player = {
        x: canvas.width/2 - 16,
        y: canvas.height - 150,
        width: 32,
        height: 32,
        color: 'red',
        dy: 0,
        jumpPower: 15,
        gravity: 0.6
    };

    platforms = [];
    const platformWidth = 100;
    const platformHeight = 20;

    for(let i=0; i<10; i++){
        let yPos = canvas.height - i*100;

        if(i === 0){
            platforms.push({
                x: player.x - platformWidth/2 + player.width/2,
                y: player.y + player.height,
                width: platformWidth,
                height: platformHeight,
                color: 'green'
            });
        } else {
            platforms.push({
                x: Math.random()*(canvas.width-platformWidth),
                y: yPos,
                width: platformWidth,
                height: platformHeight,
                color: 'green'
            });
        }
    }

    score = 0;
    left = false;
    right = false;
    gameOver = false;
    replayBtn.style.display = 'none';
    menuBtn.style.display = 'none';
}

// Touch controls pour les boutons
leftBtn.addEventListener('touchstart', ()=>{ left=true; });
leftBtn.addEventListener('touchend', ()=>{ left=false; });
rightBtn.addEventListener('touchstart', ()=>{ right=true; });
rightBtn.addEventListener('touchend', ()=>{ right=false; });

// Bouton rejouer
replayBtn.addEventListener('click', () => {
    initGame();
    update();
});

// Bouton menu
menuBtn.addEventListener('click', () => {
    canvas.style.display = 'none';
    leftBtn.style.display = 'none';
    rightBtn.style.display = 'none';
    menu.style.display = 'flex';
});

// Bouton démarrer depuis le menu
startBtn.addEventListener('click', () => {
    menu.style.display = 'none';
    canvas.style.display = 'block';
    leftBtn.style.display = 'block';
    rightBtn.style.display = 'block';
    initGame();
    update();
});

// Boucle du jeu
function update(){
    if(gameOver){
        ctx.clearRect(0,0,canvas.width,canvas.height);
        ctx.fillStyle = 'black';
        ctx.font = '50px Arial';
        ctx.textAlign = 'center';
        ctx.fillText('MORT', canvas.width/2, canvas.height/2 - 30);
        ctx.font = '25px Arial';
        ctx.fillText('Score: '+Math.floor(score), canvas.width/2, canvas.height/2 + 10);

        replayBtn.style.display = 'block';
        menuBtn.style.display = 'block';

        // Mettre à jour meilleur score
        if(score > bestScore){
            bestScore = Math.floor(score);
            localStorage.setItem('bestScore', bestScore);
            bestScoreDisplay.textContent = bestScore;
        }
        return;
    }

    ctx.clearRect(0,0,canvas.width,canvas.height);

    // Mouvement horizontal
    if(left) player.x -= 5;
    if(right) player.x += 5;

    // Gravité
    player.dy += player.gravity;
    player.y += player.dy;

    // Collision avec plateformes
    platforms.forEach(p => {
        if(player.dy > 0 &&
           player.x + player.width > p.x &&
           player.x < p.x + p.width &&
           player.y + player.height > p.y &&
           player.y + player.height < p.y + p.height) {
            player.y = p.y - player.height;
            player.dy = -player.jumpPower;
        }
    });

    // Remonter la caméra
    if(player.y < canvas.height/2){
        const diff = canvas.height/2 - player.y;
        player.y = canvas.height/2;
        platforms.forEach(p => p.y += diff);
        score += diff;
    }

    // Générer de nouvelles plateformes
    platforms = platforms.filter(p => p.y < canvas.height);
    while(platforms.length < 10){
        const highest = Math.min(...platforms.map(p=>p.y));
        platforms.push({
            x: Math.random()*(canvas.width-100),
            y: highest - 100,
            width: 100,
            height: 20,
            color: 'green'
        });
    }

    // Dessiner plateformes
    platforms.forEach(p => {
        ctx.fillStyle = p.color;
        ctx.fillRect(p.x,p.y,p.width,p.height);
    });

    // Dessiner joueur
    ctx.fillStyle = player.color;
    ctx.fillRect(player.x, player.y, player.width, player.height);

    // Dessiner score
    ctx.fillStyle = 'black';
    ctx.font = '20px Arial';
    ctx.textAlign = 'left';
    ctx.fillText('Score: '+Math.floor(score), 10, 30);

    // Détection chute
    if(player.y > canvas.height){
        gameOver = true;
    }

    requestAnimationFrame(update);
}
</script>
</body>
</html>
