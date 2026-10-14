<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NUMBER PUZZLE</title>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-height: 100vh;
  font-family: sans-serif;
  background: #f5f1eb;
  color: #39382f;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 24px 12px;
}

.container {
  width: 100%;
  max-width: 440px;
  text-align: center;
}

.label {
  font-size: 11px;
  letter-spacing: 4px;
  color: #99917e;
}

h1 {
  font-family: Georgia, serif;
  font-size: 36px;
  letter-spacing: 2px;
  margin: 12px 0;
}

.description {
  font-size: 13px;
  color: #827e73;
  margin-bottom: 28px;
}

.stats {
  display: flex;
  justify-content: space-around;
  margin-bottom: 20px;
  background: #fffdf9;
  border-radius: 16px;
  padding: 16px 8px;
}

.stat-label {
  font-size: 11px;
  color: #99917e;
  margin-bottom: 6px;
}

.stat-value {
  font-family: Georgia, serif;
  font-size: 24px;
}

.board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
  padding: 12px;
  background: #ded8cc;
  border-radius: 18px;
  box-shadow: 0 8px 20px #b9ae9c40;
}

.tile {
  aspect-ratio: 1;
  border: none;
  border-radius: 12px;
  background: #fffdf8;
  color: #49483e;
  font-family: Georgia, serif;
  font-size: clamp(20px, 6vw, 30px);
  font-weight: bold;
  cursor: pointer;
  transition: background 0.15s, transform 0.15s;
}

.tile:hover {
  background: #f1e5ce;
}

.tile:active {
  transform: scale(0.95);
}

.tile.empty {
  background: #c9c1b3;
  cursor: default;
}

.tile:focus-visible,
button:focus-visible {
  outline: 3px solid #8a8d6d;
  outline-offset: 2px;
}

.controls {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.controls button {
  flex: 1;
  padding: 14px 8px;
  border-radius: 30px;
  border: none;
  font-size: 13px;
  cursor: pointer;
}

.shuffle {
  background: #626750;
  color: white;
}

.reset {
  background: #e6e0d5;
  color: #555247;
}

.message {
  min-height: 26px;
  margin-top: 20px;
  font-size: 14px;
  color: #626750;
  line-height: 1.7;
}

footer {
  margin-top: 24px;
  color: #aaa18f;
  font-size: 10px;
  letter-spacing: 2px;
}

@media (max-width: 360px) {
  .board {
    gap: 5px;
    padding: 8px;
  }

  .tile {
    border-radius: 8px;
  }
}
</style>
</head>

<body>
<div class="container">

  <div class="label">BRAIN TRAINING GAME</div>
  <h1>NUMBER PUZZLE</h1>
  <p class="description">
    数字を順番に並べて、パズルを完成させよう。
  </p>

  <div class="stats">
    <div>
      <div class="stat-label">MOVES</div>
      <div class="stat-value" id="moves">0</div>
    </div>
    <div>
      <div class="stat-label">TIME</div>
      <div class="stat-value" id="time">00:00</div>
    </div>
  </div>

  <div class="board" id="board" aria-label="数字パズル"></div>

  <div class="controls">
    <button class="shuffle" id="shuffle">シャッフル</button>
    <button class="reset" id="reset">最初から</button>
  </div>

  <div class="message" id="message" aria-live="polite">
    空いているマスの隣の数字を動かそう！
  </div>

  <footer>TAKE YOUR TIME · ENJOY THE PUZZLE</footer>

</div>

<script>
const boardElement = document.getElementById("board");
const movesElement = document.getElementById("moves");
const timeElement = document.getElementById("time");
const messageElement = document.getElementById("message");

let tiles = [];
let moves = 0;
let seconds = 0;
let timer = null;
let started = false;
let finished = false;

function isSolved() {
  return tiles.every((value, index) =>
    index === 15 ? value === 0 : value === index + 1
  );
}

function render() {
  boardElement.innerHTML = "";

  tiles.forEach((value, index) => {
    const button = document.createElement("button");
    button.className = value === 0 ? "tile empty" : "tile";
    button.textContent = value === 0 ? "" : value;
    button.setAttribute(
      "aria-label",
      value === 0 ? "空白" : `数字 ${value}`
    );

    if (value === 0) {
      button.disabled = true;
    } else {
      button.addEventListener("click", () => moveTile(index));
    }

    boardElement.appendChild(button);
  });

  movesElement.textContent = moves;
}

function startTimer() {
  if (timer !== null) return;

  timer = setInterval(() => {
    seconds++;
    const minutes = String(Math.floor(seconds / 60)).padStart(2, "0");
    const secs = String(seconds % 60).padStart(2, "0");
    timeElement.textContent = `${minutes}:${secs}`;
  }, 1000);
}

function stopTimer() {
  clearInterval(timer);
  timer = null;
}

function moveTile(index) {
  if (finished) return;

  const emptyIndex = tiles.indexOf(0);
  const row = Math.floor(index / 4);
  const col = index % 4;
  const emptyRow = Math.floor(emptyIndex / 4);
  const emptyCol = emptyIndex % 4;

  const distance =
    Math.abs(row - emptyRow) + Math.abs(col - emptyCol);

  if (distance !== 1) return;

  if (!started) {
    started = true;
    startTimer();
    messageElement.textContent = "いい調子！完成を目指そう。";
  }

  [tiles[index], tiles[emptyIndex]] =
    [tiles[emptyIndex], tiles[index]];

  moves++;
  render();

  if (isSolved()) {
    finished = true;
    stopTimer();
    messageElement.textContent =
      `🎉 クリア！ ${moves}手・${timeElement.textContent}で完成！`;
  }
}

function shuffleBoard() {
  stopTimer();

  tiles = Array.from({ length: 16 }, (_, i) =>
    i === 15 ? 0 : i + 1
  );

  let previousEmpty = -1;

  // 正解状態から合法な移動を繰り返すため、
  // 必ず解ける盤面になる。
  for (let i = 0; i < 300; i++) {
    const empty = tiles.indexOf(0);
    const neighbors = [
      empty - 4,
      empty + 4,
      ...(empty % 4 !== 0 ? [empty - 1] : []),
      ...(empty % 4 !== 3 ? [empty + 1] : [])
    ].filter(index => index >= 0 && index < 16);

    let choices = neighbors.filter(index => index !== previousEmpty);

    if (choices.length === 0) choices = neighbors;

    const chosen =
      choices[Math.floor(Math.random() * choices.length)];

    [tiles[empty], tiles[chosen]] =
      [tiles[chosen], tiles[empty]];

    previousEmpty = empty;
  }

  moves = 0;
  seconds = 0;
  started = false;
  finished = false;

  movesElement.textContent = "0";
  timeElement.textContent = "00:00";
  messageElement.textContent =
    "空いているマスの隣の数字を動かそう！";

  render();
}

function resetBoard() {
  stopTimer();

  tiles = Array.from({ length: 16 }, (_, i) =>
    i === 15 ? 0 : i + 1
  );

  moves = 0;
  seconds = 0;
  started = false;
  finished = false;

  movesElement.textContent = "0";
  timeElement.textContent = "00:00";
  messageElement.textContent =
    "空いているマスの隣の数字を動かそう！";

  render();
}

document.getElementById("shuffle")
  .addEventListener("click", shuffleBoard);

document.getElementById("reset")
  .addEventListener("click", resetBoard);

resetBoard();
shuffleBoard();
</script>
</body>
</html>
