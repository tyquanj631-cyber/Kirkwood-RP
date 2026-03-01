# Kirkwood-RP
Every holiday update every taco Tuesday every using UnityEngine;

public class Builder : MonoBehaviour
{
    public GameObject wallPrefab; // The item you want to build

    void Update()
    {
        // If the player clicks the left mouse button
        if (Input.GetMouseButtonDown(0))
        {
            BuildObject();
        }
    }

    void BuildObject()
    {
        // This creates the wall at the current script's position
        Instantiate(wallPrefab, transform.position, transform.rotation);
    }
}<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Realistic Town Life Simulator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { 
            margin: 0; 
            overflow: hidden; 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            user-select: none; 
            background: #87CEEB;
            touch-action: none;
        }
        #ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; padding: env(safe-area-inset-top) env(safe-area-inset-right) env(safe-area-inset-bottom) env(safe-area-inset-left); }
        .interactive { pointer-events: auto; }
        
        #joystick-wrapper {
            position: absolute;
            bottom: 40px;
            left: 40px;
            width: 100px;
            height: 100px;
            background: rgba(255, 255, 255, 0.2);
            border: 2px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            backdrop-filter: blur(8px);
        }
        #joystick-knob {
            width: 44px;
            height: 44px;
            background: white;
            border-radius: 50%;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }

        #chat-log { height: 100px; overflow-y: auto; scrollbar-width: none; mask-image: linear-gradient(transparent, black 20%); }
        #chat-log::-webkit-scrollbar { display: none; }
        .btn-hover { transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1); -webkit-tap-highlight-color: transparent; }
        .btn-hover:active { transform: scale(0.9); background-color: #f3f4f6; }
        .game-font { font-family: 'Segoe UI', system-ui, sans-serif; }
    </style>
</head>
<body>

    <div id="ui-layer" class="game-font">
        <!-- Top Stats -->
        <div class="flex justify-between p-4">
            <div class="flex flex-col gap-1">
                <div class="bg-black/40 text-white px-4 py-2 rounded-2xl backdrop-blur-xl border border-white/10 interactive flex items-center gap-2 text-sm shadow-lg">
                    <span id="time-emoji">☀️</span> <span id="time-text">08:00 AM</span>
                </div>
                <div class="bg-blue-600/60 text-white px-4 py-1.5 rounded-xl backdrop-blur-md border border-white/10 interactive text-xs flex items-center gap-2 shadow-md">
                    💼 <span id="job-text">Citizen</span>
                </div>
            </div>
            <div class="bg-white/90 text-slate-900 font-bold px-5 py-2 rounded-2xl border-b-4 border-slate-300 interactive shadow-xl flex items-center gap-2">
                💰 <span class="text-green-600">$</span><span id="money-text">1500</span>
            </div>
        </div>

        <!-- Chat System -->
        <div class="absolute bottom-6 left-4 w-64 interactive flex flex-col gap-1">
            <div id="chat-log" class="bg-black/30 text-white p-3 rounded-t-2xl text-[11px] backdrop-blur-sm">
                <div class="opacity-70 italic text-[9px]">Welcome to the town! Drag to look around.</div>
                <div class="text-blue-200">System: Use the joystick to move.</div>
            </div>
            <input id="chat-input" type="text" placeholder="Type to chat..." class="w-full bg-black/50 text-white px-4 py-3 rounded-b-2xl border border-white/10 outline-none text-sm focus:bg-black/70 transition-colors">
        </div>

        <!-- Right Action Menu -->
        <div class="absolute right-4 bottom-32 flex flex-col gap-4">
            <button onclick="changeOutfit()" class="bg-white/95 text-indigo-600 w-14 h-14 rounded-full shadow-2xl interactive btn-hover flex items-center justify-center text-2xl border-b-4 border-indigo-200">👕</button>
            <button onclick="spawnVehicle()" class="bg-white/95 text-rose-500 w-14 h-14 rounded-full shadow-2xl interactive btn-hover flex items-center justify-center text-2xl border-b-4 border-rose-200">🚗</button>
            <button onclick="toggleHouse()" class="bg-white/95 text-emerald-600 w-14 h-14 rounded-full shadow-2xl interactive btn-hover flex items-center justify-center text-2xl border-b-4 border-emerald-200">🏠</button>
            <button onclick="openJobMenu()" class="bg-white/95 text-amber-600 w-14 h-14 rounded-full shadow-2xl interactive btn-hover flex items-center justify-center text-2xl border-b-4 border-amber-200">👷</button>
        </div>

        <!-- Action Button -->
        <div class="absolute right-6 bottom-8 w-20 h-20 bg-white/10 rounded-full border-2 border-white/30 interactive btn-hover flex items-center justify-center text-white font-bold text-lg backdrop-blur-md" onclick="triggerJump()">
            JUMP
        </div>

        <!-- Joystick -->
        <div id="joystick-wrapper" class="interactive">
            <div id="joystick-knob"></div>
        </div>

        <!-- Modals -->
        <div id="modal-container" class="hidden absolute inset-0 bg-slate-950/60 backdrop-blur-md flex items-center justify-center p-6 interactive">
            <div id="modal-content" class="bg-white rounded-[32px] p-8 w-full max-w-xs text-center shadow-2xl border-t-8 border-indigo-500">
                <h2 id="modal-title" class="text-xl font-bold text-slate-800 mb-6 uppercase tracking-wider">Select Option</h2>
                <div id="modal-body" class="flex flex-col gap-3 mb-8"></div>
                <button onclick="closeModal()" class="bg-slate-100 text-slate-500 w-full py-4 rounded-2xl font-bold hover:bg-slate-200 transition-colors">Close</button>
            </div>
        </div>
    </div>

    <script>
        let scene, camera, renderer, player, clock, sun;
        let moveForward = 0, moveRight = 0;
        let canJump = true, velocity = new THREE.Vector3();
        let yaw = 0, pitch = 0.3; 
        let money = 1500, currentJob = "Citizen";
        let joystickActive = false;
        
        let gameTime = 8; 
        const streetLights = [];
        const npcs = []; 

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87CEEB);
            scene.fog = new THREE.Fog(0x87CEEB, 1, 1200);

            camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.1, 3000);
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setPixelRatio(window.devicePixelRatio);
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            document.body.appendChild(renderer.domElement);

            clock = new THREE.Clock();

            const ambientLight = new THREE.AmbientLight(0xffffff, 0.7); 
            scene.add(ambientLight);

            sun = new THREE.DirectionalLight(0xffffff, 1.3);
            sun.position.set(100, 200, 100);
            sun.castShadow = true;
            sun.shadow.mapSize.set(1024, 1024);
            sun.shadow.camera.left = -300;
            sun.shadow.camera.right = 300;
            sun.shadow.camera.top = 300;
            sun.shadow.camera.bottom = -300;
            scene.add(sun);

            createEnvironment();
            createPlayer();
            spawnNPCs(15);
            setupMobileControls();
            
            updateTime(0);
            animate();
        }

        function createEnvironment() {
            const grassGeo = new THREE.PlaneGeometry(4000, 4000);
            const grassMat = new THREE.MeshStandardMaterial({ color: 0x4d7c0f });
            const floor = new THREE.Mesh(grassGeo, grassMat);
            floor.rotation.x = -Math.PI / 2;
            floor.receiveShadow = true;
            scene.add(floor);

            createRealisticRoad(0, 0, 1500, 50);
            
            for (let i = -7; i <= 7; i++) {
                if (Math.abs(i) < 1) continue;
                spawnModernHouse(i * 120, 0, 80);
                spawnStreetLight(i * 120, 30);
            }

            for (let i = 0; i < 60; i++) {
                const rx = (Math.random() - 0.5) * 1500;
                const rz = (Math.random() - 0.5) * 1500;
                if (Math.abs(rz) > 70) spawnTree(rx, rz);
            }
        }

        function spawnNPCs(count) {
            for (let i = 0; i < count; i++) {
                const npc = new THREE.Group();
                const colors = [0xff5555, 0x55ff55, 0x5555ff, 0xffff55, 0xff55ff];
                const mat = new THREE.MeshStandardMaterial({ color: colors[Math.floor(Math.random()*colors.length)] });
                const body = new THREE.Mesh(new THREE.CapsuleGeometry(0.8, 1.8, 4, 8), mat);
                body.position.y = 1.7;
                npc.add(body);
                
                npc.position.set((Math.random()-0.5)*800, 0, (Math.random() > 0.5 ? 28 : -28));
                npc.userData = { 
                    speed: 0.1 + Math.random() * 0.1, 
                    direction: Math.random() > 0.5 ? 1 : -1 
                };
                
                scene.add(npc);
                npcs.push(npc);
            }
        }

        function createRealisticRoad(x, z, w, d) {
            const roadGroup = new THREE.Group();
            const asphalt = new THREE.Mesh(new THREE.BoxGeometry(w, 0.2, d), new THREE.MeshStandardMaterial({ color: 0x222222 }));
            asphalt.receiveShadow = true;
            roadGroup.add(asphalt);

            const side1 = new THREE.Mesh(new THREE.BoxGeometry(w, 0.6, 10), new THREE.MeshStandardMaterial({ color: 0xbbbbbb }));
            side1.position.z = d/2 + 5;
            roadGroup.add(side1);

            const side2 = side1.clone();
            side2.position.z = -(d/2 + 5);
            roadGroup.add(side2);

            roadGroup.position.set(x, 0.1, z);
            scene.add(roadGroup);
        }

        function spawnModernHouse(x, y, z) {
            const house = new THREE.Group();
            const bodyColor = new THREE.Color().setHSL(Math.random(), 0.3, 0.6);
            const base = new THREE.Mesh(new THREE.BoxGeometry(50, 25, 40), new THREE.MeshStandardMaterial({ color: bodyColor }));
            base.position.y = 12.5;
            base.castShadow = true;
            house.add(base);

            const roof = new THREE.Mesh(new THREE.BoxGeometry(55, 2, 45), new THREE.MeshStandardMaterial({ color: 0x333333 }));
            roof.position.y = 26;
            house.add(roof);

            house.position.set(x, y, z);
            scene.add(house);
        }

        function spawnStreetLight(x, z) {
            const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.6, 30), new THREE.MeshStandardMaterial({ color: 0x111111 }));
            pole.position.set(x, 15, z);
            const light = new THREE.PointLight(0xffaa44, 0, 100);
            light.position.set(0, 14, -2);
            pole.add(light);
            streetLights.push(light);
            scene.add(pole);
        }

        function spawnTree(x, z) {
            const tree = new THREE.Group();
            const trunk = new THREE.Mesh(new THREE.CylinderGeometry(1, 1.5, 15), new THREE.MeshStandardMaterial({ color: 0x4b3621 }));
            trunk.position.y = 7.5;
            tree.add(trunk);
            const leaves = new THREE.Mesh(new THREE.SphereGeometry(10, 8, 8), new THREE.MeshStandardMaterial({ color: 0x1b4d3e }));
            leaves.position.y = 20;
            tree.add(leaves);
            tree.position.set(x, 0, z);
            scene.add(tree);
        }

        function createPlayer() {
            player = new THREE.Group();
            const body = new THREE.Mesh(new THREE.CapsuleGeometry(1, 2, 4, 8), new THREE.MeshStandardMaterial({ color: 0x3b82f6 }));
            body.position.y = 2;
            body.castShadow = true;
            player.add(body);
            scene.add(player);
        }

        function setupMobileControls() {
            const wrapper = document.getElementById('joystick-wrapper');
            const knob = document.getElementById('joystick-knob');
            
            const handleMove = (e) => {
                const touch = e.touches ? e.touches[0] : e;
                const rect = wrapper.getBoundingClientRect();
                const cx = rect.left + rect.width / 2;
                const cy = rect.top + rect.height / 2;
                let dx = touch.clientX - cx, dy = touch.clientY - cy;
                const dist = Math.sqrt(dx*dx + dy*dy);
                const max = 40;
                if (dist > max) { dx = (dx/dist)*max; dy = (dy/dist)*max; }
                knob.style.transform = `translate(${dx}px, ${dy}px)`;
                moveForward = -dy / max; moveRight = dx / max;
            };

            wrapper.addEventListener('touchstart', (e) => { joystickActive = true; handleMove(e); }, {passive: false});
            document.addEventListener('touchmove', (e) => { if (joystickActive) { handleMove(e); e.preventDefault(); } }, {passive: false});
            document.addEventListener('touchend', () => { joystickActive = false; knob.style.transform = `translate(0,0)`; moveForward = 0; moveRight = 0; });

            let lastX = 0, lastY = 0, isRotating = false;
            document.addEventListener('touchstart', (e) => { 
                if(!joystickActive && e.target.tagName === 'CANVAS') { 
                    isRotating = true; 
                    lastX = e.touches[0].clientX; 
                    lastY = e.touches[0].clientY;
                } 
            });
            document.addEventListener('touchmove', (e) => { 
                if(isRotating) { 
                    yaw -= (e.touches[0].clientX - lastX) * 0.01; 
                    pitch += (e.touches[0].clientY - lastY) * 0.01;
                    pitch = Math.max(0.1, Math.min(Math.PI/2.1, pitch)); 
                    lastX = e.touches[0].clientX; 
                    lastY = e.touches[0].clientY;
                } 
            });
            document.addEventListener('touchend', () => isRotating = false);
        }

        function triggerJump() { if(canJump) { velocity.y = 12; canJump = false; } }

        function updateTime(delta) {
            gameTime += delta * 0.05;
            if (gameTime >= 24) gameTime = 0;

            const hour = Math.floor(gameTime);
            const mins = Math.floor((gameTime % 1) * 60);
            document.getElementById('time-text').innerText = `${hour.toString().padStart(2, '0')}:${mins.toString().padStart(2, '0')} ${hour >= 12 ? 'PM' : 'AM'}`;
            
            const sunAngle = (gameTime / 24) * Math.PI * 2 - Math.PI/2;
            sun.position.set(Math.cos(sunAngle) * 400, Math.sin(sunAngle) * 400, 100);
            
            const isNight = gameTime < 6 || gameTime > 19;
            const skyColor = new THREE.Color();
            if (isNight) {
                skyColor.setHSL(0.6, 0.5, 0.1);
                sun.intensity = 0.1;
            } else {
                skyColor.setHSL(0.55, 0.5, 0.4 + Math.max(0, Math.sin(sunAngle)) * 0.4);
                sun.intensity = 0.5 + Math.max(0, Math.sin(sunAngle)) * 1.0;
            }
            scene.background = skyColor;
            scene.fog.color.copy(skyColor);
            document.getElementById('time-emoji').innerText = isNight ? '🌙' : '☀️';
            streetLights.forEach(l => { l.intensity = isNight ? 1.5 : 0; });
        }

        function animate() {
            requestAnimationFrame(animate);
            const delta = clock.getDelta();
            updateTime(delta);

            npcs.forEach(npc => {
                npc.position.x += npc.userData.speed * npc.userData.direction;
                if (Math.abs(npc.position.x) > 700) npc.userData.direction *= -1;
                npc.rotation.y = npc.userData.direction > 0 ? Math.PI/2 : -Math.PI/2;
            });

            velocity.x -= velocity.x * 10.0 * delta;
            velocity.z -= velocity.z * 10.0 * delta;
            velocity.y -= 30.0 * delta;

            const speed = 30.0;
            if (Math.abs(moveForward) > 0.1 || Math.abs(moveRight) > 0.1) {
                const forward = new THREE.Vector3(0, 0, -1).applyAxisAngle(new THREE.Vector3(0,1,0), yaw);
                const right = new THREE.Vector3(1, 0, 0).applyAxisAngle(new THREE.Vector3(0,1,0), yaw);
                const moveDir = forward.multiplyScalar(moveForward).add(right.multiplyScalar(moveRight));
                player.position.add(moveDir.multiplyScalar(speed * delta));
                player.rotation.y = Math.atan2(moveDir.x, moveDir.z);
            }

            player.position.y += velocity.y * delta;
            if (player.position.y < 0) { velocity.y = 0; player.position.y = 0; canJump = true; }

            const dist = 20;
            const camX = player.position.x + dist * Math.sin(yaw) * Math.cos(pitch);
            const camY = player.position.y + 10 + dist * Math.sin(pitch);
            const camZ = player.position.z + dist * Math.cos(yaw) * Math.cos(pitch);
            
            camera.position.set(camX, camY, camZ);
            camera.lookAt(player.position.x, player.position.y + 3, player.position.z);

            renderer.render(scene, camera);
        }

        function addMoney(v, m) { money += v; document.getElementById('money-text').innerText = money; if(m) systemMessage(m); }
        function systemMessage(t) { const log = document.getElementById('chat-log'); const d = document.createElement('div'); d.innerHTML = `> ${t}`; log.appendChild(d); log.scrollTop = log.scrollHeight; }
        function sendChat() { const i = document.getElementById('chat-input'); if(!i.value) return; systemMessage(`You: ${i.value}`); i.value = ""; i.blur(); }
        function changeOutfit() { player.children[0].material.color.setHex(Math.random()*0xffffff); systemMessage("Style updated."); }
        function spawnVehicle() { const car = new THREE.Mesh(new THREE.BoxGeometry(6, 2, 10), new THREE.MeshStandardMaterial({color: 0x222222})); car.position.copy(player.position).add(new THREE.Vector3(10, 1, 0)); scene.add(car); systemMessage("Car spawned."); }
        function toggleHouse() { if(money >= 500) { addMoney(-500); spawnModernHouse(player.position.x+25, 0, player.position.z); systemMessage("Home purchased!"); } }
        function openJobMenu() {
            const b = document.getElementById('modal-body');
            b.innerHTML = `
                <button onclick="setJob('Paramedic')" class="p-4 bg-red-50 text-red-700 rounded-2xl font-bold">🚑 Paramedic</button>
                <button onclick="setJob('Architect')" class="p-4 bg-indigo-50 text-indigo-700 rounded-2xl font-bold">📏 Architect</button>
            `;
            showModal("Careers");
        }
        function setJob(j) { currentJob = j; document.getElementById('job-text').innerText = j; closeModal(); }
        function showModal(t) { document.getElementById('modal-title').innerText = t; document.getElementById('modal-container').classList.remove('hidden'); }
        function closeModal() { document.getElementById('modal-container').classList.add('hidden'); }

        window.onload = init;
    </script>
</body>
</html>





