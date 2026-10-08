<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Аркада — 25 мини-игр</title>
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,560&family=Manrope:wght@400;600;700&display=swap" rel="stylesheet" />
<style>
  :root {
    --bg:#14120f; --card:#1e1b16; --ink:#f6f1e7; --muted:#a89b88;
    --line:#343026; --gold:#e2b15a; --gold-ink:#24180a; --good:#7dcea0; --bad:#e07a6a;
  }
  * { box-sizing:border-box; }
  html,body { margin:0; background:var(--bg); color:var(--ink); font-family:Manrope,system-ui,sans-serif; }
  header { padding:22px 20px 8px; max-width:1100px; margin:0 auto; }
  h1 { font-family:Fraunces,Georgia,serif; font-weight:560; font-size:clamp(2rem,5vw,3.2rem); margin:0; letter-spacing:-.03em; }
  header p { color:var(--muted); margin:6px 0 0; }
  .layout { max-width:1100px; margin:0 auto; padding:12px 16px 48px; display:grid; grid-template-columns:220px 1fr; gap:16px; }
  @media (max-width:800px) { .layout { grid-template-columns:1fr; } }
  nav { display:flex; flex-direction:column; gap:6px; max-height:78vh; overflow:auto; }
  @media (max-width:800px) { nav { flex-direction:row; flex-wrap:wrap; max-height:none; } }
  nav button, .pad button, .row button, .keys button {
    font:inherit; cursor:pointer; border:1px solid var(--line); background:var(--card); color:var(--ink);
    border-radius:12px; min-height:44px; padding:8px 12px; text-align:left;
  }
  nav button.on, .pad button:hover, button.primary { background:var(--gold); color:var(--gold-ink); border-color:var(--gold); font-weight:700; }
  .stage { background:var(--card); border:1px solid var(--line); border-radius:20px; padding:18px; min-height:460px; }
  h2 { font-family:Fraunces,Georgia,serif; margin:0 0 6px; font-size:1.8rem; }
  .hint { color:var(--muted); margin:0 0 14px; font-size:.92rem; }
  .score { font-weight:700; margin-bottom:10px; }
  canvas { background:#0e0d0b; border-radius:14px; display:block; max-width:100%; touch-action:none; }
  .grid { display:grid; gap:6px; }
  .cell, .tile { border-radius:10px; min-height:48px; display:grid; place-items:center; background:#2a261f; border:0; color:var(--ink); font:inherit; font-weight:700; cursor:pointer; }
  .row, .keys { display:flex; gap:8px; flex-wrap:wrap; margin-top:10px; }
  input { font:inherit; background:#0e0d0b; color:var(--ink); border:1px solid var(--line); border-radius:12px; padding:12px; min-height:44px; }
  .word { font-family:Fraunces,serif; font-size:2rem; letter-spacing:.12em; }
</style>
</head>
<body>
<header>
  <h1>Аркада</h1>
  <p>25 коротких игр. Счёт этой сессии не уходит с устройства.</p>
</header>
<div class="layout">
  <nav id="nav"></nav>
  <section class="stage" id="stage"></section>
</div>
<script>
const GAMES = [];
function game(name, hint, mount) { GAMES.push({ name, hint, mount }); }
const rand = (n) => Math.floor(Math.random() * n);
const pick = (a) => a[rand(a.length)];
function el(html) { const d = document.createElement("div"); d.innerHTML = html; return d; }
function canvasBox(w, h) {
  const c = document.createElement("canvas");
  c.width = w; c.height = h;
  const ctx = c.getContext("2d");
  return { c, ctx };
}
function loop(fn) {
  let id;
  const step = () => { fn(); id = requestAnimationFrame(step); };
  id = requestAnimationFrame(step);
  return () => cancelAnimationFrame(id);
}
game("Змейка", "Стрелки или свайп. Не врежьтесь в себя.", (root) => {
  const { c, ctx } = canvasBox(360, 360);
  root.append(c);
  const N = 18, cell = 20;
  let snake = [{x:8,y:8}], dir = {x:1,y:0}, food = {x:12,y:8}, alive = true, acc = 0, score = 0;
  const scoreEl = document.createElement("p"); scoreEl.className = "score"; root.prepend(scoreEl);
  const setDir = (x,y) => { if (snake.length < 2 || snake[0].x + x !== snake[1].x || snake[0].y + y !== snake[1].y) dir = {x,y}; };
  const keys = (e) => {
    if (e.key === "ArrowUp") setDir(0,-1);
    if (e.key === "ArrowDown") setDir(0,1);
    if (e.key === "ArrowLeft") setDir(-1,0);
    if (e.key === "ArrowRight") setDir(1,0);
  };
  window.addEventListener("keydown", keys);
  let sx, sy;
  c.addEventListener("pointerdown", e => { sx = e.clientX; sy = e.clientY; });
  c.addEventListener("pointerup", e => {
    const dx = e.clientX - sx, dy = e.clientY - sy;
    if (Math.abs(dx) > Math.abs(dy)) setDir(dx > 0 ? 1 : -1, 0); else setDir(0, dy > 0 ? 1 : -1);
  });
  let last = performance.now();
  const stop = loop(() => {
    const now = performance.now(); acc += now - last; last = now;
    if (acc > 130 && alive) {
      acc = 0;
      const h = { x: snake[0].x + dir.x, y: snake[0].y + dir.y };
      if (h.x < 0 || h.y < 0 || h.x >= N || h.y >= N || snake.some(p => p.x === h.x && p.y === h.y)) alive = false;
      else {
        snake.unshift(h);
        if (h.x === food.x && h.y === food.y) { score++; do { food = { x: rand(N), y: rand(N) }; } while (snake.some(p => p.x === food.x && p.y === food.y)); }
        else snake.pop();
      }
    }
    ctx.fillStyle = "#0e0d0b"; ctx.fillRect(0,0,360,360);
    ctx.fillStyle = "#e2b15a"; ctx.fillRect(food.x*cell, food.y*cell, cell-2, cell-2);
    ctx.fillStyle = "#f6f1e7";
    snake.forEach(p => ctx.fillRect(p.x*cell, p.y*cell, cell-2, cell-2));
    scoreEl.textContent = alive ? "Счёт: " + score : "Столкновение. Счёт: " + score;
  });
  return () => { stop(); window.removeEventListener("keydown", keys); };
});
game("Крестики-нолики", "Сыграйте против простого соперника.", (root) => {
  let board = Array(9).fill(""), over = "";
  const g = document.createElement("div"); g.className = "grid"; g.style.gridTemplateColumns = "repeat(3,72px)";
  const msg = document.createElement("p"); msg.className = "score";
  const win = (b, p) => [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]].some(l => l.every(i => b[i] === p));
  function draw() {
    g.innerHTML = "";
    board.forEach((v,i) => {
      const b = document.createElement("button"); b.className = "cell"; b.textContent = v || "·";
      b.onclick = () => {
        if (board[i] || over) return;
        board[i] = "X";
        if (win(board,"X")) over = "Вы выиграли";
        else if (board.every(Boolean)) over = "Ничья";
        else {
          const free = board.map((c,j)=>c?null:j).filter(x=>x!==null);
          const move = free.find(j => { const t=board.slice(); t[j]="O"; return win(t,"O"); })
            ?? free.find(j => { const t=board.slice(); t[j]="X"; return win(t,"X"); })
            ?? pick(free);
          board[move] = "O";
          if (win(board,"O")) over = "Соперник выиграл";
          else if (board.every(Boolean)) over = "Ничья";
        }
        msg.textContent = over || "Ваш ход";
        draw();
      };
      g.append(b);
    });
  }
  msg.textContent = "Ваш ход"; root.append(msg, g); draw();
});
game("Память", "Откройте пары одинаковых знаков.", (root) => {
  const icons = "ABCDEFGH".split("");
  let deck = icons.concat(icons).sort(() => Math.random()-.5).map((s)=>({s,open:false,done:false}));
  let open = [], lock = false, left = 8;
  const msg = document.createElement("p"); msg.className = "score"; msg.textContent = "Осталось пар: 8";
  const g = document.createElement("div"); g.className = "grid"; g.style.gridTemplateColumns = "repeat(4,64px)";
  function draw() {
    g.innerHTML = "";
    deck.forEach(card => {
      const b = document.createElement("button"); b.className = "cell";
      b.textContent = card.open || card.done ? card.s : "";
      b.onclick = () => {
        if (lock || card.open || card.done) return;
        card.open = true; open.push(card); draw();
        if (open.length === 2) {
          lock = true;
          setTimeout(() => {
            if (open[0].s === open[1].s) { open.forEach(c => c.done = true); left--; }
            open.forEach(c => c.open = false); open = []; lock = false;
            msg.textContent = left ? "Осталось пар: " + left : "Все пары найдены";
            draw();
          }, 500);
        }
      };
      g.append(b);
    });
  }
  root.append(msg, g); draw();
});
game("Реакция", "Ждите зелёный, затем нажмите.", (root) => {
  const b = document.createElement("button"); b.className = "primary"; b.style.minHeight = "160px"; b.style.width = "100%";
  b.textContent = "Нажмите, чтобы начать";
  let t0 = 0, wait, state = "idle";
  b.onclick = () => {
    if (state === "idle" || state === "done") {
      state = "wait"; b.textContent = "Ждите…"; b.style.background = "#2a261f";
      wait = setTimeout(() => { state = "go"; t0 = performance.now(); b.textContent = "Сейчас!"; b.style.background = "#7dcea0"; b.style.color = "#14281c"; }, 700 + rand(1800));
    } else if (state === "wait") {
      clearTimeout(wait); state = "done"; b.textContent = "Рано. Ещё раз"; b.style.background = "#e07a6a";
    } else if (state === "go") {
      state = "done"; b.style.background = "#e2b15a"; b.style.color = "#24180a";
      b.textContent = Math.round(performance.now() - t0) + " мс";
    }
  };
  root.append(b);
  return () => clearTimeout(wait);
});
game("2048", "Стрелки. Соберите 2048.", (root) => {
  let g = Array.from({length:4}, () => Array(4).fill(0)), score = 0;
  const msg = document.createElement("p"); msg.className = "score";
  const box = document.createElement("div"); box.className = "grid"; box.style.gridTemplateColumns = "repeat(4,64px)";
  const spawn = () => {
    const empty = [];
    g.forEach((row,y)=>row.forEach((v,x)=>{ if(!v) empty.push([x,y]); }));
    if (!empty.length) return;
    const [x,y] = pick(empty); g[y][x] = Math.random()<.9 ? 2 : 4;
  };
  const slide = (row) => {
    const a = row.filter(Boolean);
    for (let i=0;i<a.length-1;i++) if (a[i]===a[i+1]) { a[i]*=2; score+=a[i]; a.splice(i+1,1); }
    while (a.length<4) a.push(0);
    return a;
  };
  function move(dir) {
    const before = JSON.stringify(g);
    if (dir==="l") g = g.map(slide);
    if (dir==="r") g = g.map(r => slide(r.slice().reverse()).reverse());
    if (dir==="u" || dir==="d") {
      const cols = [0,1,2,3].map(x => g.map(r => r[x]));
      const next = cols.map(c => dir==="u" ? slide(c) : slide(c.slice().reverse()).reverse());
      g = [0,1,2,3].map(y => [0,1,2,3].map(x => next[x][y]));
    }
    if (JSON.stringify(g)!==before) spawn();
    draw();
  }
  function draw() {
    box.innerHTML = "";
    g.flat().forEach(v => { const t = document.createElement("div"); t.className = "tile"; t.textContent = v||""; t.style.background = v ? "#e2b15a" : "#2a261f"; t.style.color = v ? "#24180a" : "#f6f1e7"; box.append(t); });
    msg.textContent = g.flat().includes(2048) ? "2048 собрано. Счёт " + score : "Счёт: " + score;
  }
  const keys = (e) => {
    const map = {ArrowLeft:"l",ArrowRight:"r",ArrowUp:"u",ArrowDown:"d"};
    if (map[e.key]) { e.preventDefault(); move(map[e.key]); }
  };
  window.addEventListener("keydown", keys);
  spawn(); spawn(); draw();
  root.append(msg, box);
  const keysRow = el('<div class="keys"></div>');
  [["←","l"],["→","r"],["↑","u"],["↓","d"]].forEach(([l,d]) => {
    const b = document.createElement("button"); b.textContent = l; b.onclick = () => move(d); keysRow.append(b);
  });
  root.append(keysRow);
  return () => window.removeEventListener("keydown", keys);
});
game("Сапёр", "Откройте клетки. Флажок — правый клик или долгое нажатие.", (root) => {
  const W=8,H=8,M=10;
  const cells = Array.from({length:H*W}, (_,i)=>({i, mine:false, open:false, flag:false, n:0}));
  const mines = new Set();
  while (mines.size<M) mines.add(rand(H*W));
  mines.forEach(i => cells[i].mine = true);
  const nb = (i) => [-1,0,1].flatMap(dy => [-1,0,1].map(dx => {
    const x=i%W+dx, y=(i/W|0)+dy;
    return dx||dy ? (x>=0&&y>=0&&x<W&&y<H ? y*W+x : -1) : -1;
  })).filter(v=>v>=0);
  cells.forEach((c,i)=>{ if(!c.mine) c.n = nb(i).filter(j=>cells[j].mine).length; });
  const msg = document.createElement("p"); msg.className="score"; msg.textContent="Мин: 10";
  const grid = document.createElement("div"); grid.className="grid"; grid.style.gridTemplateColumns="repeat(8,40px)";
  let dead=false;
  function open(i) {
    const c = cells[i]; if (c.open || c.flag || dead) return;
    c.open = true;
    if (c.mine) { dead=true; cells.forEach(x=>{ if(x.mine) x.open=true; }); msg.textContent="Мина."; return; }
    if (c.n===0) nb(i).forEach(open);
  }
  function draw() {
    grid.innerHTML="";
    cells.forEach(c => {
      const b=document.createElement("button"); b.className="cell"; b.style.minHeight="40px";
      b.textContent = c.flag && !c.open ? "F" : c.open ? (c.mine ? "•" : (c.n||"")) : "";
      if (c.open && !c.mine) b.style.background = "#3a342b";
      b.oncontextmenu = (e)=>{ e.preventDefault(); if(!c.open&&!dead){ c.flag=!c.flag; draw(); } };
      let hold;
      b.onpointerdown = () => { hold = setTimeout(()=>{ if(!c.open){ c.flag=!c.flag; draw(); } }, 400); };
      b.onpointerup = () => clearTimeout(hold);
      b.onclick = () => { open(c.i); if (!dead && cells.every(x=>x.mine||x.open)) msg.textContent="Поле чистое"; draw(); };
      grid.append(b);
    });
  }
  root.append(msg,grid); draw();
});
game("Понг", "Мышь или палец. Отбейте мяч.", (root) => {
  const {c,ctx}=canvasBox(420,260); root.append(c);
  let py=100, oy=100, bx=210, by=130, vx=3.2, vy=2.2, s=0, o=0;
  c.addEventListener("pointermove", e => { const r=c.getBoundingClientRect(); py = (e.clientY-r.top)*(c.height/r.height)-30; });
  return loop(() => {
    bx+=vx; by+=vy;
    if (by<0||by>c.height-10) vy*=-1;
    oy += Math.sign(by-oy-20)*2.4;
    if (bx<16 && by>py && by<py+60) { vx=Math.abs(vx)+.1; s++; }
    if (bx>c.width-26 && by>oy && by<oy+60) vx=-Math.abs(vx);
    if (bx<0) { o++; bx=210; by=130; vx=3.2; }
    if (bx>c.width) { s++; bx=210; by=130; vx=-3.2; }
    ctx.fillStyle="#0e0d0b"; ctx.fillRect(0,0,c.width,c.height);
    ctx.fillStyle="#f6f1e7"; ctx.fillRect(8,py,8,60); ctx.fillRect(c.width-16,oy,8,60); ctx.fillRect(bx,by,10,10);
    ctx.fillStyle="#e2b15a"; ctx.font="16px sans-serif"; ctx.fillText(s+" : "+o, 190, 24);
  });
});
game("Арканоид", "Двигайте платформу. Сбейте кирпичи.", (root) => {
  const {c,ctx}=canvasBox(420,280); root.append(c);
  let px=170, bx=200, by=240, vx=3, vy=-3;
  const bricks=[];
  for(let y=0;y<4;y++) for(let x=0;x<8;x++) bricks.push({x:12+x*50,y:20+y*22,on:true});
  c.addEventListener("pointermove", e => { const r=c.getBoundingClientRect(); px=(e.clientX-r.left)*(c.width/r.width)-40; });
  return loop(()=>{
    bx+=vx; by+=vy;
    if(bx<0||bx>c.width-8) vx*=-1;
    if(by<0) vy*=-1;
    if(by>250 && bx>px && bx<px+80) vy=-Math.abs(vy);
    bricks.forEach(br=>{ if(br.on && bx>br.x && bx<br.x+46 && by>br.y && by<br.y+16){ br.on=false; vy*=-1; }});
    if(by>c.height){ bx=200; by=200; vy=-3; }
    ctx.fillStyle="#0e0d0b"; ctx.fillRect(0,0,c.width,c.height);
    ctx.fillStyle="#e2b15a"; bricks.forEach(br=>{ if(br.on) ctx.fillRect(br.x,br.y,46,16); });
    ctx.fillStyle="#f6f1e7"; ctx.fillRect(px,258,80,10); ctx.fillRect(bx,by,8,8);
    if(bricks.every(b=>!b.on)) ctx.fillText("Поле чистое", 160, 150);
  });
});
game("Саймон", "Повторите вспышки.", (root) => {
  const colors=["#e07a6a","#e2b15a","#7dcea0","#8eb6e0"];
  let seq=[], step=0, lock=true;
  const msg=document.createElement("p"); msg.className="score"; msg.textContent="Смотрите";
  const g=document.createElement("div"); g.className="grid"; g.style.gridTemplateColumns="repeat(2,120px)";
  const pads=colors.map((col)=>{
    const b=document.createElement("button"); b.style.background=col; b.style.minHeight="80px"; b.style.border="0"; b.style.borderRadius="12px"; b.style.opacity=".55";
    g.append(b); return b;
  });
  function flash(i){ pads[i].style.opacity="1"; setTimeout(()=>pads[i].style.opacity=".55", 280); }
  function next(){
    seq.push(rand(4)); step=0; lock=true; msg.textContent="Длина "+seq.length;
    seq.forEach((n,i)=>setTimeout(()=>flash(n), 400*(i+1)));
    setTimeout(()=>{ lock=false; msg.textContent="Повторите"; }, 400*(seq.length+1));
  }
  pads.forEach((b,i)=> b.onclick=()=>{
    if(lock) return;
    flash(i);
    if(i!==seq[step]) { msg.textContent="Сбой на длине "+seq.length; seq=[]; setTimeout(next,600); return; }
    step++;
    if(step===seq.length){ msg.textContent="Длина "+seq.length; setTimeout(next,500); }
  });
  root.append(msg,g); next();
});
game("Камень, ножницы, бумага", "Один раунд против поля.", (root) => {
  const msg=document.createElement("p"); msg.className="score"; msg.textContent="Выберите";
  const row=document.createElement("div"); row.className="row";
  ["Камень","Ножницы","Бумага"].forEach((name,idx)=>{
    const b=document.createElement("button"); b.textContent=name; b.onclick=()=>{
      const ai=rand(3); const names=["Камень","Ножницы","Бумага"];
      const win=(idx===0&&ai===1)||(idx===1&&ai===2)||(idx===2&&ai===0);
      msg.textContent = idx===ai ? "Ничья, у обоих "+names[ai] : (win?"Вы: ":"Поле: ")+name+" против "+names[ai];
    };
    row.append(b);
  });
  root.append(msg,row);
});
game("Выше или ниже", "Угадайте, больше следующая карта или меньше.", (root) => {
  const faces="23456789TJQKA";
  let cur=rand(13), score=0;
  const msg=document.createElement("p"); msg.className="word";
  const line=document.createElement("p"); line.className="score";
  function show(){ msg.textContent=faces[cur]; line.textContent="Серия: "+score; }
  const row=document.createElement("div"); row.className="row";
  ["Выше","Ниже"].forEach((label,hi)=>{
    const b=document.createElement("button"); b.textContent=label; b.onclick=()=>{
      const n=rand(13); const ok=hi? n>cur : n<cur;
      if(n===cur) line.textContent="Та же карта";
      else if(ok){ score++; cur=n; show(); }
      else { line.textContent="Было "+faces[cur]+", стало "+faces[n]+". Серия обнулена"; score=0; cur=n; msg.textContent=faces[cur]; }
    };
    row.append(b);
  });
  root.append(msg,line,row); show();
});
game("Прицел", "Попадите по кругам за 20 секунд.", (root) => {
  const {c,ctx}=canvasBox(420,280); root.append(c);
  const msg=document.createElement("p"); msg.className="score"; root.prepend(msg);
  let hits=0, left=20, t=0, x=80, y=80, r=22;
  const place=()=>{ x=30+rand(360); y=30+rand(210); r=16+rand(16); };
  c.addEventListener("pointerdown", e=>{
    const box=c.getBoundingClientRect();
    const px=(e.clientX-box.left)*(c.width/box.width), py=(e.clientY-box.top)*(c.height/box.height);
    if((px-x)**2+(py-y)**2 < r*r){ hits++; place(); }
  });
  let last=performance.now();
  return loop(()=>{
    const now=performance.now(); t+=(now-last)/1000; last=now;
    ctx.fillStyle="#0e0d0b"; ctx.fillRect(0,0,c.width,c.height);
    ctx.fillStyle="#e2b15a"; ctx.beginPath(); ctx.arc(x,y,r,0,7); ctx.fill();
    msg.textContent = t<left ? "Попадания: "+hits+" · "+Math.ceil(left-t)+" с" : "Итог: "+hits;
  });
});
game("Пятнашки", "Поставьте числа по порядку.", (root) => {
  let a=[...Array(15).keys()].map(n=>n+1).concat(0);
  function nb15(i){ const x=i%4,y=i/4|0,o=[]; if(x)o.push(i-1); if(x<3)o.push(i+1); if(y)o.push(i-4); if(y<3)o.push(i+4); return o; }
  for(let i=0;i<80;i++){ const z=a.indexOf(0); const n=nb15(z); const j=pick(n); [a[z],a[j]]=[a[j],a[z]]; }
  const g=document.createElement("div"); g.className="grid"; g.style.gridTemplateColumns="repeat(4,64px)";
  const msg=document.createElement("p"); msg.className="score";
  function draw(){
    g.innerHTML="";
    a.forEach((n,i)=>{
      const b=document.createElement("button"); b.className="cell"; b.textContent=n||"";
      b.onclick=()=>{
        const z=a.indexOf(0); if(!nb15(z).includes(i)) return;
        [a[z],a[i]]=[a[i],a[z]];
        msg.textContent=a.slice(0,15).every((v,k)=>v===k+1)?"Собрано":"Ещё ход";
        draw();
      };
      g.append(b);
    });
  }
  root.append(msg,g); draw();
});
game("Четыре в ряд", "Соберите линию из четырёх. Ваш цвет — золотой.", (root) => {
  const W=7,H=6; let b=Array.from({length:H},()=>Array(W).fill(0)), over="";
  const msg=document.createElement("p"); msg.className="score"; msg.textContent="Ваш ход";
  const g=document.createElement("div"); g.className="grid"; g.style.gridTemplateColumns="repeat(7,42px)";
  function line(board,p){
    const dirs=[[1,0],[0,1],[1,1],[1,-1]];
    for(let y=0;y<H;y++) for(let x=0;x<W;x++) for(const [dx,dy] of dirs){
      let ok=true;
      for(let k=0;k<4;k++){ const nx=x+dx*k, ny=y+dy*k; if(nx<0||ny<0||nx>=W||ny>=H||board[ny][nx]!==p) ok=false; }
      if(ok) return true;
    }
    return false;
  }
  function dropOn(board,col,p){ for(let y=H-1;y>=0;y--) if(!board[y][col]){ board[y][col]=p; return true;} return false; }
  function draw(){
    g.innerHTML="";
    for(let y=0;y<H;y++) for(let x=0;x<W;x++){
      const cell=document.createElement("button"); cell.className="cell"; cell.style.minHeight="42px";
      cell.style.background = b[y][x]===1?"#e2b15a": b[y][x]===2?"#f6f1e7":"#2a261f";
      cell.onclick=()=>{
        if(over||b[0][x]) return;
        dropOn(b,x,1);
        if(line(b,1)) over="Вы собрали четыре";
        else if(b[0].every(Boolean)) over="Ничья";
        else {
          const opts=[...Array(W).keys()].filter(c=>!b[0][c]);
          let choice=opts.find(c=>{ const t=b.map(r=>r.slice()); return dropOn(t,c,2)&&line(t,2); })
            ?? opts.find(c=>{ const t=b.map(r=>r.slice()); return dropOn(t,c,1)&&line(t,1); })
            ?? pick(opts);
          dropOn(b,choice,2);
          if(line(b,2)) over="Поле собрало четыре";
        }
        msg.textContent=over||"Ваш ход"; draw();
      };
      g.append(cell);
    }
  }
  root.append(msg,g); draw();
});
game("Прыжок", "Пробел или касание — прыжок через столбики.", (root) => {
  const {c,ctx}=canvasBox(420,240); root.append(c);
  let y=160, vy=0, pipes=[], t=0, score=0, alive=true;
  const msg=document.createElement("p"); msg.className="score"; root.prepend(msg);
  const jump=()=>{ if(alive) vy=-6; else { y=160; vy=0; pipes=[]; score=0; alive=true; } };
  const key=(e)=>{ if(e.code==="Space"){ e.preventDefault(); jump(); } };
  window.addEventListener("keydown", key); c.addEventListener("pointerdown", jump);
  const stop=loop(()=>{
    if(alive){
      t++; vy+=.28; y+=vy;
      if(t%90===0) pipes.push({x:420, h:40+rand(100)});
      pipes.forEach(p=>p.x-=2.4);
      pipes=pipes.filter(p=>p.x>-40);
      pipes.forEach(p=>{
        const gap=y>p.h && y<p.h+78;
        if(p.x<70 && p.x>30 && !gap) alive=false;
        if(!p.scored && p.x<40){ p.scored=true; score++; }
      });
      if(y>220||y<0) alive=false;
    }
    ctx.fillStyle="#0e0d0b"; ctx.fillRect(0,0,420,240);
    ctx.fillStyle="#e2b15a";
    pipes.forEach(p=>{ ctx.fillRect(p.x,0,36,p.h); ctx.fillRect(p.x,p.h+86,36,240); });
    ctx.fillStyle="#f6f1e7"; ctx.beginPath(); ctx.arc(56,y,10,0,7); ctx.fill();
    msg.textContent = alive ? "Счёт: "+score : "Столкновение: "+score+". Прыжок — заново";
  });
  return ()=>{ stop(); window.removeEventListener("keydown", key); };
});
game("Огни", "Погасите все клетки. Ход переключает соседей.", (root) => {
  let a=Array.from({length:25},()=>rand(2));
  const g=document.createElement("div"); g.className="grid"; g.style.gridTemplateColumns="repeat(5,52px)";
  const msg=document.createElement("p"); msg.className="score";
  function tog(i){ const x=i%5,y=i/5|0; [[x,y],[x-1,y],[x+1,y],[x,y-1],[x,y+1]].forEach(([cx,cy])=>{
    if(cx>=0&&cy>=0&&cx<5&&cy<5) a[cy*5+cx]^=1;
  }); }
  function draw(){
    g.innerHTML="";
    a.forEach((v,i)=>{
      const b=document.createElement("button"); b.className="cell";
      b.style.background=v?"#e2b15a":"#2a261f";
      b.onclick=()=>{ tog(i); msg.textContent=a.every(x=>!x)?"Все погасли":"Ещё горят"; draw(); };
      g.append(b);
    });
  }
  root.append(msg,g); draw();
});
game("Счёт в уме", "Десять примеров. Напишите ответ и нажмите Enter.", (root) => {
  let n=0, ok=0, a=0,b=0,op="+";
  const q=document.createElement("p"); q.className="word";
  const input=document.createElement("input"); input.inputMode="numeric";
  const msg=document.createElement("p"); msg.className="score";
  function next(){
    if(n===10){ q.textContent="Готово"; msg.textContent="Верно: "+ok+" из 10"; input.disabled=true; return; }
    a=2+rand(10); b=2+rand(10); op=Math.random()<.5?"+":"×";
    q.textContent=(n+1)+". "+a+" "+op+" "+b; input.value=""; msg.textContent="Верно: "+ok;
  }
  input.onchange=()=>{
    const need=op==="+"?a+b:a*b;
    if(+input.value===need) ok++;
    n++; next();
  };
  root.append(q,input,msg); next();
});
game("Цвет слова", "Нажмите цвет чернил, не читая слово.", (root) => {
  const names=[["золото","#e2b15a"],["мел","#f6f1e7"],["мята","#7dcea0"],["коралл","#e07a6a"]];
  let score=0, round=0, ink=0;
  const word=document.createElement("p"); word.className="word";
  const msg=document.createElement("p"); msg.className="score";
  const row=document.createElement("div"); row.className="row";
  function deal(){
    if(round===12){ word.textContent="Конец"; msg.textContent="Точно: "+score+" из 12"; row.innerHTML=""; return; }
    ink=rand(4); const label=rand(4);
    word.textContent=names[label][0]; word.style.color=names[ink][1];
    msg.textContent="Раунд "+(round+1);
  }
  names.forEach((n,i)=>{
    const b=document.createElement("button"); b.textContent=n[0];
    b.onclick=()=>{ if(round>=12)return; if(i===ink) score++; round++; deal(); };
    row.append(b);
  });
  root.append(word,msg,row); deal();
});
game("Двадцать одно", "Наберите ближе к 21, чем сдача.", (root) => {
  const card=()=>2+rand(10);
  let you=[card(),card()], dealer=[card()];
  const msg=document.createElement("p"); msg.className="score";
  const row=document.createElement("div"); row.className="row";
  const sum=a=>a.reduce((s,n)=>s+n,0);
  function paint(extra){ msg.textContent="Вы: "+you.join(" ")+" = "+sum(you)+" · Сдача: "+dealer.join(" ")+(extra||""); }
  function end(){
    while(sum(dealer)<17) dealer.push(card());
    const y=sum(you), d=sum(dealer);
    const text = y>21 ? "Перебор" : d>21||y>d ? "Вы ближе" : y===d ? "Ничья" : "Сдача ближе";
    paint(" · "+text+" ("+d+")");
    row.innerHTML="";
  }
  const hit=document.createElement("button"); hit.textContent="Ещё";
  const stand=document.createElement("button"); stand.textContent="Хватит";
  hit.onclick=()=>{ you.push(card()); if(sum(you)>=21) end(); else paint(); };
  stand.onclick=end;
  row.append(hit,stand); root.append(msg,row); paint();
});
game("Ловец", "Двигайте корзину и ловите круги.", (root) => {
  const {c,ctx}=canvasBox(420,260); root.append(c);
  let x=180, items=[], t=0, score=0, miss=0;
  c.addEventListener("pointermove", e=>{ const r=c.getBoundingClientRect(); x=(e.clientX-r.left)*(c.width/r.width)-30; });
  return loop(()=>{
    t++; if(t%40===0) items.push({x:20+rand(360), y:-10, v:2+Math.random()*2});
    items.forEach(it=>it.y+=it.v);
    items=items.filter(it=>{
      if(it.y>230 && it.x>x && it.x<x+60){ score++; return false; }
      if(it.y>260){ miss++; return false; }
      return true;
    });
    ctx.fillStyle="#0e0d0b"; ctx.fillRect(0,0,420,260);
    ctx.fillStyle="#e2b15a"; items.forEach(it=>{ ctx.beginPath(); ctx.arc(it.x,it.y,8,0,7); ctx.fill(); });
    ctx.fillStyle="#f6f1e7"; ctx.fillRect(x,236,60,12);
    ctx.fillText("Поймано "+score+"  мимо "+miss, 12, 20);
  });
});
game("Виселица", "Шесть ошибок.", (root) => {
  const words=["весна","книга","море","город","свет","камень","река","поле","дом","мост"];
  const word=pick(words), guessed=new Set(); let bad=0;
  const view=document.createElement("p"); view.className="word";
  const msg=document.createElement("p"); msg.className="score";
  const letters="абвгдежзийклмнопрстуфхцчшщыэюя";
  const row=document.createElement("div"); row.className="keys";
  function paint(){
    view.textContent=[...word].map(ch=>guessed.has(ch)?ch:"_").join(" ");
    msg.textContent = [...word].every(ch=>guessed.has(ch)) ? "Слово открыто" : bad>=6 ? "Слово: "+word : "Ошибки: "+bad+"/6";
  }
  [...letters].forEach(ch=>{
    const b=document.createElement("button"); b.textContent=ch; b.onclick=()=>{
      if(bad>=6||[...word].every(c=>guessed.has(c))) return;
      guessed.add(ch); if(!word.includes(ch)) bad++; b.disabled=true; paint();
    };
    row.append(b);
  });
  root.append(view,msg,row); paint();
});
game("Ним", "13 камней. Берите 1–3. Кто берёт последний — проиграл.", (root) => {
  let n=13;
  const msg=document.createElement("p"); msg.className="score";
  const row=document.createElement("div"); row.className="row";
  function paint(){ msg.textContent = n>0 ? "Камней: "+n : "Партия окончена"; }
  [1,2,3].forEach(k=>{
    const b=document.createElement("button"); b.textContent="Взять "+k;
    b.onclick=()=>{
      if(n<=0) return;
      if(k>=n){ n=0; msg.textContent="Вы взяли последний камень"; row.innerHTML=""; return; }
      n-=k;
      let take = n%4===1 ? 1 : (n%4===0?3:(n%4)-1);
      if(take<1) take=1;
      n-=take;
      if(n<=0){ msg.textContent="Поле взяло последний камень. Вы выиграли"; row.innerHTML=""; return; }
      paint();
    };
    row.append(b);
  });
  root.append(msg,row); paint();
});
game("Набор", "Печатайте слова 30 секунд.", (root) => {
  const bank=["тишина","бумага","север","лампа","мост","облако","река","песок","искра","ветка"];
  let left=30, score=0, current=pick(bank), acc=0;
  const word=document.createElement("p"); word.className="word"; word.textContent=current;
  const input=document.createElement("input");
  const msg=document.createElement("p"); msg.className="score";
  input.oninput=()=>{
    if(input.value.trim().toLowerCase()===current){ score++; current=pick(bank); word.textContent=current; input.value=""; }
  };
  let last=performance.now();
  const stop=loop(()=>{
    const now=performance.now(); acc+=now-last; last=now;
    if(acc>1000 && left>0){ acc=0; left--; }
    msg.textContent = left>0 ? "Слов: "+score+" · "+left+" с" : "Итог: "+score;
    input.disabled=left<=0;
  });
  root.append(word,input,msg);
  return stop;
});
game("Три в ряд", "Меняйте соседние клетки и собирайте тройку.", (root) => {
  const C=6, colors=["#e2b15a","#e07a6a","#7dcea0","#8eb6e0"];
  let a=Array.from({length:C*C},()=>rand(colors.length));
  let sel=null, score=0;
  const msg=document.createElement("p"); msg.className="score";
  const g=document.createElement("div"); g.className="grid"; g.style.gridTemplateColumns="repeat(6,46px)";
  const at=(x,y)=>y*C+x;
  function clearMatches(){
    const kill=new Set();
    for(let y=0;y<C;y++) for(let x=0;x<C;x++){
      const v=a[at(x,y)];
      if(x<C-2 && v===a[at(x+1,y)] && v===a[at(x+2,y)]) [0,1,2].forEach(k=>kill.add(at(x+k,y)));
      if(y<C-2 && v===a[at(x,y+1)] && v===a[at(x,y+2)]) [0,1,2].forEach(k=>kill.add(at(x,y+k)));
    }
    if(!kill.size) return false;
    score+=kill.size;
    kill.forEach(i=>a[i]=-1);
    for(let x=0;x<C;x++){
      const col=[];
      for(let y=C-1;y>=0;y--) if(a[at(x,y)]!==-1) col.push(a[at(x,y)]);
      while(col.length<C) col.push(rand(colors.length));
      for(let y=C-1;y>=0;y--) a[at(x,y)]=col[C-1-y];
    }
    return true;
  }
  function draw(){
    g.innerHTML="";
    a.forEach((v,i)=>{
      const b=document.createElement("button"); b.className="cell"; b.style.background=colors[v]; b.style.minHeight="46px";
      if(sel===i) b.style.outline="2px solid #f6f1e7";
      b.onclick=()=>{
        if(sel===null){ sel=i; draw(); return; }
        const dx=Math.abs(sel%C-i%C), dy=Math.abs((sel/C|0)-(i/C|0));
        if(dx+dy===1){ [a[sel],a[i]]=[a[i],a[sel]]; if(!clearMatches()) [a[sel],a[i]]=[a[i],a[sel]]; else while(clearMatches()){} }
        sel=null; msg.textContent="Снято клеток: "+score; draw();
      };
      g.append(b);
    });
  }
  while(clearMatches()){}
  score=0; root.append(msg,g); draw();
});
game("Башня", "Нажмите, когда блок над башней.", (root) => {
  const {c,ctx}=canvasBox(320,360); root.append(c);
  const msg=document.createElement("p"); msg.className="score"; root.prepend(msg);
  let w=140, x=90, dir=2.2, y=40, stack=[{x:90,w:140,y:300}], lost=false;
  const drop=()=>{
    if(lost){ w=140; x=90; y=40; stack=[{x:90,w:140,y:300}]; lost=false; return; }
    const top=stack[stack.length-1];
    const left=Math.max(x, top.x), right=Math.min(x+w, top.x+top.w);
    const nw=right-left;
    if(nw<12){ lost=true; return; }
    x=left; w=nw; stack.push({x,w,y:300-stack.length*18});
    y=40; dir=dir>0?2.2+stack.length*.05:-(2.2+stack.length*.05);
  };
  c.addEventListener("pointerdown", drop);
  const key=(e)=>{ if(e.code==="Space"){ e.preventDefault(); drop(); } };
  window.addEventListener("keydown", key);
  const stop=loop(()=>{
    if(!lost){ x+=dir; if(x<0||x+w>320) dir*=-1; }
    ctx.fillStyle="#0e0d0b"; ctx.fillRect(0,0,320,360);
    ctx.fillStyle="#e2b15a"; stack.forEach(b=>ctx.fillRect(b.x,b.y,b.w,16));
    ctx.fillStyle="#f6f1e7"; if(!lost) ctx.fillRect(x,y,w,16);
    msg.textContent = lost ? "Башня упала на "+(stack.length-1)+". Нажмите ещё раз" : "Этажей: "+(stack.length-1);
  });
  return ()=>{ stop(); window.removeEventListener("keydown", key); };
});
const nav=document.getElementById("nav");
const stage=document.getElementById("stage");
let cleanup=()=>{};
function openGame(i){
  cleanup(); cleanup=()=>{};
  [...nav.children].forEach((b,idx)=>b.classList.toggle("on", idx===i));
  const g=GAMES[i];
  stage.innerHTML="<h2>"+g.name+"</h2><p class='hint'>"+g.hint+"</p>";
  const ret=g.mount(stage);
  if(typeof ret==="function") cleanup=ret;
}
GAMES.forEach((g,i)=>{
  const b=document.createElement("button");
  b.textContent=(i+1)+". "+g.name;
  b.onclick=()=>openGame(i);
  nav.append(b);
});
openGame(0);
</script>
</body>
</html>
