<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Flappy Bird Mobile</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: skyblue; font-family: sans-serif; overflow: hidden; touch-action: manipulation; }

    canvas {
      display: block;
      margin: auto;
      background: #70c5ce;
    }

    #scoreBoard {
      position: absolute;
      top: 10px;
      left: 50%;
      transform: translateX(-50%);
      color: white;
      font-size: 20px;
      font-weight: bold;
      text-align: center;
    }

    #gameOver {
      display: none;
      position: absolute;
      top: 40%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: red;
      font-size: 32px;
      font-weight: bold;
    }

    #restartBtn {
      display: none;
      position: absolute;
      top: 52%;
      left: 50%;
      transform: translate(-50%, -50%);
      padding: 10px 20px;
      font-size: 20px;
      cursor: pointer;
      border: none;
      background-color: white;
      color: #333;
      border-radius: 10px;
    }
  </style>
</head>
<body>
  <div id="scoreBoard">
    Score: <span id="score">0</span><br>
    High Score: <span id="highScore">0</span>
  </div>
  <div id="gameOver">Game Over!</div>
  <button id="restartBtn">Restart</button>
  <canvas id="gameCanvas" width="400" height="600"></canvas>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const birdImg = new Image();
    birdImg.src = 'bird.png';

    const flapSound = new Audio('flap.wav');
    const hitSound = new Audio('hit.wav');
    const pointSound = new Audio('point.wav');

    let bird = {
      x: 80,
      y: 150,
      width: 34,
      height: 34,
      gravity: 0.6,
      lift: -10,
      velocity: 0
    };

    let pipes = [];
    let score = 0;
    let highScore = localStorage.getItem("highScore") || 0;
    document.getElementById("highScore").innerText = highScore;
    let gameRunning = true;

    function drawBird() {
      ctx.drawImage(birdImg, bird.x, bird.y, bird.width, bird.height);
    }

    function createPipe() {
      const gap = 140;
      const top = Math.floor(Math.random() * (canvas.height - gap - 100)) + 20;
      pipes.push({
        x: canvas.width,
        top: top,
        bottom: top + gap,
        width: 50
      });
    }

    function drawPipes() {
      ctx.fillStyle = 'green';
      pipes.forEach(pipe => {
        ctx.fillRect(pipe.x, 0, pipe.width, pipe.top);
        ctx.fillRect(pipe.x, pipe.bottom, pipe.width, canvas.height - pipe.bottom);
      });
    }

    function updatePipes() {
      pipes.forEach(pipe => {
        pipe.x -= 2;

        // Increase score
        if (pipe.x + pipe.width === bird.x) {
          score++;
          pointSound.play();
          document.getElementById('score').innerText = score;
          if (score > highScore) {
            highScore = score;
            localStorage.setItem("highScore", highScore);
            document.getElementById("highScore").innerText = highScore;
          }
        }

        // Collision detection
        if (
          bird.x < pipe.x + pipe.width &&
          bird.x + bird.width > pipe.x &&
          (bird.y < pipe.top || bird.y + bird.height > pipe.bottom)
        ) {
          hitSound.play();
          endGame();
        }
      });

      pipes = pipes.filter(pipe => pipe.x + pipe.width > 0);
    }

    function endGame() {
      gameRunning = false;
      document.getElementById('gameOver').style.display = 'block';
      document.getElementById('restartBtn').style.display = 'block';
    }

    function gameLoop() {
      if (!gameRunning) return;

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      bird.velocity += bird.gravity;
      bird.y += bird.velocity;

      if (bird.y + bird.height > canvas.height || bird.y < 0) {
        hitSound.play();
        endGame();
      }

      drawBird();
      drawPipes();
      updatePipes();

      requestAnimationFrame(gameLoop);
    }

    function flap() {
      bird.velocity = bird.lift;
      flapSound.play();
    }

    document.addEventListener('keydown', e => {
      if (e.code === 'Space') flap();
    });

    canvas.addEventListener('click', flap);
    canvas.addEventListener('touchstart', flap);

    document.getElementById('restartBtn').addEventListener('click', () => {
      bird.y = 150;
      bird.velocity = 0;
      pipes = [];
      score = 0;
      document.getElementById('score').innerText = score;
      document.getElementById('gameOver').style.display = 'none';
      document.getElementById('restartBtn').style.display = 'none';
      gameRunning = true;
      gameLoop();
    });

    setInterval(() => {
      if (gameRunning) createPipe();
    }, 1600);

    gameLoop();
  </script>
</body>
</html>
