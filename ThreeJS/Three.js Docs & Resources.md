## Official Resources
- [Three.js Documentation](https://threejs.org/docs/)
- [Three.js Manual](https://threejs.org/manual/)
- [Three.js Examples](https://threejs.org/examples/)
- [Three.js GitHub](https://github.com/mrdoob/three.js/)

## Video Tutorials & Channels
- [Three.js Journey](https://threejs-journey.com/) - Bruno Simon
- [Three.js Fundamentals](https://www.youtube.com/watch?v=Q7AOvWpIVHU) - Fireship
- [Three.js Tutorial Series](https://www.youtube.com/playlist?list=PLjcjAqAnHd1EIxV4FSZIiJZvsdrBc1Xho) - Chris Courses
- [Wael Yasmina](https://www.youtube.com/@WaelYasmina) - Three.js Tutorials

## Learning Resources
### Fundamentals
- [Three.js Fundamentals](https://threejsfundamentals.org/)
- [Discover Three.js](https://discoverthreejs.com/)
- [WebGL Fundamentals](https://webglfundamentals.org/)

### Interactive Learning
- [Three.js Editor](https://threejs.org/editor/)
- [Shadertoy](https://www.shadertoy.com/)
- [CodeSandbox Three.js](https://codesandbox.io/s/threejs-starter-forked-qdxbs)

## Basic Setup
```javascript
// Basic Scene Setup
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();

renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Animation Loop
function animate() {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
}
animate();
```

## Popular Add-ons & Controls
```javascript
// OrbitControls
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
const controls = new OrbitControls(camera, renderer.domElement);

// GLTF Loader
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
const loader = new GLTFLoader();
```

## Essential Tools
- [React Three Fiber](https://docs.pmnd.rs/react-three-fiber) - React renderer for Three.js
- [Drei](https://github.com/pmndrs/drei) - Useful helpers for React Three Fiber
- [Cannon.js](https://github.com/schteppe/cannon.js) - Physics engine
- [Tween.js](https://github.com/tweenjs/tween.js/) - Animation library
- [Stats.js](https://github.com/mrdoob/stats.js/) - Performance monitor

## 3D Model Resources
- [Sketchfab](https://sketchfab.com/)
- [Google Poly](https://poly.google.com/)
- [TurboSquid](https://www.turbosquid.com/)
- [CGTrader](https://www.cgtrader.com/)

## Texture Resources
- [ambientCG](https://ambientcg.com/)
- [3D Textures](https://3dtextures.me/)
- [Poliigon](https://www.poliigon.com/)

## Performance Optimization
- [Performance Tips](https://discoverthreejs.com/tips-and-tricks/)
- [Memory Management](https://threejs.org/docs/#manual/en/introduction/How-to-dispose-of-objects)
- [WebGL Best Practices](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/WebGL_best_practices)

## Common Patterns
### Responsive Design
```javascript
window.addEventListener('resize', onWindowResize, false);

function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
}
```

### Loading Models
```javascript
const loader = new GLTFLoader();

loader.load(
    'path/to/model.gltf',
    function (gltf) {
        scene.add(gltf.scene);
    },
    function (xhr) {
        console.log((xhr.loaded / xhr.total * 100) + '% loaded');
    },
    function (error) {
        console.error('An error happened', error);
    }
);
```

## Post-Processing
- [EffectComposer](https://threejs.org/docs/#examples/en/postprocessing/EffectComposer)
- [Common Post-Processing Effects](https://threejs.org/examples/?q=post)

## Shaders
- [Shader Introduction](https://threejsfundamentals.org/threejs/lessons/threejs-shadertoy.html)
- [GLSL Shaders](https://thebookofshaders.com/)
- [ShaderMaterial](https://threejs.org/docs/#api/en/materials/ShaderMaterial)

## Community Resources
- [Three.js Discord](https://discord.gg/threejs)
- [Three.js Discourse](https://discourse.threejs.org/)
- [Stack Overflow - Three.js](https://stackoverflow.com/questions/tagged/three.js)

## Debugging Tools
- [Three.js Inspector](https://chrome.google.com/webstore/detail/threejs-inspector/dnhjfclbfhcbcdfpjaeacomhbdfjbebi)
- [Spector.js](https://spector.babylonjs.com/)

## Best Practices
1. Scene Management
```javascript
// Cleanup
function dispose() {
    scene.traverse((object) => {
        if (object.geometry) object.geometry.dispose();
        if (object.material) {
            if (object.material.length) {
                for (let material of object.material) material.dispose();
            } else {
                object.material.dispose();
            }
        }
    });
}
```

2. Asset Loading
```javascript
const loadingManager = new THREE.LoadingManager();
loadingManager.onProgress = (url, loaded, total) => {
    console.log(`Loading: ${(loaded / total * 100)}%`);
};
```

## Advanced Topics
- Physics Integration
- Custom Shaders
- Particle Systems
- Ray Casting
- Shadow Mapping
- Environment Maps

Remember to:
- Optimize performance
- Handle cleanup properly
- Implement responsive design
- Use proper asset management
- Consider mobile performance
- Implement proper lighting
- Handle user interactions
- Test cross-browser compatibility