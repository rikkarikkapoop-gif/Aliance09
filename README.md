<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Project Alliance | AI Learning Platform</title>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #eef2f7;
}

header {
  background: #1e3a8a;
  color: white;
  padding: 18px;
  text-align: center;
}

.container {
  padding: 16px;
  max-width: 900px;
  margin: auto;
}

.card {
  background: white;
  border-radius: 12px;
  padding: 16px;
  margin-bottom: 16px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.08);
  animation: fade 0.3s ease;
}

button {
  width: 100%;
  padding: 10px;
  margin-top: 8px;
  border-radius: 8px;
  border: none;
  background: #2563eb;
  color: white;
  cursor: pointer;
}

button:hover {
  background: #1d4ed8;
}

.nav button {
  background: #f3f4f6;
  color: #111;
  border: 1px solid #ddd;
}

.progress-bar {
  height: 10px;
  background: #ddd;
  border-radius: 6px;
  overflow: hidden;
}

.progress {
  height: 100%;
  background: #22c55e;
  width: 0%;
  transition: 0.3s;
}

input {
  width: 100%;
  padding: 10px;
  margin-top: 8px;
  border-radius: 8px;
  border: 1px solid #ddd;
}

.muted {
  color: gray;
  font-size: 14px;
}

.hidden {
  display: none;
}

@keyframes fade {
  from { opacity: 0; transform: translateY(5px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>
</head>

<body>

<header>
<h1>Project Alliance</h1>
<p>AI-Nurturing Content Education</p>
</header>

<div class="container">

<!-- LOGIN -->
<div id="login" class="card">
<h2>Student Login</h2>
<input id="name" placeholder="Enter your name"/>
<input id="section" placeholder="Enter your section"/>
<button onclick="login()">Login</button>
</div>

<!-- APP -->
<div id="app" class="hidden">

<div class="card">
<h2>Dashboard</h2>
<p id="studentInfo" class="muted"></p>
<p>Total Score: <b id="score">0</b></p>
<p>Overall Progress:</p>
<div class="progress-bar">
  <div class="progress" id="overallProgress"></div>
</div>
<p>Remark: <b id="remark">—</b></p>
</div>

<div class="card nav">
<h2>Subjects</h2>
<button onclick="loadSubject('eapp')">EAPP</button>
<button onclick="loadSubject('research')">Practical Research</button>
<button onclick="loadSubject('rw')">Reading & Writing</button>
<button onclick="showSummary()">View Analytics</button>
<button onclick="showLeaderboard()">Leaderboard</button>
<button onclick="generateQR()">Generate QR Code</button>
<button onclick="downloadOffline()">Download Offline Copy</button>
</div>

<div id="content"></div>
<div id="analytics"></div>
<div id="leaderboard"></div>
<div id="qr"></div>

</div>
</div>

<script>
let studentName = "";
let studentSection = "";
let scores = { eapp:0, research:0, rw:0 };
let totals = { eapp:2, research:2, rw:2 };
let answered = {};

const weekNumber = Math.ceil((new Date() - new Date(new Date().getFullYear(),0,1)) / 604800000);

const weeklyBanks = [
{
  eapp:[
    { q:"Academic writing tone is:", options:["Formal","Casual"], correct:"Formal" },
    { q:"EAPP improves:", options:["Texting","Professional writing"], correct:"Professional writing" }
  ],
  research:[
    { q:"Qualitative research uses:", options:["Numbers","Words"], correct:"Words" },
    { q:"Ethnography studies:", options:["Culture","Weather"], correct:"Culture" }
  ],
  rw:[
    { q:"Hypertext reading is:", options:["Linear","Non-linear"], correct:"Non-linear" },
    { q:"Critical reading means:", options:["Analyze","Skim only"], correct:"Analyze" }
  ]
},
{
  eapp:[
    { q:"Formal language avoids:", options:["Slang","Structure"], correct:"Slang" },
    { q:"Academic texts are:", options:["Objective","Random"], correct:"Objective" }
  ],
  research:[
    { q:"Case study focuses on:", options:["Single case","Nationwide data"], correct:"Single case" },
    { q:"Primary sources are:", options:["First-hand","Edited"], correct:"First-hand" }
  ],
  rw:[
    { q:"Linear reading is:", options:["Sequential","Jumping"], correct:"Sequential" },
    { q:"Parody is:", options:["Humorous imitation","Scientific report"], correct:"Humorous imitation" }
  ]
}
];

const questionBank = weeklyBanks[weekNumber % weeklyBanks.length];

function login(){
  studentName = name.value.trim();
  studentSection = section.value.trim();
  if(!studentName || !studentSection) return alert("Enter name & section");

  loadLocalScores();

  login.classList.add("hidden");
  app.classList.remove("hidden");
  studentInfo.innerText = `Student: ${studentName} | Section: ${studentSection}`;

  updateDashboard();
}

function updateDashboard(){
  const totalScore = scores.eapp + scores.research + scores.rw;
  const totalItems = totals.eapp + totals.research + totals.rw;
  const percent = Math.round((totalScore / totalItems) * 100);

  score.innerText = `${totalScore} / ${totalItems}`;
  overallProgress.style.width = percent + "%";
  remark.innerText = getRemark(percent);

  saveLocalScores();
}

function getRemark(percent){
  if(percent === 100) return "Excellent 🎉";
  if(percent >= 70) return "Very Good 👍";
  if(percent >= 40) return "Needs Improvement 📘";
  return "Keep Practicing ✏️";
}

function shuffle(arr){ return arr.sort(()=>Math.random()-0.5); }

function loadSubject(subject){
  leaderboard.innerHTML = "";
  analytics.innerHTML = "";

  const questions = shuffle([...questionBank[subject]]);

  content.innerHTML = `
    <div class='card'>
      <h2>${subject.toUpperCase()}</h2>
      ${questions.map((q,i)=>quiz(`${subject}_${i}`, subject, q)).join("")}
    </div>`;
}

function quiz(id, subject, item){
  return `
    <p>${item.q}</p>
    ${shuffle([...item.options]).map(opt =>
      `<button onclick="answer('${id}','${subject}','${opt}','${item.correct}')">${opt}</button>`
    ).join("")}
    <div id="${id}" class="muted"></div>`;
}

function answer(id, subject, selected, correct){
  if(answered[id]) return;
  answered[id] = true;

  const box = document.getElementById(id);
  if(selected === correct){
    scores[subject]++;
    box.innerHTML = "✅ Correct";
  } else {
    box.innerHTML = "❌ Incorrect";
  }

  updateDashboard();
}

function showSummary(){
  analytics.innerHTML = `
  <div class='card'>
    <h2>Subject Analytics</h2>
    <p>EAPP: <b>${scores.eapp}</b></p>
    <p>Research: <b>${scores.research}</b></p>
    <p>Reading & Writing: <b>${scores.rw}</b></p>
  </div>`;
}

function showLeaderboard(){
  const board = JSON.parse(localStorage.getItem("leaderboard") || "[]");

  leaderboard.innerHTML = `
  <div class='card'>
    <h2>Leaderboard 🏆</h2>
    ${board.length ? board.map((p,i)=>`<p>${i+1}. <b>${p.name}</b> — ${p.score}%</p>`).join("") : "No scores yet"}
  </div>`;
}

function saveLocalScores(){
  localStorage.setItem("projectAllianceScores", JSON.stringify({studentName, studentSection, scores}));
  saveToLeaderboard();
}

function loadLocalScores(){
  const saved = JSON.parse(localStorage.getItem("projectAllianceScores") || "null");
  if(saved && saved.studentName === studentName && saved.studentSection === studentSection){
    scores = saved.scores;
  }
}

function saveToLeaderboard(){
  const totalScore = scores.eapp + scores.research + scores.rw;
  const percent = Math.round((totalScore / 6) * 100);

  let board = JSON.parse(localStorage.getItem("leaderboard") || "[]");
  board.push({name: studentName, score: percent});
  board = board.sort((a,b)=>b.score-a.score).slice(0,5);

  localStorage.setItem("leaderboard", JSON.stringify(board));
}

function generateQR(){
  const url = window.location.href;
  qr.innerHTML = `
  <div class='card'>
    <h2>Scan QR Code</h2>
    <img src="https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=${url}">
  </div>`;
}

function downloadOffline(){
  const blob = new Blob([document.documentElement.outerHTML], {type:"text/html"});
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "ProjectAlliance_Offline.html";
  link.click();
}
</script>

</body>
</html>
