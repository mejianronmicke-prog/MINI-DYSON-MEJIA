<!doctype html>
<html lang="en">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mini-Dyson Cosmic Visualization</title>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.167/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.167/examples/js/controls/OrbitControls.js"></script>
  <style>
    body {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
      background: linear-gradient(135deg, #0a0e27 0%, #1a1a2e 50%, #16213e 100%);
      height: 100%;
      overflow: hidden;
      color: #ffffff;
    }

    html {
      height: 100%;
    }

    #canvas-container {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 1;
    }

    #ui-overlay {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 2;
      pointer-events: none;
    }

    #header {
      padding: 32px 40px;
      background: linear-gradient(180deg, rgba(10, 14, 39, 0.95) 0%, rgba(10, 14, 39, 0) 100%);
    }

    #title {
      font-size: 36px;
      font-weight: 700;
      margin: 0 0 8px 0;
      color: #00d4ff;
      text-shadow: 0 0 20px rgba(0, 212, 255, 0.5);
      letter-spacing: 1px;
    }

    #subtitle {
      font-size: 18px;
      font-weight: 400;
      margin: 0;
      color: #ffa500;
      text-shadow: 0 0 15px rgba(255, 165, 0, 0.4);
    }

    #controls-panel {
      position: absolute;
      bottom: 32px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(26, 26, 46, 0.9);
      backdrop-filter: blur(10px);
      border: 2px solid rgba(0, 212, 255, 0.3);
      border-radius: 16px;
      padding: 24px 32px;
      pointer-events: auto;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
    }

    #instruction-text {
      font-size: 14px;
      color: #b8c5d6;
      margin: 0 0 16px 0;
      text-align: center;
    }

    .button-row {
      display: flex;
      gap: 12px;
      justify-content: center;
    }

    .control-button {
      padding: 12px 24px;
      border: none;
      border-radius: 8px;
      font-size: 14px;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.3s ease;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      pointer-events: auto;
    }

    #sound-button {
      background: linear-gradient(135deg, #ff6b35 0%, #ff8c42 100%);
      color: #ffffff;
      box-shadow: 0 4px 15px rgba(255, 107, 53, 0.4);
    }

    #sound-button:hover {
      background: linear-gradient(135deg, #ff8c42 0%, #ffa500 100%);
      box-shadow: 0 6px 20px rgba(255, 107, 53, 0.6);
      transform: translateY(-2px);
    }

    #sound-button.active {
      background: linear-gradient(135deg, #00d4ff 0%, #0099cc 100%);
      box-shadow: 0 4px 15px rgba(0, 212, 255, 0.5);
    }

    #reset-button {
      background: linear-gradient(135deg, #4a5568 0%, #2d3748 100%);
      color: #ffffff;
      box-shadow: 0 4px 15px rgba(74, 85, 104, 0.3);
    }

    #reset-button:hover {
      background: linear-gradient(135deg, #2d3748 0%, #1a202c 100%);
      box-shadow: 0 6px 20px rgba(74, 85, 104, 0.5);
      transform: translateY(-2px);
    }

    #stats-panel {
      position: absolute;
      top: 140px;
      right: 32px;
      background: rgba(26, 26, 46, 0.85);
      backdrop-filter: blur(10px);
      border: 2px solid rgba(0, 212, 255, 0.2);
      border-radius: 12px;
      padding: 20px;
      pointer-events: none;
      min-width: 200px;
    }

    .stat-row {
      display: flex;
      justify-content: space-between;
      margin-bottom: 12px;
      font-size: 14px;
    }

    .stat-label {
      color: #b8c5d6;
      font-weight: 500;
    }

    .stat-value {
      color: #00d4ff;
      font-weight: 700;
      font-family: 'Courier New', monospace;
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
  <script src="https://cdn.tailwindcss.com" type="text/javascript"></script>
 </head>
 <body>
  <div id="canvas-container"></div>
  <div id="ui-overlay">
   <div id="header">
    <h1 id="title">MINI-DYSON MEJÍA</h1>
    <p id="subtitle">EO = 0.01 - Orbital Stability Visualization</p>
   </div>
   <div id="stats-panel">
    <div class="stat-row"><span class="stat-label">Rotation:</span> <span class="stat-value" id="rotation-value">0.00°</span>
    </div>
    <div class="stat-row"><span class="stat-label">Time:</span> <span class="stat-value" id="time-value">0.00s</span>
    </div>
    <div class="stat-row"><span class="stat-label">Frequency:</span> <span class="stat-value">699 Hz</span>
    </div>
   </div>
   <div id="controls-panel">
    <p id="instruction-text">Click Sound to activate 699 Hz tone • Drag to rotate view</p>
    <div class="button-row"><button id="sound-button" class="control-button">Start Sound</button> <button id="reset-button" class="control-button">Reset</button>
    </div>
   </div>
  </div>
  <script>
    const defaultConfig = {
      title_text: "MINI-DYSON MEJÍA",
      subtitle_text: "EO = 0.01 - Orbital Stability Visualization",
      instruction_text: "Click Sound to activate 699 Hz tone • Drag to rotate view",
      primary_color: "#00d4ff",
      secondary_color: "#ffa500",
      background_start: "#0a0e27",
      background_end: "#16213e",
      button_color: "#ff6b35"
    };

    let config = {};

    async function onConfigChange(newConfig) {
      config = newConfig;
      
      const titleEl = document.getElementById('title');
      const subtitleEl = document.getElementById('subtitle');
      const instructionEl = document.getElementById('instruction-text');
      
      titleEl.textContent = config.title_text || defaultConfig.title_text;
      titleEl.style.color = config.primary_color || defaultConfig.primary_color;
      titleEl.style.textShadow = `0 0 20px ${config.primary_color || defaultConfig.primary_color}80`;
      
      subtitleEl.textContent = config.subtitle_text || defaultConfig.subtitle_text;
      subtitleEl.style.color = config.secondary_color || defaultConfig.secondary_color;
      subtitleEl.style.textShadow = `0 0 15px ${config.secondary_color || defaultConfig.secondary_color}66`;
      
      instructionEl.textContent = config.instruction_text || defaultConfig.instruction_text;
      
      document.body.style.background = `linear-gradient(135deg, ${config.background_start || defaultConfig.background_start} 0%, ${config.background_end || defaultConfig.background_end} 100%)`;
      
      const soundBtn = document.getElementById('sound-button');
      if (!soundBtn.classList.contains('active')) {
        soundBtn.style.background = `linear-gradient(135deg, ${config.button_color || defaultConfig.button_color} 0%, #ff8c42 100%)`;
      }
    }

    if (window.elementSdk) {
      window.elementSdk.init({
        defaultConfig,
        onConfigChange,
        mapToCapabilities: (config) => ({
          recolorables: [
            {
              get: () => config.primary_color || defaultConfig.primary_color,
              set: (value) => {
                config.primary_color = value;
                window.elementSdk.setConfig({ primary_color: value });
              }
            },
            {
              get: () => config.secondary_color || defaultConfig.secondary_color,
              set: (value) => {
                config.secondary_color = value;
                window.elementSdk.setConfig({ secondary_color: value });
              }
            },
            {
              get: () => config.background_start || defaultConfig.background_start,
              set: (value) => {
                config.background_start = value;
                window.elementSdk.setConfig({ background_start: value });
              }
            },
            {
              get: () => config.background_end || defaultConfig.background_end,
              set: (value) => {
                config.background_end = value;
                window.elementSdk.setConfig({ background_end: value });
              }
            },
            {
              get: () => config.button_color || defaultConfig.button_color,
              set: (value) => {
                config.button_color = value;
                window.elementSdk.setConfig({ button_color: value });
              }
            }
          ],
          borderables: [],
          fontEditable: undefined,
          fontSizeable: undefined
        }),
        mapToEditPanelValues: (config) => new Map([
          ["title_text", config.title_text || defaultConfig.title_text],
          ["subtitle_text", config.subtitle_text || defaultConfig.subtitle_text],
          ["instruction_text", config.instruction_text || defaultConfig.instruction_text]
        ])
      });
      
      config = window.elementSdk.config;
      onConfigChange(config);
    } else {
      config = defaultConfig;
      onConfigChange(config);
    }

    // THREE.JS SETUP
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setClearColor(0x000000, 0);
    document.getElementById('canvas-container').appendChild(renderer.domElement);

    // MINI-DYSON TORUS
    const dysonGeometry = new THREE.TorusGeometry(3, 0.8, 32, 100);
    const dysonMaterial = new THREE.MeshPhongMaterial({
      color: 0x44aaff,
      opacity: 0.3,
      transparent: true,
      side: THREE.DoubleSide,
      emissive: 0x2277cc,
      emissiveIntensity: 0.3
    });
    const torus = new THREE.Mesh(dysonGeometry, dysonMaterial);
    scene.add(torus);

    // FLOWER OF LIFE PATTERN
    for (let i = 0; i < 19; i++) {
      const angle = i * Math.PI * 2 / 19;
      const x = Math.cos(angle) * 2.5;
      const z = Math.sin(angle) * 2.5;
      const circleGeometry = new THREE.CircleGeometry(0.4, 32);
      const circleMaterial = new THREE.MeshBasicMaterial({
        color: 0xffaa00,
        side: THREE.DoubleSide,
        transparent: true,
        opacity: 0.6
      });
      const circle = new THREE.Mesh(circleGeometry, circleMaterial);
      circle.position.set(x, 0, z);
      circle.rotation.x = Math.PI / 2;
      scene.add(circle);
    }

    // DOUBLE HELIX (2H)
    const helixGeometry = new THREE.SphereGeometry(0.2, 16, 16);
    const helix1Material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
    const helix1 = new THREE.Mesh(helixGeometry, helix1Material);
    helix1.position.set(1, 0, 0);
    scene.add(helix1);

    const helix2Material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
    const helix2 = new THREE.Mesh(helixGeometry, helix2Material);
    helix2.position.set(-1, 0, 0);
    scene.add(helix2);

    // 4 PLANETS
    const planets = [];
    const planetColors = [0xff0000, 0x00ff00, 0x0000ff, 0xffff00];
    for (let i = 0; i < 4; i++) {
      const planetGeometry = new THREE.SphereGeometry(0.15, 16, 16);
      const planetMaterial = new THREE.MeshBasicMaterial({ color: planetColors[i] });
      const planet = new THREE.Mesh(planetGeometry, planetMaterial);
      const angle = i * Math.PI / 2;
      planet.position.set(Math.cos(angle) * 2, 0, Math.sin(angle) * 2);
      scene.add(planet);
      planets.push(planet);
    }

    // LIGHTING
    const ambientLight = new THREE.AmbientLight(0x404040);
    scene.add(ambientLight);
    const pointLight = new THREE.PointLight(0xffaa00, 2);
    pointLight.position.set(0, 5, 0);
    scene.add(pointLight);

    camera.position.z = 10;
    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;

    // AUDIO SYSTEM
    let audioContext = null;
    let oscillator = null;
    let isPlaying = false;
    const soundButton = document.getElementById('sound-button');
    const resetButton = document.getElementById('reset-button');

    soundButton.addEventListener('click', () => {
      if (!isPlaying) {
        if (!audioContext) {
          audioContext = new AudioContext();
        }
        oscillator = audioContext.createOscillator();
        oscillator.frequency.value = 699;
        oscillator.connect(audioContext.destination);
        oscillator.start();
        isPlaying = true;
        soundButton.textContent = 'Stop Sound';
        soundButton.classList.add('active');
      } else {
        if (oscillator) {
          oscillator.stop();
          oscillator = null;
        }
        isPlaying = false;
        soundButton.textContent = 'Start Sound';
        soundButton.classList.remove('active');
      }
    });

    resetButton.addEventListener('click', () => {
      time = 0;
    });

    // ANIMATION LOOP
    let time = 0;
    function animate() {
      requestAnimationFrame(animate);
      time += 0.01;

      // ROTATE DYSON TORUS
      torus.rotation.y = time * 0.5;

      // DOUBLE HELIX MOTION
      helix1.position.y = Math.sin(time * 3) * 0.5;
      helix2.position.y = Math.sin(time * 3 + Math.PI) * 0.5;

      // PLANETARY ORBITS (EO = 0.01)
      planets.forEach((planet, i) => {
        const angle = time * 0.3 + i * Math.PI / 2;
        const radius = 2 + Math.sin(time * 0.1) * 0.1; // EO dampening
        planet.position.x = Math.cos(angle) * radius;
        planet.position.z = Math.sin(angle) * radius;
      });

      // UPDATE STATS
      document.getElementById('rotation-value').textContent = ((torus.rotation.y * 180 / Math.PI) % 360).toFixed(2) + '°';
      document.getElementById('time-value').textContent = time.toFixed(2) + 's';

      controls.update();
      renderer.render(scene, camera);
    }
    animate();

    // RESPONSIVE RESIZE
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9a1297c9b5cd7b05',t:'MTc2MzU4NTU2My4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
