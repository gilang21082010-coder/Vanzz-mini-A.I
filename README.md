<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>~VANZZ~</title>
<style>
body {
  font-family: Arial, sans-serif;
  background: linear-gradient(to right, #0b0f19, #1a2a45);
  color: #9bead8;
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
  padding: 10px;
}
h1 { margin: 10px 0; text-align:center; }
#datetime { margin-bottom: 10px; font-size: 16px; }
canvas { background: #111; border: 3px solid #9bead8; margin-bottom: 10px; }
.controls { display: flex; flex-wrap: wrap; gap: 10px; margin-bottom: 10px; }
.controls button { font-size: 18px; padding: 5px 10px; background: #222; color: #9bead8; border: 2px solid #9bead8; border-radius: 10px; }
#score { font-size: 18px; margin-bottom: 10px; }
#chat { width: 100%; max-width: 350px; background: #111; border: 2px solid #9bead8; border-radius: 10px; padding: 10px; margin-bottom: 20px; }
#messages { height: 150px; overflow-y: auto; border-bottom: 1px solid #9bead8; margin-bottom: 10px; padding: 5px; }
input[type="text"] { width: calc(100% - 60px); padding: 5px; border-radius: 5px; border: 1px solid #9bead8; background: #222; color: #9bead8; }
button.send { width: 50px; padding: 5px; border-radius: 5px; border: 1px solid #9bead8; background: #222; color: #9bead8; }
table { border-collapse: collapse; margin-top: 20px; background: #111; }
table, th, td { border: 1px solid #9bead8; padding: 5px 10px; text-align: center; }
</style>
</head>
<body>
<h1>~VANZZ~ Snake Game & A.I Chat</h1>

<div id="datetime">--:--:--</div>
<div id="score">Score: 0</div>
<canvas id="gameCanvas" width="300" height="300"></canvas>

<div class="controls">
  <button onclick="changeDirection('up')">↑</button>
  <button onclick="changeDirection('left')">←</button>
  <button onclick="changeDirection('down')">↓</button>
  <button onclick="changeDirection('right')">→</button>
  <button onclick="playGame()">Play</button>
  <button onclick="pauseGame()">Pause</button>
  <button onclick="stopGame()">Stop</button>
</div>

<div id="chat">
  <div id="messages"></div>
  <input type="text" id="userInput" placeholder="Ketik sesuatu...">
  <button class="send" onclick="sendMessage()">Kirim</button>
</div>

<h2>Tabel Perkalian 1-9</h2>
<table id="multiplicationTable"></table>

<script>
// =====================
// Jam & Tanggal Realtime
// =====================
const datetimeDiv = document.getElementById('datetime');
function updateDateTime(){
  const now = new Date();
  const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
  datetimeDiv.textContent = now.toLocaleTimeString() + " | " + now.toLocaleDateString('id-ID', options);
}
setInterval(updateDateTime, 1000);
updateDateTime();

// =====================
// Snake Game
// =====================
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const grid = 15;
let snake = [{x: grid*5, y: grid*5}];
let food = {x: grid*10, y: grid*10};
let dx = grid, dy = 0;
let score = 0;
let gameInterval = null;
let isPaused = false;

function gameLoop() {
  if(isPaused) return;
  const head = {x: snake[0].x + dx, y: snake[0].y + dy};
  if(head.x < 0 || head.y < 0 || head.x >= canvas.width || head.y >= canvas.height || snake.some(s => s.x === head.x && s.y === head.y)) {
    alert("Game Over! Score: " + score);
    stopGame();
    return;
  }
  snake.unshift(head);
  if(head.x === food.x && head.y === food.y) {
    placeFood();
    score++;
    updateScore();
  } else { snake.pop(); }
  ctx.fillStyle = '#111';
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  ctx.fillStyle = '#0f0';
  snake.forEach(s => ctx.fillRect(s.x, s.y, grid-1, grid-1));
  ctx.fillStyle = '#f00';
  ctx.fillRect(food.x, food.y, grid-1, grid-1);
}

function placeFood() {
  food.x = Math.floor(Math.random() * (canvas.width/grid)) * grid;
  food.y = Math.floor(Math.random() * (canvas.height/grid)) * grid;
}

function changeDirection(dir) {
  if(dir === 'up' && dy === 0) { dx = 0; dy = -grid; }
  if(dir === 'down' && dy === 0) { dx = 0; dy = grid; }
  if(dir === 'left' && dx === 0) { dx = -grid; dy = 0; }
  if(dir === 'right' && dx === 0) { dx = grid; dy = 0; }
}

function updateScore() { document.getElementById('score').textContent = "Score: " + score; }
function playGame(){ if(!gameInterval) gameInterval = setInterval(gameLoop, 150); isPaused = false; }
function pauseGame(){ isPaused = true; }
function stopGame(){
  clearInterval(gameInterval); gameInterval = null;
  snake = [{x: grid*5, y: grid*5}]; dx = grid; dy = 0;
  score = 0; updateScore(); ctx.clearRect(0,0,canvas.width,canvas.height);
}

// Swipe Mobile
let touchStartX=0, touchStartY=0;
canvas.addEventListener('touchstart', e => { const t = e.touches[0]; touchStartX=t.clientX; touchStartY=t.clientY; });
canvas.addEventListener('touchend', e => {
  const t = e.changedTouches[0];
  const dxSwipe = t.clientX - touchStartX;
  const dySwipe = t.clientY - touchStartY;
  if(Math.abs(dxSwipe) > Math.abs(dySwipe)){
    if(dxSwipe > 0) changeDirection('right'); else changeDirection('left');
  } else {
    if(dySwipe > 0) changeDirection('down'); else changeDirection('up');
  }
});

// =====================
// Tabel Perkalian
// =====================
const table = document.getElementById('multiplicationTable');
let html = '<tr><th>x</th>';
for(let i=1;i<=9;i++) html += `<th>${i}</th>`; html += '</tr>';
for(let i=1;i<=9;i++){
  html += `<tr><th>${i}</th>`;
  for(let j=1;j<=9;j++){ html += `<td>${i*j}</td>`; }
  html += '</tr>';
}
table.innerHTML = html;

// =====================
// A.I Chat
// =====================
const messages = document.getElementById('messages');
let userName = "";

function sendMessage(){
  const input = document.getElementById('userInput');
  const text = input.value.trim().toLowerCase();
  if(!text) return;
  addMessage("Kamu: " + input.value);
  input.value = '';
  let reply = "Maaf, saya tidak mengerti.";

  // Sapaan
  const greetings = ["oyy","coy","oy","halo","hai"];
  if(greetings.includes(text)) reply = "Halo" + (userName ? ", "+userName+"!" : "!");

  // Pertanyaan kabar
  const questions = ["apa kabar","gimana kabarnya"];
  if(questions.includes(text)){
    const answers = ["Baik, mas!", "Sehat selalu!", "Alhamdulillah baik."];
    reply = answers[Math.floor(Math.random() * answers.length)];
  }

  // Memanggil nama user
  if(text.startsWith("panggil aku ")){
    userName = input.value.substring(11).trim();
    if(userName) reply = "Oke, aku akan memanggilmu " + userName + "!";
  }

  // Pertanyaan waktu & tanggal
  const timeQuestions = ["jam","waktu"];
  const dateQuestions = ["tanggal","bulan","tahun","hari","hari ini"];
  const now = new Date();
  if(timeQuestions.some(q => text.includes(q))){
    reply = "Sekarang jam " + now.toLocaleTimeString() + ".";
  } else if(dateQuestions.some(q => text.includes(q))){
    const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
    reply = "Oke, hari ini " + now.toLocaleDateString('id-ID', options) + ".";
  }

  addMessage("A.I: " + reply);
}

function addMessage(msg){
  const p = document.createElement('p');
  p.textContent = msg;
  messages.appendChild(p);
  messages.scrollTop = messages.scrollHeight;
}
</script>
</body>
</html>
