<html lang="en"><head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Home - Classroom</title>
  <link rel="icon" type="image/x-icon" href="favicon.png">
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: Arial, sans-serif;
      background: black;
    }

    #menu {
      position: fixed;
      inset: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      background: rgba(0,0,0,0.8);
      color: white;
      z-index: 100;
    }

    button {
      padding: 14px 28px;
      font-size: 20px;
      cursor: pointer;
      border: none;
      border-radius: 10px;
      background: #4caf50;
      color: white;
    }

    #crosshair {
      position: fixed;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      color: white;
      font-size: 28px;
      pointer-events: none;
      z-index: 50;
    }

    #hotbar {
      position: fixed;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 8px;
      z-index: 50;
    }

    .slot {
      width: 50px;
      height: 50px;
      border: 3px solid white;
      background: rgba(255,255,255,0.15);
    }

    .active {
      border-color: yellow;
    }
  </style>
</head>
<body>
  <div id="menu">
    <h1>HimothyCraft</h1>
    <p>A Minecraft-inspired survival sandbox</p>
    <div style="display:flex;gap:12px;margin-top:10px;">
      <button id="survivalBtn">Survival</button>
      <button id="creativeBtn">Creative</button>
    </div>
    <button id="playBtn">Play</button>
  </div>

  <div id="crosshair">+</div>

  <div id="hud" style="position:fixed;bottom:80px;left:50%;transform:translateX(-50%);z-index:50;font-family:monospace;pointer-events:none;">
    <div id="hearts" style="font-size:28px;text-shadow:3px 3px 0 black;letter-spacing:2px;">❤️ ❤️ ❤️ ❤️ ❤️ ❤️ ❤️ ❤️ ❤️ ❤️ </div>
    <div id="hunger" style="font-size:28px;text-shadow:3px 3px 0 black;letter-spacing:2px;">🍖 🍖 🍖 🍖 🍖 🍖 🍖 🍖 🍖 </div>
  </div>

  <div id="inventory" style="position:fixed;bottom:15px;left:50%;transform:translateX(-50%);display:flex;gap:6px;background:rgba(0,0,0,0.45);padding:8px;border:4px solid #555;z-index:50;">
    <div class="javaSlot selectedSlot">⛏️</div>
    <div class="javaSlot">🪓</div>
    <div class="javaSlot">⚔️</div>
    <div class="javaSlot">🟩</div>
    <div class="javaSlot">🟫</div>
    <div class="javaSlot">⬜</div>
    <div class="javaSlot">🪵</div>
    <div class="javaSlot">🍎</div>
    <div class="javaSlot">🌾</div>
  </div>

  <style>
    .javaSlot {
      width: 52px;
      height: 52px;
      background: rgba(255,255,255,0.1);
      border: 3px solid #888;
      display:flex;
      align-items:center;
      justify-content:center;
      font-size:28px;
      color:white;
      text-shadow:2px 2px 0 black;
    }

    .selectedSlot {
      border-color: white;
      transform: scale(1.08);
    }
  </style>
    <div>🟫 Dirt x64</div>
    <div>⬜ Stone x64</div>
    <div>🪵 Wood x32</div>
    <div>⛏️ Pickaxe</div>
    <div>🪓 Axe</div>
    <div>⚔️ Sword</div>
  
  <div id="hotbar"></div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);

const renderer = new THREE.WebGLRenderer({ powerPreference: 'high-performance' });
renderer.setPixelRatio(1);
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

camera.position.set(16, 12, 16);

let velocityY = 0;
let onGround = false;
camera.rotation.order = 'YXZ';

const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(10, 20, 10);
scene.add(light);
scene.add(new THREE.AmbientLight(0xffffff, 0.6));

const blocks = [];
const geometry = new THREE.BoxGeometry(1,1,1);

function makeTexture(base, dark) {
  const c = document.createElement('canvas');
  c.width = 16;
  c.height = 16;
  const ctx = c.getContext('2d');

  ctx.fillStyle = base;
  ctx.fillRect(0,0,16,16);

  for (let i = 0; i < 40; i++) {
    ctx.fillStyle = Math.random() > 0.5 ? dark : base;
    ctx.fillRect(Math.random()*16, Math.random()*16, 2, 2);
  }

  const tex = new THREE.CanvasTexture(c);
  tex.magFilter = THREE.NearestFilter;
  return tex;
}

const materials = {
  grass: new THREE.MeshLambertMaterial({
    map: makeTexture('#55aa33', '#4a992b')
  }),
  dirt: new THREE.MeshLambertMaterial({
    map: makeTexture('#8b5a2b', '#6f431d')
  }),
  stone: new THREE.MeshLambertMaterial({
    map: makeTexture('#888888', '#666666')
  })
};

const worldSize = 48;
const solidBlocks = [];

for (let x = 0; x < worldSize; x++) {
  for (let z = 0; z < worldSize; z++) {
    const height = Math.floor(Math.sin(x * 0.3) * 2 + Math.cos(z * 0.25) * 2 + Math.random() * 2 + 6);

    for (let y = 0; y < height; y++) {
      let material = materials.dirt;

      if (y === height - 1) material = materials.grass;
      if (y < 2) material = materials.stone;

      const cube = new THREE.Mesh(geometry, material);
      cube.position.set(x, y, z);
      scene.add(cube);
      blocks.push(cube);
      solidBlocks.push(cube);
    }

    // Trees
    if (Math.random() < 0.015) {
      const treeHeight = Math.floor(Math.random() * 3) + 3;

      for (let t = 0; t < treeHeight; t++) {
        const trunk = new THREE.Mesh(
          geometry,
          new THREE.MeshLambertMaterial({
            map: makeTexture('#7a4b22', '#5f3817')
          })
        );

        trunk.position.set(x, height + t, z);
        scene.add(trunk);
        solidBlocks.push(trunk);
      }

      for (let lx = -1; lx <= 1; lx++) {
        for (let ly = -1; ly <= 1; ly++) {
          for (let lz = -1; lz <= 1; lz++) {
            if (Math.random() > 0.3) {
              const leaf = new THREE.Mesh(
                geometry,
                new THREE.MeshLambertMaterial({
                  map: makeTexture('#2e8b57', '#206040')
                })
              );

              leaf.position.set(
                x + lx,
                height + treeHeight + ly,
                z + lz
              );

              scene.add(leaf);
              solidBlocks.push(leaf);
            }
          }
        }
      }
    }

    // Crops
    if (Math.random() < 0.03) {
      const crop = new THREE.Mesh(
        new THREE.BoxGeometry(0.5, 0.8, 0.5),
        new THREE.MeshLambertMaterial({ color: 0xa8d64f })
      );

      crop.position.set(x, height, z);
      scene.add(crop);
    }
  }
}

// Monsters
const monsters = [];

for (let i = 0; i < 5; i++) {
  const monster = new THREE.Mesh(
    new THREE.BoxGeometry(1, 2, 1),
    new THREE.MeshLambertMaterial({ color: 0x55ff55 })
  );

  monster.position.set(
    Math.random() * worldSize,
    8,
    Math.random() * worldSize
  );

  scene.add(monster);
  monsters.push(monster);
}

// Hotbar inventory
const inventory = ['Grass', 'Dirt', 'Stone', 'Wood'];

// Basic crafting recipes
const crafting = {
  'Wood + Wood': 'Stick',
  'Stick + Stone': 'Stone Sword',
  'Stick + Wood': 'Wooden Axe',
  'Stick + Dirt': 'Hoe'
};

console.log('Crafting Recipes:', crafting);
const slots = document.querySelectorAll('.slot');
slots.forEach((s, i) => {
  s.innerHTML = `<div style="color:white;font-size:12px;text-align:center;padding-top:14px">${inventory[i]}</div>`;
});

const menu = document.getElementById('menu');
const playBtn = document.getElementById('playBtn');
const survivalBtn = document.getElementById('survivalBtn');
const creativeBtn = document.getElementById('creativeBtn');

let gameMode = 'survival';

survivalBtn.onclick = () => {
  gameMode = 'survival';
  survivalBtn.style.border = '3px solid yellow';
  creativeBtn.style.border = 'none';
};

creativeBtn.onclick = () => {
  gameMode = 'creative';
  creativeBtn.style.border = '3px solid yellow';
  survivalBtn.style.border = 'none';
};

let locked = false;

playBtn.onclick = () => {
  renderer.domElement.requestPointerLock();
};

document.addEventListener('pointerlockchange', () => {
  locked = document.pointerLockElement === renderer.domElement;
  menu.style.display = locked ? 'none' : 'flex';
});

let rotX = 0;
let rotY = 0;

document.addEventListener('mousemove', e => {
  if (!locked) return;

  rotY -= e.movementX * 0.002;
  rotX -= e.movementY * 0.002;

  rotX = Math.max(-Math.PI/2, Math.min(Math.PI/2, rotX));

  camera.rotation.x = rotX;
  camera.rotation.y = rotY;
});

const keys = {};

document.addEventListener('keydown', e => {
  keys[e.key.toLowerCase()] = true;
});

document.addEventListener('keyup', e => {
  keys[e.key.toLowerCase()] = false;
});

// Mining and placing blocks
const raycaster = new THREE.Raycaster();

window.addEventListener('mousedown', e => {
  if (!locked) return;

  raycaster.setFromCamera(new THREE.Vector2(0,0), camera);
  const hits = raycaster.intersectObjects(solidBlocks);

  if (hits.length > 0) {
    const hit = hits[0];

    if (e.button === 0) {
      scene.remove(hit.object);
      const i = solidBlocks.indexOf(hit.object);
      if (i >= 0) solidBlocks.splice(i, 1);
    }

    if (e.button === 2) {
      const pos = hit.point.clone().add(hit.face.normal.multiplyScalar(0.5));

      const block = new THREE.Mesh(
        geometry,
        materials.grass
      );

      block.position.set(
        Math.round(pos.x),
        Math.round(pos.y),
        Math.round(pos.z)
      );

      scene.add(block);
      solidBlocks.push(block);
    }
  }
});

window.addEventListener('contextmenu', e => e.preventDefault());

function collides(x, y, z) {
  for (const b of solidBlocks) {
    const bx = b.position.x;
    const by = b.position.y;
    const bz = b.position.z;

    if (
      Math.abs(x - bx) < 0.45 &&
      Math.abs(y - by) < 1.5 &&
      Math.abs(z - bz) < 0.45
    ) {
      return true;
    }
  }

  return false;
}

let health = 10;
let hunger = 10;

setInterval(() => {
  if (gameMode === 'survival') {
    hunger = Math.max(0, hunger - 0.02);

    if (hunger <= 0) {
      health = Math.max(0, health - 0.05);
    }

    document.getElementById('hearts').innerHTML = '❤️ '.repeat(Math.ceil(health));
    document.getElementById('hunger').innerHTML = '🍖 '.repeat(Math.ceil(hunger));

    if (health <= 0) {
      camera.position.set(16, 20, 16);
      health = 10;
      hunger = 10;
    }
  }
}, 100);

function move() {
  const speed = 0.1;

  const forward = new THREE.Vector3();
  camera.getWorldDirection(forward);
  forward.y = 0;
  forward.normalize();

  const right = new THREE.Vector3();
  right.crossVectors(forward, new THREE.Vector3(0,1,0)).normalize();

  let moveX = 0;
  let moveZ = 0;

  if (keys['w']) {
    moveX += forward.x * speed;
    moveZ += forward.z * speed;
  }

  if (keys['s']) {
    moveX -= forward.x * speed;
    moveZ -= forward.z * speed;
  }

  if (keys['a']) {
    moveX += right.x * speed;
    moveZ += right.z * speed;
  }

  if (keys['d']) {
    moveX -= right.x * speed;
    moveZ -= right.z * speed;
  }

  const nextX = camera.position.x + moveX;
  const nextZ = camera.position.z + moveZ;

  if (!collides(nextX, camera.position.y - 1, camera.position.z)) {
    camera.position.x = nextX;
  }

  if (!collides(camera.position.x, camera.position.y - 1, nextZ)) {
    camera.position.z = nextZ;
  }

  if (gameMode !== 'creative') {
    velocityY -= 0.008;
  }
  camera.position.y += velocityY;

  onGround = false;

  for (const b of solidBlocks) {
    const bx = b.position.x;
    const by = b.position.y;
    const bz = b.position.z;

    if (
      Math.abs(camera.position.x - bx) < 0.6 &&
      Math.abs(camera.position.z - bz) < 0.6
    ) {
      const top = by + 1.7;

      if (
        camera.position.y <= top &&
        camera.position.y >= by + 0.8 &&
        velocityY <= 0
      ) {
        camera.position.y = top;
        velocityY = 0;
        onGround = true;
      }
    }
  }

  if (gameMode === 'creative') {
    if (keys[' ']) camera.position.y += 0.15;
    if (keys['shift']) camera.position.y -= 0.15;
  }

  if (keys[' '] && onGround && gameMode !== 'creative') {
    velocityY = 0.18;
  }

  if (camera.position.y < 2) {
    camera.position.y = 20;
  }
}

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// Simple distance culling for performance
function updateVisibility() {
  for (const b of solidBlocks) {
    const dx = b.position.x - camera.position.x;
    const dz = b.position.z - camera.position.z;
    b.visible = (dx * dx + dz * dz) < 500;
  }
}

function animate() {
  requestAnimationFrame(animate);
  move();
  updateVisibility();
  renderer.render(scene, camera);
}

// Day/night cycle
let sunAngle = 0;

function updateDayNight() {
  sunAngle += 0.001;
  light.position.x = Math.sin(sunAngle) * 50;
  light.position.y = Math.cos(sunAngle) * 50;

  if (light.position.y < 0) {
    scene.background = new THREE.Color(0x111133);
  } else {
    scene.background = new THREE.Color(0x87ceeb);
  }
}

const title = document.createElement('div');
title.innerHTML = '<div style="position:fixed;top:20px;width:100%;text-align:center;color:white;font-size:48px;font-weight:bold;text-shadow:4px 4px 6px black;z-index:60;pointer-events:none;">HIMOTHYCRAFT</div>';
document.body.appendChild(title);

const oldAnimate = animate;
animate = function() {
  requestAnimationFrame(animate);
  move();
  updateVisibility();
  updateDayNight();
  renderer.render(scene, camera);
};

animate();
</script><canvas data-engine="three.js r160" width="1045" height="773" style="display: block; width: 1045px; height: 773px;"></canvas><div><div style="position:fixed;top:20px;width:100%;text-align:center;color:white;font-size:48px;font-weight:bold;text-shadow:4px 4px 6px black;z-index:60;pointer-events:none;">HIMOTHYCRAFT</div></div>
<div id="credit" style="position:fixed;bottom:8px;right:8px;color:white;font-family:monospace;font-size:12px;background:rgba(0,0,0,0.5);padding:4px 8px;border-radius:4px;z-index:9999;">Made by Isaac Yoon</div>
<div id="credit" style="position:fixed;bottom:8px;right:8px;color:white;font-family:monospace;font-size:12px;background:rgba(0,0,0,0.5);padding:4px 8px;border-radius:4px;z-index:9999;">Made by Isaac Yoon</div>


</body></html>
