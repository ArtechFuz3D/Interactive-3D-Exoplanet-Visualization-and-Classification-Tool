# Interactive 3D Exoplanet Visualization and Classification Tool

## Project Overview

The *Interactive 3D Exoplanet Visualization and Classification Tool* is a web-based application that enables users to explore and classify exoplanet candidates using data from NASA’s *Planet Hunters TESS* citizen science project. Built with Three.js, Blender, and AstroJS, the tool visualizes exoplanet systems in 3D, simulates planetary orbits and light curves, and gamifies the classification of light curves to identify potential exoplanets. The project aims to enhance public engagement with NASA’s exoplanet research while providing a visually stunning, SpaceX/NASA-inspired interface. It is designed for solo development, with potential for recognition through NASA’s citizen science community and events like the NASA Space Apps Challenge.

### Goals

- **Scientific Contribution**: Assist *Planet Hunters TESS* by providing an intuitive tool for classifying exoplanet candidates based on TESS light curve data.
- **Visualization**: Create immersive 3D visualizations of exoplanet systems, including stars, planets, and orbits, using Blender models and Three.js.
- **Gamification**: Engage users with a game-like interface for classifying light curves, featuring scores, feedback, and tutorials.
- **Accessibility**: Deliver a browser-based app with model fallbacks for performance on various devices, hosted on a platform like Vercel.
- **Recognition**: Share the tool with NASA’s citizen science community and submit it to events like the NASA Space Apps Challenge for visibility.

### Target Audience

- Citizen scientists participating in *Planet Hunters TESS* on Zooniverse.
- Space enthusiasts interested in exoplanet discovery.
- Educators and students learning about astronomy and data visualization.
- Developers and artists inspired by 3D visualization and open-source projects.

---

## Technical Architecture

### Tech Stack

- **Frontend**:
  - **AstroJS**: Lightweight framework for building the web app, handling static and dynamic content (aligned with your familiarity from March 10, 2025).
  - **Three.js**: Renders 3D scenes, including exoplanet systems, orbits, and light curve visualizations.
  - **Web Audio API**: Adds subtle audio feedback for user interactions (e.g., clicks, as in your audio visualizer from March 17, 2025).
- **3D Modeling**:
  - **Blender**: Creates high-quality 3D models of stars, planets, and orbit paths, exported as `.glb` with fallbacks for low-poly versions.
- **Backend/Data Processing**:
  - **Python**: Processes TESS light curve data using `lightkurve` and `astroPy`, generating JSON for Three.js integration.
  - **C++ (Optional, 1%)**: Used for performance-critical tasks, like optimizing light curve simulations, if Python proves too slow.
- **Data Source**:
  - NASA TESS data from the Mikulski Archive for Space Telescopes (MAST).
- **Hosting**:
  - Vercel or GitHub Pages for deployment, ensuring accessibility.
- **Version Control**:
  - GitHub for code, assets, and documentation, with an open-source license to align with NASA’s ethos.

### System Components

1. **Data Ingestion Module** (Python)
   - Downloads and processes TESS light curve data from MAST.
   - Extracts orbital parameters (e.g., period, radius) and light curve points.
   - Outputs JSON files with star/planet metadata and light curve data for Three.js.
2. **3D Visualization Engine** (Three.js)
   - Renders a 3D scene with a starfield background, central star, orbiting planets, and animated orbit paths.
   - Supports model fallbacks: high-poly `.glb` models for modern devices, low-poly for older hardware.
   - Simulates light curves based on planet transits, visualized as a 2D graph or 3D effect.
3. **Classification Interface** (AstroJS + Three.js)
   - Displays light curves and allows users to classify them as “potential exoplanet,” “no transit,” or “noise.”
   - Includes sliders to adjust planet parameters (e.g., size, orbit distance) and see real-time light curve changes.
   - Features a gamified UI with scores, feedback, and a leaderboard.
4. **3D Asset Pipeline** (Blender)
   - Models stars (emissive textures), planets (varied textures), and orbits (curved paths).
   - Exports assets as `.glb` with multiple LODs (Levels of Detail) for performance.
5. **Audio Feedback** (Web Audio API)
   - Plays subtle click sounds or ambient effects during interactions, enhancing the game-like feel.

### Data Flow

1. Python scripts fetch TESS data from MAST and process it into JSON.
2. AstroJS frontend loads JSON data and passes it to Three.js.
3. Three.js renders the 3D scene, using high-poly or low-poly models based on device detection.
4. Users interact with the scene (e.g., rotate, zoom, adjust parameters) and classify light curves.
5. Classifications are stored locally or sent to a backend (future scope) for aggregation.

---

## Three.js Scene Setup

### Scene Structure

- **Camera**: PerspectiveCamera with OrbitControls for 360-degree exploration (as in your March 17, 2025 starfield project).
- **Lighting**: PointLight at the star’s position for realistic illumination, plus ambient light for visibility.
- **Objects**:
  - **Star**: A SphereGeometry with emissive material, textured in Blender.
  - **Planets**: SphereGeometries with varied textures, positioned based on TESS orbital data.
  - **Orbit Paths**: LineGeometry or TubeGeometry to visualize elliptical orbits.
  - **Starfield**: InstancedMesh for distant stars, reused from your “Idiot’s Guide to the Universe” (March 17, 2025).
- **Light Curve Display**: A 2D canvas or 3D plane showing the light curve graph, updated during transits.
- **UI**: HTML/CSS overlays (via AstroJS) for classification buttons, sliders, and scores.

### Model Fallbacks

- **High-Poly Models**: Detailed `.glb` assets with PBR textures, used on modern devices (e.g., desktops with WebGL 2.0).
- **Low-Poly Models**: Simplified `.glb` assets with basic textures, used on low-end devices (e.g., mobiles).
- **Detection Logic**: Uses `navigator.gpu` for WebGPU detection and `gl.getParameter(gl.MAX_TEXTURE_SIZE)` to assess WebGL capabilities, selecting appropriate models.

### Initial Code

Below is a starter Three.js scene, ready to accept TESS data, with model fallbacks and AstroJS integration.

#### `src/main.js`

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

export class ExoplanetScene {
  constructor(canvasId) {
    this.canvas = document.getElementById(canvasId);
    this.scene = new THREE.Scene();
    this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    this.renderer = new THREE.WebGLRenderer({ canvas: this.canvas, antialias: true });
    this.controls = new OrbitControls(this.camera, this.renderer.domElement);
    this.gltfLoader = new GLTFLoader();
    this.star = null;
    this.planets = [];
    this.orbits = [];
    this.starfield = null;
    this.deviceLevel = this.detectDeviceLevel();
  }

  init() {
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.camera.position.set(0, 5, 10);
    this.controls.update();

    // Lighting
    const pointLight = new THREE.PointLight(0xffffff, 1, 100);
    pointLight.position.set(0, 0, 0);
    this.scene.add(pointLight);
    this.scene.add(new THREE.AmbientLight(0x404040, 0.2));

    // Starfield
    this.createStarfield();

    // Load initial star model
    this.loadStarModel();

    // Animation loop
    this.animate();
  }

  detectDeviceLevel() {
    const isWebGPUSupported = !!navigator.gpu;
    const gl = this.renderer.getContext();
    const maxTextureSize = gl.getParameter(gl.MAX_TEXTURE_SIZE);
    return isWebGPUSupported && maxTextureSize > 8192 ? 'high' : 'low';
  }

  createStarfield() {
    const starGeometry = new THREE.SphereGeometry(0.05, 8, 8);
    const starMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff });
    const starMesh = new THREE.InstancedMesh(starGeometry, starMaterial, 1000);
    for (let i = 0; i < 1000; i++) {
      const matrix = new THREE.Matrix4();
      matrix.setPosition(
        (Math.random() - 0.5) * 100,
        (Math.random() - 0.5) * 100,
        (Math.random() - 0.5) * 100
      );
      starMesh.setMatrixAt(i, matrix);
    }
    this.scene.add(starMesh);
    this.starfield = starMesh;
  }

  loadStarModel() {
    const modelPath = this.deviceLevel === 'high' ? '/models/star_high.glb' : '/models/star_low.glb';
    this.gltfLoader.load(modelPath, (gltf) => {
      this.star = gltf.scene;
      this.star.position.set(0, 0, 0);
      this.scene.add(this.star);
    }, undefined, (error) => console.error('Error loading star model:', error));
  }

  loadPlanetData(data) {
    // Example: data = [{ radius, orbitRadius, period, texture }, ...]
    data.forEach((planetData, index) => {
      const modelPath = this.deviceLevel === 'high' ? `/models/planet_${index}_high.glb` : `/models/planet_${index}_low.glb`;
      this.gltfLoader.load(modelPath, (gltf) => {
        const planet = gltf.scene;
        planet.position.set(planetData.orbitRadius, 0, 0);
        this.planets.push({ mesh: planet, data: planetData });
        this.scene.add(planet);
        this.createOrbit(planetData.orbitRadius);
      });
    });
  }

  createOrbit(radius) {
    const curve = new THREE.EllipseCurve(0, 0, radius, radius, 0, 2 * Math.PI, false, 0);
    const points = curve.getPoints(50);
    const geometry = new THREE.BufferGeometry().setFromPoints(points);
    const material = new THREE.LineBasicMaterial({ color: 0x00ff00 });
    const orbit = new THREE.Line(geometry, material);
    orbit.rotation.x = Math.PI / 2;
    this.orbits.push(orbit);
    this.scene.add(orbit);
  }

  animate() {
    requestAnimationFrame(() => this.animate());
    this.planets.forEach((planet) => {
      const angle = Date.now() * 0.001 / planet.data.period;
      planet.mesh.position.set(
        planet.data.orbitRadius * Math.cos(angle),
        0,
        planet.data.orbitRadius * Math.sin(angle)
      );
    });
    this.controls.update();
    this.renderer.render(this.scene, this.camera);
  }
}
```

#### `src/pages/index.astro`

```astro
---
import { ExoplanetScene } from '../main.js';
---

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Exoplanet Visualizer</title>
    <style>
      body { margin: 0; background: #000; color: #fff; font-family: Arial; }
      canvas { display: block; }
      #ui { position: absolute; top: 10px; left: 10px; }
      button { padding: 10px; margin: 5px; background: #1a1a1a; color: #fff; border: 1px solid #00ff00; }
    </style>
  </head>
  <body>
    <canvas id="scene"></canvas>
    <div id="ui">
      <button id="classify">Classify Light Curve</button>
      <div id="light-curve"></div>
    </div>
    <script>
      const scene = new ExoplanetScene('scene');
      scene.init();
      // Example data load
      const sampleData = [
        { radius: 0.1, orbitRadius: 2, period: 10, texture: 'earth' },
        { radius: 0.15, orbitRadius: 3, period: 15, texture: 'jupiter' }
      ];
      scene.loadPlanetData(sampleData);
    </script>
  </body>
</html>
```

#### `process_data.py`

```python
import lightkurve as lk
import json

def process_tess_data(target_id, output_file):
    # Fetch TESS light curve
    search_result = lk.search_lightcurve(target_id, mission='TESS')
    lc = search_result[0].download().normalize()
    
    # Extract relevant data
    data = {
        'target_id': target_id,
        'time': lc.time.value.tolist(),
        'flux': lc.flux.value.tolist(),
        'planets': []  # Placeholder for orbital parameters
    }
    
    # Save to JSON
    with open(output_file, 'w') as f:
        json.dump(data, f, indent=2)

if __name__ == '__main__':
    process_tess_data('TIC 123456789', 'public/data/tess_data.json')
```

### Notes

- **Scene**: The `ExoplanetScene` class sets up a basic Three.js scene with a starfield, star, and placeholder for planets. It supports model fallbacks via `detectDeviceLevel`.
- **Data**: The Python script fetches TESS data using `lightkurve` and outputs JSON. Replace `'TIC 123456789'` with a real TESS ID from MAST.
- **AstroJS**: The `index.astro` file provides a minimal UI, ready for expansion with classification features.
- **C++**: If needed, you can write a WebAssembly module (e.g., with Emscripten) for light curve simulation, but Python should suffice initially.

---

## Development Plan

### Phase 1: Setup and Scene (2-3 weeks)

- **Tasks**:
  - Set up AstroJS project (`npm create astro@latest`).
  - Implement the Three.js scene above, testing with sample data.
  - Create basic Blender models (star, one planet, orbit path) and export as `.glb` (high/low poly).
  - Write Python script to process one TESS light curve into JSON.
- **Milestone**: A 3D scene with one star, one planet orbiting, and a static light curve display.

### Phase 2: Visualization and Simulation (3-4 weeks)

- **Tasks**:
  - Load real TESS data into Three.js, rendering multiple planets.
  - Simulate light curves based on planet transits (using transit equations or `lightkurve` outputs).
  - Add model fallbacks and optimize performance (e.g., instancing for starfield).
- **Milestone**: Accurate 3D visualization of a TESS system with dynamic light curve updates.

### Phase 3: Classification and Gamification (3-4 weeks)

- **Tasks**:
  - Build classification UI (buttons, sliders, leaderboard) in AstroJS.
  - Implement gamified feedback (scores, audio cues via Web Audio API).
  - Add a tutorial mode explaining exoplanet detection.
- **Milestone**: Fully functional classification tool with a polished UI.

### Phase 4: Testing and Outreach (2 weeks)

- **Tasks**:
  - Test with 10-20 TESS systems, validating against *Planet Hunters TESS* guidelines.
  - Deploy to Vercel and share on GitHub.
  - Submit to *Planet Hunters TESS* and post on X, tagging @NASA360.
- **Milestone**: Publicly available tool with NASA community feedback.

---

## Asset Creation (Blender)

- **Star**: Sphere with emissive texture (e.g., yellow glow). High-poly: 32x32 segments; low-poly: 8x8.
- **Planets**: Spheres with textures (e.g., Earth-like, gas giant). High-poly: 16x16 segments; low-poly: 6x6.
- **Orbits**: TubeGeometry in Three.js, but model a curved path in Blender for reference.
- **Export**: Use `.glb` format, with separate files for high/low LODs. Test exports in Three.js early.

---

## Future Enhancements

- Integrate with Zooniverse API to submit classifications directly.
- Add WebGPU support for enhanced rendering (as explored in your March 21, 2025 lunar project).
- Expand to other NASA projects (e.g., *Exoplanet Watch*).
- Create a desktop app with Tauri, reusing your “Planetary Scout” setup (March 9, 2025).

---

## Getting Started

### Prerequisites

- Node.js (v18+)
- Python (v3.8+)
- Blender (v3.6+)
- Git

### Setup

1. **Clone Repository**:

   ```bash
   git clone https://github.com/yourusername/exoplanet-visualizer.git
   cd exoplanet-visualizer
   ```

2. **Install Frontend**:

   ```bash
   npm install
   npm run dev
   ```

3. **Install Python Dependencies**:

   ```bash
   pip install lightkurve astropy
   ```

4. **Run Data Processing**:

   ```bash
   python process_data.py
   ```

5. **Create Blender Models**:
   - Model a star and planet in Blender.
   - Export as `public/models/star_high.glb`, `star_low.glb`, etc.
6. **Access**:
   - Open `http://localhost:4321` to view the scene.

### Directory Structure

```
exoplanet-visualizer/
├── public/
│   ├── data/              # JSON TESS data
│   ├── models/            # Blender .glb files
├── src/
│   ├── pages/
│   │   └── index.astro    # Main page
│   ├── main.js            # Three.js scene
├── scripts/
│   └── process_data.py    # Python data processing
├── package.json
├── astro.config.mjs
├── README.md
```

---

## Contributing

This project is open-source under the MIT License. Contributions are welcome, especially for additional TESS data processing, 3D model enhancements, or UI improvements. Contact the NASA *Planet Hunters TESS* team for data validation or collaboration.

## Contact

- GitHub: [yourusername]
- X: @yourhandle
- NASA Citizen Science: science.nasa.gov/citizenscience

---

This design doc provides a clear roadmap for your project, with a functional Three.js scene to start development. You can copy this directly into your README.md, adjusting placeholders (e.g., GitHub username) as needed. Next steps:

1. Set up the AstroJS project and test the provided `main.js` and `index.astro`.
2. Create placeholder Blender models and export them.
3. Run the Python script with a real TESS ID from MAST.

Let me know if you need help with specific tasks, like rigging models in Blender, optimizing Three.js performance, or fetching TESS data!
