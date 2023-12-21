<template>
  <div ref="mount" class="second-animation">
    <p class="second-text-animation">Drawing lines</p>
  </div>
</template>

<script>
import * as THREE from "three";

export default {
  mounted() {
    this.initThree(); // Initialize the Three.js scene
    this.animate(); // Start the animation
    this.onWindowResize(); // Adjust the scene to the window size
    window.addEventListener("resize", this.onWindowResize); // Listen for window resize events
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.onWindowResize); // Stop listening for window resize events
    this.renderer.dispose(); // Dispose the renderer
    this.scene.dispose(); // Dispose the scene
    this.line.material.dispose(); // Dispose the line material
    this.line.geometry.dispose(); // Dispose the line geometry
    cancelAnimationFrame(this.animationFrameId); // Cancel the animation frame
  },
  methods: {
    initThree() {
      this.scene = new THREE.Scene(); // Create a new scene
      this.camera = new THREE.PerspectiveCamera(
        75,
        this.$refs.mount.clientWidth / this.$refs.mount.clientHeight,
        0.1,
        1000
      ); // Create a new camera
      this.renderer = new THREE.WebGLRenderer(); // Create a new renderer
      this.renderer.setClearColor(0xffffff); // Set the clear color to white
      this.renderer.setSize(
        this.$refs.mount.clientWidth,
        this.$refs.mount.clientHeight
      ); // Set the size of the renderer
      this.renderer.setPixelRatio(window.devicePixelRatio * 2); // Set the pixel ratio of the renderer
      this.$refs.mount.appendChild(this.renderer.domElement); // Append the renderer to the mount element

      const material = new THREE.LineBasicMaterial({ color: 0x0000ff }); // Create a new line material
      const points = []; // Create an array for the points
      points.push(new THREE.Vector3(-1, 0, 0)); // Add the first point
      points.push(new THREE.Vector3(0, 1, 0)); // Add the second point
      points.push(new THREE.Vector3(1, 0, 0)); // Add the third point
      const geometry = new THREE.BufferGeometry().setFromPoints(points); // Create a new geometry from the points
      this.line = new THREE.Line(geometry, material); // Create a new line from the geometry and material
      this.scene.add(this.line); // Add the line to the scene
      this.camera.position.z = 2; // Set the z position of the camera
    },
    animate() {
      this.animationFrameId = requestAnimationFrame(this.animate); // Request a new animation frame
      this.renderer.render(this.scene, this.camera); // Render the scene with the camera
    },
    onWindowResize() {
      this.camera.aspect =
        this.$refs.mount.clientWidth / this.$refs.mount.clientHeight; // Update the camera aspect ratio
      this.camera.updateProjectionMatrix(); // Update the camera projection matrix
      this.renderer.setSize(
        this.$refs.mount.clientWidth,
        this.$refs.mount.clientHeight
      ); // Update the size of the renderer
    },
  },
};
</script>

<style lang="scss">
@import "./style.scss";
</style>
