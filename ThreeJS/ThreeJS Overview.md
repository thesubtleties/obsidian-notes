## Core Concepts
```javascript
// Essential setup
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
```

## Scene Setup
```javascript
// Basic scene elements
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshBasicMaterial({color: 0x00ff00});
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

camera.position.z = 5;
```

## Common Geometries
```javascript
// Basic shapes
const box = new THREE.BoxGeometry(width, height, depth);
const sphere = new THREE.SphereGeometry(radius, segments, rings);
const cylinder = new THREE.CylinderGeometry(radiusTop, radiusBottom, height);
const plane = new THREE.PlaneGeometry(width, height);
```

## Materials
```javascript
// Common materials
const basicMaterial = new THREE.MeshBasicMaterial({color: 0xff0000});
const phongMaterial = new THREE.MeshPhongMaterial({
    color: 0xff0000,
    specular: 0x444444,
    shininess: 30
});
const standardMaterial = new THREE.MeshStandardMaterial({
    color: 0xff0000,
    roughness: 0.5,
    metalness: 0.5
});
```

## Lighting
```javascript
// Essential lights
const ambientLight = new THREE.AmbientLight(0x404040);
const pointLight = new THREE.PointLight(0xffffff, 1, 100);
const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
const spotLight = new THREE.SpotLight(0xffffff);

scene.add(ambientLight);
```

## Animation Loop
```javascript
function animate() {
    requestAnimationFrame(animate);
    
    // Update objects
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    
    renderer.render(scene, camera);
}
animate();
```

## Controls
```javascript
// Orbit controls
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.screenSpacePanning = false;
```

## Responsive Design
```javascript
// Handle window resizing
window.addEventListener('resize', onWindowResize, false);

function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
}
```

## Loading Models
```javascript
// GLTF Loader
const loader = new THREE.GLTFLoader();
loader.load('path/to/model.gltf', function(gltf) {
    scene.add(gltf.scene);
}, undefined, function(error) {
    console.error(error);
});
```

## Key Points & Gotchas
- Always dispose of geometries and materials when removing objects
- Use `renderer.setPixelRatio(window.devicePixelRatio)` for better display on high-DPI screens
- Implement proper cleanup in your dispose() method
- Use `renderer.shadowMap.enabled = true` for shadows
- WebGL has context limits - watch memory usage
- Consider using `BufferGeometry` instead of `Geometry` for better performance
- Use `Group` to manage multiple objects together

## Performance Tips
```javascript
// Merge geometries when possible
const geometries = [];
meshes.forEach(mesh => {
    mesh.updateMatrix();
    geometries.push(mesh.geometry.clone().applyMatrix4(mesh.matrix));
});
const mergedGeometry = BufferGeometryUtils.mergeBufferGeometries(geometries);
```

## Debug Helpers
```javascript
// Add helpers for development
const axesHelper = new THREE.AxesHelper(5);
scene.add(axesHelper);

const gridHelper = new THREE.GridHelper(10, 10);
scene.add(gridHelper);
```

## Common Patterns
- Use `requestAnimationFrame` for smooth animations
- Implement proper cleanup with `dispose()`
- Use object pooling for particle systems
- Implement level-of-detail (LOD) for complex scenes
- Use `raycaster` for object interaction

## Security Considerations
- Validate model files before loading
- Be cautious with user-provided URLs in loaders
- Implement proper error handling for asset loading
- Use content security policy (CSP) headers
- Be mindful of memory leaks in animation loops

Remember:
- Three.js coordinates are right-handed
- Y-axis is up by default
- Units are arbitrary but consistent within your scene
- WebGL context can be lost - implement proper handling
- Monitor GPU memory usage in production

This is a basic overview - Three.js is extensive and has many more features. Always check the official documentation for detailed information and updates.