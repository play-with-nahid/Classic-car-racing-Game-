<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-select=none">
    <title>Highway Racer 3D </title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; touch-action: manipulation; }
        body { background: #020617; font-family: 'Segoe UI', Roboto, sans-serif; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; width: 100vw; }
        
        /* সম্পূর্ণ রেসপন্সিভ মেইন গেম কন্টেইনার */
        #gameContainer {
            position: relative;
            width: 100%; height: 100%;
            max-width: 450px; max-height: 880px;
            background: #000; overflow: hidden;
            border-radius: 16px; box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.8);
        }

        #canvas3d { width: 100%; height: 100%; display: block; }
        .hud { position: absolute; z-index: 10; display: none; width: 100%; pointer-events: none; }
        
        /* টপ ড্যাশবোর্ড HUD */
        #topDashboard { top: 16px; left: 0; padding: 0 16px; display: none; justify-content: space-between; align-items: center; pointer-events: auto; }
        .hud-card { background: rgba(15, 23, 42, 0.85); backdrop-filter: blur(12px); border-radius: 14px; padding: 10px 16px; font-family: monospace; font-weight: 900; font-size: 15px; box-shadow: 0 4px 12px rgba(0,0,0,0.4); min-width: 110px; text-align: center; }
        #scoreCard { color: #22c55e; border: 2px solid rgba(34, 197, 94, 0.5); }
        #speedoCard { color: #06b6d4; border: 2px solid rgba(6, 182, 212, 0.5); }
        .btn-pause { background: #f59e0b; border: none; color: white; padding: 10px 16px; border-radius: 12px; font-weight: bold; cursor: pointer; pointer-events: auto; box-shadow: 0 4px 6px rgba(0,0,0,0.2); }

        /* স্ক্রিন ওভারলে সিস্টেম */
        .overlay-screen {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            z-index: 100; display: flex; flex-direction: column; justify-content: center; align-items: center; padding: 24px;
        }
        
        #mainMenu { background: linear-gradient(180deg, #0f172a 0%, #020617 100%); }
        #mainMenu h1 { color: #fff; font-size: 30px; text-align: center; margin-bottom: 40px; font-weight: 900; text-shadow: 0 0 20px rgba(56,189,248,0.6); letter-spacing: 1px; font-style: italic; }
        
        .menu-btn {
            width: 85%; max-width: 260px; background: linear-gradient(135deg, #1e293b, #0f172a); color: #f8fafc;
            border: 2px solid #334155; padding: 14px; font-size: 15px; font-weight: 700; margin-bottom: 14px;
            border-radius: 14px; cursor: pointer; text-align: center; box-shadow: 0 4px 10px rgba(0,0,0,0.3); transition: transform 0.1s;
        }
        .menu-btn:active { transform: scale(0.96); }
        #startRaceBtn { background: linear-gradient(135deg, #10b981, #059669); border-color: #34d399; text-shadow: 0 1px 3px rgba(0,0,0,0.4); }

        /* গ্যারেজ এবং সেটিংস */
        #settingsMenu { background: #090d16; justify-content: flex-start; padding-top: 35px; overflow-y: auto; }
        #settingsMenu h2 { color: #fff; margin-bottom: 20px; font-size: 22px; text-shadow: 0 0 10px rgba(56,189,248,0.3); }
        
        .section-box { width: 100%; background: rgba(30, 41, 59, 0.4); border: 2px solid #1e293b; border-radius: 16px; padding: 16px; margin-bottom: 16px; }
        .section-title { color: #94a3b8; font-size: 12px; font-weight: 800; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 12px; text-align: center; }
        
        .preview-box { width: 100%; height: 90px; background: #020617; border-radius: 12px; display: flex; flex-direction: column; justify-content: center; align-items: center; margin-bottom: 12px; border: 2px dashed #38bdf8; }
        #previewIcon { font-size: 36px; margin-bottom: 4px; }
        #previewText { color: #f1f5f9; font-size: 12px; font-weight: 700; text-align: center; padding: 0 6px; }

        .tab-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 6px; width: 100%; }
        .tab-btn { background: #1e293b; color: #94a3b8; border: 2px solid #334155; padding: 12px 2px; font-size: 11px; font-weight: 700; border-radius: 10px; cursor: pointer; text-align: center; }
        .tab-btn.active { background: #38bdf8; color: #0f172a; border-color: #0284c7; font-weight: 800; box-shadow: 0 0 12px rgba(56,189,248,0.4); }

        .track-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; width: 100%; }
        .track-btn { background: #1e293b; color: #cbd5e1; border: 2px solid #334155; padding: 14px 4px; font-size: 12px; font-weight: 700; border-radius: 10px; cursor: pointer; text-align: center; }
        .track-btn.active { background: #ec4899; color: white; border-color: #f472b6; box-shadow: 0 0 12px rgba(236,72,153,0.4); }

        /* পজ এবং গেম ওভার স্ক্রিন */
        .modal-menu { display: none; background: rgba(15, 23, 42, 0.96); backdrop-filter: blur(8px); }
        .modal-title { color: #fff; font-size: 26px; font-weight: 900; margin-bottom: 8px; letter-spacing: 1px; text-align: center; }
        .modal-score-lbl { color: #94a3b8; font-size: 15px; margin-bottom: 30px; }

        /* ড্রাইভ টাচ প্যানেল */
        #controlPanel { bottom: 30px; left: 0; padding: 0 35px; display: none; justify-content: space-between; pointer-events: none; }
        .ctrl-btn { width: 72px; height: 72px; background: rgba(30, 41, 59, 0.85); backdrop-filter: blur(8px); border: 3px solid #64748b; border-radius: 50%; color: white; font-size: 26px; display: flex; align-items: center; justify-content: center; cursor: pointer; pointer-events: auto; box-shadow: 0 10px 20px rgba(0,0,0,0.5); }
        .ctrl-btn:active { background: #38bdf8; color: #0f172a; border-color: #0284c7; }
    </style>
</head>
<body>

    <div id="gameContainer">
        <div id="canvas3d"></div>

        <div id="mainMenu" class="overlay-screen">
            <h1>HIGHWAY RACER 3D<br><span style="font-size:13px; color:#38bdf8; font-weight:700; letter-spacing:4px;">CINEMATIC SIMULATION</span></h1>
            <button id="startRaceBtn" class="menu-btn">START RACE 🏁</button>
            <button id="openGarageBtn" class="menu-btn">GARAGE & TRACKS ⚙</button>
        </div>

        <div id="pauseMenu" class="overlay-screen modal-menu">
            <div class="modal-title" style="text-shadow: 0 0 15px rgba(245,158,11,0.5);">RACE PAUSED</div>
            <div class="modal-score-lbl">Current Session Distance</div>
            <button id="resumeBtn" class="menu-btn" style="background: linear-gradient(135deg, #10b981, #059669); border-color:#34d399;">RESUME RACE ▶</button>
            <button id="pauseRestartBtn" class="menu-btn" style="background: linear-gradient(135deg, #3b82f6, #1d4ed8); border-color:#60a5fa;">RESTART ↻</button>
            <button id="pauseQuitBtn" class="menu-btn" style="background: linear-gradient(135deg, #ef4444, #b91c1c); border-color:#f87171;">QUIT TO MENU ✕</button>
        </div>

        <div id="gameOverMenu" class="overlay-screen modal-menu">
            <div class="modal-title" style="color:#ef4444; text-shadow: 0 0 15px rgba(239,68,68,0.5);">💥 RACE CRASHED</div>
            <div class="modal-score-lbl">Total Covered: <span id="finalScoreVal" style="color:#fff; font-weight:bold;">0.00</span> KM</div>
            <button id="govRestartBtn" class="menu-btn" style="background: linear-gradient(135deg, #10b981, #059669); border-color:#34d399;">TRY AGAIN ↻</button>
            <button id="govQuitBtn" class="menu-btn" style="background: linear-gradient(135deg, #3b82f6, #1d4ed8); border-color:#60a5fa;">MAIN MENU ✕</button>
        </div>

        <div id="settingsMenu" class="overlay-screen" style="display:none;">
            <h2>GARAGE & MAPS</h2>
            
            <div class="section-box">
                <div class="section-title">Select Supercar</div>
                <div class="preview-box">
                    <div id="previewIcon">🏎️</div>
                    <div id="previewText">Supra MK5</div>
                </div>
                <div class="tab-grid">
                    <button id="car-supra" class="tab-btn active">SUPRA MK5</button>
                    <button id="car-police" class="tab-btn">POLICE CAR</button>
                    <button id="car-rover" class="tab-btn">LAND ROVER</button>
                </div>
            </div>

            <div class="section-box">
                <div class="section-title">Select Environment Track</div>
                <div class="track-grid">
                    <button id="track-mountain" class="track-btn active">🏞️ Hill</button>
                    <button id="track-snow" class="track-btn">🏔️ Snow</button>
                    <button id="track-village" class="track-btn">🏡 Village</button>
                    <button id="track-arabic" class="track-btn">🕌 Arabic community</button>
                </div>
            </div>

            <button id="saveConfigBtn" class="menu-btn" style="background:linear-gradient(135deg, #38bdf8, #0284c7); color:#020617; border-color:#7dd3fc; width:100%; font-weight:800;">CONFIRM & SAVE</button>
        </div>

        <div id="topDashboard" class="hud">
            <div id="scoreCard" class="hud-card">DIS: <span id="distVal">0.00</span> KM</div>
            <button id="gamePauseBtn" class="btn-pause">PAUSE ⏸</button>
            <div id="speedoCard" class="hud-card"><span id="speedVal">0</span> KM/H</div>
        </div>

        <div id="controlPanel" class="hud">
            <div id="steerLeft" class="ctrl-btn">◀</div>
            <div id="steerRight" class="ctrl-btn">▶</div>
        </div>
    </div>

    <script>
        // কোড আর্কিটেকচার ইঞ্জিন ভ্যারিয়েবলস
        let scene, camera, renderer, container;
        let ground, asphaltRoad, dynamicSun;
        let playerCar = null, enemyCar = null;
        
        let roadLines = [];
        let sceneryObjects = [];
        
        // স্টেট মেকানিজম ম্যানেজার (Strict State Control)
        let isGameRunning = false;
        let isPaused = false;
        let isGameOver = false;

        let totalDistance = 0.00;
        let currentVelocity = 0.0;
        let targetX = 0.0;
        let currentX = 0.0;
        
        let enemyZ = -95;
        let enemyLane = 1;

        // ট্রাফিকের ৪টি সুনির্দিষ্ট লেনের কোঅর্ডিনেটস
        const laneXCoords = [-1.4, -0.45, 0.45, 1.4];
        
        // থিম অনুযায়ী এক্সক্লুসিভ লাইটিং এবং অ্যাটমোসফিয়ার প্যারামিটার্স
        const environmentThemes = {
            mountain: { groundColor: 0x1e88e5, skyColor: 0xbae6fd, fogColor: 0xbae6fd, fogDensity: 0.007 },
            snow: { groundColor: 0xe2e8f0, skyColor: 0x93c5fd, fogColor: 0x93c5fd, fogDensity: 0.013 },
            village: { groundColor: 0x2e7d32, skyColor: 0x7dd3fc, fogColor: 0x7dd3fc, fogDensity: 0.006 },
            arabic: { groundColor: 0xc29b68, skyColor: 0xffedd5, fogColor: 0xffedd5, fogDensity: 0.014 }
        };
        let currentTheme = "mountain";

        const vehicleCatalog = {
            supra: { title: "Supra MK5", icon: "🏎️", color: 0xe11d48, maxSpeed: 1.55, scaleDist: 50 },
            police: { title: "Police Car", icon: "🚓", color: 0x1d4ed8, maxSpeed: 1.25, scaleDist: 100 },
            rover: { title: "Land Rover", icon: "🚙", color: 0x047857, maxSpeed: 0.98, scaleDist: 170 }
        };
        let selectedCarType = "supra";

        function start3DEngine() {
            container = document.getElementById('gameContainer');

            scene = new THREE.Scene();
            applyEnvironmentAtmosphere();

            camera = new THREE.PerspectiveCamera(54, container.clientWidth / container.clientHeight, 0.1, 1000);
            camera.position.set(0, 3.7, 6.8);
            camera.lookAt(0, 0.8, -2.5);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(container.clientWidth, container.clientHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.outputEncoding = THREE.sRGBEncoding;
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            renderer.toneMappingExposure = 1.35;
            document.getElementById('canvas3d').appendChild(renderer.domElement);

            // ইউনিফর্ম লাইটিং এনভায়রনমেন্ট
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.85);
            scene.add(ambientLight);
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.75);
            directionalLight.position.set(15, 45, 20);
            scene.add(directionalLight);

            // মেইন ইনফিনিটি গ্রাউন্ড গ্রিড জ্যামিতি
            ground = new THREE.Mesh(new THREE.PlaneGeometry(250, 600), new THREE.MeshStandardMaterial({ roughness: 0.9 }));
            ground.rotation.x = -Math.PI / 2;
            ground.position.y = -0.02;
            scene.add(ground);

            // হাইওয়ে পিচ অ্যাসফাল্ট ট্র‍্যাক
            asphaltRoad = new THREE.Mesh(new THREE.PlaneGeometry(5.2, 600), new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.45 }));
            asphaltRoad.rotation.x = -Math.PI / 2;
            scene.add(asphaltRoad);

            // হাইওয়ে লেন মার্কার ড্যাশড স্ট্রিপস
            for (let i = 0; i < 28; i++) {
                const centerLine = new THREE.Mesh(new THREE.PlaneGeometry(0.14, 5.0), new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 }));
                centerLine.rotation.x = -Math.PI / 2;
                centerLine.position.set(0, 0.01, -i * 15.5);
                scene.add(centerLine);
                roadLines.push(centerLine);
            }

            // সিনেমাটিক স্কাই সান গ্লোব
            dynamicSun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 32, 32), new THREE.MeshBasicMaterial({ color: 0xfffbeb }));
            dynamicSun.position.set(-35, 14, -130);
            scene.add(dynamicSun);

            // ইঞ্জিন লোডিং ও বাইন্ডিং সিকোয়েন্স
            syncThemeColors();
            buildProceduralSceneryWorld();
            spawnPlayerCar();
            spawnEnemyCar();
            bindUserInterfaceEvents();

            // ব্রাউজার উইন্ডো রিসাইজ হ্যান্ডলিং লুপ (নিচের কালো অংশ ফিক্স করার মূল কোড)
            window.addEventListener('resize', () => {
                camera.aspect = container.clientWidth / container.clientHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(container.clientWidth, container.clientHeight);
            });

            // রেন্ডার ফ্রেম লুপ অ্যানিমেশন ফিজিক্স
            function animateLoop() {
                requestAnimationFrame(animateLoop);

                let meta = vehicleCatalog[selectedCarType];
                
                // 📈 কিলোমিটার প্রগ্রেসের সাথে রিয়েল-টাইম ভেলোসিটি ম্যাচিং সূত্র
                let progressFactor = Math.min(totalDistance / meta.scaleDist, 1.0);
                let allowedMaxVelocity = 0.28 + (meta.maxSpeed - 0.28) * progressFactor;

                if (isGameRunning && !isPaused && !isGameOver) {
                    if (currentVelocity < allowedMaxVelocity) currentVelocity += 0.004;
                    totalDistance += (currentVelocity * 0.007);
                    
                    document.getElementById('distVal').innerText = totalDistance.toFixed(2);
                    document.getElementById('speedVal').innerText = Math.floor(currentVelocity * 245);
                } else if (!isGameRunning && !isPaused && !isGameOver) {
                    currentVelocity = 0.035; // মেইন হোমপেজে থাকার সময় ব্যাকগ্রাউন্ড স্ক্রোলিং স্পিড
                } else {
                    currentVelocity = 0; // পজ বা গেম ওভার থাকলে স্পিড শূন্য হবে
                }

                // 🏎️ সুনিশ্চিত গাড়ির বাউন্ডারি লক (Strict Clamping - স্ক্রিনের বাইরে যাবে না)
                currentX += (targetX - currentX) * 0.16;
                currentX = Math.max(-1.70, Math.min(1.70, currentX)); 
                if (playerCar) playerCar.position.x = currentX;

                // রাস্তা ও ট্র‍্যাকের অবজেক্ট মোশন স্ক্রোলিং এফেক্ট
                roadLines.forEach(line => {
                    line.position.z += currentVelocity;
                    if (line.position.z > 15) line.position.z = -150;
                });

                sceneryObjects.forEach(obj => {
                    obj.position.z += currentVelocity;
                    if (obj.position.z > 15) {
                        obj.position.z = -165;
                        let side = (Math.random() > 0.5) ? 1 : -1;
                        obj.position.x = side * (5.8 + Math.random() * 16.0); // ৫.৮ অফসেট পজিশন যাতে কখনো রাস্তার ওপর না আসে
                    }
                });

                // কোলিশন ডিটেকশন ও রিয়েল-টাইম ক্র্যাশ সিকোয়েন্স
                if (isGameRunning && !isPaused && !isGameOver) {
                    enemyZ += currentVelocity * 1.35;
                    if (enemyZ > 15) {
                        enemyZ = -95;
                        enemyLane = Math.floor(Math.random() * laneXCoords.length);
                    }
                    if (enemyCar) enemyCar.position.set(laneXCoords[enemyLane], 0, enemyZ);

                    // বাউন্ডারি বক্স বাফার হিট ক্যালকুলেশন
                    if (Math.abs(currentX - laneXCoords[enemyLane]) < 0.65 && Math.abs(2.0 - enemyZ) < 1.6) {
                        executeGameOverSequence();
                    }
                }

                renderer.render(scene, camera);
            }
            animateLoop();
        }

        function applyEnvironmentAtmosphere() {
            let config = environmentThemes[currentTheme];
            scene.background = new THREE.Color(config.skyColor);
            scene.fog = new THREE.FogExp2(config.fogColor, config.fogDensity);
        }

        function syncThemeColors() {
            ground.material.color.setHex(environmentThemes[currentTheme].groundColor);
        }

        // 🏞️ ৪টি সম্পূর্ণ ইউনিক এনভায়রনমেন্টাল প্রোপস ও ম্যাপ জেনারেটর (No Copy-Paste Colors)
        function buildProceduralSceneryWorld() {
            sceneryObjects.forEach(obj => scene.remove(obj));
            sceneryObjects = [];

            dynamicSun.visible = (currentTheme === "mountain"); // সূর্য শুধুমাত্র মাউন্টেন ট্র‍্যাকেই দৃশ্যমান হবে

            // রাস্তার দুই পাশে প্রসিডিউরাল অবজেক্ট স্পনিং লুপ
            for (let i = 0; i < 42; i++) {
                let side = (i % 2 === 0) ? 1 : -1;
                
                // 🛑 রাস্তা থেকে মিনিমাম ৫.৫ ইউনিট বাইরে অবজেক্ট লক রাখা হয়েছে (রাস্তার ওপরে আসবে না)
                let calculatedX = side * (5.5 + Math.random() * 15.0); 
                let calculatedZ = -i * 5.0 - (Math.random() * 4.0);

                // 🚏 ট্রাফিক সাইন পোস্ট এবং হাইওয়ে সিগন্যাল বোর্ড রেন্ডারিং
                if (i % 7 === 0) {
                    const signPost = new THREE.Group();
                    const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.04, 0.05, 2.4), new THREE.MeshStandardMaterial({ color: 0x64748b }));
                    pole.position.y = 1.2; signPost.add(pole);
                    
                    const board = new THREE.Mesh(new THREE.BoxGeometry(0.7, 0.5, 0.08), new THREE.MeshStandardMaterial({ color: 0xef4444 }));
                    board.position.y = 2.1; signPost.add(board);
                    
                    signPost.position.set(side * 2.85, 0, calculatedZ);
                    scene.add(signPost);
                    sceneryObjects.push(signPost);
                }

                // থিম ভিত্তিক কাস্টম জ্যামিতিক অবজেক্ট আর্কিটেকচার মেথডস
                if (currentTheme === "mountain") {
                    // ★ ১. মাউন্টেন এরিয়া: মাল্টি-লেয়ার্ড জটিল পাথুরে পাহাড় (র্যান্ডম স্কেল)
                    const mountainRockGroup = new THREE.Group();
                    const mtnBase = new THREE.Mesh(new THREE.ConeGeometry(5 + Math.random()*2, 9 + Math.random()*4, 5), new THREE.MeshStandardMaterial({ color: 0x4e3629, roughness: 0.95 }));
                    mountainRockGroup.add(mtnBase);
                    
                    const subRock = new THREE.Mesh(new THREE.DodecahedronGeometry(2.0), new THREE.MeshStandardMaterial({ color: 0x3d2b20 }));
                    subRock.position.set(2, -1, -1); mountainRockGroup.add(subRock);
                    
                    mountainRockGroup.position.set(calculatedX * 1.4, 3.5, calculatedZ);
                    scene.add(mountainRockGroup);
                    sceneryObjects.push(mountainRockGroup);

                } else if (currentTheme === "snow") {
                    // ★ ২. বরফের এলাকা: পিরামিড মুক্ত ন্যাচারাল অসম স্নো-পাহাড় (আইকোসাহেড্রন শেপ)
                    const snowPeak = new THREE.Mesh(new THREE.IcosahedronGeometry(3.5 + Math.random()*2, 1), new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.4 }));
                    snowPeak.scale.set(1, 2.2, 1); // Y অক্ষ বরাবর লম্বা করে ন্যাচারাল খাড়া পাহাড়ের লোক দেওয়া হলো
                    snowPeak.position.set(calculatedX * 1.3, 2.5, calculatedZ);
                    
                    // স্নো ট্র‍্যাকের জন্য ক্রিসমাস ঝাউ গাছ
                    const winterTree = new THREE.Group();
                    const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.1, 1.2), new THREE.MeshStandardMaterial({ color: 0x27160c }));
                    trunk.position.y = 0.6; winterTree.add(trunk);
                    const leaves = new THREE.Mesh(new THREE.ConeGeometry(0.8, 1.8, 6), new THREE.MeshStandardMaterial({ color: 0x064e3b }));
                    leaves.position.y = 1.6; winterTree.add(leaves);
                    winterTree.position.set(side * (3.5 + Math.random()*2), 0, calculatedZ + 1);

                    scene.add(snowPeak, winterTree);
                    sceneryObjects.push(snowPeak, winterTree);

                } else if (currentTheme === "village") {
                    // ★ ৩. গ্রামীণ এলাকা: মডার্ন কান্ট্রি সাইড কটেজ ভিলা এবং ঘন গাছের অরণ্য
                    if (i % 2 === 0) {
                        const luxuryVilla = new THREE.Group();
                        const foundation = new THREE.Mesh(new THREE.BoxGeometry(2.0, 1.4, 2.0), new THREE.MeshStandardMaterial({ color: 0xf1f5f9, roughness: 0.5 }));
                        foundation.position.y = 0.7; luxuryVilla.add(foundation);
                        
                        const roofSlanted = new THREE.Mesh(new THREE.ConeGeometry(1.6, 1.1, 4), new THREE.MeshStandardMaterial({ color: 0x991b1b, roughness: 0.6 }));
                        roofSlanted.position.y = 1.9; roofSlanted.rotation.y = Math.PI/4; luxuryVilla.add(roofSlanted);
                        
                        luxuryVilla.position.set(calculatedX, 0, calculatedZ);
                        scene.add(luxuryVilla);
                        sceneryObjects.push(luxuryVilla);
                    } else {
                        const villageForestTree = new THREE.Group();
                        const coreTrunk = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.15, 1.5), new THREE.MeshStandardMaterial({ color: 0x451a03 }));
                        coreTrunk.position.y = 0.75; villageForestTree.add(coreTrunk);
                        
                        const greenFoliage = new THREE.Mesh(new THREE.SphereGeometry(0.85, 10, 10), new THREE.MeshStandardMaterial({ color: 0x15803d, roughness: 0.7 }));
                        greenFoliage.position.y = 1.9; villageForestTree.add(greenFoliage);
                        
                        villageForestTree.position.set(side * (3.6 + Math.random()*2), 0, calculatedZ);
                        scene.add(villageForestTree);
                        sceneryObjects.push(villageForestTree);
                    }

                } else if (currentTheme === "arabic") {
                    // ★ ৪. এরাবিক এলাকা: সূক্ষ্ম কারুকার্যময় হোয়াইট মার্বেল মসজিদ ও স্যান্ড-গোল্ড প্যালেস
                    const premiumMosque = new THREE.Group();
                    const mainHall = new THREE.Mesh(new THREE.BoxGeometry(3.0, 2.8, 3.0), new THREE.MeshStandardMaterial({ color: 0xfffbeb, roughness: 0.4 }));
                    mainHall.position.y = 1.4; premiumMosque.add(mainHall);
                    
                    const traditionalDome = new THREE.Mesh(new THREE.SphereGeometry(1.05, 20, 20), new THREE.MeshStandardMaterial({ color: 0xd97706, metalness: 0.5, roughness: 0.2 }));
                    traditionalDome.position.y = 2.8; traditionalDome.scale.set(1, 1.3, 1); premiumMosque.add(traditionalDome);
                    
                    const tallMinaret = new THREE.Mesh(new THREE.CylinderGeometry(0.14, 0.22, 4.8), new THREE.MeshStandardMaterial({ color: 0xfef3c7 }));
                    tallMinaret.position.set(1.8 * side, 2.4, 0); premiumMosque.add(tallMinaret);
                    
                    premiumMosque.position.set(side * (5.8 + Math.random()*2), 0, calculatedZ);
                    scene.add(premiumMosque);
                    sceneryObjects.push(premiumMosque);
                }
            }
        }

        // 🏎️ হাই-ফাইডেলিটি ৩ডি সুপারকার মেকার মডিউল
        function craftAdvanced3DVehicle(colorCode, architectureType) {
            const compositeCarGroup = new THREE.Group();
            const glossBodyMat = new THREE.MeshStandardMaterial({ color: colorCode, metalness: 0.85, roughness: 0.12 });
            const darkGlassMat = new THREE.MeshStandardMaterial({ color: 0x090d16, roughness: 0.0, metalness: 0.95 });
            const rubberTyreMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.9 });

            if (architectureType === 'supra') {
                const aeroChassis = new THREE.Mesh(new THREE.BoxGeometry(0.82, 0.22, 1.72), glossBodyMat); aeroChassis.position.y = 0.22; compositeCarGroup.add(aeroChassis);
                const cockpit = new THREE.Mesh(new THREE.SphereGeometry(0.33, 16, 16), darkGlassMat); cockpit.scale.set(0.85, 0.55, 1.5); cockpit.position.set(0, 0.32, 0); compositeCarGroup.add(cockpit);
                const racingSpoiler = new THREE.Mesh(new THREE.BoxGeometry(0.86, 0.04, 0.16), glossBodyMat); racingSpoiler.position.set(0, 0.39, 0.76); compositeCarGroup.add(racingSpoiler);
            } else if (architectureType === 'police') {
                const interceptorChassis = new THREE.Mesh(new THREE.BoxGeometry(0.86, 0.28, 1.76), glossBodyMat); interceptorChassis.position.y = 0.25; compositeCarGroup.add(interceptorChassis);
                const policeCabin = new THREE.Mesh(new THREE.BoxGeometry(0.74, 0.28, 0.88), darkGlassMat); policeCabin.position.set(0, 0.46, -0.05); compositeCarGroup.add(policeCabin);
                const flasherLightbar = new THREE.Mesh(new THREE.BoxGeometry(0.32, 0.08, 0.12), new THREE.MeshBasicMaterial({ color: 0xef4444 })); flasherLightbar.position.set(0, 0.63, -0.05); compositeCarGroup.add(flasherLightbar);
            } else if (architectureType === 'rover') {
                const heavyChassis = new THREE.Mesh(new THREE.BoxGeometry(0.92, 0.54, 1.82), glossBodyMat); heavyChassis.position.y = 0.42; compositeCarGroup.add(heavyChassis);
                const suvCabin = new THREE.Mesh(new THREE.BoxGeometry(0.84, 0.35, 1.05), darkGlassMat); suvCabin.position.set(0, 0.80, 0.05); compositeCarGroup.add(suvCabin);
            }

            // চাকা সংযোজন লুপ
            const wheelGeometry = new THREE.CylinderGeometry(0.23, 0.23, 0.15, 18);
            const positionsArray = [[-0.48, 0.22, 0.48], [0.48, 0.22, 0.48], [-0.48, 0.22, -0.48], [0.48, 0.22, -0.48]];
            positionsArray.forEach(p => {
                const wMesh = new THREE.Mesh(wheelGeometry, rubberTyreMat); wMesh.rotation.z = Math.PI / 2; wMesh.position.set(p[0], p[1], p[2]);
                compositeCarGroup.add(wMesh);
            });

            return compositeCarGroup;
        }

        function spawnPlayerCar() {
            if (playerCar) scene.remove(playerCar);
            let meta = vehicleCatalog[selectedCarType];
            playerCar = craftAdvanced3DVehicle(meta.color, selectedCarType);
            playerCar.position.set(currentX, 0, 2.0);
            scene.add(playerCar);
        }

        function spawnEnemyCar() {
            if (enemyCar) scene.remove(enemyCar);
            enemyCar = craftAdvanced3DVehicle(0xdc2626, 'supra'); // এনিমি কার অবজেক্ট লাল রঙে স্পন হবে
            enemyCar.position.set(laneXCoords[enemyLane], 0, enemyZ);
            scene.add(enemyCar);
        }

        function refreshSettingsPanelMetaData() {
            let meta = vehicleCatalog[selectedCarType];
            document.getElementById('previewIcon').innerHTML = meta.icon;
            document.getElementById('previewText').innerHTML = `${meta.title} (ধাপে ধাপে সর্বোচ্চ গতি উঠবে ${meta.scaleDist} KM এ)`;
        }

        // 🎯 ইন্টারঅ্যাক্টিভ কোর ইউজার ইন্টারফেস ইভেন্ট হ্যান্ডলিং লজিকস (Strict Menu Isolation)
        function bindUserInterfaceEvents() {
            document.getElementById('startRaceBtn').onclick = function() {
                document.getElementById('mainMenu').style.display = 'none';
                toggleHUDSplashScreen(true);
                clearAndResetAllStates();
                isGameRunning = true;
            };

            document.getElementById('openGarageBtn').onclick = function() {
                document.getElementById('mainMenu').style.display = 'none';
                document.getElementById('settingsMenu').style.display = 'flex';
                refreshSettingsPanelMetaData();
            };

            document.getElementById('gamePauseBtn').onclick = function() {
                if (!isGameRunning || isGameOver) return; // গেম ওভার থাকলে পজ বাটন কাজ করবে না
                isPaused = true;
                toggleHUDSplashScreen(false);
                document.getElementById('pauseMenu').style.display = 'flex';
            };

            // পজ মেনুর বাটন অ্যাকশনস
            document.getElementById('resumeBtn').onclick = function() {
                if (isGameOver) return; // ডিফেন্সিভ প্রটেকশন চেক
                document.getElementById('pauseMenu').style.display = 'none';
                toggleHUDSplashScreen(true);
                isPaused = false;
            };

            document.getElementById('pauseRestartBtn').onclick = function() {
                document.getElementById('pauseMenu').style.display = 'none';
                toggleHUDSplashScreen(true);
                clearAndResetAllStates();
                isPaused = false;
                isGameRunning = true;
            };

            document.getElementById('pauseQuitBtn').onclick = function() {
                document.getElementById('pauseMenu').style.display = 'none';
                toggleHUDSplashScreen(false);
                document.getElementById('mainMenu').style.display = 'flex';
                clearAndResetAllStates();
            };

            // গেম ওভার মেনুর সুনির্দিষ্ট বাটন অ্যাকশনস (লজিক সেপারেশন)
            document.getElementById('govRestartBtn').onclick = function() {
                document.getElementById('gameOverMenu').style.display = 'none';
                toggleHUDSplashScreen(true);
                clearAndResetAllStates();
                isGameRunning = true;
            };

            document.getElementById('govQuitBtn').onclick = function() {
                document.getElementById('gameOverMenu').style.display = 'none';
                toggleHUDSplashScreen(false);
                document.getElementById('mainMenu').style.display = 'flex';
                clearAndResetAllStates();
            };

            // ক্যারেক্টার ও কার সুইচিং ইভেন্ট লিসেনারস
            const vehicleIds = ['supra', 'police', 'rover'];
            vehicleIds.forEach(id => {
                document.getElementById(`car-${id}`).onclick = function() {
                    vehicleIds.forEach(v => document.getElementById(`car-${v}`).classList.remove('active'));
                    document.getElementById(`car-${id}`).classList.add('active');
                    selectedCarType = id;
                    spawnPlayerCar();
                    refreshSettingsPanelMetaData();
                    clearAndResetAllStates(); // কার চেঞ্জ করলে স্টেট রিসেট হবে
                };
            });

            // ট্র্যাক ও এনভায়রনমেন্ট ল্যান্ডস্কেপ সুইচ লিসেনারস
            const trackIds = ['mountain', 'snow', 'village', 'arabic'];
            trackIds.forEach(tId => {
                document.getElementById(`track-${tId}`).onclick = function() {
                    trackIds.forEach(t => document.getElementById(`track-${t}`).classList.remove('active'));
                    document.getElementById(`track-${tId}`).classList.add('active');
                    currentTheme = tId;
                    
                    applyEnvironmentAtmosphere();
                    syncThemeColors();
                    buildProceduralSceneryWorld();
                    clearAndResetAllStates(); // ট্র্যাক চেঞ্জ করলেও সেশন নতুন করে শুরু হবে
                };
            });

            document.getElementById('saveConfigBtn').onclick = function() {
                document.getElementById('settingsMenu').style.display = 'none';
                document.getElementById('mainMenu').style.display = 'flex';
            };

            // স্টিয়ারিং টাচ মোড লজিক
            document.getElementById('steerLeft').onclick = function() { targetX -= 0.92; };
            document.getElementById('steerRight').onclick = function() { targetX += 0.92; };
        }

        function executeGameOverSequence() {
            isGameOver = true;
            isGameRunning = false;
            toggleHUDSplashScreen(false);
            document.getElementById('finalScoreVal').innerText = totalDistance.toFixed(2);
            document.getElementById('gameOverMenu').style.display = 'flex';
        }

        function clearAndResetAllStates() {
            isGameOver = false;
            isPaused = false;
            isGameRunning = false;
            totalDistance = 0.00; targetX = 0.0; currentX = 0.0; enemyZ = -95; currentVelocity = 0.05;
            document.getElementById('distVal').innerText = "0.00";
            if (playerCar) playerCar.position.set(0, 0, 2.0);
            spawnEnemyCar();
        }

        function toggleHUDSplashScreen(show) {
            let displayMode = show ? 'flex' : 'none';
            document.getElementById('topDashboard').style.display = displayMode;
            document.getElementById('controlPanel').style.display = displayMode;
        }

        window.onload = start3DEngine;
    </script>
</body>
</html>
