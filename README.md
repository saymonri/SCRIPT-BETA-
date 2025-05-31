/**
 * Script para criar e acionar download do arquivo Pac-Man HTML.
 * 
 * Para usar, cole o conteÃºdo inteiro do arquivo pacman.html
 * dentro da variÃ¡vel `pacmanHTML` (entre as crases).
 * 
 * Chame a funÃ§Ã£o downloadPacman() para iniciar o download.
 */

const pacmanHTML = `<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Jogo Pac-Man</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

  body {
    margin: 0;
    background: radial-gradient(circle at center, #000000, #111111, #222222);
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    font-family: 'Press Start 2P', cursive;
    color: #fff;
    user-select: none;
  }

  canvas {
    background-color: #000;
    border: 5px solid #fce903;
    border-radius: 16px;
    box-shadow:
      0 0 20px #fce903,
      inset 0 0 30px #fce903;
  }

  #scoreboard {
    position: absolute;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 1.2rem;
    letter-spacing: 2px;
    text-shadow:
      2px 2px 8px #fce903,
      0 0 5px #fce903;
  }

  #message {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 1.5rem;
    background: rgba(0,0,0,0.75);
    padding: 20px 40px;
    border-radius: 12px;
    border: 3px solid #fce903;
    text-align: center;
    display: none;
    user-select: none;
    box-shadow:
      0 0 20px #fce903;
  }

  #restartBtn {
    margin-top: 12px;
    background: #fce903;
    border: none;
    border-radius: 8px;
    padding: 8px 22px;
    font-weight: 700;
    cursor: pointer;
    color: #000;
    font-family: inherit;
    letter-spacing: 1.5px;
    box-shadow:
      0 0 12px #fce903;
    transition: background-color 0.3s ease;
  }
  #restartBtn:hover {
    background-color: #ffea00;
  }
</style>
</head>
<body>
<div id="scoreboard">PontuaÃ§Ã£o: 0</div>
<div id="message">
  <div id="messageText"></div>
  <button id="restartBtn">Reiniciar</button>
</div>
<canvas id="gameCanvas" width="560" height="620" tabindex="0"></canvas>
<script>
(() => {
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');
  const scoreBoard = document.getElementById('scoreboard');
  const messageBox = document.getElementById('message');
  const messageText = document.getElementById('messageText');
  const restartBtn = document.getElementById('restartBtn');

  // ConfiguraÃ§Ãµes do jogo
  const TILE_SIZE = 20;
  const ROWS = 31;
  const COLS = 28;

  // Labirinto: 0=vide, 1=parede, 2=ponto
  // Mapa baseado no layout clÃ¡ssico do Pac-Man (simplificado)
  const map = [
    [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
    [1,2,2,2,2,2,2,2,2,2,2,2,1,1,1,2,2,2,2,2,2,2,2,2,2,2,2,1],
    [1,2,1,1,1,1,2,1,1,1,1,2,1,1,1,2,1,1,1,1,2,1,1,1,1,1,2,1],
    [1,2,1,1,1,1,2,1,1,1,1,2,1,1,1,2,1,1,1,1,2,1,1,1,1,1,2,1],
    [1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,1],
    [1,2,1,1,1,1,2,1,1,2,1,1,1,1,1,1,1,1,2,1,1,2,1,1,1,1,2,1],
    [1,2,2,2,2,2,2,1,1,2,2,2,2,1,1,2,2,2,2,1,1,2,2,2,2,2,2,1],
    [1,1,1,1,1,1,2,1,1,1,1,1,0,0,0,0,1,1,1,1,1,2,1,1,1,1,1,1],
    [0,0,0,0,0,1,2,1,1,0,0,0,0,0,0,0,0,0,1,1,2,1,0,0,0,0,0,0],
    [1,1,1,1,1,1,2,1,1,0,1,1,1,0,0,1,1,1,0,1,2,1,1,1,1,1,1,1],
    [0,0,0,0,0,0,2,0,0,0,1,0,0,0,0,0,0,1,0,0,2,0,0,0,0,0,0,0],
    [1,1,1,1,1,1,2,1,1,0,1,1,1,0,0,1,1,1,0,1,2,1,1,1,1,1,1,1],
    [0,0,0,0,0,1,2,1,1,0,0,0,0,0,0,0,0,0,0,1,2,1,0,0,0,0,0,0],
    [1,1,1,1,1,1,2,1,1,1,1,1,1,2,2,1,1,1,1,1,2,1,1,1,1,1,1,1],
    [1,2,2,2,2,2,2,2,2,2,2,2,1,1,1,2,2,2,2,2,2,2,2,2,2,2,2,1],
    [1,2,1,1,1,1,2,1,1,1,1,2,1,1,1,2,1,1,1,1,2,1,1,1,1,1,2,1],
    [1,2,2,2,1,1,2,2,2,2,1,2,2,2,2,2,1,2,2,2,2,1,1,2,2,2,2,1],
    [1,1,1,2,1,1,2,1,1,0,0,0,0,0,0,0,0,0,1,1,2,1,1,2,1,1,1,1],
    [1,1,1,2,1,1,2,1,1,0,1,1,1,0,0,1,1,0,1,1,2,1,1,2,1,1,1,1],
    [1,2,2,2,2,2,2,2,2,0,1,0,0,0,0,0,0,0,2,2,2,2,2,2,2,2,2,1],
    [1,2,1,1,1,1,2,1,1,0,1,1,1,1,1,1,1,0,1,1,2,1,1,1,1,1,2,1],
    [1,2,2,2,2,2,2,1,1,0,0,0,0,1,1,0,0,0,2,1,2,2,2,2,2,2,2,1],
    [1,1,1,1,1,1,2,1,1,1,1,1,2,1,1,2,1,1,1,1,2,1,1,1,1,1,1,1],
    [1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,1],
    [1,2,1,1,1,1,2,1,1,1,1,1,1,1,1,1,1,1,1,1,2,1,1,1,1,1,2,1],
    [1,2,2,2,1,1,2,2,2,2,2,2,1,1,1,2,2,2,2,2,2,1,1,2,2,2,2,1],
    [1,1,1,2,1,1,1,1,1,1,1,2,1,1,1,2,1,1,1,1,1,1,1,2,1,1,1,1],
    [1,2,2,2,2,2,2,2,2,2,1,2,2,2,2,2,1,2,2,2,2,2,2,2,2,2,2,1],
    [1,2,1,1,1,1,1,1,1,2,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,2,1],
    [1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,1],
    [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
  ];

  // Estado inicial
  let pacman = {
    x: 13,
    y: 23,
    dir: 'left',
    nextDir: null,
    mouth: 0,
    mouthOpen: true,
    speed: 5, // frames entre movimentaÃ§Ãµes
    frameCount: 0,
  };

  const ghosts = [
    { x: 13, y: 14, dir: 'left', color: '#FF0000', frightened: false, frightCount: 0 }, // Blinky
    { x: 14, y: 14, dir: 'right', color: '#FFB8FF', frightened: false, frightCount: 0 }, // Pinky
    { x: 12, y: 14, dir: 'up', color: '#00FFFF', frightened: false, frightCount: 0 }, // Inky
    { x: 15, y: 14, dir: 'down', color: '#FFB852', frightened: false, frightCount: 0 }, // Clyde
  ];

  let score = 0;
  let gameOver = false;
  let dotsTotal = 0;

  // Contabilizar pontos inicialmente
  map.forEach(row => row.forEach(tile => {
    if (tile === 2) dotsTotal++;
  }));

  // FunÃ§Ãµes auxiliares do jogo
  function canMove(x, y) {
    if (x < 0 || x >= COLS || y < 0 || y >= ROWS) return false;
    return map[y][x] !== 1;
  }

  function drawWall(x, y) {
    const posX = x * TILE_SIZE;
    const posY = y * TILE_SIZE;

    ctx.fillStyle = '#0a0a6c';
    ctx.fillRect(posX, posY, TILE_SIZE, TILE_SIZE);
    // paredes com brilho
    ctx.strokeStyle = '#ffffffcc';
    ctx.lineWidth = 1.5;
    ctx.strokeRect(posX + 2, posY + 2, TILE_SIZE - 4, TILE_SIZE - 4);
  }

  function drawDot(x, y) {
    const posX = x * TILE_SIZE + TILE_SIZE / 2;
    const posY = y * TILE_SIZE + TILE_SIZE / 2;

    ctx.fillStyle = '#fce903';
    ctx.beginPath();
    ctx.arc(posX, posY, 4, 0, Math.PI * 2);
    ctx.fill();
  }

  // Desenha o labirinto inteiro
  function drawMaze() {
    for (let y=0; y<ROWS; y++) {
      for (let x=0; x<COLS; x++) {
        if (map[y][x] === 1) {
          drawWall(x, y);
        } else {
          ctx.fillStyle = '#111';
          ctx.fillRect(x * TILE_SIZE, y * TILE_SIZE, TILE_SIZE, TILE_SIZE);
          if (map[y][x] === 2) {
            drawDot(x, y);
          }
        }
      }
    }
  }

  // Desenhar Pac-Man
  function drawPacman() {
    const posX = pacman.x * TILE_SIZE + TILE_SIZE / 2;
    const posY = pacman.y * TILE_SIZE + TILE_SIZE / 2;

    const radius = TILE_SIZE / 2 - 2;

    // Alterna boca aberta e fechada a cada frame
    if (pacman.frameCount % (pacman.speed * 2) === 0) {
      pacman.mouthOpen = !pacman.mouthOpen;
    }

    let mouthAngle = pacman.mouthOpen ? 0.25 : 0;

    // DireÃ§Ã£o da boca
    let startAngle, endAngle;
    switch(pacman.dir) {
      case 'right':
        startAngle = mouthAngle * Math.PI;
        endAngle = (2 - mouthAngle) * Math.PI;
        break;
      case 'left':
        startAngle = (1 + mouthAngle) * Math.PI;
        endAngle = (1 - mouthAngle) * Math.PI;
        break;
      case 'up':
        startAngle = (1.5 + mouthAngle) * Math.PI;
        endAngle = (1.5 - mouthAngle) * Math.PI;
        break;
      case 'down':
        startAngle = (0.5 + mouthAngle) * Math.PI;
        endAngle = (0.5 - mouthAngle) * Math.PI;
        break;
      default:
        startAngle = mouthAngle * Math.PI;
        endAngle = (2 - mouthAngle) * Math.PI;
    }

    ctx.fillStyle = '#fce903';
    ctx.beginPath();
    ctx.moveTo(posX, posY);
    ctx.arc(posX, posY, radius, startAngle, endAngle, false);
    ctx.closePath();
    ctx.fill();

    // Olho
    ctx.fillStyle = '#000';
    let eyeX = posX;
    let eyeY = posY;
    switch(pacman.dir) {
      case 'right':
        eyeX += radius / 2;
        eyeY -= radius / 1.5;
        break;
      case 'left':
        eyeX -= radius / 2;
        eyeY -= radius / 1.5;
        break;
      case 'up':
        eyeX += radius / 1.5;
        eyeY -= radius / 2;
        break;
      case 'down':
        eyeX += radius / 1.5;
        eyeY += radius / 2;
        break;
    }
    ctx.beginPath();
    ctx.arc(eyeX, eyeY, radius / 6, 0, Math.PI * 2);
    ctx.fill();
  }

  // Desenhar fantasmas
  function drawGhost(g) {
    const posX = g.x * TILE_SIZE + TILE_SIZE / 2;
    const posY = g.y * TILE_SIZE + TILE_SIZE / 2;

    const radius = TILE_SIZE / 2 - 2;

    // Corpo do fantasma
    ctx.fillStyle = g.frightened ? '#0000FF' : g.color;
    ctx.beginPath();
    ctx.moveTo(posX - radius, posY + radius/2);
    ctx.lineTo(posX - radius, posY - radius/2);
    ctx.quadraticCurveTo(posX, posY - radius, posX + radius, posY - radius/2);
    ctx.lineTo(posX + radius, posY + radius/2);
    ctx.quadraticCurveTo(posX + radius*0.7, posY + radius, posX + radius/2, posY + radius*0.3);
    ctx.lineTo(posX + radius/4, posY + radius);
    ctx.lineTo(posX, posY + radius*0.3);
    ctx.lineTo(posX - radius/4, posY + radius);
    ctx.lineTo(posX - radius/2, posY + radius*0.3);
    ctx.quadraticCurveTo(posX - radius*0.7, posY + radius, posX - radius, posY + radius/2);
    ctx.fill();

    // Olhos
    ctx.fillStyle = '#fff';
    const eyeOffset = radius / 2.2;
    const eyeRadius = radius / 5;

    ctx.beginPath();
    ctx.ellipse(posX - eyeOffset, posY - radius/6, eyeRadius, eyeRadius*1.4, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.beginPath();
    ctx.ellipse(posX + eyeOffset, posY - radius/6, eyeRadius, eyeRadius*1.4, 0, 0, Math.PI * 2);
    ctx.fill();

    // Pupilas
    ctx.fillStyle = g.frightened ? '#fff' : '#000';

    let pupilXOffset = 0;
    let pupilYOffset = 0;

    switch(g.dir) {
      case 'left': pupilXOffset = -eyeRadius/2; break;
      case 'right': pupilXOffset = eyeRadius/2; break;
      case 'up': pupilYOffset = -eyeRadius/3; break;
      case 'down': pupilYOffset = eyeRadius/3; break;
    }
    ctx.beginPath();
    ctx.ellipse(posX - eyeOffset + pupilXOffset, posY - radius/6 + pupilYOffset, eyeRadius/2, eyeRadius/2, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.beginPath();
    ctx.ellipse(posX + eyeOffset + pupilXOffset, posY - radius/6 + pupilYOffset, eyeRadius/2, eyeRadius/2, 0, 0, Math.PI * 2);
    ctx.fill();
  }

  function movePacman() {
    // Confirma se direÃ§Ã£o desejada pode ser tomada
    if (pacman.nextDir) {
      let nextX = pacman.x;
      let nextY = pacman.y;
      switch(pacman.nextDir) {
        case 'left': nextX--; break;
        case 'right': nextX++; break;
        case 'up': nextY--; break;
        case 'down': nextY++; break;
      }
      if (canMove(nextX, nextY)) {
        pacman.dir = pacman.nextDir;
        pacman.nextDir = null;
      }
    }

    let newX = pacman.x;
    let newY = pacman.y;
    switch(pacman.dir) {
      case 'left': newX--; break;
      case 'right': newX++; break;
      case 'up': newY--; break;
      case 'down': newY++; break;
    }

    if (canMove(newX, newY)) {
      pacman.x = newX;
      pacman.y = newY;

      // Controle tÃºnel lateral
      if (pacman.x < 0) pacman.x = COLS - 1;
      if (pacman.x >= COLS) pacman.x = 0;

      // Come pontos
      if (map[pacman.y][pacman.x] === 2) {
        map[pacman.y][pacman.x] = 0;
        score++;
        scoreBoard.textContent = 'PontuaÃ§Ã£o: ' + score;

        if (score === dotsTotal) {
          endGame(true);
        }
      }
    }
  }

  // Movimenta fantasmas com lÃ³gica simples aleatÃ³ria com preferÃªncia na direÃ§Ã£o atual
  function moveGhost(g) {
    if (g.frightened) { // modo assustado: fugir do Pacman
      fleePacman(g);
    } else {
      chasePacman(g);
    }
  }

  // FunÃ§Ã£o para fugir do Pacman mantendo movimento vÃ¡lido
  function fleePacman(g) {
    let dx = g.x - pacman.x;
    let dy = g.y - pacman.y;
    // tenta mover na direÃ§Ã£o que aumenta distÃ¢ncia
    let possibleDirs = [];
    if (canMove(g.x + 1, g.y)) possibleDirs.push({dir:'right', dist:Math.hypot(g.x+1-pacman.x,g.y-pacman.y)});
    if (canMove(g.x - 1, g.y)) possibleDirs.push({dir:'left', dist:Math.hypot(g.x-1-pacman.x,g.y-pacman.y)});
    if (canMove(g.x, g.y + 1)) possibleDirs.push({dir:'down', dist:Math.hypot(g.x-pacman.x,g.y+1-pacman.y)});
    if (canMove(g.x, g.y -1)) possibleDirs.push({dir:'up', dist:Math.hypot(g.x-pacman.x,g.y-1-pacman.y)});

    if (possibleDirs.length === 0) return;

    possibleDirs.sort((a,b) => b.dist - a.dist);
    let chosen = possibleDirs[0];

    g.dir = chosen.dir;
    moveInDir(g);
  }

  // FunÃ§Ã£o para perseguir Pacman com movimentaÃ§Ã£o simples
  function chasePacman(g) {
    const directions = ['left','right','up','down'];
    // Tenta mover na direÃ§Ã£o que reduz distÃ¢ncia
    let possibleDirs = [];
    directions.forEach(dir => {
      let nx = g.x;
      let ny = g.y;
      switch(dir) {
        case 'left': nx--; break;
        case 'right': nx++; break;
        case 'up': ny--; break;
        case 'down': ny++; break;
      }
      if(canMove(nx, ny)) {
        const dist = Math.hypot(nx - pacman.x, ny - pacman.y);
        possibleDirs.push({dir, dist});
      }
    });

    if (possibleDirs.length === 0) return;

    possibleDirs.sort((a,b) => a.dist - b.dist);
    // Escolhe a direÃ§Ã£o que mais se aproxima de Pacman
    let chosen = possibleDirs[0];
    g.dir = chosen.dir;
    moveInDir(g);
  }

  function moveInDir(g) {
    switch(g.dir) {
      case 'left': if (canMove(g.x - 1, g.y)) g.x--; break;
      case 'right': if (canMove(g.x + 1, g.y)) g.x++; break;
      case 'up': if (canMove(g.x, g.y - 1)) g.y--; break;
      case 'down': if (canMove(g.x, g.y + 1)) g.y++; break;
    }

    // Controle tÃºnel lateral
    if (g.x < 0) g.x = COLS - 1;
    if (g.x >= COLS) g.x = 0;
  }

  // Detecta colisÃ£o Pacman-Fantasma
  function checkCollision() {
    for (let g of ghosts) {
      if (g.x === pacman.x && g.y === pacman.y) {
        if (g.frightened) {
          // Fantasma Ã© comido, voltar ao centro apÃ³s um tempo
          g.x = 13;
          g.y = 14;
          g.frightened = false;
          score += 10;
          scoreBoard.textContent = 'PontuaÃ§Ã£o: ' + score;
        } else {
          endGame(false);
        }
      }
    }
  }

  // FunÃ§Ã£o para finalizar jogo
  function endGame(win) {
    gameOver = true;
    messageBox.style.display = 'block';
    messageText.textContent = win ? 'VocÃª venceu! ParabÃ©ns!' : 'Game Over! Tente novamente.';
  }

  // Atualizar estado do jogo
  function update() {
    if (gameOver) return;

    pacman.frameCount++;
    if (pacman.frameCount % pacman.speed === 0) {
      movePacman();

      ghosts.forEach(g => {
        if (!g.frightened) {
          // A cada 20 frames, aleatoriamente fantasma entra em modo assustado por pouco tempo
          if (Math.random() < 0.01) {
            g.frightened = true;
            g.frightCount = 60; // duraÃ§Ã£o do modo assustado (frames)
          }
        }
        if (g.frightened) {
          g.frightCount--;
          if (g.frightCount <= 0) {
            g.frightened = false;
          }
        }
        moveGhost(g);
      });

      checkCollision();
    }
  }

  // Limpar Tela
  function clear() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  }

  // FunÃ§Ã£o principal de desenho
  function draw() {
    clear();
    drawMaze();
    drawPacman();
    ghosts.forEach(drawGhost);
  }

  // Loop do jogo
  function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
  }

  // Controle teclado para movimentar Pacman
  canvas.addEventListener('keydown', (e) => {
    if (gameOver) return;
    switch(e.key) {
      case 'ArrowLeft':
        pacman.nextDir = 'left';
        break;
      case 'ArrowRight':
        pacman.nextDir = 'right';
        break;
      case 'ArrowUp':
        pacman.nextDir = 'up';
        break;
      case 'ArrowDown':
        pacman.nextDir = 'down';
        break;
    }
  });

  // Focar canvas para permitir captura de teclado
  canvas.focus();

  restartBtn.addEventListener('click', () => {
    // Resetar jogo
    for(let y=0; y<ROWS; y++) {
      for(let x=0; x<COLS; x++) {
        if(map[y][x] === 0) map[y][x] = 2;
      }
    }
    // Recontar pontos
    dotsTotal = 0;
    map.forEach(row => row.forEach(tile => {
      if (tile === 2) dotsTotal++;
    }));

    pacman = {
      x: 13,
      y: 23,
      dir: 'left',
      nextDir: null,
      mouth: 0,
      mouthOpen: true,
      speed: 5,
      frameCount: 0,
    };
    ghosts.forEach(g => {
      g.x = 13;
      g.y = 14;
      g.dir = 'left';
      g.frightened = false;
      g.frightCount = 0;
    });
    score = 0;
    gameOver = false;
    messageBox.style.display = 'none';
    scoreBoard.textContent = 'PontuaÃ§Ã£o: 0';
    canvas.focus();
  });

  // Inicia loop do jogo
  loop();

})();
<\/script>
</body>
</html>
`;

/**
 * Cria e baixa o arquivo pacman.html com conteÃºdo pacmanHTML
 */
function downloadPacman() {
  const blob = new Blob([pacmanHTML], {type: 'text/html'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');

  a.href = url;
  a.download = 'pacman.html';
  document.body.appendChild(a);
  a.click();

  setTimeout(() => {
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }, 100);
}

// Exporta funÃ§Ã£o para uso global
window.downloadPacman = downloadPacman;
