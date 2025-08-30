
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Tic Tac Toe — HTML + JS</title>
  <style>
    :root{
      --bg:#0f1724; --card:#0b1220; --accent:#06b6d4; --accent2:#f472b6; --muted:#9ca3af;
    }
    *{box-sizing:border-box}
    body{margin:0; font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue"; background:linear-gradient(180deg,#071026 0%, #071a2a 100%); color:#e6eef6; min-height:100vh; display:flex; align-items:center; justify-content:center; padding:20px}
    .app{width:100%; max-width:980px; display:grid; grid-template-columns:1fr 320px; gap:20px; align-items:start}
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border:1px solid rgba(255,255,255,0.04); padding:18px; border-radius:12px; box-shadow: 0 8px 30px rgba(2,6,23,0.6)}
    h1{margin:0 0 8px 0; font-size:20px}
    .status{font-weight:600; color:var(--accent); margin-bottom:10px}
    .board{display:grid; grid-template-columns:repeat(3,1fr); gap:10px; padding:10px; background:rgba(255,255,255,0.01); border-radius:10px}
    .cell{aspect-ratio:1 / 1; display:flex; align-items:center; justify-content:center; font-size:48px; font-weight:800; color:#fff; background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:8px; cursor:pointer; user-select:none; transition:transform .08s ease, box-shadow .12s ease}
    .cell:hover{transform:translateY(-2px)}
    .cell.disabled{cursor:default; opacity:0.7}
    .controls{display:flex; gap:8px; flex-wrap:wrap; margin-top:12px}
    button.btn{background:transparent; border:1px solid rgba(255,255,255,0.06); color:inherit; padding:8px 12px; border-radius:8px; cursor:pointer}
    .btn.primary{background:linear-gradient(90deg,var(--accent),var(--accent2)); color:#041025; border:none; font-weight:700}
    .right .section{margin-bottom:14px}
    label.small{display:block; font-size:12px; color:var(--muted); margin-bottom:6px}
    .flex{display:flex; gap:8px; align-items:center}
    .score{display:grid; grid-template-columns:repeat(3,1fr); gap:8px; margin-top:10px}
    .score .item{background:rgba(255,255,255,0.02); padding:10px; border-radius:8px; text-align:center}
    .muted{color:var(--muted)}
    .footer{font-size:12px; color:var(--muted); margin-top:12px}
    @media (max-width:880px){ .app{grid-template-columns:1fr; max-width:700px} .right{order:2} }
  </style>
</head>
<body>
  <div class="app">
    <div class="card">
      <h1>Tic Tac Toe</h1>
      <div class="status" id="status">Your turn — X</div>
      <div class="board" id="board" aria-label="Tic Tac Toe board">
        <!-- 9 cells -->
      </div>
      <div class="controls">
        <button class="btn primary" id="restart">Restart</button>
        <button class="btn" id="undo">Undo</button>
        <div style="flex:1"></div>
        <button class="btn" id="share">Share</button>
      </div>
      <div class="footer muted">Built with plain HTML, CSS and JavaScript — includes AI (minimax)</div>
    </div>

    <div class="right">
      <div class="card section">
        <label class="small">Mode</label>
        <div class="flex">
          <select id="mode" class="btn" style="padding:8px;border-radius:8px;background:transparent;border:1px solid rgba(255,255,255,0.04)">
            <option value="AI">Vs AI</option>
            <option value="PVP">Two Players</option>
          </select>
          <div style="flex:1"></div>
          <label class="small muted" title="First player">Starter</label>
          <select id="starter" class="btn" style="padding:8px;border-radius:8px;background:transparent;border:1px solid rgba(255,255,255,0.04)">
            <option value="X">X</option>
            <option value="O">O</option>
          </select>
        </div>
        <label class="small" style="margin-top:10px">Difficulty</label>
        <div class="flex">
          <select id="difficulty" class="btn">
            <option>Easy</option>
            <option>Medium</option>
            <option selected>Impossible</option>
          </select>
          <div style="flex:1"></div>
          <label class="small muted">AI plays as</label>
          <select id="aiMark" class="btn">
            <option value="O">O (second)</option>
            <option value="X">X (first)</option>
          </select>
        </div>
      </div>

      <div class="card section">
        <div class="flex" style="justify-content:space-between;align-items:center">
          <div><label class="small">Scoreboard</label></div>
          <button class="btn" id="resetScore">Reset</button>
        </div>
        <div class="score">
          <div class="item"><div class="muted">X</div><div id="scoreX" style="font-weight:700;font-size:20px">0</div></div>
          <div class="item"><div class="muted">Draw</div><div id="scoreD" style="font-weight:700;font-size:20px">0</div></div>
          <div class="item"><div class="muted">O</div><div id="scoreO" style="font-weight:700;font-size:20px">0</div></div>
        </div>
      </div>

      <div class="card section">
        <label class="small">Tips</label>
        <div class="muted">Impossible uses perfect play (minimax). Use Medium for balanced play, Easy for fun.</div>
      </div>
    </div>
  </div>

<script>
// Game logic (vanilla JS) with minimax AI (alpha-beta)
const boardEl = document.getElementById('board');
const statusEl = document.getElementById('status');
const restartBtn = document.getElementById('restart');
const undoBtn = document.getElementById('undo');
const modeSelect = document.getElementById('mode');
const difficultySelect = document.getElementById('difficulty');
const aiMarkSelect = document.getElementById('aiMark');
const starterSelect = document.getElementById('starter');
const shareBtn = document.getElementById('share');
const resetScoreBtn = document.getElementById('resetScore');
const scoreX = document.getElementById('scoreX');
const scoreO = document.getElementById('scoreO');
const scoreD = document.getElementById('scoreD');

let board = Array(9).fill(null);
let history = [];
let xStarts = starterSelect.value === 'X';
let xIsNext = xStarts;
let mode = modeSelect.value; // AI or PVP
let aiMark = aiMarkSelect.value; // X or O when AI plays
let difficulty = difficultySelect.value; // Easy, Medium, Impossible
let scores = {X:0, O:0, D:0};

const LINES = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];

// Render board cells
function renderBoard(){
  boardEl.innerHTML = '';
  board.forEach((v,i)=>{
    const cell = document.createElement('div');
    cell.className = 'cell' + (v ? ' disabled' : '');
    cell.setAttribute('data-index', i);
    cell.setAttribute('role','button');
    cell.setAttribute('aria-label', 'Cell ' + (i+1));
    cell.innerText = v ? v : '';
    cell.addEventListener('click', ()=> onCellClick(i));
    boardEl.appendChild(cell);
  });
  updateStatus();
}

// Evaluate board
function evaluate(bd){
  for(const [a,b,c] of LINES){
    if(bd[a] && bd[a] === bd[b] && bd[b] === bd[c]) return {winner: bd[a], line:[a,b,c]};
  }
  if(bd.every(Boolean)) return {draw:true};
  return null;
}

function emptyIndices(bd){ return bd.map((v,i)=>v==null?i:null).filter(x=>x!=null); }

// Minimax with alpha-beta pruning
function minimax(bd, isMaximizing, ai, human, depth=0, alpha=-Infinity, beta=Infinity){
  const res = evaluate(bd);
  if(res){
    if(res.winner === ai) return {score: 10 - depth};
    if(res.winner === human) return {score: depth - 10};
    if(res.draw) return {score: 0};
  }
  const empties = emptyIndices(bd);
  let bestMove = -1;
  if(isMaximizing){
    let bestScore = -Infinity;
    for(const i of empties){
      bd[i] = ai;
      const {score} = minimax(bd, false, ai, human, depth+1, alpha, beta);
      bd[i] = null;
      if(score > bestScore){ bestScore = score; bestMove = i; }
      alpha = Math.max(alpha, score);
      if(beta <= alpha) break;
    }
    return {score: bestScore, move: bestMove};
  } else {
    let bestScore = Infinity;
    for(const i of empties){
      bd[i] = human;
      const {score} = minimax(bd, true, ai, human, depth+1, alpha, beta);
      bd[i] = null;
      if(score < bestScore){ bestScore = score; bestMove = i; }
      beta = Math.min(beta, score);
      if(beta <= alpha) break;
    }
    return {score: bestScore, move: bestMove};
  }
}

// AI move decision based on difficulty
function aiMove(){
  const human = aiMark === 'X' ? 'O' : 'X';
  const empties = emptyIndices(board);
  let move = null;
  if(difficulty === 'Easy'){
    move = empties[Math.floor(Math.random()*empties.length)];
  } else if(difficulty === 'Medium'){
    if(Math.random() < 0.5) move = empties[Math.floor(Math.random()*empties.length)];
    else move = minimax([...board], true, aiMark, human).move;
  } else {
    move = minimax([...board], true, aiMark, human).move;
  }
  if(move != null) makeMove(move);
}

// Handle click
function onCellClick(i){
  const res = evaluate(board);
  if(res) return; // game over
  if(board[i]) return;
  if(mode === 'AI'){
    const current = xIsNext ? 'X' : 'O';
    const human = aiMark === 'X' ? 'O' : 'X';
    if(current !== human) return; // not human's turn
  }
  makeMove(i);
}

// Make move and update
function makeMove(i){
  board[i] = xIsNext ? 'X' : 'O';
  history.push(board.slice());
  xIsNext = !xIsNext;
  renderBoard();
  const res = evaluate(board);
  if(res){
    if(res.winner){
      statusEl.innerText = res.winner + ' wins!';
      if(res.winner === 'X') scores.X++; else scores.O++;
      updateScores();
    } else if(res.draw){
      statusEl.innerText = "It's a draw!";
      scores.D++; updateScores();
    }
    highlightLine(res && res.line);
    return;
  }
  // If vs AI and it's AI's turn -> AI move
  if(mode === 'AI'){
    const current = xIsNext ? 'X' : 'O';
    const human = aiMark === 'X' ? 'O' : 'X';
    if(current === aiMark){
      // small delay for nicer UX
      setTimeout(aiMove, 250);
    }
  }
}

// Highlight winning line
function highlightLine(line){
  if(!line) return;
  line.forEach(i=>{
    const cell = boardEl.querySelector('[data-index="'+i+'"]');
    if(cell) cell.style.boxShadow = '0 6px 24px rgba(99,102,241,0.28)';
  });
}

// Update status text
function updateStatus(){
  const res = evaluate(board);
  if(res){
    if(res.winner) statusEl.innerText = res.winner + ' wins!';
    else statusEl.innerText = "It's a draw!";
    return;
  }
  const current = xIsNext ? 'X' : 'O';
  if(mode === 'AI'){
    const human = aiMark === 'X' ? 'O' : 'X';
    statusEl.innerText = (current === human ? 'Your turn — ' + human : 'AI thinking — ' + (xIsNext? 'X':'O'));
  } else {
    statusEl.innerText = current + "'s turn";
  }
}

// Reset board
function resetBoard(swapStarter=false){
  board = Array(9).fill(null);
  history = [];
  if(swapStarter) xStarts = !xStarts;
  xIsNext = xStarts;
  // clear highlights
  boardEl.querySelectorAll('.cell').forEach(c=>c.style.boxShadow='');
  renderBoard();
}

// Undo last
function undo(){
  if(history.length < 1) return;
  history.pop(); // remove current
  const prev = history.pop(); // previous board to restore
  if(prev){
    board = prev.slice();
    history.push(board.slice());
  } else {
    board = Array(9).fill(null);
  }
  xIsNext = !xIsNext;
  renderBoard();
}

// Update scores UI
function updateScores(){
  scoreX.innerText = scores.X;
  scoreO.innerText = scores.O;
  scoreD.innerText = scores.D;
}

// Event listeners
restartBtn.addEventListener('click', ()=> resetBoard(false));
undoBtn.addEventListener('click', undo);
modeSelect.addEventListener('change', (e)=>{
  mode = e.target.value;
  resetBoard(false);
});
difficultySelect.addEventListener('change', (e)=>{ difficulty = e.target.value; });
aiMarkSelect.addEventListener('change', (e)=>{ aiMark = e.target.value; resetBoard(false); });
starterSelect.addEventListener('change', (e)=>{ xStarts = e.target.value === 'X'; xIsNext = xStarts; resetBoard(false); });
shareBtn.addEventListener('click', ()=>{
  const url = location.href;
  navigator.clipboard?.writeText(url).then(()=> alert('Page URL copied to clipboard — share with friends!'), ()=> alert('Could not copy'));
});
resetScoreBtn.addEventListener('click', ()=>{ scores = {X:0,O:0,D:0}; updateScores(); });

// Initial render (create 9 cells)
for(let i=0;i<9;i++){ const d = document.createElement('div'); d.className='cell'; d.setAttribute('data-index',i); d.addEventListener('click', ()=>onCellClick(i)); boardEl.appendChild(d); }
renderBoard();

// If AI starts and mode is AI, and AI is starter, make first move
function maybeInitialAI(){
  if(mode==='AI'){
    const human = aiMark === 'X' ? 'O' : 'X';
    if(xIsNext && aiMark === 'X'){ setTimeout(aiMove, 350); }
    if(!xIsNext && aiMark === 'O'){ /* if O starts but X starts by default, handled by starter select */ }
  }
}
maybeInitialAI();

</script>
</body>
</html>
