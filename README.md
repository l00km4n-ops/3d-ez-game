<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Game Perang 3D</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
    #instructions {
      position: absolute;
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,0.8);
      color: #fff;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 24px;
      cursor: pointer;
      z-index: 1;
    }
  </style>
</head>
<body>
<div id="instructions">Klik untuk mulai</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/PointerLockControls.js"></script>

<script>
let camera, scene, renderer, controls;
let bullets = [], enemies = [];
let moveF = false, moveB = false, moveL = false, moveR = false;
const clock = new THREE.Clock();
const velocity = new THREE.Vector3();

init();

function init() {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x222222);

  camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 1, 1000);
  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  // Controls
  controls = new THREE.PointerLockControls(camera, document.body);
  const instructions = document.getElementById('instructions');
  instructions.addEventListener('click', () => controls.lock());
  controls.addEventListener('lock', () => instructions.style.display = 'none');
  controls.addEventListener('unlock', () => instructions.style.display = '');

  scene.add(controls.getObject());

  // Ground
  const ground = new THREE.Mesh(
    new THREE.PlaneGeometry(200, 200),
    new THREE.MeshBasicMaterial({color: 0x333333})
  );
  ground.rotation.x = -Math.PI / 2;
  scene.add(ground);

  // Light
  const light = new THREE.HemisphereLight(0xffffff, 0x444444);
  scene.add(light);

  // Enemies
  for (let i = 0; i < 5; i++) {
    const box = new THREE.Mesh(
      new THREE.BoxGeometry(1, 1, 1),
      new THREE.MeshBasicMaterial({color: 0xff0000})
    );
    box.position.set(Math.random() * 100 - 50, 0.5, Math.random() * -50);
    scene.add(box);
    enemies.push(box);
  }

  document.addEventListener('keydown', keyDown);
  document.addEventListener('keyup', keyUp);
  document.addEventListener('click', shoot);

  animate();
}

function keyDown(e) {
  if (e.code === 'KeyW') moveF = true;
  if (e.code === 'KeyS') moveB = true;
  if (e.code === 'KeyA') moveL = true;
  if (e.code === 'KeyD') moveR = true;
}
function keyUp(e) {
  if (e.code === 'KeyW') moveF = false;
  if (e.code === 'KeyS') moveB = false;
  if (e.code === 'KeyA') moveL = false;
  if (e.code === 'KeyD') moveR = false;
}

function shoot() {
  const bullet = new THREE.Mesh(
    new THREE.SphereGeometry(0.1),
    new THREE.MeshBasicMaterial({color: 0xffff00})
  );
  bullet.position.copy(controls.getObject().position);
  const dir = new THREE.Vector3();
  camera.getWorldDirection(dir);
  bullet.userData.velocity = dir.clone().multiplyScalar(2);
  bullets.push(bullet);
  scene.add(bullet);
}

function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta();

  velocity.set(0, 0, 0);
  if (moveF) velocity.z -= 10 * delta;
  if (moveB) velocity.z += 10 * delta;
  if (moveL) velocity.x -= 10 * delta;
  if (moveR) velocity.x += 10 * delta;
  controls.moveRight(velocity.x);
  controls.moveForward(velocity.z);

  bullets.forEach((b, i) => {
    b.position.add(b.userData.velocity);
    enemies.forEach((e, ei) => {
      if (b.position.distanceTo(e.position) < 0.6) {
        scene.remove(e);
        enemies.splice(ei, 1);
        scene.remove(b);
        bullets.splice(i, 1);
      }
    });
    if (b.position.length() > 200) {
      scene.remove(b);
      bullets.splice(i, 1);
    }
  });

  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.render(scene, camera);
}
</script>
</body>
</html>

