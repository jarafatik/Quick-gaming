const gameArea = document.getElementById('gameArea');
const scoreEl = document.getElementById('score');
const timerEl = document.getElementById('timer');
const bestEl = document.getElementById('best');
const startBtn = document.getElementById('startBtn');
const messageEl = document.getElementById('message');

let score = 0;
let timeLeft = 30;
let bestScore = Number(localStorage.getItem('quickGamingBest')) || 0;
let isRunning = false;
let timerId = null;
let targetTimeout = null;

bestEl.textContent = bestScore;

function setMessage(text, isVisible = true) {
  messageEl.innerHTML = text;
  messageEl.classList.toggle('visible', isVisible);
}

function updateHud() {
  scoreEl.textContent = score;
  timerEl.textContent = timeLeft;
  bestEl.textContent = bestScore;
}

function clearTarget() {
  const existingTarget = document.querySelector('.target');
  if (existingTarget) existingTarget.remove();
}

function spawnTarget() {
  if (!isRunning) return;

  clearTarget();

  const target = document.createElement('button');
  target.type = 'button';
  target.className = 'target';

  const areaWidth = gameArea.clientWidth;
  const areaHeight = gameArea.clientHeight;
  const size = 52 + Math.random() * 34;

  const x = Math.random() * (areaWidth - size - 12);
  const y = Math.random() * (areaHeight - size - 12);

  target.style.width = `${size}px`;
  target.style.height = `${size}px`;
  target.style.left = `${x}px`;
  target.style.top = `${y}px`;

  target.addEventListener('click', () => {
    score += 1;
    updateHud();
    spawnTarget();
  });

  gameArea.appendChild(target);

  const nextDelay = Math.max(500, 1100 - score * 20);
  targetTimeout = setTimeout(() => {
    if (isRunning) {
      clearTarget();
      spawnTarget();
    }
  }, nextDelay);
}

function finishGame() {
  isRunning = false;
  clearInterval(timerId);
  clearTimeout(targetTimeout);
  clearTarget();

  if (score > bestScore) {
    bestScore = score;
    localStorage.setItem('quickGamingBest', String(bestScore));
  }

  updateHud();
  setMessage(`Hra skončila! Dosáhl jsi <strong>${score}</strong> bodů. Zkus to znovu!`);
  startBtn.textContent = 'Hrát znovu';
}

function startGame() {
  score = 0;
  timeLeft = 30;
  isRunning = true;
  clearInterval(timerId);
  clearTimeout(targetTimeout);
  clearTarget();
  updateHud();
  setMessage('Hra běží! Klikni na co nejvíc kroužků.', true);
  startBtn.textContent = 'Restartovat';

  spawnTarget();

  timerId = setInterval(() => {
    timeLeft -= 1;
    updateHud();

    if (timeLeft <= 0) {
      finishGame();
    }
  }, 1000);
}

startBtn.addEventListener('click', startGame);
updateHud();
setMessage('Klikni na <strong>Začít hru</strong> a zkus překonat svůj rekord.');

window.addEventListener('resize', () => {
  if (isRunning) {
    clearTarget();
    spawnTarget();
  }
});
