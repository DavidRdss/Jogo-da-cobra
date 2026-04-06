# Jogo-da-cobra
é um codigo baseado no jogo da cobrinha onde vc controla a cobra para que ela coma as maças e vá aumentando de tamanho

<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Jogo da Cobrinha</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #4e1616;
      color: white;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }

   .box {
      background: #181a3d;
      padding: 10px;
      border-radius: 10px;
      text-align: center;
      width: 80%;
      max-width: 420px;
    }

   .info, .buttons {
      display: flex;
      justify-content: space-between;
      gap: 10px;
      margin: 10px 0;
      flex-wrap: wrap;
    }

   button {
      flex: 1;
      padding: 10px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
    }

   canvas {
      background: #0f172a;
      display: block;
      margin: 15px auto;
      border: 2px solid #334155;
      width: 100%;
      max-width: 500px;
    }

   .status {
      margin-top: 10px;
      min-height: 20px;
    }
  </style>
</head>
<body>
  <div class="box">
    <h1>Jogo da Cobrinha</h1>

   <div class="info">
      <div>Pontos: <span id="score">0</span></div>
    </div>

   <div class="buttons">
      <button id="startBtn">Iniciar</button>
      <button id="solverBtn">Solver: OFF</button>
      <button id="resetBtn">Reiniciar</button>
    </div>

   <canvas id="game" width="600" height="600"></canvas>

   <p>Use as setas para jogar. O solver tenta ir até a comida sozinho.</p>
    <div class="status" id="status"></div>
  </div>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");

    const scoreEl = document.getElementById("score");
    const statusEl = document.getElementById("status");
    const startBtn = document.getElementById("startBtn");
    const solverBtn = document.getElementById("solverBtn");
    const resetBtn = document.getElementById("resetBtn");

    const grid = 20;
    const size = canvas.width / grid;
    const speed = 120;

    let snake, food, direction, nextDirection;
    let score = 0;
    let running = false;
    let solverOn = false;
    let gameOver = false;
    let gameLoop;

    function startGame() {
      snake = [
        { x: 10, y: 10 },
        { x: 9, y: 10 },
        { x: 8, y: 10 }
      ];

      food = { x: 0, y: 0 };
      direction = { x: 1, y: 0 };
      nextDirection = { x: 1, y: 0 };
      score = 0;
      running = false;
      solverOn = false;
      gameOver = false;

      scoreEl.textContent = score;
      statusEl.textContent = "";
      startBtn.textContent = "Iniciar";
      solverBtn.textContent = "Solver: OFF";

      createFood();
      drawGame();

      clearInterval(gameLoop);
    }

    function createFood() {
      do {
        food.x = Math.floor(Math.random() * grid);
        food.y = Math.floor(Math.random() * grid);
      } while (snake.some(p => p.x === food.x && p.y === food.y));
    }

    function insideBoard(x, y) {
      return x >= 0 && x < grid && y >= 0 && y < grid;
    }

    function hitBody(x, y) {
      return snake.some((p, i) => i < snake.length - 1 && p.x === x && p.y === y);
    }

    function solverMove() {
      const head = snake[0];
      const dx = food.x - head.x;
      const dy = food.y - head.y;

      let moves = [];

      if (Math.abs(dx) > Math.abs(dy)) {
        moves = [
          { x: dx > 0 ? 1 : -1, y: 0 },
          { x: 0, y: dy > 0 ? 1 : -1 }
        ];
      } else {
        moves = [
          { x: 0, y: dy > 0 ? 1 : -1 },
          { x: dx > 0 ? 1 : -1, y: 0 }
        ];
      }

      moves.push(
        { x: 1, y: 0 },
        { x: -1, y: 0 },
        { x: 0, y: 1 },
        { x: 0, y: -1 }
      );

      for (let move of moves) {
        let nx = head.x + move.x;
        let ny = head.y + move.y;

        if (insideBoard(nx, ny) && !hitBody(nx, ny)) {
          return move;
        }
      }

      return direction;
    }

    function updateGame() {
      if (!running || gameOver) return;

      if (solverOn) {
        nextDirection = solverMove();
      }

      if (nextDirection.x !== -direction.x || nextDirection.y !== -direction.y) {
        direction = nextDirection;
      }

      let newHead = {
        x: snake[0].x + direction.x,
        y: snake[0].y + direction.y
      };

      if (!insideBoard(newHead.x, newHead.y) || hitBody(newHead.x, newHead.y)) {
        gameOver = true;
        running = false;
        clearInterval(gameLoop);
        startBtn.textContent = "Iniciar";
        statusEl.textContent = "Fim de jogo!";
        drawGame();
        return;
      }

      snake.unshift(newHead);

      if (newHead.x === food.x && newHead.y === food.y) {
        score++;
        scoreEl.textContent = score;
        createFood();
      } else {
        snake.pop();
      }

      drawGame();
    }

    function drawGame() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = "#0f172a";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      ctx.strokeStyle = "#1e293b";
      for (let i = 0; i <= grid; i++) {
        ctx.beginPath();
        ctx.moveTo(i * size, 0);
        ctx.lineTo(i * size, canvas.height);
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(0, i * size);
        ctx.lineTo(canvas.width, i * size);
        ctx.stroke();
      }

      ctx.fillStyle = "#f43f5e";
      ctx.fillRect(food.x * size + 3, food.y * size + 3, size - 6, size - 6);

      snake.forEach((part, i) => {
        ctx.fillStyle = i === 0 ? "#86efac" : "#4ade80";
        ctx.fillRect(part.x * size + 2, part.y * size + 2, size - 4, size - 4);
      });

      if (!running && !gameOver) {
        ctx.fillStyle = "rgba(0,0,0,0.45)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "white";
        ctx.font = "bold 26px Arial";
        ctx.textAlign = "center";
        ctx.fillText("Clique em Iniciar", canvas.width / 2, canvas.height / 2);
      }

      if (gameOver) {
        ctx.fillStyle = "rgba(0,0,0,0.55)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "white";
        ctx.font = "bold 30px Arial";
        ctx.textAlign = "center";
        ctx.fillText("HAHA Perdeu!", canvas.width / 2, canvas.height / 2);
      }
    }

    function toggleGame() {
      if (gameOver) return;

      if (!running) {
        running = true;
        startBtn.textContent = "Pausar";
        statusEl.textContent = solverOn ? "Solver ligado." : "Jogo iniciado.";
        gameLoop = setInterval(updateGame, speed);
      } else {
        running = false;
        startBtn.textContent = "Continuar";
        statusEl.textContent = "Pausado.";
        clearInterval(gameLoop);
        drawGame();
      }
    }

    document.addEventListener("keydown", (event) => {
      if (event.key === "ArrowUp") nextDirection = { x: 0, y: -1 };
      if (event.key === "ArrowDown") nextDirection = { x: 0, y: 1 };
      if (event.key === "ArrowLeft") nextDirection = { x: -1, y: 0 };
      if (event.key === "ArrowRight") nextDirection = { x: 1, y: 0 };
      if (event.key === " ") toggleGame();
    });

    startBtn.onclick = toggleGame;

    solverBtn.onclick = () => {
      solverOn = !solverOn;
      solverBtn.textContent = solverOn ? "Solver: ON" : "Solver: OFF";
      statusEl.textContent = solverOn ? "A cobra está jogando sozinha." : "Solver desligado.";
    };

    resetBtn.onclick = startGame;

    startGame();
  </script>
</body>
</html>
