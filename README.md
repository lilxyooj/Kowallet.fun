# Kowallet.fun
crypto 
<!DOCTYPE html>
<html>
  <head>
    <title>Kowallet</title>
  </head>
  <body style="font-family:sans-serif; text-align:center; margin-top:100px;">
    <h1>Hello, world!</h1>
    <p>Welcome to Kowallet + Vercel!</p>
  </body>
</html>
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>KoWallet (Simulation)</title>
  <style>
    body{font-family:sans-serif;max-width:900px;margin:40px auto;padding:0 20px}
    header{display:flex;justify-content:space-between;align-items:center}
    .card{border:1px solid #ddd;padding:12px;border-radius:8px;margin:12px 0}
    button{padding:8px 12px;border-radius:6px;border:0;background:#2563eb;color:white;cursor:pointer}
  </style>
</head>
<body>
  <header>
    <h1>Token Launch Simulator</h1>
    <div>Credits: <span id="credits">1000</span></div>
  </header>

  <section class="card">
    <h2>Create Token</h2>
    <input id="name" placeholder="Token name" />
    <input id="sym" placeholder="SYM" style="width:80px"/>
    <button id="create">Create (cost: 10 credits)</button>
    <div id="createMsg"></div>
  </section>

  <section class="card">
    <h2>Active Launches</h2>
    <div id="launches"></div>
  </section>

  <section class="card">
    <h2>Leaderboard</h2>
    <div id="board"></div>
  </section>

<script>
/* Simple in-memory simulation. Not persistent across devices. */
const startCredits = () => Number(localStorage.getItem('credits')||1000);
let credits = startCredits();
document.getElementById('credits').innerText = credits;

const saveCredits = () => { localStorage.setItem('credits', credits); document.getElementById('credits').innerText = credits; }
const launches = JSON.parse(localStorage.getItem('launches')||'[]');
const board = JSON.parse(localStorage.getItem('board')||'[]');

function renderLaunches(){
  const container = document.getElementById('launches');
  container.innerHTML = '';
  launches.forEach((l,i)=>{
    const el = document.createElement('div');
    el.style.margin='8px 0';
    el.innerHTML = `<strong>${l.name} (${l.sym})</strong> — target ${l.target} — raised ${l.raised} <button data-i="${i}" class="buy">Buy (10)</button>`;
    container.appendChild(el);
  });
  document.querySelectorAll('.buy').forEach(b=>{
    b.onclick = e=>{
      const i = e.target.dataset.i|0;
      if(credits < 10){ alert('Not enough credits'); return; }
      credits -= 10; saveCredits();
      launches[i].raised += 10;
      // instant check: if reached, resolve success else random chance to rug
      if(launches[i].raised >= launches[i].target){
        resolveSuccess(i);
      } else {
        // small chance to rug each buy
        if(Math.random() < 0.12){ resolveRug(i); }
      }
      persist();
      renderLaunches(); renderBoard();
    }
  });
}

function resolveSuccess(i){
  const l = launches[i];
  // winners: creator + buyers (simple split)
  board.push({who:'You', token:l.sym, result:'moon', profit: l.target});
  alert(`${l.sym} launched! You earned ${l.target} credits on leaderboard.`);
  launches.splice(i,1);
}
function resolveRug(i){
  const l = launches[i];
  board.push({who:'You', token:l.sym, result:'rug', profit:-Math.floor(l.raised/2)});
  alert(`${l.sym} got rugged. You lost some credits.`);
  launches.splice(i,1);
}
function persist(){
  localStorage.setItem('launches', JSON.stringify(launches));
  localStorage.setItem('board', JSON.stringify(board));
}
function renderBoard(){
  const b = document.getElementById('board');
  b.innerHTML = board.slice(-10).map(x=>`<div>${x.token} — ${x.result} — ${x.profit}</div>`).join('');
}

document.getElementById('create').onclick = ()=>{
  const name = document.getElementById('name').value||'Meme';
  const sym = (document.getElementById('sym').value||('M'+Math.floor(Math.random()*900))).toUpperCase();
  if(credits < 10) return alert('Not enough credits to create');
  credits -= 10; saveCredits();
  const target = 100 + Math.floor(Math.random()*400);
  launches.push({name, sym, target, raised:0});
  persist(); renderLaunches(); renderBoard();
  document.getElementById('createMsg').innerText = `Created ${sym} — target ${target}`;
}

renderLaunches(); renderBoard();
</script>
</body>
</html>

