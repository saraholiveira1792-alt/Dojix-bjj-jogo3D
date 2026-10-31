# Dojix-bjj-jogo3D
Index.html
[31/10, 11:05] Chat Gpt: 🥋 Aqui está o código do *DOJIX BJJ — versão 1.0*.

No GitHub, abre o ficheiro `index.html` e cola TODO o código abaixo.

```html
<!DOCTYPE html>
<html lang="pt">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>DOJIX BJJ</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#101827;color:white;font-family:Arial;overflow:hidden;touch-action:none}
canvas{display:block}
#hud{position:absolute;top:12px;left:12px;right:12px;pointer-events:none}
.logo{font-size:25px;font-weight:bold;color:#f4b400}
.stats{background:#101d;border-radius:12px;padding:10px;margin-top:8px}
.bar{height:8px;background:#444;border-radius:8px;margin-top:5px;overflow:hidden}
.fill{height:100%;background:#20cf75;width:100%}
#controls{position:absolute;bottom:25px;left:15px;right:15px;display:none;justify-content:space-between;align-items:end}
#joy{width:115px;height:115px;border-radius:50%;background:#ffffff30;border:2px solid #fff8;position:relative}
#knob{width:45px;height:45px;background:#f4b400;border-radius:50%;position:absolute;left:33px;top:33px}
.buttons{display:grid;grid-template-columns:repeat(2,75px);gap:9px}
button{border:0;border-radius:12px;padding:14px 5px;background:#f4b400;color:#111;font-weight:bold;font-size:12px}
#menu{position:absolute;inset:0;background:#101827ee;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;padding:20px}
#menu h1{font-size:43px;color:#f4b400;margin-bottom:0}
#menu button{width:230px;margin:8px;font-size:15px}
#message{position:absolute;top:44%;width:100%;text-align:center;font-size:23px;font-weight:bold;pointer-events:none}
#topbuttons{position:absolute;right:10px;top:10px;display:none}
#topbuttons button{padding:9px;font-size:10px}
</style>
</head>
<body>

<div id="hud">
<div class="logo">🥋 DOJIX BJJ</div>
<div class="stats">
<div id="mode">Modo: Preparado</div>
<div id="score">XP: 0 | Pontos: 0</div>
<div id="belt">Faixa: Branca</div>
<div>Energia</div>
<div class="bar"><div class="fill" id="energy"></div></div>
</div>
</div>

<div id="topbuttons">
<button onclick="start('treino')">TREINO</button>
<button onclick="start('combate')">COMBATE</button>
</div>

<div id="message"></div>

<div id="controls">
<div id="joy"><div id="knob"></div></div>
<div class="buttons">
<button id="attack">TÉCNICA</button>
<button id="defend">DEFESA</button>
<button id="takedown">QUEDA</button>
<button id="escape">ESCAPE</button>
</div>
</div>

<div id="menu">
<h1>DOJIX BJJ</h1>
<h3>APRENDE. TREINA. EVOLUI. CONQUISTA.</h3>
<p>Entra no teu tatami virtual!</p>
<button onclick="start('treino')">COMEÇAR TREINO</button>
<button onclick="start('combate')">INICIAR COMBATE</button>
<p>Jiu-Jitsu Brasileiro 3D</p>
</div>

<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

let scene,camera,renderer,player,enemy;
let mode='treino',active=false,defending=false;
let xp=Number(localStorage.getItem('dojixXP')||0);
let points=0,energy=100,aiCooldown=0;
let joy={x:0,y:0};
let msgTime=0;

const clock=new THREE.Clock();

function mat(color){
 return new THREE.MeshStandardMaterial({color,roughness:.8});
}

function box(parent,x,y,z,w,h,d,color){
 let m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat(color));
 m.position.set(x,y,z);
 m.castShadow=true;
 parent.add(m);
 return m;
}

function ball(parent,x,y,z,r,color){
 let m=new THREE.Mesh(new THREE.SphereGeometry(r,16,12),mat(color));
 m.position.set(x,y,z);
 m.castShadow=true;
 parent.add(m);
 return m;
}

function limb(parent,x,y,z,r,h,color){
 let m=new THREE.Mesh(new THREE.CylinderGeometry(r*.8,r,h,12),mat(color));
 m.position.set(x,y,z);
 m.castShadow=true;
 parent.add(m);
 return m;
}
[31/10, 11:05] Chat Gpt: function createFighter(gi,belt,skin){
 let root=new THREE.Group();
 scene.add(root);

 box(root,0,1.35,0,.72,.8,.4,gi);
 ball(root,0,2.02,0,.23,skin);

 // Gola do kimono
 box(root,-.12,1.65,.215,.15,.65,.07,'#eeeeee');
 box(root,.12,1.65,.215,.15,.65,.07,'#eeeeee');

 // Cinto
 box(root,0,1.02,0,.85,.09,.46,belt);

 let arms=[],legs=[];

 for(let x of [-.52,.52]){
  let arm=new THREE.Group();
  arm.position.set(x,1.7,0);
  root.add(arm);

  limb(arm,0,-.25,0,.12,.5,gi);
  limb(arm,0,-.62,.02,.09,.3,gi);
  ball(arm,0,-.81,.02,.1,skin);
  arms.push(arm);
 }

 for(let x of [-.2,.2]){
  let leg=new THREE.Group();
  leg.position.set(x,.9,0);
  root.add(leg);

  limb(leg,0,-.22,0,.15,.45,gi);
  limb(leg,0,-.57,.02,.12,.35,gi);
  box(leg,0,-.78,.09,.22,.12,.35,skin);
  legs.push(leg);
 }

 return {root,arms,legs};
}

function init(){
 scene=new THREE.Scene();
 scene.background=new THREE.Color('#172638');

 camera=new THREE.PerspectiveCamera(55,innerWidth/innerHeight,.1,100);
 camera.position.set(0,8,11);
 camera.lookAt(0,0,0);

 renderer=new THREE.WebGLRenderer({antialias:true});
 renderer.setSize(innerWidth,innerHeight);
 renderer.setPixelRatio(Math.min(devicePixelRatio,1.5));
 renderer.shadowMap.enabled=true;
 document.body.appendChild(renderer.domElement);

 scene.add(new THREE.HemisphereLight(0xffffff,0x536078,2));

 let light=new THREE.DirectionalLight(0xffffff,2);
 light.position.set(4,10,5);
 light.castShadow=true;
 scene.add(light);

 // Tatami
 let floor=new THREE.Mesh(
  new THREE.PlaneGeometry(18,18),
  mat('#17677e')
 );
 floor.rotation.x=-Math.PI/2;
 floor.receiveShadow=true;
 scene.add(floor);

 let center=new THREE.Mesh(
  new THREE.PlaneGeometry(11,11),
  mat('#25869b')
 );
 center.rotation.x=-Math.PI/2;
 center.position.y=.02;
 scene.add(center);

 let grid=new THREE.GridHelper(18,18,0xffffff,0xffffff);
 grid.position.y=.04;
 scene.add(grid);

 // Lutadores
 player=createFighter('#eeeeee','#ffffff','#bd8965');
 enemy=createFighter('#2459a6','#1565c0','#a87555');

 player.root.position.set(0,0,3);
 enemy.root.position.set(0,0,-3);
 enemy.root.rotation.y=Math.PI;

 window.addEventListener('resize',resize);
 animate();
}

function resize(){
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
}

window.start=function(selected){
 mode=selected;
 active=true;
 energy=100;

 player.root.position.set(0,0,3);
 enemy.root.position.set(0,0,-3);

 document.getElementById('menu').style.display='none';
 document.getElementById('controls').style.display='flex';
 document.getElementById('topbuttons').style.display='block';

 show(mode==='treino'?'MODO TREINO':'MODO COMBATE');
 updateHUD();
};

function show(text){
 document.getElementById('message').innerText=text;
 msgTime=1.5;
}

function updateHUD(){
 document.getElementById('mode').innerText='Modo: '+mode;
 document.getElementById('score').innerText=
 'XP: '+xp+' | Pontos: '+points;

 let belt='Branca';
 if(xp>=1000)belt='Azul';
 if(xp>=3000)belt='Roxa';
 if(xp>=6000)belt='Castanha';
 if(xp>=10000)belt='Preta';

 document.getElementById('belt').innerText=
 'Faixa virtual: '+belt;

 document.getElementById('energy').style.width=energy+'%';
}

function distance(){
 return player.root.position.distanceTo(enemy.root.position);
}

function technique(){
 if(!active)return;

 if(distance()>2.4){
  show('APROXIMA-TE DO PARCEIRO!');
  return;
 }

 points+=2;
 xp+=15;
 enemy.root.position.z-=.4;

 show('TÉCNICA EXECUTADA! +15 XP');
 save();
}

function takedown(){
 if(!active)return;

 if(distance()>2.4){
  show('APROXIMA-TE!');
  return;
 }

 points+=3;
 xp+=20;
 enemy.root.position.x+=.8;

 show('QUEDA EXECUTADA! +20 XP');
 save();
}

function escape(){
 if(!active)return;

 player.root.position.x+=.5;
 xp+=5;
[31/10, 11:05] Chat Gpt: show('ESCAPE EXECUTADO!');
 save();
}

function save(){
 localStorage.setItem('dojixXP',xp);
 updateHUD();
}

// Controlos tácteis
const joyArea=document.getElementById('joy');
const knob=document.getElementById('knob');

joyArea.addEventListener('pointerdown',e=>{
 joyArea.setPointerCapture(e.pointerId);
 moveJoy(e);
});

joyArea.addEventListener('pointermove',e=>{
 if(e.buttons)moveJoy(e);
});

function moveJoy(e){
 let r=joyArea.getBoundingClientRect();
 let x=(e.clientX-r.left-r.width/2)/43;
 let y=(e.clientY-r.top-r.height/2)/43;

 let d=Math.hypot(x,y);
 if(d>1){x/=d;y/=d;}

 joy={x,y};
 knob.style.left=(33+x*30)+'px';
 knob.style.top=(33+y*30)+'px';
}

function resetJoy(){
 joy={x:0,y:0};
 knob.style.left='33px';
 knob.style.top='33px';
}

joyArea.addEventListener('pointerup',resetJoy);
joyArea.addEventListener('pointercancel',resetJoy);

document.getElementById('attack').addEventListener('pointerdown',technique);
document.getElementById('takedown').addEventListener('pointerdown',takedown);
document.getElementById('escape').addEventListener('pointerdown',escape);

let db=document.getElementById('defend');

db.addEventListener('pointerdown',()=>{
 defending=true;
 show('DEFESA ATIVA');
});

db.addEventListener('pointerup',()=>defending=false);
db.addEventListener('pointercancel',()=>defending=false);
db.addEventListener('pointerleave',()=>defending=false);

// Jogo
function animate(){
 requestAnimationFrame(animate);

 let dt=Math.min(clock.getDelta(),.05);
 let t=clock.elapsedTime;

 if(active){

  // Movimento do atleta
  player.root.position.x+=joy.x*3*dt;
  player.root.position.z+=joy.y*3*dt;

  player.root.position.x=THREE.MathUtils.clamp(
   player.root.position.x,-4.5,4.5);

  player.root.position.z=THREE.MathUtils.clamp(
   player.root.position.z,-4.5,4.5);

  let moving=Math.abs(joy.x)+Math.abs(joy.y)>.1;

  player.arms.forEach((a,i)=>{
   a.rotation.x=moving?Math.sin(t*8+i)*.3:0;
  });

  // Inteligência artificial
  if(mode==='combate'){
   let dx=player.root.position.x-enemy.root.position.x;
   let dz=player.root.position.z-enemy.root.position.z;
   let d=Math.hypot(dx,dz);

   if(d>1.4){
    enemy.root.position.x+=dx/d*dt*1.1;
    enemy.root.position.z+=dz/d*dt*1.1;
   }

   aiCooldown-=dt;

   if(d<1.6 && aiCooldown<=0){
    aiCooldown=3;

    if(defending){
     show('BOA DEFESA!');
     xp+=5;
    }else{
     energy=Math.max(0,energy-10);
     show('O ADVERSÁRIO ATACOU!');
    }
    updateHUD();
   }

   if(energy<=0){
    active=false;
    show('COMBATE TERMINADO!');
   }
  }

  // Orientação dos lutadores
  let dx=enemy.root.position.x-player.root.position.x;
  let dz=enemy.root.position.z-player.root.position.z;

  player.root.rotation.y=Math.atan2(dx,dz);
  enemy.root.rotation.y=Math.atan2(-dx,-dz);

  // Câmara
  camera.position.x=player.root.position.x*.3;
  camera.lookAt(0,0,0);
 }

 if(msgTime>0){
  msgTime-=dt;
  if(msgTime<=0)
   document.getElementById('message').innerText='';
 }

 renderer.render(scene,camera);
}

init();
</script>
</body>
</html>
```

Agora faz isto:

1. Copia o código inteiro.
2. Cola no `index.html` do GitHub.
3. Carrega em *Commit changes*.
4. Avisa-me quando terminares.

Depois vamos ativar o *GitHub Pages* para teres o teu próprio link do DOJIX BJJ! 🥋
