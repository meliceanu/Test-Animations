<template>
  <div ref="mount" class="third-animation">
    <p class="third-text-animation">Creating text</p>
  </div>
</template>

<script>
import * as THREE from "three";
import { TextGeometry } from "three/examples/jsm/geometries/TextGeometry.js";
import { FontLoader } from "three/examples/jsm/loaders/FontLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

const CAMERA_PARAMS = { fov: 75, near: 0.1, far: 1000 };
const TEXT_PARAMS = { size: 1, height: 0.2 };
const CONTROLS_PARAMS = { minDistance: 1, maxDistance: 10 };
const ROTATION_SPEED = 0.001;

export default {
  mounted() {
    this.initScene(); // Initialize the scene
    window.addEventListener("resize", this.onWindowResize); // Listen for window resize events
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.onWindowResize); // Stop listening for window resize events
    this.cleanup(); // Cleanup the scene
  },
  methods: {
    initScene() {
      this.setupScene(); // Setup the scene
      this.setupCamera(); // Setup the camera
      this.setupRenderer(); // Setup the renderer
      this.loadText(); // Load the text
    },
    setupScene() {
      this.scene = new THREE.Scene(); // Create a new scene
    },
    setupCamera() {
      this.camera = new THREE.PerspectiveCamera(
        CAMERA_PARAMS.fov,
        this.$refs.mount.clientWidth / this.$refs.mount.clientHeight,
        CAMERA_PARAMS.near,
        CAMERA_PARAMS.far
      ); // Create a new camera
    },
    setupRenderer() {
      this.renderer = new THREE.WebGLRenderer({ alpha: true }); // Create a new renderer with alpha
      this.renderer.setClearColor(0xffffff, 1); // Set the clear color to white
      this.renderer.setSize(
        this.$refs.mount.clientWidth,
        this.$refs.mount.clientHeight
      ); // Set the size of the renderer
      this.renderer.setPixelRatio(window.devicePixelRatio * 2); // Set the pixel ratio of the renderer

      this.$refs.mount.appendChild(this.renderer.domElement); // Append the renderer to the mount element
    },
    loadText() {
      const loader = new FontLoader(); // Create a new font loader

      loader.load(
        "https://threejs.org/examples/fonts/helvetiker_regular.typeface.json",
        (font) => {
          const geometry = new TextGeometry("Hello three.js!", {
            font: font,
            size: TEXT_PARAMS.size,
            height: TEXT_PARAMS.height,
          }); // Create a new text geometry

          geometry.computeBoundingBox(); // Compute the bounding box of the geometry
          geometry.center(); // Center the geometry

          const material = new THREE.MeshPhongMaterial({ color: 0x000000 }); // Create a new phong material

          this.text = new THREE.Mesh(geometry, material); // Create a new mesh with the geometry and material
          this.scene.add(this.text); // Add the text to the scene

          this.camera.position.z = 5; // Set the z position of the camera

          this.controls = new OrbitControls(
            this.camera,
            this.renderer.domElement
          ); // Create new orbit controls
          this.controls.minDistance = CONTROLS_PARAMS.minDistance; // Set the minimum distance of the controls
          this.controls.maxDistance = CONTROLS_PARAMS.maxDistance; // Set the maximum distance of the controls

          // Start the animation loop after the text geometry is created and added to the scene
          this.animate();
        },
        undefined,
        (error) => {
          console.error("An error occurred while loading the font:", error);
          // Show an error message to the user or stop the execution of the initScene method
        }
      );
    },
    animate() {
      this.animationFrameId = requestAnimationFrame(this.animate); // Request a new animation frame

      if (this.text) {
        this.text.rotation.y = Math.sin(Date.now() * ROTATION_SPEED) * 0.5; // Rotate the text around the y-axis
        this.text.rotation.x = Math.sin(Date.now() * ROTATION_SPEED) * 0.5; // Rotate the text around the x-axis
      }

      if (this.controls) {
        this.controls.update(); // Update the controls
      }

      this.renderer.render(this.scene, this.camera); // Render the scene with the camera
    },
    onWindowResize() {
      if (this.camera && this.renderer) {
        this.camera.aspect =
          this.$refs.mount.clientWidth / this.$refs.mount.clientHeight; // Update the camera aspect ratio
        this.camera.updateProjectionMatrix(); // Update the camera projection matrix
        this.renderer.setSize(
          this.$refs.mount.clientWidth,
          this.$refs.mount.clientHeight
        ); // Update the size of the renderer
      }
    },
    cleanup() {
      cancelAnimationFrame(this.animationFrameId); // Cancel the animation frame
      this.renderer.dispose(); // Dispose the renderer
      this.scene.dispose(); // Dispose the scene
      this.controls.dispose(); // Dispose the controls
      if (this.text) {
        this.text.geometry.dispose(); // Dispose the text geometry
        this.text.material.dispose(); // Dispose the text material
      }
    },
  },
};
</script>

<style lang="scss">
@import "./style.scss";
</style>
