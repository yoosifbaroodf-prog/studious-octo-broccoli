<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bubble Shooter Pro MAX</title>

<style>
body{
margin:0;
overflow:hidden;
background:radial-gradient(circle,#0f2027,#203a43,#2c5364);
font-family:Tahoma;
}

canvas{
display:block;
margin:auto;
background:linear-gradient(#0b1d2a,#1b3b5a);
border:4px solid #fff;
border-radius:18px;
box-shadow:0 0 40px rgba(0,0,0,.6);
}

#ui{
position:absolute;
top:10px;
width:100%;
text-align:center;
color:white;
font-size:22px;
font-weight:bold;
text-shadow:2px 2px 5px black;
}

#menu{
position:absolute;
inset:0;
background:rgba(0,0,0,.75);
display:flex;
flex-direction:column;
justify-content:center;
align-items:center;
color:white;
}

button{
padding:15px 35px;
border:none;
border-radius:12px;
font-size:20px;
cursor:pointer;
background:#ff9800;
color:white;
margin-top:15px;
}
button:hover{transform:scale(1.05)}
</style>
</head>

<body>

<div id="ui">
⭐ النقاط: <span id="score">0</span> |
🏆 أفضل نتيجة: <span id="best">0</span> |
🎯 المرحلة: <span id="level">1</span>
</div>

<div id="menu">
<h1>Bubble Shooter Pro MAX</h1>
<p>صوّب الكرات وامسح اللون نفسه!</p>
<button onclick="startGame()">ابدأ</button>
</div>

<canvas id="game" width="520" height="760"></canvas>

<script>
const c=document.getElementById("game");
const ctx=c.getContext("2d");

let score=0;
let level=1;
let grid=[];
let particles=[];
let running=false;

const R=20;
const colors=["#ff5252","#ffd600","#00e676","#40c4ff","#ab47bc","#ff9100"];

let shooter={
x:c.width/2,
y:c.height-60,
angle:0,
bubble:randomBubble()
};

function randomBubble(){
return {
color:colors[Math.floor(Math.random()*colors.length)],
x:0,y:0,vx:0,vy:0,moving:false
};
}

function init(){
grid=[];
for(let r=0;r<7+level;r++){
let cols=10-(r%2);
for(let i=0;i<cols;i++){
grid.push({
x:i*R*2+(r%2?R:0)+R,
y:r*R*1.8+40,
color:colors[Math.floor(Math.random()*colors.length)]
});
}
}
}

function drawBubble(x,y,color){
let g=ctx.createRadialGradient(x-5,y-5,5,x,y,25);
g.addColorStop(0,"white");
g.addColorStop(1,color);

ctx.fillStyle=g;
ctx.beginPath();
ctx.arc(x,y,R,0,Math.PI*2);
ctx.fill();
}

function drawGrid(){
for(let b of grid){
drawBubble(b.x,b.y,b.color);
}
}

function drawShooter(){
ctx.save();
ctx.translate(shooter.x,shooter.y);
ctx.rotate(shooter.angle);

ctx.strokeStyle="white";
ctx.lineWidth=4;
ctx.beginPath();
ctx.moveTo(0,0);
ctx.lineTo(0,-70);
ctx.stroke();

ctx.restore();

drawBubble(shooter.x,shooter.y,shooter.bubble.color);
}

function updateBubble(){
let b=shooter.bubble;
if(!b.moving)return;

b.x+=b.vx;
b.y+=b.vy;

if(b.x<R||b.x>c.width-R)b.vx*=-1;

if(b.y<R){
attach(b);
return;
}

for(let g of grid){
let dx=b.x-g.x;
let dy=b.y-g.y;
if(Math.sqrt(dx*dx+dy*dy)<R*2-2){
attach(b);
return;
}
}
}

function attach(b){
b.moving=false;
grid.push({x:b.x,y:b.y,color:b.color});
check(b);
shooter.bubble=randomBubble();
}

function check(b){
let matched=grid.filter(g=>{
let dx=g.x-b.x;
let dy=g.y-b.y;
return Math.sqrt(dx*dx+dy*dy)<R*2.2 && g.color===b.color;
});

if(matched.length>=3){
matched.forEach(m=>{
grid.splice(grid.indexOf(m),1);
createParticles(m.x,m.y,m.color);
score+=10;
});
document.getElementById("score").innerText=score;

if(score> (localStorage.best||0)){
localStorage.best=score;
document.getElementById("best").innerText=score;
}

if(grid.length<5){
level++;
document.getElementById("level").innerText=level;
init();
}
}
}

function createParticles(x,y,color){
for(let i=0;i<10;i++){
particles.push({
x,y,
vx:(Math.random()-0.5)*4,
vy:(Math.random()-0.5)*4,
life:30,
color
});
}
}

function drawParticles(){
for(let p of particles){
ctx.fillStyle=p.color;
ctx.globalAlpha=p.life/30;
ctx.beginPath();
ctx.arc(p.x,p.y,3,0,Math.PI*2);
ctx.fill();

p.x+=p.vx;
p.y+=p.vy;
p.life--;
}

particles=particles.filter(p=>p.life>0);
ctx.globalAlpha=1;
}

function loop(){
if(!running)return;

ctx.clearRect(0,0,c.width,c.height);

drawGrid();
updateBubble();
drawParticles();
drawShooter();

requestAnimationFrame(loop);
}

c.addEventListener("mousemove",(e)=>{
let r=c.getBoundingClientRect();
let mx=e.clientX-r.left;
let my=e.clientY-r.top;
shooter.angle=Math.atan2(my-shooter.y,mx-shooter.x)+Math.PI/2;
});

c.addEventListener("click",()=>{
if(shooter.bubble.moving)return;
let b=shooter.bubble;
b.moving=true;
b.x=shooter.x;
b.y=shooter.y;
b.vx=Math.sin(shooter.angle)*7;
b.vy=-Math.cos(shooter.angle)*7;
});

function startGame(){
document.getElementById("menu").style.display="none";
score=0;
level=1;
document.getElementById("score").innerText=0;
document.getElementById("best").innerText=localStorage.best||0;
document.getElementById("level").innerText=1;
init();
running=true;
loop();
}
</script>

</body>
</html>
