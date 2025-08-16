<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>3D Sky Jump</title>
<style>
  body { margin:0; overflow:hidden; }
  canvas { display:block; }
  #score { position:absolute; top:20px; left:20px; color:white; font-size:24px; font-family:sans-serif; z-index:10; }
</style>
</head>
<body>
<div id="score">Score: 0</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r152/three.min.js"></script>
<script>
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Lighting
const light = new THREE.DirectionalLight(0xffffff,1);
light.position.set(10,20,10);
scene.add(light);
scene.add(new THREE.AmbientLight(0x404040));

// Player
const playerGeo = new THREE.BoxGeometry(1,1,1);
const playerMat = new THREE.MeshStandardMaterial({color:0xff0000});
const player = new THREE.Mesh(playerGeo,playerMat);
scene.add(player);
player.position.y = 2;

// Platforms
let platforms = [];
function createPlatform(y){
  const geo = new THREE.BoxGeometry(5,1,5);
  const mat = new THREE.MeshStandardMaterial({color:0xffffff});
  const p = new THREE.Mesh(geo,mat);
  p.position.y = y;
  p.position.x = Math.random()*20-10;
  platforms.push(p);
  scene.add(p);
}
for(let i=0;i<10;i++) createPlatform(i*5);

// Coins
let coins = [];
function createCoin(x,y,z){
  const geo = new THREE.TorusGeometry(0.3,0.1,16,100);
  const mat = new THREE.MeshStandardMaterial({color:0xffff00});
  const coin = new THREE.Mesh(geo,mat);
  coin.position.set(x,y+1,z);
  coins.push(coin);
  scene.add(coin);
}
platforms.forEach(p=>{
  if(Math.random()<0.5) createCoin(p.position.x,p.position.y,p.position.z);
});

// Camera
camera.position.z = 10;
camera.position.y = player.position.y + 5;
camera.lookAt(player.position);

// Jump mechanics
let velocityY = 0;
const gravity = -0.5;
let onGround = true;
let score = 0;

function jump(){
  if(onGround){
    velocityY = 0.2;
    onGround = false;
  }
}

document.addEventListener('mousedown', jump);
document.addEventListener('touchstart', jump);

function animate(){
  requestAnimationFrame(animate);

  // Physics
  velocityY += gravity*0.02;
  player.position.y += velocityY;

  // Collision with platforms
  platforms.forEach(p=>{
    if(velocityY<=0 && player.position.x +0.5 > p.position.x-2.5 && player.position.x -0.5 < p.position.x+2.5 &&
       player.position.y-0.5 <= p.position.y+0.5 && player.position.y-0.5 > p.position.y-0.5){
      player.position.y = p.position.y +1;
      velocityY = 0.15;
      onGround = true;
      score++;
      document.getElementById('score').innerText = 'Score: '+score;
    }
  });

  // Coin collection
  coins.forEach(c=>{
    if(c.visible && player.position.distanceTo(c.position)<1){
      c.visible = false;
      score +=5;
      document.getElementById('score').innerText = 'Score: '+score;
    }
  });

  // Scroll camera
  if(player.position.y>camera.position.y-5) camera.position.y = player.position.y+5;

  // Animate coins
  coins.forEach(c=>{ if(c.visible) c.rotation.y +=0.05; });

  // Game Over
  if(player.position.y<-5){
    alert('Game Over! Score: '+score);
    location.reload();
  }

  renderer.render(scene,camera);
}

animate();
</script>
</body>
</html>
