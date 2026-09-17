<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Cyber Shooter</title>
<style>
body{
    margin:0;
    overflow:hidden;
    background:#000;
}
canvas{
    display:block;
    background:#050510;
}
#score{
    position:absolute;
    top:10px;
    left:10px;
    color:#00ffcc;
    font-family:Arial;
    font-size:24px;
}
</style>
</head>
<body>

<div id="score">Score: 0</div>
<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let score = 0;

const player = {
    x: canvas.width/2,
    y: canvas.height/2,
    size: 20,
    speed: 5
};

const keys = {};
const bullets = [];
const enemies = [];

document.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
document.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

canvas.addEventListener("click", e => {
    const rect = canvas.getBoundingClientRect();
    const mx = e.clientX - rect.left;
    const my = e.clientY - rect.top;

    const angle = Math.atan2(my-player.y, mx-player.x);

    bullets.push({
        x: player.x,
        y: player.y,
        dx: Math.cos(angle)*8,
        dy: Math.sin(angle)*8,
        size: 5
    });
});

function spawnEnemy(){
    let side = Math.floor(Math.random()*4);
    let x,y;

    if(side===0){
        x=0; y=Math.random()*canvas.height;
    }else if(side===1){
        x=canvas.width; y=Math.random()*canvas.height;
    }else if(side===2){
        x=Math.random()*canvas.width; y=0;
    }else{
        x=Math.random()*canvas.width; y=canvas.height;
    }

    enemies.push({
        x,y,
        size:18,
        speed:1.5
    });
}

setInterval(spawnEnemy,1200);

function update(){

    if(keys["w"]) player.y -= player.speed;
    if(keys["s"]) player.y += player.speed;
    if(keys["a"]) player.x -= player.speed;
    if(keys["d"]) player.x += player.speed;

    bullets.forEach((b,i)=>{
        b.x += b.dx;
        b.y += b.dy;

        if(
            b.x<0 || b.x>canvas.width ||
            b.y<0 || b.y>canvas.height
        ){
            bullets.splice(i,1);
        }
    });

    enemies.forEach((e,ei)=>{

        const angle = Math.atan2(
            player.y-e.y,
            player.x-e.x
        );

        e.x += Math.cos(angle)*e.speed;
        e.y += Math.sin(angle)*e.speed;

        bullets.forEach((b,bi)=>{

            const dist = Math.hypot(
                b.x-e.x,
                b.y-e.y
            );

            if(dist < e.size){
                enemies.splice(ei,1);
                bullets.splice(bi,1);

                score++;
                document.getElementById("score").innerText =
                "Score: " + score;
            }
        });
    });
}

function draw(){

    ctx.clearRect(0,0,canvas.width,canvas.height);

    // Neon grid
    ctx.strokeStyle="rgba(0,255,255,0.08)";
    for(let x=0;x<canvas.width;x+=50){
        ctx.beginPath();
        ctx.moveTo(x,0);
        ctx.lineTo(x,canvas.height);
        ctx.stroke();
    }

    for(let y=0;y<canvas.height;y+=50){
        ctx.beginPath();
        ctx.moveTo(0,y);
        ctx.lineTo(canvas.width,y);
        ctx.stroke();
    }

    // Player
    ctx.shadowBlur=20;
    ctx.shadowColor="#00ffff";
    ctx.fillStyle="#00ffff";
    ctx.beginPath();
    ctx.arc(player.x,player.y,player.size,0,Math.PI*2);
    ctx.fill();

    // Bullets
    bullets.forEach(b=>{
        ctx.shadowColor="#00ff00";
        ctx.fillStyle="#00ff00";
        ctx.beginPath();
        ctx.arc(b.x,b.y,b.size,0,Math.PI*2);
        ctx.fill();
    });

    // Enemies
    enemies.forEach(e=>{
        ctx.shadowColor="#ff0000";
        ctx.fillStyle="#ff0000";
        ctx.beginPath();
        ctx.arc(e.x,e.y,e.size,0,Math.PI*2);
        ctx.fill();
    });
}

function gameLoop(){
    update();
    draw();
    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
