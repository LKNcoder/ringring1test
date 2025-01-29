import * as THREE from 'three';
import {
    FontLoader
} from 'three/addons/loaders/FontLoader.js';
import {
    TextGeometry
} from 'three/addons/geometries/TextGeometry.js';
import {
    OrbitControls
} from 'three/addons/controls/OrbitControls.js';
import {
    VRButton
} from 'three/addons/webxr/VRButton.js';
import {
    XRControllerModelFactory
} from 'three/addons/webxr/XRControllerModelFactory.js';
import {
    EffectComposer
} from 'three/addons/postprocessing/EffectComposer.js';
import {
    RenderPass
} from 'three/addons/postprocessing/RenderPass.js';
import {
    UnrealBloomPass
} from 'three/addons/postprocessing/UnrealBloomPass.js';
import {
    SMAAPass
} from 'three/addons/postprocessing/SMAAPass.js';

// Initialize loading manager
let assetsLoaded = false;
const loadingManager = new THREE.LoadingManager(
    // onLoad callback
    function() {
        console.log('Loading complete!');
        assetsLoaded = true;
        loadingScreen.style.display = 'none';
        // Add camera listener for audio after loading
        camera.add(audioListener);
    },
    // onProgress callback
    function(url, itemsLoaded, itemsTotal) {
        const progress = (itemsLoaded / itemsTotal * 100).toFixed(0);
        loadingScreen.innerHTML = `Loading... ${progress}%`;
    },
    // onError callback
    function(url) {
        console.error('Error loading ' + url);
        assetsLoaded = true;
        loadingScreen.style.display = 'none';
    }
);
const textureLoader = new THREE.TextureLoader(loadingManager);
// Get the main render canvas and parent div first
const parentDiv = document.getElementById('renderDiv');
let canvas = document.getElementById('threeRenderCanvas');
if (!canvas) {
    canvas = document.createElement('canvas');
    canvas.id = 'threeRenderCanvas';
    parentDiv.appendChild(canvas);
}
// Create text variables
let levelMesh;
let timerMesh;
let textMaterial;
// Function to update timer text
function updateTextDisplay(timerString) {
    if (timerMesh) {
        ringMesh.remove(timerMesh);
    }

    const loader = new FontLoader();
    loader.load('https://threejs.org/examples/fonts/helvetiker_regular.typeface.json', function(font) {
        const textGeometry = new TextGeometry(timerString, {
            font: font,
            size: 0.1,
            height: 0.01,
            curveSegments: 32
        });

        textGeometry.computeBoundingBox();
        textGeometry.center();

        timerMesh = new THREE.Mesh(textGeometry, textMaterial);
        timerMesh.position.set(0, 0.4, 0);
        ringMesh.add(timerMesh);
    });
}
// Create loading screen
const loadingScreen = document.createElement('div');
loadingScreen.style.position = 'absolute';
loadingScreen.style.top = '0';
loadingScreen.style.left = '0';
loadingScreen.style.width = '100%';
loadingScreen.style.height = '100%';
loadingScreen.style.backgroundColor = 'rgba(0, 0, 0, 0.8)';
loadingScreen.style.color = 'white';
loadingScreen.style.display = 'flex';
loadingScreen.style.justifyContent = 'center';
loadingScreen.style.alignItems = 'center';
loadingScreen.style.fontSize = '24px';
loadingScreen.innerHTML = 'Loading... 0%';
parentDiv.appendChild(loadingScreen);

// Loading manager events
// Loading screen setup only
loadingScreen.innerHTML = 'Loading... 0%';

// Initialize the scene
const scene = new THREE.Scene();
const world = new CANNON.World();
// Audio setup
const audioListener = new THREE.AudioListener();
const sound = new THREE.PositionalAudio(audioListener);
const audioLoader = new THREE.AudioLoader();
// Game state
let currentLevel = 1;
let isRingGrabbed = false;
let startTime = 0;
let bestTime = Infinity;
let currentTime = 0;

function formatTime(timeInMs) {
    const minutes = Math.floor(timeInMs / 60000);
    const seconds = Math.floor((timeInMs % 60000) / 1000);
    const milliseconds = Math.floor((timeInMs % 1000) / 10);
    return `${minutes}:${seconds.toString().padStart(2, '0')}:${milliseconds.toString().padStart(2, '0')}`;
}
const startPosition = new THREE.Vector3(0, 1.4, 0.75); // Position ring to encircle the wire
world.gravity.set(0, 0, 0); // Remove gravity
world.defaultContactMaterial.friction = 0.01; // Reduced global friction
world.defaultContactMaterial.restitution = 0; // No bounce by default
world.solver.iterations = 10;
world.solver.tolerance = 0.001;

// Define physics materials
const groundPhysMaterial = new CANNON.Material('ground');
const ballPhysMaterial = new CANNON.Material('ballMaterial');

// Create contact material between ball and ground with high restitution
const ballGroundContactMaterial = new CANNON.ContactMaterial(
    ballPhysMaterial,
    groundPhysMaterial, {
        friction: 0.05, // Lower friction for smoother rolling
        restitution: 0.99 // Very high restitution for bouncy ball
    }
);
world.addContactMaterial(ballGroundContactMaterial);

const clock = new THREE.Clock();

// Initialize the camera
const camera = new THREE.PerspectiveCamera(
    75,
    canvas.offsetWidth / canvas.offsetHeight,
    0.1,
    1000
);
// Adjusted camera position to better view the big ball
camera.position.set(0, 2, 5);
camera.lookAt(0, 1.6, 0);

// Initialize the renderer with HDR
const renderer = new THREE.WebGLRenderer({
    antialias: false, // Disable antialiasing initially
    canvas: canvas,
    alpha: true,
    powerPreference: "default",
    xr: {
        enabled: true
    },
    failIfMajorPerformanceCaveat: false,
    precision: "mediump",
    depth: true,
    stencil: false
});
document.body.appendChild(VRButton.createButton(renderer));
// VR Controller setup
const controllerModelFactory = new XRControllerModelFactory();
// Controllers
const controller1 = renderer.xr.getController(0);
const controller2 = renderer.xr.getController(1);
const controllerGrip1 = renderer.xr.getControllerGrip(0);
const controllerGrip2 = renderer.xr.getControllerGrip(1);
controllerGrip1.add(controllerModelFactory.createControllerModel(controllerGrip1));
controllerGrip2.add(controllerModelFactory.createControllerModel(controllerGrip2));
scene.add(controller1);
scene.add(controller2);
scene.add(controllerGrip1);
scene.add(controllerGrip2);
// Add ray visualizers
const geometry = new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(0, 0, 0),
    new THREE.Vector3(0, 0, -1)
]);
const material = new THREE.LineBasicMaterial({
    color: 0xffffff
});
const line1 = new THREE.Line(geometry, material);
const line2 = new THREE.Line(geometry, material);
line1.scale.z = 5;
line2.scale.z = 5;
controller1.add(line1);
controller2.add(line2);
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setSize(canvas.offsetWidth, canvas.offsetHeight);
renderer.shadowMap.enabled = false; // Disable shadows initially
renderer.shadowMap.type = THREE.BasicShadowMap; // Use basic shadow map
renderer.outputEncoding = THREE.sRGBEncoding;
renderer.toneMapping = THREE.NoToneMapping; // Disable tone mapping initially
renderer.toneMappingExposure = 1.0;

// Initialize post-processing
const composer = new EffectComposer(renderer);

// Regular scene render pass
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// Add subtle bloom effect
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    0.2, // bloom strength
    0.4, // radius
    0.9 // threshold
);
composer.addPass(bloomPass);

// Add anti-aliasing
const smaaPass = new SMAAPass(
    window.innerWidth * renderer.getPixelRatio(),
    window.innerHeight * renderer.getPixelRatio()
);
composer.addPass(smaaPass);

// Initialize composer size
composer.setSize(parentDiv.clientWidth, parentDiv.clientHeight);



// Remove sky mesh as we want complete darkness
// scene.add(sky);

// Modify OrbitControls setup
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.25; // Adjusted damping factor
controls.screenSpacePanning = false;
controls.maxPolarAngle = Math.PI / 2;

// Create sci-fi environment
function createEnvironment() {
    // Load trippy sci-fi skybox
    try {
        // Load texture with error callback
        const texture = textureLoader.load(
            'https://play.rosebud.ai/assets/amazing trippy scifi skybox.png?zRdV',
            function(texture) {
                texture.mapping = THREE.EquirectangularReflectionMapping;
                scene.environment = texture;
                renderer.toneMappingExposure = 0.05;
                scene.background = new THREE.Color(0x000000);
            }
        );
    } catch (error) {
        console.error('Error in texture loading setup:', error);
        renderer.toneMappingExposure = 0.05;
        scene.background = new THREE.Color(0x000000);
    }

    // Add some floating particles for ambiance
    const particlesGeometry = new THREE.BufferGeometry();
    const particlesCount = 3000;
    const posArray = new Float32Array(particlesCount * 3);
    for (let i = 0; i < particlesCount * 3; i += 3) {
        // Generate position
        let x = (Math.random() - 0.5) * 100;
        let y = (Math.random() - 0.5) * 100;
        let z = (Math.random() - 0.5) * 100;

        // Calculate distance from wire center (0, 1.4, 0)
        const distanceFromWire = Math.sqrt(
            Math.pow(x - 0, 2) +
            Math.pow(y - 1.4, 2) +
            Math.pow(z - 0, 2)
        );

        // If too close to wire, move particle further out
        if (distanceFromWire < 10) {
            const scale = 10 / distanceFromWire;
            x *= scale;
            y *= scale;
            z *= scale;
        }

        posArray[i] = x;
        posArray[i + 1] = y;
        posArray[i + 2] = z;
    }

    particlesGeometry.setAttribute('position', new THREE.BufferAttribute(posArray, 3));
    const particlesMaterial = new THREE.PointsMaterial({
        size: 0.2,
        color: 0x00ffff,
        transparent: true,
        opacity: 0.8,
        blending: THREE.AdditiveBlending
    });

    const particlesMesh = new THREE.Points(particlesGeometry, particlesMaterial);
    scene.add(particlesMesh);

    return particlesMesh;
}

// Setup improved lighting system
const sunLight = new THREE.DirectionalLight(0xffffff, 1.0);
sunLight.position.set(5, 5, 5);
sunLight.castShadow = true;
sunLight.shadow.mapSize.width = 2048;
sunLight.shadow.mapSize.height = 2048;
sunLight.shadow.camera.near = 10;
sunLight.shadow.camera.far = 400;
sunLight.shadow.camera.left = -100;
sunLight.shadow.camera.right = 100;
sunLight.shadow.camera.top = 100;
sunLight.shadow.camera.bottom = -100;
sunLight.shadow.bias = -0.001;
scene.add(sunLight);

// Add hemisphere light to simulate sky and ground bounce light
// Add strong ambient light to ensure basic visibility
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);
const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 0.8);
scene.add(hemiLight);

// Create initial scene
const environment = createEnvironment();
// Create glowing materials
const wireMaterial = new THREE.MeshStandardMaterial({
    color: new THREE.Color(0x00ff00), // Start with green
    emissive: new THREE.Color(0x00ff00),
    emissiveIntensity: 20,
    metalness: 1.0,
    roughness: 0.0
});
const ringMaterial = new THREE.MeshStandardMaterial({
    color: 0x800080, // Pure purple
    emissive: 0xff69b4, // Hot pink emissive
    emissiveIntensity: 1.0,
    metalness: 1.0,
    roughness: 0.0
});
// Create a randomly bent wire
function createWire() {
    const points = [];
    const segments = 200;
    const radius = 0.75;

    for (let i = 0; i <= segments; i++) {
        const t = i / segments;
        const angle = t * Math.PI * 2;

        // Add wave variations (matching green particle path)
        const waveX = Math.sin(angle * 8) * 0.05;
        const waveY = Math.cos(angle * 6) * 0.05;

        // Calculate position with waves
        const x = (Math.cos(angle) * radius) + waveX;
        const y = 1.4 + waveY;
        const z = (Math.sin(angle) * radius) + waveX;

        points.push(new THREE.Vector3(x, y, z));
    }

    const curve = new THREE.CatmullRomCurve3(points);
    const tubeGeometry = new THREE.TubeGeometry(curve, segments, 0.0125, 16, false); // 0.025m/2 radius, more radial segments
    const wireMesh = new THREE.Mesh(tubeGeometry, wireMaterial);
    scene.add(wireMesh);

    // Create physics body for the wire
    const wireShape = new CANNON.Cylinder(0.2, 0.2, 100, 8);
    const wireBody = new CANNON.Body({
        mass: 0
    });
    wireBody.addShape(wireShape);
    world.addBody(wireBody);

    return {
        mesh: wireMesh,
        body: wireBody
    };
}
// Create a ring
function createRing() {
    const ringGeometry = new THREE.TorusGeometry(0.1, 0.015, 32, 32);
    const ringMesh = new THREE.Mesh(ringGeometry, ringMaterial);
    scene.add(ringMesh);

    // Create physics body for the ring
    const ringShape = new CANNON.Sphere(0.09); // Slightly larger than visual size for better collision
    const ringBody = new CANNON.Body({
        mass: 1
    });
    ringBody.addShape(ringShape);
    // Position ring so wire passes through its center
    ringBody.position.copy(startPosition); // Match the start position
    ringBody.quaternion.setFromAxisAngle(new CANNON.Vec3(0, 1, 0), Math.PI / 2); // Rotate ring 90 degrees around Y axis
    world.addBody(ringBody);

    return {
        mesh: ringMesh,
        body: ringBody
    };
}
// Create particle system for objects
function createObjectParticles(object, color) {
    const particlesGeometry = new THREE.BufferGeometry();
    const particleCount = 100;
    const positions = new Float32Array(particleCount * 3);
    const velocities = new Float32Array(particleCount * 3);

    for (let i = 0; i < particleCount * 3; i += 3) {
        positions[i] = 0;
        positions[i + 1] = 0;
        positions[i + 2] = 0;

        velocities[i] = (Math.random() - 0.5) * 0.01;
        velocities[i + 1] = (Math.random() - 0.5) * 0.01;
        velocities[i + 2] = (Math.random() - 0.5) * 0.01;
    }

    particlesGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    particlesGeometry.setAttribute('velocity', new THREE.BufferAttribute(velocities, 3));

    const particlesMaterial = new THREE.PointsMaterial({
        size: 0.01,
        color: color,
        transparent: true,
        opacity: 0.6,
        blending: THREE.AdditiveBlending
    });

    const particles = new THREE.Points(particlesGeometry, particlesMaterial);
    object.add(particles);
    return particles;
}

// Create hello text variables
let helloMesh;
let spriteText;

function createHelloText() {
    // Create an emissive material
    // Create global material for all text
    textMaterial = new THREE.MeshStandardMaterial({
        color: 0x00ffff,
        emissive: 0x00ffff,
        emissiveIntensity: 20,
        metalness: 0,
        roughness: 0,
        side: THREE.DoubleSide
    });
    // Load font and create 3D text
    const loader = new FontLoader();
    // Timer text creation moved to updateTextDisplay function
}
// Create game objects
const wire = createWire();
const ring = createRing();
// Create green particle path
function createGreenParticlePath() {
    const particlesGeometry = new THREE.BufferGeometry();
    const particleCount = 1;
    const positions = new Float32Array(particleCount * 3);
    return new THREE.Points(particlesGeometry, new THREE.PointsMaterial({
        color: 0x00ff00,
        size: 0.01
    }));
}
const greenParticlePath = createGreenParticlePath();
// Create sprite text
let timerSprite;

function createSpriteText() {
    // Create hello sprite
    const canvas = document.createElement('canvas');
    const context = canvas.getContext('2d');
    canvas.width = 256;
    canvas.height = 256;
    // Setup text style
    context.font = 'Bold 40px Arial';
    context.fillStyle = '#00ffff';
    context.textAlign = 'center';
    context.textBaseline = 'middle';
    // Draw text with current level
    context.fillText(`Level ${currentLevel}`, 128, 128);
    // Create sprite
    const texture = new THREE.CanvasTexture(canvas);
    const spriteMaterial = new THREE.SpriteMaterial({
        map: texture,
        transparent: true,
        opacity: 1
    });
    spriteText = new THREE.Sprite(spriteMaterial);
    spriteText.scale.set(0.35, 0.35, 1); // Scaled down from 0.5 to 0.35
    spriteText.position.set(0, 0.25, 0);
    ringMesh.add(spriteText);
    // Create timer sprite
    const timerCanvas = document.createElement('canvas');
    const timerContext = timerCanvas.getContext('2d');
    timerCanvas.width = 256;
    timerCanvas.height = 256;
    // Setup timer text style
    timerContext.font = 'Bold 40px Arial';
    timerContext.fillStyle = '#00ffff';
    timerContext.textAlign = 'center';
    timerContext.textBaseline = 'middle';
    // Initial timer text
    timerContext.fillText('0:00:00', 128, 128);
    // Create timer sprite
    const timerTexture = new THREE.CanvasTexture(timerCanvas);
    const timerMaterial = new THREE.SpriteMaterial({
        map: timerTexture,
        transparent: true,
        opacity: 1
    });
    timerSprite = new THREE.Sprite(timerMaterial);
    timerSprite.scale.set(0.35, 0.35, 1); // Scaled down from 0.5 to 0.35
    timerSprite.position.set(0, 0.18, 0);
    ringMesh.add(timerSprite);
}
// Variables for the game objects
let wireMesh = wire.mesh;
let wireBody = wire.body;
let ringMesh = ring.mesh;
let ringBody = ring.body;
// Now that ringMesh is initialized, create text elements
createHelloText();
createSpriteText();
// Particle systems removed
// Initialize sound placeholder
sound.setVolume(0);
ringMesh.add(sound);
// Reset ring position
function resetGameForNextLevel() {
    // Increment level
    currentLevel++;

    // Reset scene background
    scene.background = new THREE.Color(0x000000);

    // Reset wire color
    wireMaterial.color.setHex(0x00ff00);
    wireMaterial.emissive.setHex(0x00ff00);

    // Reset ring
    ringBody.position.copy(startPosition);
    ringBody.velocity.set(0, 0, 0);
    ringBody.angularVelocity.set(0, 0, 0);
    ringMesh.visible = true;

    // Reset timer
    startTime = 0;
    currentTime = 0;
    isRingGrabbed = false;

    // Update level text
    const canvas = spriteText.material.map.image;
    const context = canvas.getContext('2d');
    context.clearRect(0, 0, canvas.width, canvas.height);
    context.font = 'Bold 40px Arial';
    context.fillStyle = '#00ffff';
    context.textAlign = 'center';
    context.textBaseline = 'middle';
    context.fillText(`Level ${currentLevel}`, canvas.width / 2, canvas.height / 2);
    spriteText.material.map.needsUpdate = true;

    // Show text and timer
    spriteText.visible = true;
    timerSprite.visible = true;
}

function resetRing() {
    ringBody.position.copy(startPosition);
    ringBody.velocity.set(0, 0, 0);
    ringBody.angularVelocity.set(0, 0, 0);
    isRingGrabbed = false;
    // Note: We're not resetting startTime here to keep the timer running
}
// Controller event listeners
function onSelectStart(event) {
    const controller = event.target;
    const controllerPosition = new THREE.Vector3();
    controller.getWorldPosition(controllerPosition);

    // Check if controller is close enough to ring to grab it
    if (controllerPosition.distanceTo(ringMesh.position) < 0.5) {
        isRingGrabbed = true;
        controller.userData.selectedObject = ringBody;
        startTime = Date.now();
    }
}

function onSelectEnd(event) {
    const controller = event.target;
    if (controller.userData.selectedObject) {
        controller.userData.selectedObject = null;
        isRingGrabbed = false;
    }
}
controller1.addEventListener('selectstart', onSelectStart);
controller1.addEventListener('selectend', onSelectEnd);
controller2.addEventListener('selectstart', onSelectStart);
controller2.addEventListener('selectend', onSelectEnd);



function updateObjectParticles(particles) {
    const positions = particles.geometry.attributes.position.array;
    const velocities = particles.geometry.attributes.velocity.array;

    for (let i = 0; i < positions.length; i += 3) {
        // Update positions based on velocity
        positions[i] += velocities[i];
        positions[i + 1] += velocities[i + 1];
        positions[i + 2] += velocities[i + 2];

        // Reset particles that move too far from origin
        const distance = Math.sqrt(
            positions[i] ** 2 +
            positions[i + 1] ** 2 +
            positions[i + 2] ** 2
        );

        if (distance > 0.2) {
            positions[i] = 0;
            positions[i + 1] = 0;
            positions[i + 2] = 0;
        }
    }

    particles.geometry.attributes.position.needsUpdate = true;
}

function animate() {
    requestAnimationFrame(animate);
    const deltaTime = Math.min(clock.getDelta(), 0.1);
    // Make timer text face camera
    // Timer mesh orientation code removed

    if (assetsLoaded) {
        world.step(deltaTime);

        // Update ball position and rotation
        // Update ring position
        ringMesh.position.copy(ringBody.position);
        ringMesh.quaternion.copy(ringBody.quaternion);
        // Check for collisions between ring and wire
        const ringPosition = ringMesh.position;
        const collision = world.collisionMatrix.get(ringBody, wireBody);

        // Update ring position when grabbed
        if (isRingGrabbed) {
            const controller = controller1.userData.selectedObject ? controller1 : controller2;
            const controllerPosition = new THREE.Vector3();
            controller.getWorldPosition(controllerPosition);
            ringBody.position.copy(controllerPosition);

            // Check if player has completed the circuit (progress is 1)
            if (progress >= 0.99) { // Using 0.99 to account for floating-point precision
                // Play victory sounds
                const applauseSound = new THREE.Audio(audioListener);
                const chimeSound = new THREE.PositionalAudio(audioListener);

                // Load and play applause (global)
                audioLoader.load('https://play.rosebud.ai/assets/short_applause.mp3?pLx6', function(buffer) {
                    applauseSound.setBuffer(buffer);
                    applauseSound.setVolume(1.0);
                    applauseSound.play();
                });

                // Load and play chime (from ring position)
                audioLoader.load('https://play.rosebud.ai/assets/magic_chime-SFX.mp3?OO3H', function(buffer) {
                    chimeSound.setBuffer(buffer);
                    chimeSound.setVolume(1.0);
                    chimeSound.setRefDistance(1);
                    ringMesh.add(chimeSound);
                    chimeSound.play();
                });
                // Create and trigger confetti particles
                const particleCount = 1000;
                const colors = [0xff0000, 0x00ff00, 0x0000ff, 0xff00ff, 0xffff00, 0x00ffff];
                const particles = new THREE.BufferGeometry();
                const positions = new Float32Array(particleCount * 3);
                const velocities = new Float32Array(particleCount * 3);
                const particleColors = new Float32Array(particleCount * 3);
                for (let i = 0; i < particleCount * 3; i += 3) {
                    // Start at ring position
                    positions[i] = ringBody.position.x;
                    positions[i + 1] = ringBody.position.y;
                    positions[i + 2] = ringBody.position.z;
                    // Random velocities for explosion effect
                    velocities[i] = (Math.random() - 0.5) * 2;
                    velocities[i + 1] = Math.random() * 2; // Upward bias
                    velocities[i + 2] = (Math.random() - 0.5) * 2;
                    // Random colors
                    const color = new THREE.Color(colors[Math.floor(Math.random() * colors.length)]);
                    particleColors[i] = color.r;
                    particleColors[i + 1] = color.g;
                    particleColors[i + 2] = color.b;
                }
                particles.setAttribute('position', new THREE.BufferAttribute(positions, 3));
                particles.setAttribute('velocity', new THREE.BufferAttribute(velocities, 3));
                particles.setAttribute('color', new THREE.BufferAttribute(particleColors, 3));
                const particleMaterial = new THREE.PointsMaterial({
                    size: 0.05,
                    vertexColors: true,
                    transparent: true,
                    opacity: 1,
                    blending: THREE.AdditiveBlending
                });
                const particleSystem = new THREE.Points(particles, particleMaterial);
                scene.add(particleSystem);
                // Hide ring
                ringMesh.visible = false;
                // Stop the timer
                const finalTime = currentTime;

                // Fade to black and reset after delay
                setTimeout(() => {
                    const fadeToBlack = setInterval(() => {
                        scene.background.multiplyScalar(0.95);
                        particleMaterial.opacity *= 0.95;
                        if (scene.background.r < 0.01) {
                            clearInterval(fadeToBlack);
                            // Reset game state
                            resetGameForNextLevel();
                        }
                    }, 50);
                }, 3000);
            }
        }
        // Update timer
        if (isRingGrabbed && startTime > 0) {
            currentTime = Date.now() - startTime;
            // Update timer sprite
            if (timerSprite) {
                const canvas = timerSprite.material.map.image;
                const context = canvas.getContext('2d');

                // Clear canvas
                context.clearRect(0, 0, canvas.width, canvas.height);

                // Setup text style
                context.font = 'Bold 40px Arial';
                context.fillStyle = '#00ffff';
                context.textAlign = 'center';
                context.textBaseline = 'middle';

                // Draw updated time
                context.fillText(formatTime(currentTime), canvas.width / 2, canvas.height / 2);

                // Update texture
                timerSprite.material.map.needsUpdate = true;
            }
        }
        // Calculate progress based on ring position
        const startAngle = Math.atan2(startPosition.z - 0, startPosition.x - 0);
        const currentAngle = Math.atan2(ringBody.position.z - 0, ringBody.position.x - 0);
        let progress = ((currentAngle - startAngle) / (Math.PI * 2));
        if (progress < 0) progress += 1; // Normalize to 0-1 range
        // Interpolate wire color based on progress
        if (isRingGrabbed && !collision) {
            const startColor = new THREE.Color(0x00ff00); // Green
            const endColor = new THREE.Color(0xff69b4); // Pink
            const currentColor = new THREE.Color();
            currentColor.lerpColors(startColor, endColor, progress);

            wireMaterial.color.copy(currentColor);
            wireMaterial.emissive.copy(currentColor);
        }
        if (collision) {
            // Flash wire red
            wireMaterial.emissive.setHex(0xff0000);
            wireMaterial.color.setHex(0xff0000);
            wireMaterial.emissiveIntensity = 5;
            // Play explosion sound
            const explosionSound = new THREE.Audio(audioListener);
            audioLoader.load('https://play.rosebud.ai/assets/magic-explosion-SFX.mp3?lpSj', function(buffer) {
                explosionSound.setBuffer(buffer);
                explosionSound.setVolume(1.0);
                explosionSound.play();
            });
            // Hide ring, text and timer
            ringMesh.visible = false;
            spriteText.visible = false;
            timerSprite.visible = false;
            // Trigger haptic feedback
            if (controller1.userData.selectedObject || controller2.userData.selectedObject) {
                if (controller1.gamepad) controller1.gamepad.hapticActuators[0]?.pulse(0.5, 100);
                if (controller2.gamepad) controller2.gamepad.hapticActuators[0]?.pulse(0.5, 100);
            }
            // Reset ring position after a short delay
            setTimeout(() => {
                // Reset wire color and reset ring position
                wireMaterial.emissive.setHex(0x00ff00);
                wireMaterial.color.setHex(0x00ff00);
                wireMaterial.emissiveIntensity = 20;

                // Show ring, text and timer
                ringMesh.visible = true;
                spriteText.visible = true;
                timerSprite.visible = true;

                // Reset ring position
                resetRing();
            }, 1000); // 1 second delay
        } else {
            // Reset wire color to green on collision
            wireMaterial.emissive.setHex(0x00ff00);
            wireMaterial.color.setHex(0x00ff00);
            wireMaterial.emissiveIntensity = 20;
        }

        controls.update();
    }

    composer.render();
}

// Handle window resize
function onWindowResize() {
    const width = parentDiv.clientWidth;
    const height = parentDiv.clientHeight;

    // Update camera
    camera.aspect = width / height;
    camera.updateProjectionMatrix();

    // Update renderer and composer
    renderer.setSize(width, height);
    composer.setSize(width, height);

    // Update post-processing passes
    bloomPass.resolution.set(width, height);
    smaaPass.setSize(width, height);
}

// Add event listeners
window.addEventListener('resize', onWindowResize);
const resizeObserver = new ResizeObserver(onWindowResize);
resizeObserver.observe(parentDiv);

// Start animation
animate();
