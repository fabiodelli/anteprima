<template>
  <div>
    <div ref="threeContainer"></div>
  </div>
</template>

<script>
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import * as THREE from 'three';

export default {
  data() {
    return {
      gltf: null,
      animations: [],
      mixer: null,
      animationStarted: false,
      lastFrame: null,
    };
  },
  mounted() {
    this.setupScene();
    this.setupLights();
    this.load3DModel();
    this.addEventListener();
  },
  beforeDestroy() {
    this.removeEventListener();
  },
  methods: {
    setupScene() {
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color(0x0c0c29);

      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      this.camera.position.set(0, 20, 45);

      const gridHelper = new THREE.GridHelper(100, 10);
      this.scene.add(gridHelper);

      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.domElement.style.position = 'absolute';
      this.renderer.domElement.style.top = '0';
      this.renderer.domElement.style.left = '0';
      this.$refs.threeContainer.appendChild(this.renderer.domElement);
    },
    setupLights() {
      const ambientLight = new THREE.AmbientLight(0x404040);
      ambientLight.intensity = 2;
      this.scene.add(ambientLight);

      const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight.position.set(1, 1, 1).normalize();
      directionalLight.castShadow = true;
      this.scene.add(directionalLight);

      directionalLight.shadow.mapSize.width = 1024;
      directionalLight.shadow.mapSize.height = 1024;
      directionalLight.shadow.camera.near = 0.5;
      directionalLight.shadow.camera.far = 500;
    },
    load3DModel() {
      const loader = new GLTFLoader();
      loader.load('/models/brain.gltf', (gltf) => {
        this.gltf = gltf;
        gltf.scene.scale.set(6, 6, 6);
        gltf.scene.position.set(0, 0, 0);
        gltf.scene.traverse((child) => {
          if (child.isMesh) {
            child.castShadow = true;
            child.receiveShadow = true;
          }
        });

        if (gltf.animations && gltf.animations.length > 0) {
          this.animations = gltf.animations;
          this.mixer = new THREE.AnimationMixer(gltf.scene);
          this.animations.forEach((animation) => {
            const action = this.mixer.clipAction(animation);
            action.play();
          });
        } else {
          console.error('Errore: Nessuna animazione trovata nel modello.');
        }

        if (gltf.scene) {
          this.scene.add(gltf.scene);
        } else {
          console.error('Errore: La scena del modello non è definita correttamente.');
        }

        this.renderer.render(this.scene, this.camera);
      });
    },
    addEventListener() {
      window.addEventListener('wheel', this.handleWheel);
    },
    removeEventListener() {
      window.removeEventListener('wheel', this.handleWheel);
    },
    handleWheel() {
      if (!this.animationStarted) {
        this.animationStarted = true;
        this.animate();
      }
    },
    animate() {
      requestAnimationFrame(this.animate);

      if (this.mixer) {
        this.mixer.update(0.01);
      }

      this.renderer.render(this.scene, this.camera);
    },
}
};
</script>
