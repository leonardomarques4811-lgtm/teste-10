<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>SpinVillage Saga - Coin Master Style</title>
<style>
body {
    background: linear-gradient(to bottom, #1a1a2e, #0b0c1a);
    color: white;
    font-family: 'Comic Sans MS', Arial, sans-serif;
    text-align: center;
    margin: 0;
    padding: 20px;
    overflow-x: hidden;
}
h1 { 
    color: gold; 
    text-shadow: 2px 2px 5px #ff0000;
    font-size: 3em;
}
button {
    padding: 15px 30px;
    font-size: 20px;
    margin: 10px;
    border: none;
    border-radius: 15px;
    background: linear-gradient(45deg, #ff5f6d, #ffc371);
    color: white;
    cursor: pointer;
    box-shadow: 0 5px 15px rgba(0,0,0,0.3);
    transition: transform 0.1s, box-shadow 0.2s;
}
button:hover {
    box-shadow: 0 10px 20px rgba(255,255,255,0.4);
    transform: scale(1.05);
}
button:active { 
    transform: scale(0.95); 
    box-shadow: 0 3px 10px rgba(0,0,0,0.2);
}
.card {
    background: linear-gradient(to bottom, #16213e, #0f3460);
    padding: 20px;
    margin: 20px auto;
    border-radius: 20px;
    width: 340px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.5);
    transition: transform 0.2s;
}
.card:hover { transform: translateY(-5px); }

/* Roleta */
#wheel-container { position: relative; width: 300px; height: 300px; margin: 20px auto; }
#wheel { width: 100%; height: 100%; border-radius: 50%; position: relative; overflow: hidden; border: 5px solid #e94560; box-shadow: 0 0 20px rgba(0,0,0,0.5);}
.sector { position: absolute; width: 50%; height: 50%; top: 50%; left: 50%; transform-origin: 0% 0%; overflow: visible;}
.sector img { width: 60px; position: absolute; top: -30px; left: -30px;}
#pointer { position: absolute; top: -10px; left: 50%; width: 0; height: 0; border-left: 15px solid transparent; border-right: 15px solid transparent; border-bottom: 25px solid #e94560; transform: translateX(-50%); z-index:10;}

/* Vila animada */
#village { margin: 20px auto; width: 300px; height: 200px; position: relative; overflow: hidden; border-top:1px solid rgba(255,255,255,0.1);}
.building { width: 60px; height: 60px; position: absolute; bottom: 0; transition: bottom 0.5s, opacity 0.5s;}
.character { position: absolute; bottom: 0; width: 40px; animation: walk 3s linear infinite; }
@keyframes walk { 0%{left:-50px;} 100%{left:260px;} }

/* Moedas e escudos caindo */
.flying { position: absolute; width: 30px; height: 30px; animation: fall 1s ease-out forwards; z-index:100; }
@keyframes fall { 0% {opacity:1; transform: translateY(0px);} 100% {opacity:0; transform: translateY(200px);} }

/* Ataque animado */
.attack-effect { position:absolute; width:80px; height:80px; background:radial-gradient(circle, #ff0000 0%, transparent 70%); border-radius:50%; animation: explode 0.6s forwards; z-index:200; }
@keyframes explode { 0%{transform:scale(0);opacity:1;} 100%{transform:scale(1.5);opacity:0;} }
</style>
</head>
<body>

<h1>🏰 SpinVillage Saga 🎰</h1>

<div class="card">
    <p>💰 Moedas: <span id="coins">200</span></p>
    <p>🎰 Spins: <span id="spins">20</span></p>
    <p>🛡️ Escudos: <span id="shields">0</span></p>
</div>

<div id="wheel-container">
    <div id="pointer"></div>
    <div id="wheel"></div>
</div>

<button onclick="spinWheel()">GIRAR ROLETA 🎰</button>

<div class="card" id="village-card">
    <h2 id="villageName">🌲 Vila da Floresta</h2>
    <div id="village"></div>
    <p>Construções: <span id="buildings">0</span>/3</p>
    <button onclick="build()">Construir 🏗️</button>
</div>

<script>
let coins = 200;
let spins = 20;
let shields = 0;
let villageIndex = 0;

const villages = [
    { name: "🌲 Vila da Floresta", costs: [200,300,400], built: 0, image:["https://i.imgur.com/8P5Kf2G.png","https://i.imgur.com/3PTfI6g.png","https://i.imgur.com/YWdNmfV.png"] },
    { name: "🏜️ Vila do Deserto", costs: [400,500,600], built: 0, image:["https://i.imgur.com/6XWTiE5.png","https://i.imgur.com/ocfZ3Cn.png","https://i.imgur.com/Y8j5iMK.png"] },
    { name: "❄️ Vila Congelada", costs: [600,800,1000], built: 0, image:["https://i.imgur.com/g7ZhCqJ.png","https://i.imgur.com/IXoq2hZ.png","https://i.imgur.com/k6skzL2.png"] }
];

const rewards = [
    { type: "coins", img: "https://i.imgur.com/1KegWPz.png", value: 100 },
    { type: "attack", img: "https://i.imgur.com/2M0x1hL.png", value: 200 },
    { type: "shield", img: "https://i.imgur.com/6dX0A5b.png", value: 1 },
    { type: "coins", img: "https://i.imgur.com/1KegWPz.png", value: 50 }
];

const wheel = document.getElementById("wheel");

// Cria roleta
function createWheel(){
    rewards.forEach((reward,i)=>{
        const div = document.createElement("div");
        div.className="sector";
        div.style.transform=`rotate(${i*360/rewards.length}deg) skewY(-60deg)`;
        const img = document.createElement("img");
        img.src=reward.img;
        div.appendChild(img);
        wheel.appendChild(div);
    });
}
createWheel();

// Vila animada
function updateVillage(){
    const villageDiv = document.getElementById("village");
    villageDiv.innerHTML="";
    const village=villages[villageIndex];
    // Construções
    for(let i=0;i<village.built;i++){
        const img=document.createElement("img");
        img.src=village.image[i];
        img.className="building";
        img.style.left=`${i*70}px`;
        img.style.opacity=0;
        villageDiv.appendChild(img);
        setTimeout(()=>{img.style.opacity=1; img.style.bottom="0px";},100);
    }
    // Personagem andando
    const character=document.createElement("img");
    character.src="https://i.imgur.com/4AiXzf8.png"; // personagem genérico
    character.className="character";
    villageDiv.appendChild(character);
}

function updateUI(){
    document.getElementById("coins").innerText=coins;
    document.getElementById("spins").innerText=spins;
    document.getElementById("shields").innerText=shields;
    document.getElementById("villageName").innerText=villages[villageIndex].name;
    document.getElementById("buildings").innerText=villages[villageIndex].built;
    updateVillage();
}

// Moedas/escudos voando
function flyEffect(type,value){
    const container=document.body;
    for(let i=0;i<value;i++){
        const img=document.createElement("img");
        img.src=type==="coins"?"https://i.imgur.com/1KegWPz.png":"https://i.imgur.com/6dX0A5b.png";
        img.className="flying";
        img.style.left=Math.random()*window.innerWidth+"px";
        img.style.top="-50px";
        container.appendChild(img);
        setTimeout(()=>{img.remove();},1200);
    }
}

// Ataque animado
function attackEffect(){
    const container=document.getElementById("village");
    const atk=document.createElement("div");
    atk.className="attack-effect";
    atk.style.left="100px";
    atk.style.bottom="50px";
    container.appendChild(atk);
    setTimeout(()=>atk.remove(),600);
}

// Girar roleta
function spinWheel(){
    if(spins<=0){ alert("Sem spins!"); return;}
    spins--;
    const sectors=rewards.length;
    const randomIndex=Math.floor(Math.random()*sectors);
    const spinsCount=6;
    const rotation=spinsCount*360 + randomIndex*(360/sectors) + (360/sectors)/2;

    wheel.style.transition="transform 4s cubic-bezier(0.33, 1, 0.68, 1)";
    wheel.style.transform=`rotate(${rotation}deg)`;

    setTimeout(()=>{
        const reward=rewards[randomIndex];
        if(reward.type==="coins") { coins+=reward.value; flyEffect("coins",reward.value/50);}
        else if(reward.type==="shield") { shields+=reward.value; flyEffect("shield",reward.value);}
        else if(reward.type==="attack") { coins+=reward.value; attackEffect();}

        alert(`🎉 Você ganhou: ${reward.type==="coins"?reward.value+" moedas":reward.type==="shield"?reward.value+" escudo(s)":"Ataque! +" + reward.value}`);
        saveGame();
        updateUI();
    },4000);
}

// Construir vila
function build(){
    const village=villages[villageIndex];
    if(village.built>=3){ alert("Vila completa!"); return;}
    const cost=village.costs[village.built];
    if(coins>=cost){
        coins-=cost;
        village.built++;
        if(village.built===3 && villageIndex<villages.length-1){
            villageIndex++;
            alert("Nova vila desbloqueada!");
        }
        saveGame();
        updateUI();
    } else { alert("Moedas insuficientes!");}
}

// Salvar e carregar
function saveGame(){
    localStorage.setItem("spinVillageSave",JSON.stringify({coins,spins,shields,villageIndex,villages}));
}
function loadGame(){
    const data=localStorage.getItem("spinVillageSave");
    if(data){
        const save=JSON.parse(data);
        coins=save.coins; spins=save.spins; shields=save.shields; villageIndex=save.villageIndex;
        Object.assign(villages,save.villages);
    }
}
loadGame();
updateUI();
</script>
</body>
</html>
