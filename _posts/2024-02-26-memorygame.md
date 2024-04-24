---
layout: default
title: Memory Game
permalink: /memorygame
---
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Flag Memory Game</title>
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300;400;700&display=swap" rel="stylesheet">
<style>
  body {
    font-family: 'Montserrat', sans-serif;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    background-color: #f7f7f7;
    margin: 0;
    padding: 20px;
    box-sizing: border-box;
  }
  .grid {
    display: grid;
    gap: 15px;
    margin: 20px auto;
    perspective: 1000px;
  }
  .card {
    position: relative;
    width: 100px;
    height: 150px;
    transition: transform 0.6s;
    transform-style: preserve-3d;
    cursor: pointer;
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  }
  .card.flip {
    transform: rotateY(180deg);
  }
  .card-front, .card-back {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 10px;
    font-size: 2rem;
  }
  .card-front {
    background-color: #1e88e5;
    color: white;
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  }
  .card-back {
    background-color: #f0f0f0;
    transform: rotateY(180deg);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  }
  .info {
    display: flex;
    justify-content: space-around;
    width: 100%;
    max-width: 650px;
    margin-top: 20px;
  }
  .timer, .score {
    font-size: 1.5rem;
    background: #e1e1e1;
    padding: 10px 15px;
    border-radius: 5px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    color: black; /* Set color to black */
  }
  .difficulty-buttons {
    margin: 20px 0;
  }
  button {
    padding: 10px 20px;
    font-size: 1rem;
    margin-right: 10px;
    cursor: pointer;
    border: none;
    border-radius: 5px;
    background-color: #4CAF50;
    color: white;
    transition: background-color 0.3s, transform 0.2s;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }
  button:hover {
    background-color: #45a049;
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  }
  button:focus {
    outline: none;
  }
</style>
</head>
<body>

<div class="difficulty-buttons">
  <button onclick="selectDifficulty('easy')">Easy</button>
  <button onclick="selectDifficulty('medium')">Medium</button>
  <button onclick="selectDifficulty('hard')">Hard</button>
</div>
<div class="info">
  <div class="timer">Time: <span id="time">60</span>s</div>
  <div class="score">Score: <span id="score">0</span></div>
</div>
<div id="game" class="grid"></div>

<script>
const flagEmojis = ['🇺🇸', '🇬🇧', '🇫🇷', '🇩🇪', '🇨🇦', '🇯🇵', '🇦🇺', '🇮🇹'];
let gameSize = 8; 
let timerDuration; 
let score = 0;
let timerId; 

const game = document.getElementById('game');
const timeEl = document.getElementById('time');
const scoreEl = document.getElementById('score');

let hasFlippedCard = false;
let lockBoard = false;
let firstCard, secondCard;

function selectDifficulty(difficulty) {
  switch(difficulty) {
    case 'easy':
      gameSize = 4;
      timerDuration = 90;
      break;
    case 'medium':
      gameSize = 8;
      timerDuration = 60;
      break;
    case 'hard':
      gameSize = 12;
      timerDuration = 30;
      break;
  }
  resetGame();
}

function initGame() {
  let selectedFlags = flagEmojis.slice(0, gameSize); 
  selectedFlags = [...selectedFlags, ...selectedFlags];
  selectedFlags.sort(() => 0.5 - Math.random());
  createBoard(selectedFlags);
  resetTimer();
  startTimer();
}

function createBoard(flags) {
  game.innerHTML = ''; 
  flags.forEach(flag => {
    const card = document.createElement('div');
    card.classList.add('card');
    const cardFront = document.createElement('div');
    cardFront.classList.add('card-front');
    const cardBack = document.createElement('div');
    cardBack.classList.add('card-back');
    cardBack.innerHTML = flag;
    card.appendChild(cardFront);
    card.appendChild(cardBack);
    card.addEventListener('click', flipCard);
    game.appendChild(card);
  });
  adjustGrid(flags.length);
}

function adjustGrid(size) {
  const columns = Math.sqrt(size);
  game.style.gridTemplateColumns = `repeat(${columns}, 1fr)`;
}

function resetTimer() {
  clearInterval(timerId);
  timerId = null;
  timeEl.textContent = timerDuration;
}

function startTimer() {
  let timeLeft = timerDuration;
  timeEl.textContent = timeLeft;
  timerId = setInterval(() => {
    timeLeft--;
    timeEl.textContent = timeLeft;
    if (timeLeft <= 0) {
      clearInterval(timerId);
      alert('Time\'s up! Your score: ' + score);
      resetGame();
    }
  }, 1000);
}

function flipCard() {
  if (lockBoard) return;
  if (this === firstCard) return;

  this.classList.add('flip');

  if (!hasFlippedCard) {
    hasFlippedCard = true;
    firstCard = this;
    return;
  }

  secondCard = this;
  checkForMatch();
}

function checkForMatch() {
  const isMatch = firstCard.querySelector('.card-back').innerHTML === secondCard.querySelector('.card-back').innerHTML;
  if (isMatch) {
    score += 10;
    disableCards();
  } else {
    score = Math.max(0, score - 5);
    unflipCards();
  }
  scoreEl.textContent = score;
}

function disableCards() {
  firstCard.removeEventListener('click', flipCard);
  secondCard.removeEventListener('click', flipCard);
  resetBoard();
}

function unflipCards() {
  lockBoard = true;

  setTimeout(() => {
    firstCard.classList.remove('flip');
    secondCard.classList.remove('flip');
    resetBoard();
  }, 1500);
}

function resetBoard() {
  [hasFlippedCard, lockBoard] = [false, false];
  [firstCard, secondCard] = [null, null];
}

function resetGame() {
  score = 0;
  scoreEl.textContent = score;
  initGame(); 
}


selectDifficulty('medium');
</script>
</body>
</html>
