<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Planta 3D + Dashboards - Simulación Discreta</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
    <script type="importmap">
    {
        "imports": {
            "three": "https://cdn.jsdelivr.net/npm/three@0.169.0/build/three.module.js",
            "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.169.0/examples/jsm/"
        }
    }
    </script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: #0a0e1a;
            color: #e0e6ed;
            overflow: hidden;
            height: 100vh;
        }

        /* Layout principal */
        .main-container {
            display: flex;
            height: 100vh;
            width: 100vw;
        }

        /* Panel 3D */
        .scene-panel {
            flex: 1;
            position: relative;
            min-width: 0;
        }

        #canvas-container {
            width: 100%;
            height: 100%;
            position: relative;
        }

        canvas { display: block; }

        /* Overlay de info en 3D */
        .scene-overlay {
            position: absolute;
            top: 15px;
            left: 15px;
            background: rgba(10, 14, 26, 0.92);
            border: 1px solid #1e3a5f;
            border-radius: 10px;
            padding: 15px 20px;
            backdrop-filter: blur(8px);
            z-index: 10;
            min-width: 220px;
        }

        .scene-overlay h3 {
            color: #4fc3f7;
            font-size: 14px;
            margin-bottom: 10px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .legend-item {
            display: flex;
            align-items: center;
            gap: 8px;
            margin: 6px 0;
            font-size: 12px;
        }

        .legend-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            flex-shrink: 0;
        }

        .legend-dot.equipo { background: #4fc3f7; }
        .legend-dot.persona { background: #ff9800; }
        .legend-dot.camion { background: #e91e63; }
        .legend-dot.almacen { background: #8bc34a; }

        /* Panel de Dashboards */
        .dashboard-panel {
            width: 420px;
            background: #0d1321;
            border-left: 1px solid #1e3a5f;
            display: flex;
            flex-direction: column;
            overflow-y: auto;
            overflow-x: hidden;
        }

        .dash-header {
            background: linear-gradient(135deg, #0d1b2a 0%, #1b2838 100%);
            padding: 18px 20px;
            border-bottom: 2px solid #1e3a5f;
        }

        .dash-header h1 {
            font-size: 16px;
            color: #4fc3f7;
            margin-bottom: 4px;
        }

        .dash-header .subtitle {
            font-size: 11px;
            color: #8899aa;
        }

        /* KPIs */
        .kpi-row {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            padding: 15px;
        }

        .kpi-card {
            background: linear-gradient(135deg, #111d2e 0%, #162538 100%);
            border: 1px solid #1e3a5f;
            border-radius: 8px;
            padding: 12px 15px;
            text-align: center;
            transition: transform 0.2s, border-color 0.2s;
        }

        .kpi-card:hover {
            transform: translateY(-2px);
            border-color: #4fc3f7;
        }

        .kpi-value {
            font-size: 22px;
            font-weight: 700;
            color: #4fc3f7;
        }

        .kpi-label {
            font-size: 10px;
            color: #8899aa;
            margin-top: 4px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .kpi-change {
            font-size: 11px;
            margin-top: 4px;
        }

        .kpi-change.positive { color: #4caf50; }
        .kpi-change.negative { color: #f44336; }

        /* Gráficos */
        .chart-section {
            padding: 15px;
            border-bottom: 1px solid #1e3a5f;
        }

        .chart-section h4 {
            font-size: 12px;
            color: #a0b4c8;
            margin-bottom: 10px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .chart-container {
            background: #111d2e;
            border-radius: 8px;
            padding: 10px;
            border: 1px solid #1e3a5f;
        }

        /* Tabla de equipos */
        .table-section {
            padding: 15px;
        }

        .table-section h4 {
            font-size: 12px;
            color: #a0b4c8;
            margin-bottom: 10px;
            text-transform: uppercase;
        }

        .data-table {
            width: 100%;
            border-collapse: collapse;
            font-size: 11px;
        }

        .data-table th {
            background: #1b2838;
            color: #4fc3f7;
            padding: 8px 10px;
            text-align: left;
            font-weight: 600;
            border-bottom: 1px solid #1e3a5f;
        }

        .data-table td {
            padding: 7px 10px;
            border-bottom: 1px solid #162538;
            color: #c0d0e0;
        }

        .data-table tr:hover td {
            background: #162538;
        }

        .status-badge {
            padding: 2px 8px;
            border-radius: 10px;
            font-size: 10px;
            font-weight: 600;
        }

        .status-activo { background: #1b5e20; color: #81c784; }
        .status-pausa { background: #e65100; color: #ffb74d; }
        .status-mant { background: #b71c1c; color: #ef9a9a; }

        /* Barra de progreso */
        .progress-bar {
            width: 100%;
            height: 6px;
            background: #162538;
            border-radius: 3px;
            overflow: hidden;
        }

        .progress-fill {
            height: 100%;
            border-radius: 3px;
            transition: width 0.5s ease;
        }

        .progress-fill.high { background: #4caf50; }
        .progress-fill.medium { background: #ff9800; }
        .progress-fill.low { background: #f44336; }

        /* Tooltip flotante en 3D */
        #tooltip-3d {
            position: absolute;
            background: rgba(10, 14, 26, 0.95);
            border: 1px solid #4fc3f7;
            border-radius: 6px;
            padding: 8px 12px;
            font-size: 11px;
            pointer-events: none;
            display: none;
            z-index: 100;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        }

        #tooltip-3d .tt-title {
            color: #4fc3f7;
            font-weight: 700;
            margin-bottom: 3px;
        }

        #tooltip-3d .tt-info {
            color: #c0d0e0;
        }

        /* Scrollbar personalizada */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #0d1321; }
        ::-webkit-scrollbar-thumb { background: #1e3a5f; border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: #2a4a6f; }

        /* Responsive */
        @media (max-width: 1024px) {
            .dashboard-panel { width: 340px; }
        }
    </style>
<base target="_blank">
</head>
<body>
    <div class="main-container">
        <!-- Panel 3D -->
        <div class="scene-panel">
            <div id="canvas-container"></div>
            <div id="tooltip-3d"></div>

            <div class="scene-overlay">
                <h3>🏭 Planta San Antonio</h3>
                <div class="legend-item">
                    <div class="legend-dot equipo"></div>
                    <span>Equipos (Envasadoras, MRU)</span>
                </div>
                <div class="legend-item">
                    <div class="legend-dot persona"></div>
                    <span>Operarios (12 activos)</span>
                </div>
                <div class="legend-item">
                    <div class="legend-dot camion"></div>
                    <span>Camiones en espera</span>
                </div>
                <div class="legend-item">
                    <div class="legend-dot almacen"></div>
                    <span>Almacén / Bodega</span>
                </div>
                <div style="margin-top:10px; font-size:11px; color:#8899aa;">
                    🖱️ <b>Click</b> en objeto para ver detalle<br>
                    🔄 <b>Drag</b> para rotar vista<br>
                    📜 <b>Scroll</b> para zoom
                </div>
            </div>
        </div>

        <!-- Panel Dashboards -->
        <div class="dashboard-panel">
            <div class="dash-header">
                <h1>📊 Dashboard de Producción</h1>
                <div class="subtitle">Planta Láctea - Simulación en Tiempo Real</div>
            </div>

            <!-- KPIs -->
            <div class="kpi-row">
                <div class="kpi-card">
                    <div class="kpi-value" id="kpi-eficiencia">67.3%</div>
                    <div class="kpi-label">Eficiencia Global</div>
                    <div class="kpi-change positive">▲ +2.1% vs ayer</div>
                </div>
                <div class="kpi-card">
                    <div class="kpi-value" id="kpi-produccion">8,420</div>
                    <div class="kpi-label">Lts Producidos (hoy)</div>
                    <div class="kpi-change positive">▲ +340 lts</div>
                </div>
                <div class="kpi-card">
                    <div class="kpi-value" id="kpi-ocupacion">54%</div>
                    <div class="kpi-label">Ocupación Capacidad</div>
                    <div class="kpi-change positive">▲ Meta: 50% ✓</div>
                </div>
                <div class="kpi-card">
                    <div class="kpi-value" id="kpi-rechazo">2.1%</div>
                    <div class="kpi-label">% Rechazo MP</div>
                    <div class="kpi-change negative">▼ -0.3% OK</div>
                </div>
            </div>

            <!-- Gráfico 1: Producción por Línea -->
            <div class="chart-section">
                <h4>📈 Producción por Línea (Últimos 7 días)</h4>
                <div class="chart-container">
                    <canvas id="chart-produccion" height="180"></canvas>
                </div>
            </div>

            <!-- Gráfico 2: Ocupación por Equipo -->
            <div class="chart-section">
                <h4>⚙️ Ocupación por Equipo (%)</h4>
                <div class="chart-container">
                    <canvas id="chart-equipos" height="180"></canvas>
                </div>
            </div>

            <!-- Gráfico 3: Rechazos por Motivo -->
            <div class="chart-section">
                <h4>🚫 Rechazos de MP por Motivo</h4>
                <div class="chart-container">
                    <canvas id="chart-rechazos" height="180"></canvas>
                </div>
            </div>

            <!-- Tabla de Estado de Equipos -->
            <div class="table-section">
                <h4>🔧 Estado de Equipos en Tiempo Real</h4>
                <table class="data-table">
                    <thead>
                        <tr>
                            <th>Equipo</th>
                            <th>Estado</th>
                            <th>Uso %</th>
                            <th>Producto</th>
                        </tr>
                    </thead>
                    <tbody id="tabla-equipos">
                        <tr>
                            <td>FABMAN</td>
                            <td><span class="status-badge status-activo">ACTIVO</span></td>
                            <td>
                                <div class="progress-bar"><div class="progress-fill high" style="width:78%"></div></div>
                                <span style="font-size:10px; color:#8899aa;">78%</span>
                            </td>
                            <td>Mantequilla</td>
                        </tr>
                        <tr>
                            <td>ENVQB</td>
                            <td><span class="status-badge status-activo">ACTIVO</span></td>
                            <td>
                                <div class="progress-bar"><div class="progress-fill high" style="width:82%"></div></div>
                                <span style="font-size:10px; color:#8899aa;">82%</span>
                            </td>
                            <td>Queso</td>
                        </tr>
                        <tr>
                            <td>ENVQM</td>
                            <td><span class="status-badge status-pausa">PAUSA</span></td>
                            <td>
                                <div class="progress-bar"><div class="progress-fill low" style="width:12%"></div></div>
                                <span style="font-size:10px; color:#8899aa;">12%</span>
                            </td>
                            <td>—</td>
                        </tr>
                        <tr>
                            <td>ENYMX</td>
                            <td><span class="status-badge status-mant">MANT.</span></td>
                            <td>
                                <div class="progress-bar"><div class="progress-fill low" style="width:0%"></div></div>
                                <span style="font-size:10px; color:#8899aa;">0%</span>
                            </td>
                            <td>Yogurt</td>
                        </tr>
                        <tr>
                            <td>ENYRM</td>
                            <td><span class="status-badge status-activo">ACTIVO</span></td>
                            <td>
                                <div class="progress-bar"><div class="progress-fill medium" style="width:45%"></div></div>
                                <span style="font-size:10px; color:#8899aa;">45%</span>
                            </td>
                            <td>Yogurt</td>
                        </tr>
                        <tr>
                            <td>POLYG</td>
                            <td><span class="status-badge status-activo">ACTIVO</span></td>
                            <td>
                                <div class="progress-bar"><div class="progress-fill medium" style="width:63%"></div></div>
                                <span style="font-size:10px; color:#8899aa;">63%</span>
                            </td>
                            <td>Polietileno</td>
                        </tr>
                    </tbody>
                </table>
            </div>

            <!-- Gráfico 4: Tendencia Mensual -->
            <div class="chart-section">
                <h4>📊 Tendencia Mensual - Capacidad Utilizada</h4>
                <div class="chart-container">
                    <canvas id="chart-tendencia" height="180"></canvas>
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        // ═══════════════════════════════════════════════════════════════
        // SCENE 3D - PLANTA LÁCTEA
        // ═══════════════════════════════════════════════════════════════

        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x0a0e1a);
        scene.fog = new THREE.Fog(0x0a0e1a, 50, 200);

        const camera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
        camera.position.set(60, 50, 60);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(container.clientWidth, container.clientHeight);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        container.appendChild(renderer.domElement);

        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.maxPolarAngle = Math.PI / 2.1;
        controls.target.set(25, 0, 25);

        // Luces
        const ambientLight = new THREE.AmbientLight(0x404060, 0.6);
        scene.add(ambientLight);

        const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
        dirLight.position.set(50, 80, 30);
        dirLight.castShadow = true;
        dirLight.shadow.mapSize.width = 2048;
        dirLight.shadow.mapSize.height = 2048;
        scene.add(dirLight);

        const pointLight = new THREE.PointLight(0x4fc3f7, 0.5, 100);
        pointLight.position.set(25, 20, 25);
        scene.add(pointLight);

        // ═══════════════════════════════════════════════════════════════
        // TERRENO / PISO DE FÁBRICA
        // ═══════════════════════════════════════════════════════════════

        // Piso principal con grid
        const floorGeometry = new THREE.PlaneGeometry(100, 100, 50, 50);
        const floorMaterial = new THREE.MeshStandardMaterial({
            color: 0x1a2332,
            roughness: 0.8,
            metalness: 0.2,
            wireframe: false
        });
        const floor = new THREE.Mesh(floorGeometry, floorMaterial);
        floor.rotation.x = -Math.PI / 2;
        floor.receiveShadow = true;
        scene.add(floor);

        // Grid helper
        const gridHelper = new THREE.GridHelper(100, 50, 0x2a3a4f, 0x162538);
        gridHelper.position.y = 0.02;
        scene.add(gridHelper);

        // Líneas de zona pintadas en el piso
        function createZoneLine(x1, z1, x2, z2, color) {
            const points = [new THREE.Vector3(x1, 0.03, z1), new THREE.Vector3(x2, 0.03, z2)];
            const geometry = new THREE.BufferGeometry().setFromPoints(points);
            const material = new THREE.LineBasicMaterial({ color: color, linewidth: 2 });
            return new THREE.Line(geometry, material);
        }

        // Zonas de la planta
        const zones = [
            { name: 'Recepción', x: 5, z: 5, w: 20, d: 15, color: 0x1e3a5f },
            { name: 'MRU', x: 30, z: 5, w: 20, d: 15, color: 0x1a3a4a },
            { name: 'Producción', x: 5, z: 30, w: 45, d: 25, color: 0x1a2a3a },
            { name: 'Almacén', x: 55, z: 30, w: 20, d: 25, color: 0x1a3a2a },
            { name: 'Envase', x: 55, z: 5, w: 20, d: 20, color: 0x2a1a3a }
        ];

        zones.forEach(zone => {
            const geo = new THREE.PlaneGeometry(zone.w, zone.d);
            const mat = new THREE.MeshStandardMaterial({
                color: zone.color,
                transparent: true,
                opacity: 0.3,
                roughness: 0.9
            });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.rotation.x = -Math.PI / 2;
            mesh.position.set(zone.x + zone.w/2 - 50, 0.01, zone.z + zone.d/2 - 50);
            scene.add(mesh);

            // Borde de zona
            const edges = new THREE.EdgesGeometry(new THREE.PlaneGeometry(zone.w, zone.d));
            const lineMat = new THREE.LineBasicMaterial({ color: 0x4fc3f7, transparent: true, opacity: 0.4 });
            const line = new THREE.LineSegments(edges, lineMat);
            line.rotation.x = -Math.PI / 2;
            line.position.set(zone.x + zone.w/2 - 50, 0.04, zone.z + zone.d/2 - 50);
            scene.add(line);
        });

        // ═══════════════════════════════════════════════════════════════
        // EQUIPOS (MÁQUINAS 3D)
        // ═══════════════════════════════════════════════════════════════

        const equipos = [];
        const equiposData = [
            { name: 'FABMAN', type: 'mantequilla', x: 10, z: 35, color: 0x4fc3f7, uso: 78, estado: 'activo' },
            { name: 'ENVQB', type: 'queso', x: 20, z: 40, color: 0x4fc3f7, uso: 82, estado: 'activo' },
            { name: 'ENVQM', type: 'queso', x: 30, z: 40, color: 0xff9800, uso: 12, estado: 'pausa' },
            { name: 'ENYMX', type: 'yogurt', x: 15, z: 50, color: 0xf44336, uso: 0, estado: 'mant' },
            { name: 'ENYRM', type: 'yogurt', x: 25, z: 50, color: 0x4fc3f7, uso: 45, estado: 'activo' },
            { name: 'POLYG', type: 'polietileno', x: 60, z: 15, color: 0x4fc3f7, uso: 63, estado: 'activo' },
            { name: 'ENVYTB', type: 'yogurt', x: 35, z: 50, color: 0xff9800, uso: 5, estado: 'pausa' }
        ];

        equiposData.forEach(eq => {
            // Base del equipo
            const baseGeo = new THREE.BoxGeometry(4, 1, 3);
            const baseMat = new THREE.MeshStandardMaterial({ 
                color: 0x2a3a4f, 
                roughness: 0.5, 
                metalness: 0.6 
            });
            const base = new THREE.Mesh(baseGeo, baseMat);
            base.position.set(eq.x - 50, 0.5, eq.z - 50);
            base.castShadow = true;
            base.receiveShadow = true;
            base.userData = { type: 'equipo', ...eq };
            scene.add(base);
            equipos.push(base);

            // Cuerpo principal
            const bodyGeo = new THREE.BoxGeometry(3, 2.5, 2);
            const bodyMat = new THREE.MeshStandardMaterial({ 
                color: eq.color, 
                roughness: 0.3, 
                metalness: 0.4,
                emissive: eq.color,
                emissiveIntensity: 0.1
            });
            const body = new THREE.Mesh(bodyGeo, bodyMat);
            body.position.set(0, 1.75, 0);
            base.add(body);

            // Panel de control (pequeña caja)
            const panelGeo = new THREE.BoxGeometry(0.8, 1.2, 0.2);
            const panelMat = new THREE.MeshStandardMaterial({ color: 0x111111, emissive: 0x00ff00, emissiveIntensity: 0.3 });
            const panel = new THREE.Mesh(panelGeo, panelMat);
            panel.position.set(0, 0.5, 1.1);
            body.add(panel);

            // Indicador LED de estado
            const ledGeo = new THREE.SphereGeometry(0.15, 8, 8);
            const ledColor = eq.estado === 'activo' ? 0x00ff00 : (eq.estado === 'pausa' ? 0xff9800 : 0xff0000);
            const ledMat = new THREE.MeshBasicMaterial({ color: ledColor });
            const led = new THREE.Mesh(ledGeo, ledMat);
            led.position.set(1.2, 1.5, 0);
            body.add(led);

            // Etiqueta flotante (sprite con texto)
            const canvas = document.createElement('canvas');
            canvas.width = 128; canvas.height = 32;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = 'rgba(0,0,0,0.7)';
            ctx.fillRect(0, 0, 128, 32);
            ctx.fillStyle = '#4fc3f7';
            ctx.font = 'bold 14px Arial';
            ctx.fillText(eq.name, 8, 22);
            const texture = new THREE.CanvasTexture(canvas);
            const spriteMat = new THREE.SpriteMaterial({ map: texture });
            const sprite = new THREE.Sprite(spriteMat);
            sprite.position.set(0, 3, 0);
            sprite.scale.set(4, 1, 1);
            base.add(sprite);
        });

        // ═══════════════════════════════════════════════════════════════
        // PERSONAS (OPERARIOS)
        // ═══════════════════════════════════════════════════════════════

        const personas = [];
        const personasData = [
            { name: 'Op. Carlos', x: 12, z: 38, rol: 'Supervisor' },
            { name: 'Op. María', x: 22, z: 42, rol: 'Envasado' },
            { name: 'Op. Juan', x: 32, z: 42, rol: 'Calidad' },
            { name: 'Op. Ana', x: 16, z: 52, rol: 'Producción' },
            { name: 'Op. Luis', x: 26, z: 52, rol: 'Producción' },
            { name: 'Op. Pedro', x: 8, z: 8, rol: 'Recepción' },
            { name: 'Op. Rosa', x: 35, z: 8, rol: 'MRU' },
            { name: 'Op. Diego', x: 62, z: 18, rol: 'Almacén' }
        ];

        personasData.forEach(p => {
            // Cuerpo
            const bodyGeo = new THREE.CapsuleGeometry(0.3, 0.8, 4, 8);
            const bodyMat = new THREE.MeshStandardMaterial({ 
                color: 0xff9800, 
                roughness: 0.7 
            });
            const body = new THREE.Mesh(bodyGeo, bodyMat);
            body.position.set(p.x - 50, 0.7, p.z - 50);
            body.castShadow = true;
            body.userData = { type: 'persona', ...p };
            scene.add(body);
            personas.push(body);

            // Cabeza
            const headGeo = new THREE.SphereGeometry(0.25, 8, 8);
            const headMat = new THREE.MeshStandardMaterial({ color: 0xffcc80 });
            const head = new THREE.Mesh(headGeo, headMat);
            head.position.set(0, 0.7, 0);
            body.add(head);

            // Casco (seguridad)
            const cascoGeo = new THREE.SphereGeometry(0.28, 8, 8, 0, Math.PI * 2, 0, Math.PI / 2);
            const cascoMat = new THREE.MeshStandardMaterial({ color: 0xffd700, metalness: 0.3 });
            const casco = new THREE.Mesh(cascoGeo, cascoMat);
            casco.position.set(0, 0.05, 0);
            head.add(casco);
        });

        // ═══════════════════════════════════════════════════════════════
        // CAMIONES
        // ═══════════════════════════════════════════════════════════════

        const camiones = [];
        const camionesData = [
            { name: 'Camión A', x: 5, z: 10, carga: 5000, estado: 'espera' },
            { name: 'Camión B', x: 12, z: 10, carga: 3200, estado: 'muestreo' },
            { name: 'Camión C', x: 5, z: 18, carga: 4500, estado: 'espera' }
        ];

        camionesData.forEach(c => {
            // Caja del camión
            const truckGeo = new THREE.BoxGeometry(3, 2.5, 6);
            const truckMat = new THREE.MeshStandardMaterial({ 
                color: 0xe91e63, 
                roughness: 0.4, 
                metalness: 0.3 
            });
            const truck = new THREE.Mesh(truckGeo, truckMat);
            truck.position.set(c.x - 50, 1.25, c.z - 50);
            truck.castShadow = true;
            truck.userData = { type: 'camion', ...c };
            scene.add(truck);
            cabiones.push(truck);

            // Cabina
            const cabGeo = new THREE.BoxGeometry(2.8, 1.5, 2);
            const cabMat = new THREE.MeshStandardMaterial({ color: 0xc2185b });
            const cab = new THREE.Mesh(cabGeo, cabMat);
            cab.position.set(0, -0.5, 3.5);
            truck.add(cab);

            // Ruedas
            [-1.2, 1.2].forEach(lx => {
                [-2, 2].forEach(lz => {
                    const wheelGeo = new THREE.CylinderGeometry(0.4, 0.4, 0.3, 8);
                    const wheelMat = new THREE.MeshStandardMaterial({ color: 0x333333 });
                    const wheel = new THREE.Mesh(wheelGeo, wheelMat);
                    wheel.rotation.z = Math.PI / 2;
                    wheel.position.set(lx, -1.25, lz);
                    truck.add(wheel);
                });
            });
        });

        // ═══════════════════════════════════════════════════════════════
        // ALMACÉN / BODEGA
        // ═══════════════════════════════════════════════════════════════

        const almacenGeo = new THREE.BoxGeometry(18, 8, 22);
        const almacenMat = new THREE.MeshStandardMaterial({ 
            color: 0x8bc34a, 
            transparent: true, 
            opacity: 0.15,
            roughness: 0.1,
            metalness: 0.1
        });
        const almacen = new THREE.Mesh(almacenGeo, almacenMat);
        almacen.position.set(65 - 50, 4, 42 - 50);
        scene.add(almacen);

        // Estantes dentro del almacén
        for (let i = 0; i < 4; i++) {
            for (let j = 0; j < 3; j++) {
                const estanteGeo = new THREE.BoxGeometry(3, 0.1, 1);
                const estanteMat = new THREE.MeshStandardMaterial({ color: 0x5a7a3a });
                const estante = new THREE.Mesh(estanteGeo, estanteMat);
                estante.position.set((i * 4) - 6, (j * 2) + 1, 0);
                almacen.add(estante);

                // Cajas en estante
                for (let k = 0; k < 3; k++) {
                    const cajaGeo = new THREE.BoxGeometry(0.8, 0.6, 0.8);
                    const cajaMat = new THREE.MeshStandardMaterial({ color: 0x689f38 });
                    const caja = new THREE.Mesh(cajaGeo, cajaMat);
                    caja.position.set((k * 1) - 1, 0.35, 0);
                    estante.add(caja);
                }
            }
        }

        // ═══════════════════════════════════════════════════════════════
        // INTERACCIÓN - RAYCASTER PARA CLICK
        // ═══════════════════════════════════════════════════════════════

        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();
        const tooltip = document.getElementById('tooltip-3d');

        function onMouseMove(event) {
            const rect = renderer.domElement.getBoundingClientRect();
            mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
            mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects([...equipos, ...personas, ...camiones], true);

            if (intersects.length > 0) {
                let obj = intersects[0].object;
                while (obj.parent && !obj.userData.type) obj = obj.parent;

                if (obj.userData.type) {
                    const data = obj.userData;
                    tooltip.style.display = 'block';
                    tooltip.style.left = (event.clientX + 15) + 'px';
                    tooltip.style.top = (event.clientY + 15) + 'px';

                    let content = `<div class="tt-title">${data.name}</div>`;
                    if (data.type === 'equipo') {
                        content += `<div class="tt-info">Tipo: ${data.type}</div>`;
                        content += `<div class="tt-info">Uso: ${data.uso}%</div>`;
                        content += `<div class="tt-info">Estado: ${data.estado.toUpperCase()}</div>`;
                    } else if (data.type === 'persona') {
                        content += `<div class="tt-info">Rol: ${data.rol}</div>`;
                    } else if (data.type === 'camion') {
                        content += `<div class="tt-info">Carga: ${data.carga} lts</div>`;
                        content += `<div class="tt-info">Estado: ${data.estado}</div>`;
                    }
                    tooltip.innerHTML = content;
                    document.body.style.cursor = 'pointer';
                }
            } else {
                tooltip.style.display = 'none';
                document.body.style.cursor = 'default';
            }
        }

        renderer.domElement.addEventListener('mousemove', onMouseMove);

        // ═══════════════════════════════════════════════════════════════
        // ANIMACIÓN
        // ═══════════════════════════════════════════════════════════════

        let time = 0;
        function animate() {
            requestAnimationFrame(animate);
            time += 0.01;

            // Animar personas (caminar en círculo pequeño)
            personas.forEach((p, i) => {
                const offset = i * 1.5;
                p.position.x += Math.sin(time + offset) * 0.02;
                p.position.z += Math.cos(time + offset) * 0.02;
                p.rotation.y = Math.atan2(Math.cos(time + offset), -Math.sin(time + offset));
            });

            // Animar LEDs de equipos (parpadeo)
            equipos.forEach(eq => {
                const led = eq.children[0]?.children[1];
                if (led) {
                    led.material.emissiveIntensity = 0.3 + Math.sin(time * 3) * 0.2;
                }
            });

            controls.update();
            renderer.render(scene, camera);
        }
        animate();

        // Resize
        window.addEventListener('resize', () => {
            camera.aspect = container.clientWidth / container.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(container.clientWidth, container.clientHeight);
        });

        // ═══════════════════════════════════════════════════════════════
        // DASHBOARDS - CHART.JS
        // ═══════════════════════════════════════════════════════════════

        Chart.defaults.color = '#8899aa';
        Chart.defaults.borderColor = '#1e3a5f';
        Chart.defaults.font.family = "'Segoe UI', sans-serif";
        Chart.defaults.font.size = 11;

        // Gráfico 1: Producción por Línea
        new Chart(document.getElementById('chart-produccion'), {
            type: 'line',
            data: {
                labels: ['Lun', 'Mar', 'Mié', 'Jue', 'Vie', 'Sáb', 'Dom'],
                datasets: [
                    {
                        label: 'Línea 1001 (Leche)',
                        data: [7200, 8100, 7800, 8500, 9200, 6800, 0],
                        borderColor: '#4fc3f7',
                        backgroundColor: 'rgba(79, 195, 247, 0.1)',
                        fill: true,
                        tension: 0.4
                    },
                    {
                        label: 'Línea 1002 (Saborizada)',
                        data: [5400, 6200, 5900, 6800, 7100, 5200, 0],
                        borderColor: '#ff9800',
                        backgroundColor: 'rgba(255, 152, 0, 0.1)',
                        fill: true,
                        tension: 0.4
                    },
                    {
                        label: 'Línea 1003 (Néctar)',
                        data: [3200, 3800, 3500, 4100, 4500, 2800, 0],
                        borderColor: '#8bc34a',
                        backgroundColor: 'rgba(139, 195, 74, 0.1)',
                        fill: true,
                        tension: 0.4
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { labels: { font: { size: 10 } } } },
                scales: { y: { beginAtZero: true, grid: { color: '#162538' } } }
            }
        });

        // Gráfico 2: Ocupación por Equipo
        new Chart(document.getElementById('chart-equipos'), {
            type: 'bar',
            data: {
                labels: ['FABMAN', 'ENVQB', 'ENVQM', 'ENYMX', 'ENYRM', 'POLYG', 'ENVYTB'],
                datasets: [{
                    label: '% Uso',
                    data: [78, 82, 12, 0, 45, 63, 5],
                    backgroundColor: [
                        '#4caf50', '#4caf50', '#ff9800', '#f44336', '#ff9800', '#4caf50', '#f44336'
                    ],
                    borderRadius: 4
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { display: false } },
                scales: { y: { max: 100, grid: { color: '#162538' } } }
            }
        });

        // Gráfico 3: Rechazos por Motivo
        new Chart(document.getElementById('chart-rechazos'), {
            type: 'doughnut',
            data: {
                labels: ['Crioscopía', 'Acidez', 'Carga Bacteriana', 'Temperatura', 'Otros'],
                datasets: [{
                    data: [35, 28, 20, 12, 5],
                    backgroundColor: ['#f44336', '#ff9800', '#ffeb3b', '#4fc3f7', '#9e9e9e'],
                    borderWidth: 0
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                cutout: '60%',
                plugins: { legend: { position: 'right', labels: { font: { size: 10 }, boxWidth: 12 } } }
            }
        });

        // Gráfico 4: Tendencia Mensual
        new Chart(document.getElementById('chart-tendencia'), {
            type: 'bar',
            data: {
                labels: ['Ene', 'Feb', 'Mar', 'Abr', 'May', 'Jun'],
                datasets: [{
                    label: 'Capacidad Utilizada (%)',
                    data: [23, 27, 31, 38, 45, 54],
                    backgroundColor: '#4fc3f7',
                    borderRadius: 4
                }, {
                    label: 'Meta (%)',
                    data: [30, 35, 40, 45, 50, 50],
                    type: 'line',
                    borderColor: '#ff9800',
                    borderDash: [5, 5],
                    pointRadius: 0,
                    borderWidth: 2
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { labels: { font: { size: 10 } } } },
                scales: { y: { max: 100, grid: { color: '#162538' } } }
            }
        });

        // Simulación de actualización en tiempo real
        setInterval(() => {
            // Actualizar KPIs con pequeñas variaciones
            const ef = 67.3 + (Math.random() - 0.5) * 2;
            document.getElementById('kpi-eficiencia').textContent = ef.toFixed(1) + '%';

            const prod = 8420 + Math.floor((Math.random() - 0.5) * 200);
            document.getElementById('kpi-produccion').textContent = prod.toLocaleString();

            const occ = 54 + (Math.random() - 0.5) * 3;
            document.getElementById('kpi-ocupacion').textContent = occ.toFixed(1) + '%';

            const rec = 2.1 + (Math.random() - 0.5) * 0.3;
            document.getElementById('kpi-rechazo').textContent = rec.toFixed(1) + '%';
        }, 3000);

    </script>
</body>
</html>
