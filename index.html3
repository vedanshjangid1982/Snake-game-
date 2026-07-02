/* =========================================================
   SNAKE GAME - Vanilla JavaScript
   Features: keyboard + touch controls, difficulty levels,
   sound effects (Web Audio API - no external assets),
   localStorage high score, pause/resume/restart, responsive canvas.
========================================================= */

(() => {
  'use strict';

  // ---------- DOM references ----------
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  const scoreEl = document.getElementById('score');
  const highScoreEl = document.getElementById('highScore');
  const finalScoreEl = document.getElementById('finalScore');
  const newHighScoreMsg = document.getElementById('newHighScoreMsg');

  const startOverlay = document.getElementById('startOverlay');
  const pauseOverlay = document.getElementById('pauseOverlay');
  const gameOverOverlay = document.getElementById('gameOverOverlay');

  const startBtn = document.getElementById('startBtn');
  const resumeBtn = document.getElementById('resumeBtn');
  const playAgainBtn = document.getElementById('playAgainBtn');
  const pauseBtn = document.getElementById('pauseBtn');
  const restartBtn = document.getElementById('restartBtn');
  const muteBtn = document.getElementById('muteBtn');
  const muteIcon = document.getElementById('muteIcon');
  const diffButtons = document.querySelectorAll('.diff-btn');
  const touchButtons = document.querySelectorAll('.touch-btn');

  // ---------- Game configuration ----------
  const GRID_SIZE = 20; // number of cells per row/column
  const DIFFICULTY_SPEED = {
    easy: 150,   // ms per tick (slower)
    medium: 100,
    hard: 60     // faster
  };

  let cellPixelSize = 0;   // computed based on canvas size
  let speedMs = DIFFICULTY_SPEED.easy;
  let currentDifficulty = 'easy';

  // ---------- Game state ----------
  let snake = [];
  let direction = { x: 1, y: 0 };
  let nextDirection = { x: 1, y: 0 };
  let food = { x: 0, y: 0 };
  let score = 0;
  let highScore = 0;
  let isRunning = false;
  let isPaused = false;
  let isGameOver = false;
  let loopTimer = null;
  let isMuted = false;

  // ---------- localStorage helpers ----------
  function loadHighScore() {
    const stored = localStorage.getItem('snake_highScore');
    highScore = stored ? parseInt(stored, 10) : 0;
    highScoreEl.textContent = highScore;
  }

  function saveHighScoreIfNeeded() {
    if (score > highScore) {
      highScore = score;
      localStorage.setItem('snake_highScore', highScore.toString());
      highScoreEl.textContent = highScore;
      return true;
    }
    return false;
  }

  function loadMutePreference() {
    const stored = localStorage.getItem('snake_muted');
    isMuted = stored === 'true';
    updateMuteIcon();
  }

  function saveMutePreference() {
    localStorage.setItem('snake_muted', isMuted.toString());
  }

  // ---------- Sound (Web Audio API, no external files) ----------
  let audioCtx = null;

  function getAudioContext() {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }
    return audioCtx;
  }

  function playTone(frequency, duration, type = 'sine', volume = 0.15) {
    if (isMuted) return;
    try {
      const ctxAudio = getAudioContext();
      const oscillator = ctxAudio.createOscillator();
      const gainNode = ctxAudio.createGain();

      oscillator.type = type;
      oscillator.frequency.value = frequency;
      gainNode.gain.value = volume;

      oscillator.connect(gainNode);
      gainNode.connect(ctxAudio.destination);

      oscillator.start();
      gainNode.gain.exponentialRampToValueAtTime(0.0001, ctxAudio.currentTime + duration);
      oscillator.stop(ctxAudio.currentTime + duration);
    } catch (e) {
      // Audio not supported / blocked - fail silently
    }
  }

  function playEatSound() {
    playTone(660, 0.12, 'square', 0.12);
  }

  function playGameOverSound() {
    playTone(220, 0.15, 'sawtooth', 0.15);
    setTimeout(() => playTone(160, 0.25, 'sawtooth', 0.15), 150);
  }

  function playTurnSound() {
    playTone(440, 0.04, 'sine', 0.05);
  }

  function updateMuteIcon() {
    muteIcon.textContent = isMuted ? '🔇' : '🔊';
  }

  // ---------- Canvas sizing (responsive) ----------
  function resizeCanvas() {
    const container = canvas.parentElement;
    const size = Math.min(container.clientWidth, container.clientHeight);
    cellPixelSize = Math.floor(size / GRID_SIZE);
    const finalSize = cellPixelSize * GRID_SIZE;

    // Use devicePixelRatio for crisp rendering
    const dpr = window.devicePixelRatio || 1;
    canvas.width = finalSize * dpr;
    canvas.height = finalSize * dpr;
    canvas.style.width = finalSize + 'px';
    canvas.style.height = finalSize + 'px';
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);

    draw(); // redraw immediately after resize
  }

  // ---------- Game initialization ----------
  function resetGame() {
    const mid = Math.floor(GRID_SIZE / 2);
    snake = [
      { x: mid - 1, y: mid },
      { x: mid - 2, y: mid },
      { x: mid - 3, y: mid }
    ];
    direction = { x: 1, y: 0 };
    nextDirection = { x: 1, y: 0 };
    score = 0;
    scoreEl.textContent = score;
    isGameOver = false;
    isPaused = false;
    placeFood();
  }

  function placeFood() {
    let validPosition = false;
    let newFood;
    while (!validPosition) {
      newFood = {
        x: Math.floor(Math.random() * GRID_SIZE),
        y: Math.floor(Math.random() * GRID_SIZE)
      };
      validPosition = !snake.some(seg => seg.x === newFood.x && seg.y === newFood.y);
    }
    food = newFood;
  }

  // ---------- Game loop ----------
  function startLoop() {
    stopLoop();
    loopTimer = setInterval(tick, speedMs);
  }

  function stopLoop() {
    if (loopTimer) {
      clearInterval(loopTimer);
      loopTimer = null;
    }
  }

  function tick() {
    if (isPaused || isGameOver || !isRunning) return;

    direction = nextDirection;

    const head = snake[0];
    const newHead = { x: head.x + direction.x, y: head.y + direction.y };

    // Wall collision
    if (
      newHead.x < 0 || newHead.x >= GRID_SIZE ||
      newHead.y < 0 || newHead.y >= GRID_SIZE
    ) {
      return endGame();
    }

    // Self collision
    if (snake.some(seg => seg.x === newHead.x && seg.y === newHead.y)) {
      return endGame();
    }

    snake.unshift(newHead);

    // Food collision
    if (newHead.x === food.x && newHead.y === food.y) {
      score += 10;
      scoreEl.textContent = score;
      playEatSound();
      placeFood();
    } else {
      snake.pop();
    }

    draw();
  }

  function endGame() {
    isGameOver = true;
    isRunning = false;
    stopLoop();
    playGameOverSound();

    const isNewHigh = saveHighScoreIfNeeded();
    finalScoreEl.textContent = score;
    newHighScoreMsg.classList.toggle('hidden', !isNewHigh);

    gameOverOverlay.classList.remove('hidden');
    pauseBtn.disabled = true;
  }

  // ---------- Drawing ----------
  function draw() {
    const size = cellPixelSize * GRID_SIZE;
    ctx.clearRect(0, 0, size, size);

    // Background
    ctx.fillStyle = '#111827';
    ctx.fillRect(0, 0, size, size);

    // Grid lines (subtle)
    ctx.strokeStyle = 'rgba(255,255,255,0.03)';
    ctx.lineWidth = 1;
    for (let i = 0; i <= GRID_SIZE; i++) {
      ctx.beginPath();
      ctx.moveTo(i * cellPixelSize, 0);
      ctx.lineTo(i * cellPixelSize, size);
      ctx.stroke();
      ctx.beginPath();
      ctx.moveTo(0, i * cellPixelSize);
      ctx.lineTo(size, i * cellPixelSize);
      ctx.stroke();
    }

    // Food
    drawCell(food.x, food.y, '#f87171', true);

    // Snake
    snake.forEach((segment, index) => {
      const isHead = index === 0;
      drawCell(segment.x, segment.y, isHead ? '#4ade80' : '#22c55e', false, isHead);
    });
  }

  function drawCell(x, y, color, isFood = false, isHead = false) {
    const px = x * cellPixelSize;
    const py = y * cellPixelSize;
    const pad = isFood ? cellPixelSize * 0.15 : 2;
    const radius = isFood ? cellPixelSize / 2 - pad : 5;

    ctx.fillStyle = color;

    if (isFood) {
      // Draw food as a circle
      ctx.beginPath();
      ctx.arc(px + cellPixelSize / 2, py + cellPixelSize / 2, radius, 0, Math.PI * 2);
      ctx.fill();
    } else {
      // Draw snake segment as rounded square
      roundRect(ctx, px + pad, py + pad, cellPixelSize - pad * 2, cellPixelSize - pad * 2, radius);
      ctx.fill();

      if (isHead) {
        // Simple eyes on head for a nicer look
        ctx.fillStyle = '#052e12';
        const eyeSize = Math.max(2, cellPixelSize * 0.1);
        const cx = px + cellPixelSize / 2;
        const cy = py + cellPixelSize / 2;
        const offset = cellPixelSize * 0.2;
        ctx.beginPath();
        ctx.arc(cx - offset, cy - offset, eyeSize, 0, Math.PI * 2);
        ctx.arc(cx + offset, cy - offset, eyeSize, 0, Math.PI * 2);
        ctx.fill();
      }
    }
  }

  function roundRect(context, x, y, w, h, r) {
    context.beginPath();
    context.moveTo(x + r, y);
    context.arcTo(x + w, y, x + w, y + h, r);
    context.arcTo(x + w, y + h, x, y + h, r);
    context.arcTo(x, y + h, x, y, r);
    context.arcTo(x, y, x + w, y, r);
    context.closePath();
  }

  // ---------- Direction handling ----------
  function setDirection(newDir) {
    // Prevent reversing directly into itself
    if (newDir.x === -direction.x && newDir.y === -direction.y) return;
    // Prevent queuing an invalid direction if two keys pressed within one tick
    if (newDir.x === -nextDirection.x && newDir.y === -nextDirection.y) return;
    nextDirection = newDir;
    playTurnSound();
  }

  function handleKeyDown(e) {
    if (!isRunning || isPaused) {
      // Allow starting game with space/enter if on start screen
      return;
    }
    switch (e.key.toLowerCase()) {
      case 'arrowup':
      case 'w':
        setDirection({ x: 0, y: -1 });
        break;
      case 'arrowdown':
      case 's':
        setDirection({ x: 0, y: 1 });
        break;
      case 'arrowleft':
      case 'a':
        setDirection({ x: -1, y: 0 });
        break;
      case 'arrowright':
      case 'd':
        setDirection({ x: 1, y: 0 });
        break;
      case ' ':
        togglePause();
        break;
    }
  }

  // ---------- Game control functions ----------
  function startGame() {
    resetGame();
    isRunning = true;
    isPaused = false;
    startOverlay.classList.add('hidden');
    gameOverOverlay.classList.add('hidden');
    pauseOverlay.classList.add('hidden');
    pauseBtn.disabled = false;
    pauseBtn.textContent = 'Pause';
    draw();
    startLoop();
  }

  function togglePause() {
    if (!isRunning || isGameOver) return;
    isPaused = !isPaused;
    pauseOverlay.classList.toggle('hidden', !isPaused);
    pauseBtn.textContent = isPaused ? 'Resume' : 'Pause';
  }

  function resumeGame() {
    isPaused = false;
    pauseOverlay.classList.add('hidden');
    pauseBtn.textContent = 'Pause';
  }

  function restartGame() {
    stopLoop();
    startGame();
  }

  function setDifficulty(level) {
    currentDifficulty = level;
    speedMs = DIFFICULTY_SPEED[level];
    diffButtons.forEach(btn => {
      btn.classList.toggle('active', btn.dataset.difficulty === level);
    });
    if (isRunning && !isGameOver) {
      startLoop(); // restart interval with new speed
    }
  }

  // ---------- Event listeners ----------
  document.addEventListener('keydown', handleKeyDown);

  startBtn.addEventListener('click', startGame);
  playAgainBtn.addEventListener('click', startGame);
  resumeBtn.addEventListener('click', () => {
    resumeGame();
  });
  pauseBtn.addEventListener('click', togglePause);
  restartBtn.addEventListener('click', restartGame);

  muteBtn.addEventListener('click', () => {
    isMuted = !isMuted;
    updateMuteIcon();
    saveMutePreference();
  });

  diffButtons.forEach(btn => {
    btn.addEventListener('click', () => setDifficulty(btn.dataset.difficulty));
  });

  // Touch controls
  touchButtons.forEach(btn => {
    const dirMap = {
      up: { x: 0, y: -1 },
      down: { x: 0, y: 1 },
      left: { x: -1, y: 0 },
      right: { x: 1, y: 0 }
    };
    btn.addEventListener('click', () => {
      if (isRunning && !isPaused) {
        setDirection(dirMap[btn.dataset.dir]);
      }
    });
  });

  // Swipe gesture support
  let touchStartX = 0;
  let touchStartY = 0;

  canvas.addEventListener('touchstart', (e) => {
    const touch = e.changedTouches[0];
    touchStartX = touch.screenX;
    touchStartY = touch.screenY;
  }, { passive: true });

  canvas.addEventListener('touchend', (e) => {
    if (!isRunning || isPaused) return;
    const touch = e.changedTouches[0];
    const dx = touch.screenX - touchStartX;
    const dy = touch.screenY - touchStartY;

    if (Math.abs(dx) < 20 && Math.abs(dy) < 20) return; // ignore tiny movements

    if (Math.abs(dx) > Math.abs(dy)) {
      setDirection(dx > 0 ? { x: 1, y: 0 } : { x: -1, y: 0 });
    } else {
      setDirection(dy > 0 ? { x: 0, y: 1 } : { x: 0, y: -1 });
    }
  }, { passive: true });

  window.addEventListener('resize', resizeCanvas);

  // ---------- Init ----------
  function init() {
    loadHighScore();
    loadMutePreference();
    setDifficulty('easy');
    resetGame();
    resizeCanvas();
    draw();
  }

  init();
})();
