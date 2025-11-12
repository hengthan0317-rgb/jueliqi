Than Yu Heng陈煜恒, [8/26/2025 8:54 PM]
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>角力棋 8×8（模式切换）</title>
<style>
  :root{
    --cell: 56px;     /* 格子边长 */
    --gap:  4px;      /* 格间距 */
    --rad:  10px;     /* 格圆角 */
    --piece: calc(var(--cell) - 10px);
    --speed: 260ms;   /* 单段移动动画时间 */
  }
  *{box-sizing:border-box}
  body{margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial; background:#fafafa;}
  .wrap{max-width:1100px; margin:20px auto; padding:0 16px;}
  h1{margin:0 0 12px; font-size:20px}
  .panel{display:flex; flex-wrap:wrap; gap:10px; align-items:center; margin-bottom:12px}
  .panel label{font-size:14px}
  select,button,input[type=checkbox]{font-size:14px; padding:6px 8px}
  .stage{position:relative; display:inline-block}
  .board{
    display:grid;
    grid-template-columns: repeat(8, var(--cell));
    grid-auto-rows: var(--cell);
    gap: var(--gap);
    user-select:none;
    position:relative;
    z-index:0;
  }
  .cell{
    width:var(--cell); height:var(--cell);
    border-radius:var(--rad);
    border:1px solid #d9d9d9;
    position:relative;
  }
  .light{ background:#ffffff; }
  .dark{  background:#e9f0f8; }
  .goal{ position:absolute; inset:2px; border:2px dashed rgba(0,0,0,.18); border-radius:8px; }
  .hint{ box-shadow: inset 0 0 0 3px rgba(76,175,80,.60); }
  .swap{ box-shadow: inset 0 0 0 3px rgba(233,30,99,.65); }
  .tp{   box-shadow: inset 0 0 0 3px rgba(33,150,243,.65); }
  .overlay{
    position:absolute; inset:0; pointer-events:none;
  }
  .piece{
    position:absolute;
    width:var(--piece); height:var(--piece);
    border-radius:999px;
    color:#fff; font-weight:700; letter-spacing:.5px;
    display:flex; align-items:center; justify-content:center;
    box-shadow:0 4px 10px rgba(0,0,0,.12);
    pointer-events:auto;
    transition: left var(--speed) ease, top var(--speed) ease, transform 120ms ease;
  }
  .pieceA{ background:#e74c3c; }
  .pieceB{ background:#2ecc71; }
  .pieceC{ background:#3498db; }
  .pieceD{ background:#f1c40f; color:#333; }
  .sel{ outline:3px solid #ffb300; box-shadow: 0 0 12px rgba(255, 179, 0, 0.8); transform: scale(1.05); z-index:10; }
  .hud{margin-top:12px; font-size:14px; display:flex; gap:12px; flex-wrap:wrap; align-items:center}
  .badge{display:inline-flex; align-items:center; gap:6px; padding:4px 8px; border-radius:999px; background:#fff; border:1px solid #ddd;}
  .legend{display:flex; gap:8px; flex-wrap:wrap; margin-top:8px;}
  .legend > span{display:inline-flex; align-items:center; gap:6px; padding:3px 8px; border:1px solid #ddd; border-radius:8px; background:#fff;}
  .tag{width:14px;height:14px;border-radius:3px;}
  .controls{display:flex; gap:8px; align-items:center; flex-wrap:wrap}
  .btn{padding:6px 10px; border:1px solid #ccc; background:#fff; border-radius:8px; cursor:pointer}
  .btn:active{transform:translateY(1px)}
  .rule-highlight { background-color: #fffacd; padding: 15px; border-radius: 8px; margin-top: 15px; border-left: 4px solid #ffb300; }
  .debug-info { margin-top: 15px; padding: 10px; background: #f5f5f5; border-radius: 5px; font-size: 14px; }
/* 开关样式 */ .switch { position: relative; display: inline-block; width: 60px; height: 28px; vertical-align: middle; } .switch input {  opacity: 0; width: 0; height: 0; } .slider { position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0; background-color: #ccc; transition: .4s; border-radius: 28px; } .slider:before { position: absolute; content: ""; height: 22px; width: 22px; left: 3px; bottom: 3px; background-color: white; transition: .4s; border-radius: 50%; } input:checked + .slider { background-color: #2196F3; } input:checked + .slider:before { transform: translateX(32px); } .switch-label { margin-left: 8px; font-size: 14px; vertical-align: middle; } </style>

Than Yu Heng陈煜恒, [8/26/2025 8:54 PM]
</head>
<body>
<div class="wrap">
  <h1>角力棋 8×8（模式切换）</h1>
  <div class="panel">
    <label class="switch">
      <input type="checkbox" id="modeToggle">
      <span class="slider"></span>
    </label>
    <span class="switch-label" id="modeLabel">四人模式</span>
    <button id="resetBtn" class="btn">重置</button>
    <label>
      <input type="checkbox" id="hintsCbx" checked>
      显示提示
    </label>
  </div>
  <div class="stage" id="stage">
    <div id="board" class="board"></div>
    <div id="overlay" class="overlay"></div>
  </div>
  <div class="hud">
    <div class="badge">当前：<strong id="turn">A</strong></div>
    <div class="badge">名次：<strong id="ranks">—</strong></div>
    <div class="legend">
      <span><span class="tag" style="background:#c8e6c9"></span> 合法落点</span>
      <span><span class="tag" style="background:#f8bbd0"></span> 可强制换位</span>
      <span><span class="tag" style="background:#bbdefb"></span> 可传送落点</span>
      <span><span class="tag" style="box-shadow: inset 0 0 0 2px rgba(0,0,0,.15); background:#fff"></span> 目标九宫格</span>
    </div>
  </div>
  <div class="rule-highlight">
    <h3>游戏说明：</h3>
    <p>1. 使用开关切换二人/四人模式 - 二人模式为左上角 vs 右下角</p>
    <p>2. 强制换位机制 - 正确检查归位移动和周围棋子条件</p>
    <p>3. 传送机制 - 沿主对角线对称传送 [r,c]→[c,r]</p>
  </div>
  <div class="debug-info" id="debugInfo">
    选择棋子查看调试信息...
  </div>
</div>
<script>
/* ================== 常量与工具 ================== */
const N = 8;
const dirs4 = [[1,0],[-1,0],[0,1],[0,-1]];
const players4 = ["A","B","C","D"];
const players2 = ["A","B"];
const colorClass = {A:"pieceA",B:"pieceB",C:"pieceC",D:"pieceD"};
const inb = (r,c)=> r>=0 && r<N && c>=0 && c<N;
const key = (r,c)=> r+"_"+c;
const unkey = s => s.split("_").map(Number);
const diagSym = ([r,c])=>[N-1-r, N-1-c];
const cssCell = parseFloat(getComputedStyle(document.documentElement).getPropertyValue('--cell'));
const cssGap  = parseFloat(getComputedStyle(document.documentElement).getPropertyValue('--gap'));
/* 网格像素定位 */
function cellToXY(r,c){
  const x = c*(cssCell+cssGap);
  const y = r*(cssCell+cssGap);
  return {x,y};
}
/* 角落九宫格（3×3） */
function cornerCells(corner){
  const cs=[];
  for(let r=0;r<3;r++)for(let c=0;c<3;c++){
    if(corner==="tl") cs.push([r,c]);
    if(corner==="tr") cs.push([r, N-3+c]);
    if(corner==="bl") cs.push([N-3+r, c]);
    if(corner==="br") cs.push([N-3+r, N-3+c]);
  }
  return cs;
}
/* 角点坐标 */
const cornerPoint = { tl:[0,0], tr:[0,7], bl:[7,0], br:[7,7] };
/* ================== DOM 绑定 ================== */
const boardEl  = document.getElementById('board');
const overlayEl= document.getElementById('overlay');
const modeToggle = document.getElementById('modeToggle');
const modeLabel = document.getElementById('modeLabel');
const hintsCbx = document.getElementById('hintsCbx');
const resetBtn = document.getElementById('resetBtn');
const turnEl   = document.getElementById('turn');
const ranksEl  = document.getElementById('ranks');
const debugInfo= document.getElementById('debugInfo');
/* ================== 状态 ================== */
let state;
let nextPid = 1;
/* ================== 初始化 ================== */
function init(){
  boardEl.innerHTML = "";
  for(let r=0;r<N;r++){
    for(let c=0;c<N;c++){
      const cell = document.createElement('div');
      cell.className = "cell "+(((r+c)%2===0)?"light":"dark");
      cell.dataset.row=r; cell.dataset.col=c;
      cell.addEventListener('click', ()=>onCellClick(r,c));
      boardEl.appendChild(cell);
    }
  }
  const isTwoPlayerMode = modeToggle.checked;
  modeLabel.textContent = isTwoPlayerMode ? "二人模式" : "四人模式";
  const mode = isTwoPlayerMode ? 2 : 4;
  const order = isTwoPlayerMode ? players2.slice() : players4.slice();
  const places4 = ["tl","tr","br","bl"];
  const places2 = ["tl","br"]; // 二人模式：左上角 vs 右下角
  const placeSeq = isTwoPlayerMode ? places2 : places4;

Than Yu Heng陈煜恒, [8/26/2025 8:54 PM]
const placeFor = {};
  order.forEach((p,i)=> placeFor[p]=placeSeq[i]);
  const start = {};
  const goal  = {};
  for(const p of order){
    const startCorner = placeFor[p];
    let goalCorner;
    if (isTwoPlayerMode) {
        goalCorner = (startCorner === "tl") ? "br" : "tl";
    } else {
        let pair = (p==="A"||p==="C") ? ["A","C"] : ["B","D"];
        const other = pair.find(x=>x!==p);
        goalCorner = placeFor[other];
    }
    start[p] = cornerCells(startCorner).map(([r,c])=>({r,c,id:nextPid++}));
    goal[p]  = new Set(cornerCells(goalCorner).map(([r,c])=>key(r,c)));
  }
  const goalCornerPoint = {};
  for(const p of order){
    const any = [...goal[p]][0];
    const [gr,gc] = unkey(any);
    let nm="tl";
    if(gr<3 && gc<3) nm="tl";
    else if(gr<3 && gc>=5) nm="tr";
    else if(gr>=5 && gc<3) nm="bl";
    else nm="br";
    goalCornerPoint[p] = cornerPoint[nm];
  }
  const tpZone = {};
  if (isTwoPlayerMode) {
      // 二人模式下，传送区域为右上角和左下角
      const corners = ["tr", "bl"];
      const tpCells = new Set();
      corners.forEach(corner => {
          cornerCells(corner).forEach(([r, c]) => {
              tpCells.add(key(r, c));
          });
      });
      tpZone["A"] = tpCells;
      tpZone["B"] = tpCells;
  } else {
      // 四人模式下，传送区域为对手的九宫格
      for(const p of order){
          const group = (p==="A"||p==="C") ? ["B","D"] : ["A","C"];
          const u = new Set();
          for(const q of group){ for(const g of goal[q]) u.add(g); }
          tpZone[p] = u;
      }
  }
  state = {
    mode, order,
    pieces: start,
    goal, goalCornerPoint, tpZone,
    turnIdx:0,
    selected: null,
    ranks: [],
    finished: new Set(),
    showHints: hintsCbx.checked,
    animating: false,
    moveHistory: []
  };
  drawGoals();
  mountPieces();
  paintHUD();
  debugInfo.textContent = "选择棋子查看调试信息...";
}
/* ================== 渲染 ================== */
function drawGoals(){
  const cells = boardEl.querySelectorAll('.cell');
  cells.forEach(c=>{
    const old = c.querySelector('.goal'); if(old) old.remove();
    c.classList.remove('hint','swap','tp');
  });
  for(const p of state.order){
    for(const g of state.goal[p]){
      const [r,c] = unkey(g);
      const cell = boardEl.children[r*8+c];
      const box = document.createElement('div');
      box.className="goal";
      cell.appendChild(box);
    }
  }
}
function paintHints(sel, tips){
  const cells = boardEl.querySelectorAll('.cell');
  cells.forEach(c=>c.classList.remove('hint','swap','tp'));
  if(!sel || !state.showHints) return;
  for(const t of tips.emptyLandings){
    const cell = boardEl.children[t.r*8+t.c]; cell.classList.add('hint');
  }
  for(const t of tips.swapTargets){
    const cell = boardEl.children[t.r*8+t.c]; cell.classList.add('swap');
  }
  for(const t of tips.tpLandings){
    const cell = boardEl.children[t.r*8+t.c]; cell.classList.add('tp');
  }
}
function mountPieces(){
  overlayEl.innerHTML = "";
  for(const p of state.order){
    for(const pc of state.pieces[p]){
      const el = document.createElement('div');
      el.className = piece ${colorClass[p]};
      el.dataset.pid = pc.id;
      el.dataset.player = p;
      el.textContent = p;
      const {x,y} = cellToXY(pc.r, pc.c);
      el.style.left = x+"px"; el.style.top = y+"px";
      el.addEventListener('click',(e)=>{ e.stopPropagation(); onCellClick(pc.r, pc.c); });
      overlayEl.appendChild(el);
    }
  }
}
function pieceElById(id){ return overlayEl.querySelector(.piece[data-pid="${id}"]); }
function pieceAt(r,c){
  for(const p of state.order){
    for(const pc of state.pieces[p]){
      if(pc.r===r && pc.c===c) return {player:p, piece:pc};
    }
  }
  return null;
}
async function animateToPiece(pc, r, c){
  return new Promise(res=>{
    const el = pieceElById(pc.id);
    if(!el){ res(); return; }

Than Yu Heng陈煜恒, [8/26/2025 8:54 PM]
const {x,y} = cellToXY(r,c);
    const onEnd = ()=>{ el.removeEventListener('transitionend', onEnd); res(); };
    el.addEventListener('transitionend', onEnd);
    setTimeout(onEnd, 800);
    el.style.left = x+"px";
    el.style.top  = y+"px";
  });
}
function paintSelection(){
  overlayEl.querySelectorAll('.piece').forEach(el=>el.classList.remove('sel'));
  if(!state.selected) return;
  const here = pieceAt(state.selected.r, state.selected.c);
  if(here){
    const el = pieceElById(here.piece.id);
    if(el) el.classList.add('sel');
  }
}
/* ================== 游戏逻辑 ================== */
function currentPlayer(){ return state.order[state.turnIdx]; }
function setSelected(r,c){ 
  state.selected={r,c}; 
  paintSelection(); 
  const tips=computeAllDestinations(state.selected); 
  paintHints(state.selected, tips); 
  updateDebugInfo(r, c);
}
function updateDebugInfo(r, c) {
  const p = currentPlayer();
  const piece = state.pieces[p].find(pc=>pc.r===r && pc.c===c);
  if (!piece) return;
  let debugText = 选中棋子: ${p} [${r},${c}]<br>;
  debugText += 目标区域: ${[...state.goal[p]].join(', ')}<br>;
  debugText += 传送区域: ${[...state.tpZone[p]].join(', ')}<br>;
  debugText += "<br>四周棋子: ";
  for(const [dr,dc] of dirs4) {
    const nr = r+dr, nc = c+dc;
    if(inb(nr,nc)) {
      const occ = occupiedBy(nr,nc);
      debugText += [${nr},${nc}]: ${occ || '空'} ;
    }
  }
  debugText += "<br><br>强制换位检查: ";
  for(const [dr,dc] of dirs4) {
    const tr = r+dr, tc = c+dc;
    if(!inb(tr,tc)) continue;
    const occ = occupiedBy(tr,tc);
    if(!occ || occ===p) continue;
    if(isNextStepHome(p, {r, c}, {r:tr, c:tc})) {
      let cnt=0;
      for(const [er,ec] of dirs4){
        const ar=tr+er, ac=tc+ec;
        if(!inb(ar,ac)) continue;
        const kk=key(ar,ac);
        if(ar===pos.r && ac===pos.c) continue;
        if(mySet.has(kk)) cnt++;
      }
      debugText += [${tr},${tc}]: ${occ} (${cnt}个友方) ;
    }
  }
  debugInfo.innerHTML = debugText;
}
function occupiedBy(r,c){
  for(const p of state.order){
    for(const pc of state.pieces[p]){
      if(pc.r===r && pc.c===c) return p;
    }
  }
  return null;
}
function inOwnGoalArea(p, r, c){ return state.goal[p].has(key(r,c)); }
function isCornerStuck(p, r,c){
  const [gr,gc] = state.goalCornerPoint[p];
  return (r===gr && c===gc);
}
function allInGoal(p){
  for(const pc of state.pieces[p]){
    if(!state.goal[p].has(key(pc.r,pc.c))) return false;
  }
  return true;
}
function nextTurn(){
  let idx = state.turnIdx;
  do{
    idx = (idx + 1) % state.order.length;
  }while(state.finished.has(state.order[idx]));
  state.turnIdx = idx;
  paintHUD();
}
function paintHUD(){
  turnEl.textContent = state.order[state.turnIdx];
  ranksEl.textContent = state.ranks.length? state.ranks.join(" > ") : "—";
}
function goalMinDist(p, r,c){
  let best=Infinity;
  for(const g of state.goal[p]){
    const [gr,gc] = unkey(g);
    const d = Math.abs(gr-r)+Math.abs(gc-c);
    if(d<best) best=d;
  }
  return best;
}
function distanceToCorner(p, r, c) {
    const [cornerR, cornerC] = state.goalCornerPoint[p];
    return Math.abs(r - cornerR) + Math.abs(c - cornerC);
}
function isTowardsGoal(p, from, to) {
    const d0 = goalMinDist(p, from.r, from.c);
    const d1 = goalMinDist(p, to.r, to.c);
    if (state.goal[p].has(key(from.r, from.c))) {
        const distFrom = distanceToCorner(p, from.r, from.c);
        const distTo = distanceToCorner(p, to.r, to.c);
        return distTo < distFrom;
    }
    return d1 < d0 || state.goal[p].has(key(to.r, to.c));
}
function isNextStepHome(p, from, to) {
    if (state.goal[p].has(key(from.r, from.c))) {
        const distFrom = distanceToCorner(p, from.r, from.c);
        const distTo = distanceToCorner(p, to.r, to.c);
        return distTo < distFrom;
    }
    return state.goal[p].has(key(to.r, to.c));

Than Yu Heng陈煜恒, [8/26/2025 8:54 PM]
}
function stepMoves(p, from){
  const out=[];
  for(const [dr,dc] of dirs4){
    const r=from.r+dr, c=from.c+dc;
    if(!inb(r,c) || occupiedBy(r,c)) continue;
    if (state.goal[p].has(key(from.r, from.c)) && !state.goal[p].has(key(r, c))) continue;
    if (state.goal[p].has(key(from.r, from.c)) && !isTowardsGoal(p, from, {r, c})) continue;
    if (isCornerStuck(p, from.r, from.c)) continue;
    out.push({r,c});
  }
  return out;
}
function jumpStepsFrom(p, pos, dr, dc) {
    const out = [];
    if (isCornerStuck(p, pos.r, pos.c)) return out;
    for (let gap = 0; gap <= 2; gap++) {
        const bridgeR = pos.r + dr * (gap + 1);
        const bridgeC = pos.c + dc * (gap + 1);
        const landR = bridgeR + dr * (gap + 1);
        const landC = bridgeC + dc * (gap + 1);
        if (!inb(bridgeR, bridgeC) || !inb(landR, landC)) continue;
        if (!occupiedBy(bridgeR, bridgeC)) continue;
        if (occupiedBy(landR, landC)) continue;
        if (state.goal[p].has(key(pos.r, pos.c)) && !state.goal[p].has(key(landR, landC))) continue;
        if (state.goal[p].has(key(pos.r, pos.c)) && !isTowardsGoal(p, pos, {r: landR, c: landC})) continue;
        let pathClear = true;
        for (let i = 1; i <= gap; i++) {
            const checkR = pos.r + dr * i;
            const checkC = pos.c + dc * i;
            if (occupiedBy(checkR, checkC)) {
                pathClear = false;
                break;
            }
        }
        for (let i = 1; i <= gap; i++) {
            const checkR = bridgeR + dr * i;
            const checkC = bridgeC + dc * i;
            if (occupiedBy(checkR, checkC)) {
                pathClear = false;
                break;
            }
        }
        if (pathClear) {
            out.push({r: landR, c: landC});
        }
    }
    for (let gap = 0; gap <= 2; gap++) {
        const bridge1R = pos.r + dr * (gap + 1);
        const bridge1C = pos.c + dc * (gap + 1);
        const bridge2R = bridge1R + dr;
        const bridge2C = bridge1C + dc;
        const landR = bridge2R + dr * (gap + 1);
        const landC = bridge2C + dc * (gap + 1);
        if (!inb(bridge1R, bridge1C)  !inb(bridge2R, bridge2C)  !inb(landR, landC)) continue;
        const bridge1 = occupiedBy(bridge1R, bridge1C);
        const bridge2 = occupiedBy(bridge2R, bridge2C);
        if (!bridge1  !bridge2  bridge1 !== bridge2) continue;
        if (occupiedBy(landR, landC)) continue;
        if (state.goal[p].has(key(pos.r, pos.c)) && !state.goal[p].has(key(landR, landC))) continue;
        if (state.goal[p].has(key(pos.r, pos.c)) && !isTowardsGoal(p, pos, {r: landR, c: landC})) continue;
        let pathClear = true;
        for (let i = 1; i <= gap; i++) {
            const checkR = pos.r + dr * i;
            const checkC = pos.c + dc * i;
            if (occupiedBy(checkR, checkC)) {
                pathClear = false;
                break;
            }
        }
        for (let i = 1; i <= gap; i++) {
            const checkR = bridge2R + dr * i;
            const checkC = bridge2C + dc * i;
            if (occupiedBy(checkR, checkC)) {
                pathClear = false;
                break;
            }
        }
        if (pathClear) {
            out.push({r: landR, c: landC});
        }
    }
    return out;
}
function isTeleportPoint(p, r,c){ return state.tpZone[p].has(key(r,c)); }
function generateJumpGraph(p, start, maxDepth = 20){
  const seen = new Set();
  const tpUsed = new Set();
  const emptyLandings = new Set();
  const tpLandings = new Set();
  const pathsTo = new Map();
  let currentDepth = 0;
  function dfs(pos, path){
    if (currentDepth++ > maxDepth) return;
    const k = key(pos.r,pos.c);
    if(seen.has(k)) return;
    seen.add(k);
    if(!(pos.r===start.r && pos.c===start.c)){
      emptyLandings.add(k);

Than Yu Heng陈煜恒, [8/26/2025 8:54 PM]
pathsTo.set(k, path.slice());
    }
    for(const [dr,dc] of dirs4){
      const opts = jumpStepsFrom(p, pos, dr,dc);
      for(const to of opts){
        if (state.goal[p].has(key(pos.r, pos.c)) && !state.goal[p].has(key(to.r, to.c))) continue;
        if (state.goal[p].has(key(pos.r, pos.c)) && !isTowardsGoal(p, pos, to)) continue;
        const k2 = key(to.r,to.c);
        if(!seen.has(k2)){
          path.push({type:"jump", from:{...pos}, to});
          dfs(to, path);
          path.pop();
        }
      }
    }
    // Corrected teleport logic
    function mirrorTeleport([r,c]) {
        return [c, r];
    }
    if(isTeleportPoint(p, pos.r,pos.c)){
        const [tr,tc] = mirrorTeleport([pos.r,pos.c]);
        if(inb(tr,tc) && !occupiedBy(tr,tc)){
            const edgeKey = tp:${key(pos.r,pos.c)}->${key(tr,tc)};
            if(!tpUsed.has(edgeKey)){
                tpUsed.add(edgeKey);
                const to = {r:tr,c:tc};
                tpLandings.add(key(to.r,to.c));
                path.push({type:"tp", from:{...pos}, to});
                dfs(to, path);
                path.pop();
            }
        }
    }
  }
  dfs(start, []);
  return {
    emptyLandings:[...emptyLandings].map(s=>{const [r,c]=unkey(s); return {r,c};}),
    tpLandings:[...tpLandings].map(s=>{const [r,c]=unkey(s); return {r,c};}),
    pathTo: pathsTo
  };
}
function computeForceSwapTargets(p, start, reachableEmpty, pathTo){
  const out = new Set();
  const mySet = new Set(state.pieces[p].map(pc=>key(pc.r,pc.c)));
  const validFromPositions = new Set();
  validFromPositions.add(key(start.r, start.c));
  const basicMoves = stepMoves(p, start);
  for(const move of basicMoves){
    validFromPositions.add(key(move.r, move.c));
  }
  for(const posKey of validFromPositions){
    const [r,c] = unkey(posKey);
    const pos = {r,c};
    for(const [dr,dc] of dirs4){
      const tr = pos.r+dr, tc = pos.c+dc;
      if(!inb(tr,tc)) continue;
      const occ = occupiedBy(tr,tc);
      if(!occ || occ===p) continue;
      if(!isNextStepHome(p, pos, {r:tr,c:tc})) continue;
      let cnt=0;
      for(const [er,ec] of dirs4){
        const ar=tr+er, ac=tc+ec;
        if(!inb(ar,ac)) continue;
        const kk=key(ar,ac);
        if(ar===pos.r && ac===pos.c) continue;
        if(mySet.has(kk)) cnt++;
      }
      if(cnt>=2){
        out.add(key(tr,tc));
        const destK = key(tr,tc);
        if(!pathTo.has(destK)){
          const isStart = (pos.r === start.r && pos.c === start.c);
          const path = isStart ? [] : [{type:"step", from:start, to:pos}];
          pathTo.set(destK, path.concat([{type:"swap", from:pos, to:{r:tr,c:tc}}]));
        }
      }
    }
  }
  return [...out].map(s=>{const [r,c]=unkey(s); return {r,c};});
}
function computeAllDestinations(sel){
  const p = currentPlayer();
  const piece = state.pieces[p].find(pc=>pc.r===sel.r && pc.c===sel.c);
  if(!piece) return {emptyLandings:[], tpLandings:[], swapTargets:[], pathTo:new Map()};
  if(isCornerStuck(p, piece.r, piece.c)) return {emptyLandings:[], tpLandings:[], swapTargets:[], pathTo:new Map()};
  const steps = stepMoves(p, sel);
  const {emptyLandings, tpLandings, pathTo} = generateJumpGraph(p, sel);
  const mergedEmpty = new Map();
  for(const t of [...steps, ...emptyLandings]){
    mergedEmpty.set(key(t.r,t.c), t);
    if(!pathTo.has(key(t.r,t.c)) && (t.r !== sel.r || t.c !== sel.c)){
      pathTo.set(key(t.r,t.c), [{type:"step", from:sel, to:t}]);
    }
  }
  const swapTargets = computeForceSwapTargets(p, sel, [...mergedEmpty.values()], pathTo);
  return { emptyLandings:[...mergedEmpty.values()], tpLandings, swapTargets, pathTo };
}
/* ================== 动画与执行 ================== */
async function animatePathAndApply(p, start, path){
  if(path.length===0) return start;
  const me = state.pieces[p].find(pc=>pc.r===start.r && pc.c===start.c);

Than Yu Heng陈煜恒, [8/26/2025 8:58 PM]
let cur = {r:start.r, c:start.c};
  for(const step of path){
    await animateToPiece(me, step.to.r, step.to.c);
    me.r = step.to.r;
    me.c = step.to.c;
    cur = {r:me.r, c:me.c};
  }
  return cur;
}
async function tryMoveToEmpty(dest, pathTo){
  if(state.animating) return;
  state.animating = true;
  const p = currentPlayer();
  const start = state.selected;
  if(!start){ state.animating=false; return; }
  const k = key(dest.r,dest.c);
  const path = (pathTo.get(k) || []).filter(x=>x.type!=="swap");
  await animatePathAndApply(p, start, path);
  afterActionCommon(p);
  state.animating = false;
}
async function tryForceSwap(tgt, pathTo){
  if(state.animating) return;
  state.animating = true;
  const p = currentPlayer();
  const start = state.selected;
  if(!start){ state.animating=false; return; }
  const opp = occupiedBy(tgt.r,tgt.c);
  if(!opp || opp===p){ state.animating=false; return; }
  const path = pathTo.get(key(tgt.r,tgt.c)) || [];
  const preMoves = path.filter(x=>x.type!=="swap");
  const at = await animatePathAndApply(p, start, preMoves);
  const myPc = state.pieces[p].find(pc=>pc.r===at.r && pc.c===at.c);
  const oppPc = state.pieces[opp].find(pc=>pc.r===tgt.r && pc.c===tgt.c);
  const hadMotionBefore = preMoves.length>0;
  myPc.r = tgt.r; myPc.c = tgt.c;
  const sendTo = hadMotionBefore ? {r:start.r, c:start.c} : {r:at.r, c:at.c};
  oppPc.r = sendTo.r; oppPc.c = sendTo.c;
  await Promise.all([
    animateToPiece(myPc, myPc.r, myPc.c),
    animateToPiece(oppPc, oppPc.r, oppPc.c)
  ]);
  afterActionCommon(p);
  state.animating = false;
}
function afterActionCommon(p){
  if(allInGoal(p) && !state.finished.has(p)){
    state.finished.add(p);
    state.ranks.push(p);
  }
  nextTurn();
  state.selected = null;
  paintSelection();
  paintHints(null, {emptyLandings:[], swapTargets:[], tpLandings:[]});
  debugInfo.textContent = "选择棋子查看调试信息...";
}
/* ================== 事件处理 ================== */
function onCellClick(r,c){
  if(state.animating) return;
  const p = currentPlayer();
  const occ = occupiedBy(r,c);
  if(occ===p){
    if(isCornerStuck(p, r, c)) return;
    setSelected(r,c);
    return;
  }
  if(!state.selected) return;
  const tips = computeAllDestinations(state.selected);
  const isEmptyDest = tips.emptyLandings.some(t=>t.r===r && t.c===c);
  const isSwapDest  = tips.swapTargets.some(t=>t.r===r && t.c===c);
  if(isEmptyDest){
    tryMoveToEmpty({r,c}, tips.pathTo);
  }else if(isSwapDest){
    tryForceSwap({r,c}, tips.pathTo);
  }
}
/* ================== 交互控件 ================== */
modeToggle.addEventListener('change', init);
hintsCbx.addEventListener('change', function() {
    if (state) {
        state.showHints = hintsCbx.checked;
        if (state.selected) {
            const tips = computeAllDestinations(state.selected);
            paintHints(state.selected, tips);
        }
    }
});
resetBtn.addEventListener('click', init);
/* ================== 启动 ================== */
document.addEventListener('DOMContentLoaded', function() {
    init();
});
</script>
</body>
</html>
