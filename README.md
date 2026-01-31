<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Knocks Creative Studios - Live Intro</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #000; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        canvas { display: block; }
        
        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }

        #brand-name {
            font-size: 4rem;
            font-weight: 800;
            text-transform: uppercase;
            letter-spacing: 0.5rem;
            color: #fff;
            text-shadow: 0 0 20px rgba(255, 255, 255, 0.5);
            opacity: 0;
            transform: scale(1.2);
            transition: all 1s cubic-bezier(0.16, 1, 0.3, 1);
            text-align: center;
            mix-blend-mode: screen;
        }

        #brand-subtitle {
            font-size: 1.2rem;
            color: #aaa;
            letter-spacing: 0.8rem;
            margin-top: 10px;
            opacity: 0;
            transform: translateY(20px);
            transition: all 1.5s ease-out;
        }

        #start-btn {
            pointer-events: auto;
            background: transparent;
            color: #fff;
            border: 2px solid #fff;
            padding: 15px 40px;
            font-size: 1.2rem;
            letter-spacing: 2px;
            cursor: pointer;
            transition: all 0.3s;
            text-transform: uppercase;
            background: rgba(0,0,0,0.5);
            backdrop-filter: blur(5px);
        }

        #start-btn:hover {
            background: #fff;
            color: #000;
            box-shadow: 0 0 30px rgba(255,255,255,0.5);
        }

        .hidden {
            display: none !important;
        }

        .visible {
            opacity: 1 !important;
            transform: scale(1) translateY(0) !important;
        }

        /* Glitch effect classes could go here */
    </style>
    <!-- Import Three.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="ui-layer">
        <button id="start-btn">INITIALIZE LAUNCH SEQUENCE</button>
        <div id="brand-container" style="display:flex; flex-direction:column; align-items:center;">
            <div id="brand-name">KNOCKS</div>
            <div id="brand-subtitle">CREATIVE STUDIOS</div>
        </div>
    </div>

    <script>
        // --- AUDIO SYSTEM (Synthesized SFX) ---
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        function playIntroSound() {
            if (audioCtx.state === 'suspended') audioCtx.resume();

            const t = audioCtx.currentTime;

            // 1. The Build Up (Low drone)
            const osc1 = audioCtx.createOscillator();
            const gain1 = audioCtx.createGain();
            osc1.type = 'sawtooth';
            osc1.frequency.setValueAtTime(50, t);
            osc1.frequency.exponentialRampToValueAtTime(100, t + 2);
            gain1.gain.setValueAtTime(0, t);
            gain1.gain.linearRampToValueAtTime(0.3, t + 1);
            gain1.gain.exponentialRampToValueAtTime(0.001, t + 2.5);
            
            // Filter for sweep
            const filter = audioCtx.createBiquadFilter();
            filter.type = 'lowpass';
            filter.frequency.setValueAtTime(100, t);
            filter.frequency.exponentialRampToValueAtTime(5000, t + 2);

            osc1.connect(filter);
            filter.connect(gain1);
            gain1.connect(audioCtx.destination);
            osc1.start(t);
            osc1.stop(t + 3);

            // 2. The "KNOCK" (Impact) at 2.2s
            const impactTime = t + 2.1;
            const osc2 = audioCtx.createOscillator();
            const gain2 = audioCtx.createGain();
            osc2.type = 'sine';
            osc2.frequency.setValueAtTime(150, impactTime);
            osc2.frequency.exponentialRampToValueAtTime(40, impactTime + 0.5);
            gain2.gain.setValueAtTime(0, impactTime);
            gain2.gain.linearRampToValueAtTime(1, impactTime + 0.05);
            gain2.gain.exponentialRampToValueAtTime(0.001, impactTime + 1);
            
            osc2.connect(gain2);
            gain2.connect(audioCtx.destination);
            osc2.start(impactTime);
            osc2.stop(impactTime + 1.5);

            // 3. The Digital Shimmer (High sparkly bits)
            const osc3 = audioCtx.createOscillator();
            const gain3 = audioCtx.createGain();
            osc3.type = 'triangle';
            osc3.frequency.setValueAtTime(800, impactTime);
            osc3.frequency.linearRampToValueAtTime(1200, impactTime + 3);
            gain3.gain.setValueAtTime(0, impactTime);
            gain3.gain.linearRampToValueAtTime(0.1, impactTime + 0.5);
            gain3.gain.linearRampToValueAtTime(0, impactTime + 4);

            // Stereo panner for shimmer
            const panner = audioCtx.createStereoPanner();
            panner.pan.setValueAtTime(-1, impactTime);
            panner.pan.linearRampToValueAtTime(1, impactTime + 4);

            osc3.connect(gain3);
            gain3.connect(panner);
            panner.connect(audioCtx.destination);
            osc3.start(impactTime);
            osc3.stop(impactTime + 4);
        }

        // --- VISUAL SYSTEM (Three.js) ---
        
        // Scene Setup
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x050510, 0.04);
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

        // Lighting
        const ambientLight = new THREE.AmbientLight(0x404040, 2);
        scene.add(ambientLight);

        const pointLight1 = new THREE.PointLight(0xffaa00, 2, 50);
        pointLight1.position.set(5, 5, 5);
        scene.add(pointLight1);

        const pointLight2 = new THREE.PointLight(0x0088ff, 2, 50);
        pointLight2.position.set(-5, -5, 5);
        scene.add(pointLight2);

        const pointLight3 = new THREE.PointLight(0xff00aa, 2, 50);
        pointLight3.position.set(0, 5, -5);
        scene.add(pointLight3);

        // Objects: The "Knocks" Cubes
        const group = new THREE.Group();
        scene.add(group);

        const cubes = [];
        const cubeCount = 40;
        const colors = [0xffaa00, 0x00aaff, 0xff0088, 0xffee00, 0xdddddd]; // Brand Palette based on previous images

        const geometry = new THREE.BoxGeometry(0.8, 0.8, 0.8);

        for (let i = 0; i < cubeCount; i++) {
            const material = new THREE.MeshStandardMaterial({
                color: colors[Math.floor(Math.random() * colors.length)],
                roughness: 0.2,
                metalness: 0.8,
                emissive: 0x000000
            });
            const cube = new THREE.Mesh(geometry, material);
            
            // Random start positions (Exploded view)
            cube.position.x = (Math.random() - 0.5) * 30;
            cube.position.y = (Math.random() - 0.5) * 30;
            cube.position.z = (Math.random() - 0.5) * 30;
            
            // Random rotation
            cube.rotation.x = Math.random() * Math.PI;
            cube.rotation.y = Math.random() * Math.PI;

            // Store target destination (The clustered cube shape)
            // We want them to form a rough sphere/cluster in center
            const theta = Math.random() * Math.PI * 2;
            const phi = Math.acos(2 * Math.random() - 1);
            const radius = 2.5 + Math.random() * 1.5;

            cube.userData = {
                startPos: cube.position.clone(),
                targetPos: new THREE.Vector3(
                    radius * Math.sin(phi) * Math.cos(theta),
                    radius * Math.sin(phi) * Math.sin(theta),
                    radius * Math.cos(phi)
                ),
                rotationSpeed: {
                    x: (Math.random() - 0.5) * 0.02,
                    y: (Math.random() - 0.5) * 0.02
                },
                delay: Math.random() * 0.5 // staggered entry
            };

            cubes.push(cube);
            group.add(cube);
        }

        // Connecting Lines (The network effect)
        const lineMaterial = new THREE.LineBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.2 });
        const lineGeometry = new THREE.BufferGeometry();
        // Create initial empty positions
        const linePositions = new Float32Array(cubeCount * cubeCount * 3);
        lineGeometry.setAttribute('position', new THREE.BufferAttribute(linePositions, 3));
        const lines = new THREE.LineSegments(lineGeometry, lineMaterial);
        group.add(lines);

        // Particles
        const particleGeo = new THREE.BufferGeometry();
        const particleCount = 200;
        const pPos = new Float32Array(particleCount * 3);
        for(let i=0; i<particleCount*3; i++) {
            pPos[i] = (Math.random() - 0.5) * 20;
        }
        particleGeo.setAttribute('position', new THREE.BufferAttribute(pPos, 3));
        const particleMat = new THREE.PointsMaterial({ color: 0xffffff, size: 0.05, transparent: true, opacity: 0.6 });
        const particles = new THREE.Points(particleGeo, particleMat);
        scene.add(particles);


        // Animation Variables
        let time = 0;
        let animationState = 'waiting'; // waiting, intro, active
        let startTime = 0;

        camera.position.z = 15;

        // Interaction
        let mouseX = 0;
        let mouseY = 0;
        document.addEventListener('mousemove', (e) => {
            mouseX = (e.clientX - window.innerWidth / 2) * 0.001;
            mouseY = (e.clientY - window.innerHeight / 2) * 0.001;
        });

        // Start Sequence
        document.getElementById('start-btn').addEventListener('click', () => {
            document.getElementById('start-btn').classList.add('hidden');
            playIntroSound();
            animationState = 'intro';
            startTime = performance.now() * 0.001;
        });

        function animate() {
            requestAnimationFrame(animate);

            const now = performance.now() * 0.001;
            const elapsed = now - startTime;

            // Rotation of entire group
            group.rotation.y += 0.005;
            group.rotation.z += 0.002;

            // Mouse interaction
            if (animationState === 'active') {
                group.rotation.y += mouseX * 2;
                group.rotation.x += mouseY * 2;
            }

            // Particle float
            particles.rotation.y -= 0.001;

            if (animationState === 'intro') {
                // Intro Animation Logic
                // 0s - 2s: Suck in
                // 2.1s: Impact/Stop
                
                let progress = Math.min(elapsed / 2.0, 1);
                // Easing function for suction effect
                let ease = progress * progress * (3 - 2 * progress); 

                cubes.forEach(cube => {
                    if (elapsed > cube.userData.delay) {
                        // Interpolate from startPos to targetPos
                        cube.position.lerpVectors(cube.userData.startPos, cube.userData.targetPos, ease);
                        
                        // Spin fast during entry
                        cube.rotation.x += 0.1;
                        cube.rotation.y += 0.1;
                    }
                });

                // Camera zoom
                if (elapsed < 2.0) {
                   camera.position.z = 15 - (elapsed * 3); // Zoom from 15 to ~9
                }

                // The Impact Trigger
                if (elapsed > 2.1 && !document.getElementById('brand-name').classList.contains('visible')) {
                    // Flash background
                    scene.background = new THREE.Color(0x222233);
                    setTimeout(() => { scene.background = new THREE.Color(0x000000); }, 100);
                    
                    // Show Text
                    document.getElementById('brand-name').classList.add('visible');
                    document.getElementById('brand-subtitle').classList.add('visible');
                    
                    animationState = 'active';
                }
            } else if (animationState === 'active') {
                // Idle Animation (Breathing)
                const pulse = Math.sin(now * 2) * 0.1 + 1;
                
                cubes.forEach((cube, i) => {
                    cube.rotation.x += cube.userData.rotationSpeed.x;
                    cube.rotation.y += cube.userData.rotationSpeed.y;
                    
                    // Subtle breathing effect
                    cube.position.copy(cube.userData.targetPos).multiplyScalar(pulse);
                });

                // Draw lines between close cubes
                updateLines();
            }

            renderer.render(scene, camera);
        }

        function updateLines() {
            let lineIdx = 0;
            const posAttr = lines.geometry.attributes.position;
            const connectionDist = 3.5;

            // Only connect some cubes to avoid performance hit
            for (let i = 0; i < cubes.length; i++) {
                for (let j = i + 1; j < cubes.length; j++) {
                    const d = cubes[i].position.distanceTo(cubes[j].position);
                    if (d < connectionDist) {
                        // Add line
                        posAttr.setXYZ(lineIdx++, cubes[i].position.x, cubes[i].position.y, cubes[i].position.z);
                        posAttr.setXYZ(lineIdx++, cubes[j].position.x, cubes[j].position.y, cubes[j].position.z);
                    }
                }
            }
            lines.geometry.setDrawRange(0, lineIdx);
            lines.geometry.attributes.position.needsUpdate = true;
        }

        // Resize handler
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>

# G-QCX3G9KSPC