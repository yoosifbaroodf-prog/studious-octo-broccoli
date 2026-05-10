<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bubble Shooter Pro</title>

<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Tahoma;
}

body{
overflow:hidden;
background:linear-gradient(180deg,#0d1b2a,#1b263b,#415a77);
display:flex;
justify-content:center;
align-items:center;
height:100vh;
}

canvas{
background:linear-gradient(#102542,#1e3c72);
border:4px solid white;
border-radius:20px;
box-shadow:0 0 40px rgba(0,0,0,.5);
}

#ui{
position:absolute;
top:10px;
width:100%;
display:flex;
justify-content:center;
gap:30px;
color:white;
font-size:24px;
font-weight:bold;
text-shadow:2px 2px 5px black;
}

#menu{
position:absolute;
inset:0;
background:rgba(0,0,0,.7);
display:flex;
justify-content:center;
align-items:center;
flex-direction:column;
color:white;
z-index:10;
}

#menu h1{
font-size:60px;
margin-bottom:20px;
}

button{
padding:15px 40px;
border:none;
border-radius:15px;
font-size:24px;
cursor:pointer;
background:#ff9800;
color:white;
transition:.2s;
margin-top:15px;
font-weight:bold;
}

button:hover{
transform:scale(1.08);
background:#ffb300;
}
</style>
</head>

<body>

<div id="ui">
<div>⭐ النقاط: <span id="score">0</span></div>
<div>🏆 أعلى نتيجة: <span id="best">0</span></div>
</div>

<div id="menu">
<h1>🎯 Bubble Shooter Pro</h1>
<p>صوّب الكرات وفجّر الألوان المتشابهة</p>
<button onclick="startGame()">ابدأ اللعب</button>
</div>

<canvas id="game" width="520" height="760"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const scoreEl = document.getElementById("score");
const bestEl = document.getElementById("best");

const W = canvas.width;
const H = canvas.height;

const radius = 22;
const rowHeight = 38;

let grid = [];
let score = 0;
let gameStarted = false;

const colors = [
"#ff5252",
"#ffeb3b",
"#00e676",
"#40c4ff",
"#e040fb",
"#ff9100"
];

let shooter = {
x:W/2,
y:H-60,
angle:0,
bubble:createBubble()
};

function createBubble(){
return {
color:colors[Math.floor(Math.random()*colors.length)],
x:0,
y:0,
vx:0,
vy:0,
moving:false
};
}

function initGrid(){

grid = [];

for(let row=0; row<8; row++){

let cols = row % 2 === 0 ? 10 : 9;

for(let col=0; col<cols; col++){

let x = col * radius*2 + radius + (row%2 ? radius : 0);
let y = row * rowHeight + radius + 20;

grid.push({
x,
y,
color:colors[Math.floor(Math.random()*colors.length)],
row,
col
});

}
}
}

function drawBubble(x,y,color){
ctx.beginPath();
ctx.arc(x,y,radius,0,Math.PI*2);

let grad = ctx.createRadialGradient(
x-8,y-8,5,
x,y,30
);

grad.addColorStop(0,"white");
grad.addColorStop(1,color);

ctx.fillStyle = grad;
ctx.fill();

ctx.lineWidth=2;
ctx.strokeStyle="rgba(255,255,255,.5)";
ctx.stroke();
}

function drawGrid(){
grid.forEach(b=>{
drawBubble(b.x,b.y,b.color);
});
}

function drawShooter(){

ctx.save();

ctx.translate(shooter.x,shooter.y);

ctx.rotate(shooter.angle);

ctx.strokeStyle="white";
ctx.lineWidth=5;

ctx.beginPath();
ctx.moveTo(0,0);
ctx.lineTo(0,-70);
ctx.stroke();

ctx.restore();

drawBubble(
shooter.x,
shooter.y,
shooter.bubble.color
);
}

function drawBackground(){

for(let i=0;i<80;i++){

ctx.fillStyle="rgba(255,255,255,.15)";

ctx.beginPath();

ctx.arc(
(i*73)%W,
(i*97 + performance.now()*0.02)%H,
2,
0,
Math.PI*2
);

ctx.fill();
}
}

function updateBubble(){

let b = shooter.bubble;

if(!b.moving) return;

b.x += b.vx;
b.y += b.vy;

if(b.x <= radius || b.x >= W-radius){
b.vx *= -1;
}

if(b.y <= radius+10){
attachBubble(b);
return;
}

for(let g of grid){

let dx = b.x-g.x;
let dy = b.y-g.y;
let dist = Math.sqrt(dx*dx+dy*dy);

if(dist < radius*2-2){
attachBubble(b);
return;
}
}
}

function attachBubble(b){

b.moving = false;

grid.push({
x:b.x,
y:b.y,
color:b.color
});

checkMatches(b.x,b.y,b.color);

shooter.bubble = createBubble();
}

function checkMatches(x,y,color){

let matched=[];

for(let g of grid){

let dx = g.x-x;
let dy = g.y-y;
let dist = Math.sqrt(dx*dx+dy*dy);

if(dist < radius*2.5 && g.color===color){
matched.push(g);
}
}

if(matched.length >=3){

matched.forEach(m=>{

let index = grid.indexOf(m);

if(index>-1){
grid.splice(index,1);
score +=10;
}

});

scoreEl.innerText = score;

let best = localStorage.getItem("bubble_best") || 0;

if(score > best){
localStorage.setItem("bubble_best",score);
bestEl.innerText = score;
}
}
}

function update(){

ctx.clearRect(0,0,W,H);

drawBackground();

drawGrid();

updateBubble();

drawShooter();

requestAnimationFrame(update);
}

canvas.addEventListener("mousemove",(e)=>{

const rect = canvas.getBoundingClientRect();

const mx = e.clientX - rect.left;
const my = e.clientY - rect.top;

let dx = mx - shooter.x;
let dy = my - shooter.y;

shooter.angle = Math.atan2(dy,dx)+Math.PI/2;

});

canvas.addEventListener("click",()=>{

let b = shooter.bubble;

if(b.moving) return;

b.moving = true;

b.x = shooter.x;
b.y = shooter.y;

let speed = 8;

b.vx = Math.sin(shooter.angle) * speed;
b.vy = -Math.cos(shooter.angle) * speed;

});

function startGame(){

document.getElementById("menu").style.display="none";

score = 0;

scoreEl.innerText = 0;

bestEl.innerText = localStorage.getItem("bubble_best") || 0;

initGrid();

gameStarted = true;

update();
}

</script>

</body>
</html>
