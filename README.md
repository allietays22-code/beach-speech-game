<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Beach Speech Adventure - Research Edition</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body {
  margin: 0;
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(#87CEEB, #FFD580);
  text-align: center;
}

h1 { color: white; }

.card {
  background: rgba(255,255,255,0.95);
  padding: 20px;
  border-radius: 15px;
  display: inline-block;
  margin-top: 20px;
  box-shadow: 0 6px 15px rgba(0,0,0,0.2);
}

button {
  padding: 10px 18px;
  border-radius: 10px;
  border: none;
  background: #00aaff;
  color: white;
  cursor: pointer;
  margin: 5px;
}

#word { font-size: 40px; margin: 20px; }
canvas { max-width: 400px; margin-top: 20px; }
</style>
</head>
<body>

<h1>🏖️ Beach Speech Adventure (Research Edition)</h1>

<div class="card">
  <select id="soundSelect">
    <option value="s">/S/</option>
    <option value="r">/R/</option>
    <option value="l">/L/</option>
  </select>

  <div id="word">Press Start</div>

  <button onclick="startGame()">Start Session</button>
  <button onclick="startListening()">🎤 Speak</button>
  <button onclick="exportData()">Export Data</button>

  <div id="score">Score: 0</div>

  <canvas id="chart"></canvas>
</div>

<script>
let wordBank = {
  s: ["sun", "sand", "sea", "seal", "soup"],
  r: ["red", "run", "rope", "rain", "rock"],
  l: ["lion", "leaf", "light", "lake", "lamp"]
};

let currentWord = "";
let score = 0;
let attempts = 0;
let sessionLog = [];
let chart;

function startGame() {
  score = 0;
  attempts = 0;
  sessionLog = [];
  nextWord();
  updateDisplay();
  updateChart();
}

function nextWord() {
  let sound = document.getElementById("soundSelect").value;
  let list = wordBank[sound];
  currentWord = list[Math.floor(Math.random() * list.length)];
  document.getElementById("word").innerText = currentWord;
}

function updateDisplay() {
  document.getElementById("score").innerText = `Score: ${score}/${attempts}`;
}

const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
const recognition = new SpeechRecognition();
recognition.lang = 'en-US';

function startListening() {
  recognition.start();
}

recognition.onresult = function(event) {
  const spoken = event.results[0][0].transcript.toLowerCase();
  attempts++;

  let correct = spoken.includes(currentWord);
  if (correct) score++;

  sessionLog.push({ target: currentWord, response: spoken, correct: correct });

  updateDisplay();
  updateChart();
  nextWord();
};

function updateChart() {
  let correct = sessionLog.filter(x => x.correct).length;
  let incorrect = sessionLog.length - correct;

  if (chart) chart.destroy();

  chart = new Chart(document.getElementById('chart'), {
    type: 'bar',
    data: {
      labels: ['Correct', 'Incorrect'],
      datasets: [{ data: [correct, incorrect] }]
    }
  });
}

function exportData() {
  let data = {
    attempts,
    score,
    accuracy: (score/attempts*100 || 0).toFixed(1) + '%',
    trials: sessionLog
  };

  let blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
  let a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'session_data.json';
  a.click();
}
</script>

</body>
</html>
