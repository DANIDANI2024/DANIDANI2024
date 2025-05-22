// === Estados do jogo (para controle de telas e lógica de pausa/morte) ===
const STATE = {
    START: 'start',       // Tela inicial
    PLAYING: 'playing',   // Jogando
    PAUSED: 'paused',     // Pausado
    GAMEOVER: 'gameover'  // Fim de jogo
  };
  
  let snake;             // Instância da cobrinha
  let food;              // Vetor com a posição da comida
  let gridSize = 20;     // Tamanho de cada "bloco" da grade
  let cols, rows;        // Número de colunas e linhas na grade
  let score = 0;         // Pontuação atual
  let gameState = STATE.START; // Estado atual do jogo
  let baseSpeed = 10;    // Velocidade base (FPS)
  let foodAngle = 0;     // Ângulo de rotação da comida (efeito visual)
  
  function setup() {
    createCanvas(400,400);       // Define o tamanho da tela
    textFont('monospace');        // Define a fonte usada
    resetGame();                  // Inicializa ou reinicia o jogo
  }
  
  // === Reinicia o jogo ===
  function resetGame() {
    cols = floor(width / gridSize);  // Calcula número de colunas
    rows = floor(height / gridSize); // Calcula número de linhas
    snake = new Snake();             // Cria nova cobrinha
    food = createFood();             // Gera nova comida
    score = 0;                       // Reinicia a pontuação
    foodAngle = 0;                   // Reinicia ângulo de rotação da comida
    changeState(STATE.START);       // Vai para tela inicial
  }
  
  // === Troca de estado do jogo ===
  function changeState(newState) {
    gameState = newState;
    if (newState === STATE.PLAYING) {
      loop(); // Continua o draw()
      frameRate(baseSpeed + floor(score / 5)); // Aumenta a velocidade conforme a pontuação
    } else if (newState === STATE.PAUSED || newState === STATE.GAMEOVER) {
      noLoop(); // Pausa o draw()
    }
  }
  
  function draw() {
    background(20); 
  
    switch (gameState) {
      case STATE.START:
        drawStartScreen(); // Tela inicial
        break;
      case STATE.PLAYING:
        snake.update();    // Atualiza posição da cobra
        snake.show();      // Desenha a cobra
  
        // Verifica se a cobra comeu a comida
        if (snake.eat(food)) {
          food = createFood(); // Cria nova comida
          score++;             // Aumenta pontuação
          frameRate(baseSpeed + floor(score / 5)); // Aumenta velocidade gradualmente
        }
  
        drawFood(food);    // Desenha a comida
        drawScore();       // Exibe a pontuação
  
        // Verifica se a cobra morreu
        if (snake.dead()) {
          changeState(STATE.GAMEOVER); // Fim de jogo
        }
        break;
  
      case STATE.GAMEOVER:
        drawGameOver(); // Tela de fim de jogo
        break;
  
      case STATE.PAUSED:
        drawPauseScreen(); // Tela de pausa
        break;
    }
  }
  
  // === Controla as teclas pressionadas ===
  function keyPressed() {
    if (gameState === STATE.START && keyCode === ENTER) {
      changeState(STATE.PLAYING); // Começa o jogo
    } else if (gameState === STATE.GAMEOVER && keyCode === ENTER) {
      resetGame();                // Reinicia o jogo
      changeState(STATE.PLAYING);
    } else if (gameState === STATE.PLAYING) {
      if (key === ' ') {
        changeState(STATE.PAUSED); // Pausa
      }
      snake.handleInput(keyCode);  // Move a cobra
    } else if (gameState === STATE.PAUSED && key === ' ') {
      changeState(STATE.PLAYING); // Retoma o jogo
    }
  }
  
  // === Gera comida em uma posição aleatória ===
  function createFood() {
    let x = floor(random(cols)) * gridSize;
    let y = floor(random(rows)) * gridSize;
    return createVector(x, y); // Retorna vetor com a posição
  }
  
  // === Desenha a comida com rotação e pulsação ===
  function drawFood(pos) {
    push();
    translate(pos.x + gridSize / 2, pos.y + gridSize / 2); // Move para o centro do bloco
    rotate(foodAngle);             // Aplica rotação
    let scalePulse = 0.9 + 0.1 * sin(frameCount * 0.2); // Efeito pulsante
    scale(scalePulse);
    foodAngle += 0.05;             // Aumenta ângulo gradualmente
    fill(255, 100, 150);           // Cor da comida
    stroke(255);                   // Contorno
    strokeWeight(1);
    rectMode(CENTER);              // Desenha o retângulo a partir do centro
    rect(0, 0, gridSize * 0.8, gridSize * 0.8, 5); // Forma da comida
    pop();
  }
  
  // === Desenha a pontuação no canto superior ===
  function drawScore() {
    fill(255);
    textSize(16);
    textAlign(LEFT, TOP);
    text("Score: " + score, 10, 10);
  }
  
  // === Tela inicial ===
  function drawStartScreen() {
    fill(255);
    textAlign(CENTER, CENTER);
    textSize(24);
    text("🐍 Snake Neon 🐍", width / 2, height / 2 - 40);
    textSize(16);
    text("Setas: mover | Espaço: pausa", width / 2, height / 2);
    text("ENTER para começar", width / 2, height / 2 + 30);
  }
  
  // === Tela de fim de jogo ===
  function drawGameOver() {
    background(0, 0, 100); // Fundo azul escuro
    fill(255);
    textAlign(CENTER, CENTER);
    textSize(32);
    text("GAME OVER", width / 2, height / 2);
  }
  
  // === Tela de pausa ===
  function drawPauseScreen() {
    fill(255);
    textAlign(CENTER, CENTER);
    textSize(24);
    text("⏸️ Pausado", width / 2, height / 2 - 10);
    textSize(16);
    text("Pressione espaço para continuar", width / 2, height / 2 + 20);
  }
  
  // === Classe Snake que representa a cobrinha ===
  class Snake {
    constructor() {
      this.body = [createVector(floor(width / 2), floor(height / 2))]; // Começa no centro
      this.xdir = 0;   // Direção X inicial (parado)
      this.ydir = 0;   // Direção Y inicial (parado)
      this.len = 1;    // Tamanho inicial
    }
  
    // Define direção com base nos inputs
    setDir(x, y) {
      this.xdir = x;
      this.ydir = y;
    }
  
    // Controla a direção da cobra pelas teclas
    handleInput(keyCode) {
      if (keyCode === UP_ARROW && this.ydir !== 1) {
        this.setDir(0, -1);
      } else if (keyCode === DOWN_ARROW && this.ydir !== -1) {
        this.setDir(0, 1);
      } else if (keyCode === LEFT_ARROW && this.xdir !== 1) {
        this.setDir(-1, 0);
      } else if (keyCode === RIGHT_ARROW && this.xdir !== -1) {
        this.setDir(1, 0);
      }
    }
  
    // Atualiza posição da cabeça e movimenta o corpo
    update() {
      let head = this.body[this.body.length - 1].copy(); // Copia a cabeça
      head.x += this.xdir * gridSize; // Move na direção X
      head.y += this.ydir * gridSize; // Move na direção Y
      this.body.push(head);           // Adiciona nova cabeça
      if (this.body.length > this.len) {
        this.body.shift();           // Remove o bloco mais antigo se necessário
      }
    }
  
    // Aumenta o tamanho da cobra
    grow() {
      this.len++;
    }
  
    // Verifica se comeu a comida
    eat(pos) {
      let head = this.body[this.body.length - 1];
      if (dist(head.x, head.y, pos.x, pos.y) < 1) {
        this.grow();
        return true;
      }
      return false;
    }
  
    // Verifica se morreu (colidiu com parede ou corpo)
    dead() {
      let head = this.body[this.body.length - 1];
      // Saiu da tela
      if (head.x < 0 || head.x >= width || head.y < 0 || head.y >= height) {
        return true;
      }
      // Bateu no próprio corpo
      for (let i = 0; i < this.body.length - 1; i++) {
        if (head.x === this.body[i].x && head.y === this.body[i].y) {
          return true;
        }
      }
      return false;
    }
  
    // Desenha a cobra com efeito neon
    show() {
      noStroke();
      for (let i = 0; i < this.body.length; i++) {
        let bright = map(i, 0, this.body.length, 100, 255); // Gradiente de brilho
        if (i === this.body.length - 1) {
          fill(255, 0, 0); // Cabeça em vermelho
        } else {
          fill(0, bright, 255 - bright, 180); // Corpo em azul com transparência
        }
        rect(this.body[i].x, this.body[i].y, gridSize, gridSize, 4); // Corpo arredondado
      }
    }
  }
