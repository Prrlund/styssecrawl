<!DOCTYPE html>
<html lang="sv">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gruppkalender</title>
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: system-ui, -apple-system, sans-serif; background: #f5f5f0; color: #1a1a1a; min-height: 100vh; }
header { background: white; border-bottom: 1px solid #e5e5e0; padding: 0.75rem 1rem; display: flex; align-items: center; gap: 10px; }
header h1 { font-size: 17px; font-weight: 600; flex: 1; }
.user-pill { background: #EEEDFE; color: #3C3489; font-size: 12px; font-weight: 500; padding: 5px 12px; border-radius: 20px; display: flex; align-items: center; gap: 6px; white-space: nowrap; }
.change-btn { font-size: 11px; color: #7F77DD; cursor: pointer; background: none; border: none; text-decoration: underline; }
.main { max-width: 900px; margin: 0 auto; padding: 1rem; }
.week-nav { display: flex; align-items: center; gap: 10px; margin-bottom: 0.75rem; background: white; border-radius: 10px; padding: 10px 14px; border: 1px solid #e5e5e0; }
.nav-btn { padding: 5px 12px; border: 1px solid #d5d5d0; border-radius: 7px; background: #f5f5f0; font-size: 13px; cursor: pointer; color: #444; }
.nav-btn:hover { background: #eee; }
.week-label { font-size: 14px; font-weight: 500; flex: 1; }
.cal-wrap { background: white; border-radius: 10px; border: 1px solid #e5e5e0; overflow: hidden; }
.cal-scroll { overflow-x: auto; }
table { border-collapse: collapse; width: 100%; min-width: 500px; }
thead th { padding: 8px 3px; font-size: 11px; font-weight: 500; color: #888; text-align: center; border-bottom: 1px solid #e5e5e0; background: #fafaf8; }
thead th:first-child { width: 48px; }
tbody td { border: 0.5px solid #eee; height: 30px; position: relative; cursor: pointer; user-select: none; }
tbody td:first-child { font-size: 10px; color: #bbb; text-align: right; padding-right: 6px; cursor: default; background: #fafaf8; white-space: nowrap; }
.dot-row { display: flex; flex-wrap: wrap; gap: 2px; padding: 3px; justify-content: center; align-items: center; height: 100%; }
.dot { width: 7px; height: 7px; border-radius: 50%; flex-shrink: 0; }
.legend { display: flex; gap: 14px; flex-wrap: wrap; margin: 0.75rem 0; }
.legend-item { display: flex; align-items: center; gap: 5px; font-size: 11px; color: #666; }
.ldot { width: 9px; height: 9px; border-radius: 50%; }
.summary { background: white; border-radius: 10px; border: 1px solid #e5e5e0; padding: 1rem; margin-top: 0.75rem; }
.summary h2 { font-size: 14px; font-weight: 600; margin-bottom: 10px; }
.best-row { display: flex; justify-content: space-between; align-items: center; padding: 6px 0; border-bottom: 0.5px solid #eee; font-size: 13px; }
.best-row:last-child { border-bottom: none; }
.cpill { font-size: 11px; font-weight: 500; padding: 3px 9px; border-radius: 10px; }
.participants { background: white; border-radius: 10px; border: 1px solid #e5e5e0; padding: 1rem; margin-top: 0.75rem; }
.participants h2 { font-size: 14px; font-weight: 600; margin-bottom: 10px; }
.pchips { display: flex; flex-wrap: wrap; gap: 7px; }
.pchip { display: flex; align-items: center; gap: 5px; padding: 4px 11px; border-radius: 20px; font-size: 12px; font-weight: 500; }
.modal-bg { position: fixed; inset: 0; background: rgba(0,0,0,0.4); display: flex; align-items: center; justify-content: center; z-index: 100; padding: 1rem; }
.modal { background: white; border-radius: 14px; padding: 1.5rem; width: 100%; max-width: 320px; }
.modal h2 { font-size: 18px; font-weight: 600; margin-bottom: 6px; }
.modal p { font-size: 13px; color: #666; margin-bottom: 1.25rem; line-height: 1.5; }
.modal input { width: 100%; padding: 10px 12px; border: 1.5px solid #d5d5d0; border-radius: 9px; font-size: 15px; outline: none; font-family: inherit; }
.modal input:focus { border-color: #7F77DD; }
.modal button { width: 100%; margin-top: 10px; padding: 11px; border: none; border-radius: 9px; background: #534AB7; color: white; font-size: 14px; font-weight: 500; cursor: pointer; font-family: inherit; }
.modal button:hover { background: #3C3489; }
.modal button:disabled { background: #ccc; cursor: default; }
</style>
</head>
<body>

<header>
  <h1>Styssecrawl — när kan ni?</h1>
  <div id="userPill" style="display:none" class="user-pill">
    <span id="pillName"></span>
    <button class="change-btn" onclick="showModal(true)">byt</button>
  </div>
</header>

<div class="main">
  <div class="week-nav">
    <button class="nav-btn" onclick="changeWeek(-1)">&#8592;</button>
    <span class="week-label" id="weekLabel"></span>
    <button class="nav-btn" onclick="changeWeek(1)">&#8594;</button>
  </div>
  <div class="cal-wrap"><div class="cal-scroll"><table id="cal"><tr><td style="padding:2rem;color:#888;font-size:13px">Ansluter till databasen...</td></tr></table></div></div>
  <div class="legend">
    <div class="legend-item"><div class="ldot" style="background:#1D9E75"></div>Alla kan</div>
    <div class="legend-item"><div class="ldot" style="background:#63C4A0"></div>Flesta kan</div>
    <div class="legend-item"><div class="ldot" style="background:#FAC775"></div>Några kan</div>
    <div class="legend-item"><div class="ldot" style="background:#F09595"></div>Få kan</div>
  </div>
  <div class="summary"><h2>Bästa tider</h2><div id="best"><p style="font-size:13px;color:#888">Ingen har markerat ännu.</p></div></div>
  <div class="participants"><h2>Deltagare</h2><div class="pchips" id="plist"><span style="font-size:12px;color:#888">Ingen ännu.</span></div></div>
</div>

<div class="modal-bg" id="modalBg" style="display:none">
  <div class="modal">
    <h2>Vad heter du?</h2>
    <p>Ditt namn syns för alla i gruppen när du markerar tider du kan.</p>
    <input type="text" id="nameIn" placeholder="Ditt namn" maxlength="30" autocomplete="off" />
    <button id="nameBtn" onclick="saveName()" disabled>Fortsätt</button>
  </div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getDatabase, ref, onValue, set, remove } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-database.js";

const firebaseConfig = {
  apiKey: "AIzaSyBDwuwXiYBshJ9ae9XJYlzAJb-3nCxMAiI",
  authDomain: "styssecrawl.firebaseapp.com",
  databaseURL: "https://styssecrawl-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "styssecrawl",
  storageBucket: "styssecrawl.firebasestorage.app",
  messagingSenderId: "825829162427",
  appId: "1:825829162427:web:3cd20338fcf1e2d237189e"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

const COLORS = ['#534AB7','#0F6E56','#993C1D','#D4537E','#185FA5','#639922','#854F0B','#A32D2D','#5F5E5A','#BA7517'];
const BGS    = ['#EEEDFE','#E1F5EE','#FAECE7','#FBEAF0','#E6F1FB','#EAF3DE','#FAEEDA','#FCEBEB','#F1EFE8','#FAEEDA'];
const DAYS = ['Mån','Tis','Ons','Tor','Fre','Lör','Sön'];
const DAYSFULL = ['Måndag','Tisdag','Onsdag','Torsdag','Fredag','Lördag','Söndag'];
const TIMES = [];
for(let h=8;h<=21;h++){TIMES.push(h+':00');if(h<21)TIMES.push(h+':30');}

let myName = localStorage.getItem('gruppkalender_name') || '';
let weekOffset = 0, allData = {}, dragging = false, dragVal = null, pendingSlot = null;

const nameIn = document.getElementById('nameIn');
nameIn.oninput = () => document.getElementById('nameBtn').disabled = nameIn.value.trim().length < 2;
nameIn.onkeydown = e => { if(e.key==='Enter') saveName(); };

function showModal(force) {
  if(!myName || force) {
    document.getElementById('modalBg').style.display = 'flex';
    nameIn.value = myName || '';
    document.getElementById('nameBtn').disabled = (myName||'').length < 2;
    setTimeout(() => nameIn.focus(), 50);
  }
}
window.showModal = showModal;

function saveName() {
  const n = nameIn.value.trim();
  if(n.length < 2) return;
  myName = n;
  localStorage.setItem('gruppkalender_name', n);
  document.getElementById('modalBg').style.display = 'none';
  document.getElementById('pillName').textContent = n;
  document.getElementById('userPill').style.display = 'flex';
  if(pendingSlot) { toggleSlot(pendingSlot); pendingSlot = null; }
}
window.saveName = saveName;

function dateKey(d) { return d.toISOString().slice(0,10); }
function slotKey(dk, t) { return dk + '_' + t.replace(':','h'); }

function getWeekDates(off) {
  const now = new Date(), dow = now.getDay();
  const mon = new Date(now);
  mon.setDate(now.getDate() - (dow===0?6:dow-1) + off*7);
  mon.setHours(0,0,0,0);
  return Array.from({length:7}, (_,i) => { const d=new Date(mon); d.setDate(mon.getDate()+i); return d; });
}

function allPersons() {
  const s = new Set();
  Object.values(allData).forEach(slot => Object.keys(slot).forEach(n => s.add(n)));
  return [...s];
}
function pColor(name) { const p=allPersons(); const i=p.indexOf(name); return COLORS[i>=0?i%COLORS.length:0]; }
function pBg(name)    { const p=allPersons(); const i=p.indexOf(name); return BGS[i>=0?i%BGS.length:0]; }

function heatColor(count, total) {
  if(!count) return '#f5f5f0';
  const r = count / Math.max(total,1);
  if(r >= 1)     return '#1D9E75';
  if(r >= 0.625) return '#63C4A0';
  if(r >= 0.3)   return '#FAC775';
  return '#F09595';
}

function render() {
  const days = getWeekDates(weekOffset);
  const fmt = new Intl.DateTimeFormat('sv-SE',{month:'short',day:'numeric'});
  document.getElementById('weekLabel').textContent = fmt.format(days[0]) + ' – ' + fmt.format(days[6]) + ' ' + days[0].getFullYear();

  const persons = allPersons(), total = persons.length;
  let html = '<thead><tr><th></th>';
  days.forEach((d,i) => html += `<th>${DAYS[i]}<br><span style="font-weight:400;font-size:10px">${fmt.format(d)}</span></th>`);
  html += '</tr></thead><tbody>';

  TIMES.forEach(time => {
    html += `<tr><td>${time}</td>`;
    days.forEach(d => {
      const dk = dateKey(d);
      const key = slotKey(dk, time);
      const slot = allData[key] || {};
      const names = Object.keys(slot).filter(n => slot[n]);
      const count = names.length;
      const bg = heatColor(count, total);
      const myOn = myName && slot[myName];
      const outline = myOn ? `outline:2px solid ${pColor(myName)};outline-offset:-2px;` : '';
      let dots = names.map(n => `<div class="dot" style="background:${pColor(n)}" title="${n}"></div>`).join('');
      html += `<td data-key="${key}" style="background:${bg};${outline}" onmousedown="startDrag(event,'${key}')" onmouseover="doDrag('${key}')" onmouseup="endDrag()" ontouchstart="touchStart(event,'${key}')" ontouchmove="touchMove(event)" ontouchend="endDrag()">`;
      if(count) html += `<div class="dot-row">${dots}</div>`;
      html += '</td>';
    });
    html += '</tr>';
  });
  html += '</tbody>';
  document.getElementById('cal').innerHTML = html;
  renderSummary(days);
  renderPersons();
}

function renderSummary(days) {
  const fmt = new Intl.DateTimeFormat('sv-SE',{day:'numeric',month:'short'});
  const dayMap = {};
  days.forEach((d,i) => dayMap[dateKey(d)] = DAYSFULL[i]+' '+fmt.format(d));
  const total = allPersons().length;
  const scores = [];
  Object.entries(allData).forEach(([key, slot]) => {
    const count = Object.values(slot).filter(Boolean).length;
    if(!count) return;
    const [dk, tp] = key.split('_');
    const time = tp.replace('h',':');
    if(dayMap[dk]) scores.push({dk, time, count, key});
  });
  scores.sort((a,b) => b.count-a.count);
  const el = document.getElementById('best');
  if(!scores.length) { el.innerHTML = '<p style="font-size:13px;color:#888">Ingen har markerat ännu.</p>'; return; }
  el.innerHTML = scores.slice(0,6).map(s => {
    const col = heatColor(s.count, total||1);
    const names = Object.entries(allData[s.key]||{}).filter(([,v])=>v).map(([n])=>n).join(', ');
    return `<div class="best-row"><div><div style="font-weight:500">${dayMap[s.dk]} kl. ${s.time}</div><div style="font-size:11px;color:#888;margin-top:1px">${names}</div></div><span class="cpill" style="background:${col}30;color:${col}">${s.count}${total?'/'+total:''} kan</span></div>`;
  }).join('');
}

function renderPersons() {
  const persons = allPersons();
  const el = document.getElementById('plist');
  if(!persons.length) { el.innerHTML = '<span style="font-size:12px;color:#888">Ingen ännu.</span>'; return; }
  el.innerHTML = persons.map(n => `<div class="pchip" style="background:${pBg(n)};color:${pColor(n)}"><div class="dot" style="background:${pColor(n)}"></div>${n}</div>`).join('');
}

function toggleSlot(key, forceVal) {
  if(!myName) { pendingSlot = key; showModal(false); return; }
  if(!allData[key]) allData[key] = {};
  const newVal = forceVal !== undefined ? forceVal : !allData[key][myName];
  const dbRef = ref(db, 'slots/' + key + '/' + myName);
  if(newVal) set(dbRef, true);
  else remove(dbRef);
}

window.startDrag = function(e, key) { e.preventDefault(); dragging=true; dragVal=!(allData[key]||{})[myName]; toggleSlot(key, dragVal); };
window.doDrag = function(key) { if(dragging) toggleSlot(key, dragVal); };
window.endDrag = function() { dragging=false; dragVal=null; };
window.changeWeek = function(dir) { weekOffset+=dir; render(); };

window.touchStart = function(e, key) { e.preventDefault(); dragging=true; dragVal=!(allData[key]||{})[myName]; toggleSlot(key, dragVal); };
window.touchMove = function(e) {
  if(!dragging) return;
  const t = e.touches[0];
  const el = document.elementFromPoint(t.clientX, t.clientY);
  if(el && el.dataset.key) toggleSlot(el.dataset.key, dragVal);
};

document.addEventListener('mouseup', window.endDrag);
document.addEventListener('touchend', window.endDrag);

onValue(ref(db, 'slots'), snapshot => {
  allData = {};
  const data = snapshot.val() || {};
  Object.entries(data).forEach(([key, persons]) => { allData[key] = persons; });
  render();
});

showModal(false);
if(myName) {
  document.getElementById('pillName').textContent = myName;
  document.getElementById('userPill').style.display = 'flex';
}
</script>
</body>
</html>
