<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mini-Dyson Mejia | EO=0.01</title>
    <script src="https://cdn.jsdelivr.net/npm/three@0.167/build/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.167/examples/js/controls/OrbitControls.js"></script>
    
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            margin: 0;
            padding: 0;
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
            background: linear-gradient(135deg, #0a0e27 0%, #1a1a2e 50%, #16213e 100%);
            height: 100vh;
            overflow: hidden;
            color: #ffffff;
        }
        #canvas-container { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }
        #header { padding: 32px 40px; background: linear-gradient(180deg, rgba(10,14,39,0.95) 0%, transparent 100%); }
        #title { font-size: 42px; font-weight: 700; color: #00d4ff; text-shadow: 0 0 25px rgba(0,212,255,0.6); }
        #subtitle { font-size: 18px; color: #ffa500; }
        #controls-panel {
            position: absolute; bottom: 40px; left: 50%; transform: translateX(-50%);
            background: rgba(26,26,46,0.92); border: 2px solid rgba(0,212,255,0.4);
            border-radius: 16px; padding: 20px 32px; pointer-events: auto;
        }
        .control-button {
            padding: 12px 28px; border: none; border-radius: 10px;
            font-weight: 600; cursor: pointer; margin: 0 8px;
        }
    </style>
</head>
<body>
    <div id="canvas-container"></div>

    <div id="header">
        <h1 id="title">MINI-DYSON MEJÍA</h1>
        <p id="subtitle">EO = 0.01 — Estabilidad Orbital</p>
    </div>

    <div id="controls-panel">
        <button id="sound-button" class="control-button">Activar Sonido 699 Hz</button>
        <button id="reset-button" class="control-button">Reset</button>
    </div>

    <script>
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.getElementById('canvas-container').appendChild(renderer.domElement);

        // Torus
        const torus = new THREE.Mesh(new THREE.TorusGeometry(3, 0.8, 32, 100), 
            new THREE.MeshPhongMaterial({color: 0x44aaff, transparent:true, opacity:0.35, emissive:0x2277cc}));
        scene.add(torus);

        // Flor de la Vida
        for (let i = 0; i < 19; i++) {
            const angle = i * Math.PI * 2 / 19;
            const circle = new THREE.Mesh(new THREE.CircleGeometry(0.4, 32), 
                new THREE.MeshBasicMaterial({color: 0xffaa00, transparent:true, opacity:0.6}));
            circle.position.set(Math.cos(angle)*2.5, 0, Math.sin(angle)*2.5);
            circle.rotation.x = Math.PI / 2;
            scene.add(circle);
        }

        camera.position.z = 12;
        const controls = new THREE.OrbitControls(camera, renderer.domElement);

        let time = 0;
        function animate() {
            requestAnimationFrame(animate);
            time += 0.016;
            torus.rotation.y = time * 0.5;
            renderer.render(scene, camera);
        }
        animate();

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>