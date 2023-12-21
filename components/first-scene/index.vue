<template>
  <div ref="mount" class="first-animation">
    <p class="first-text-animation">Creating a scene</p>
  </div>
</template>

<script>
import * as THREE from "three";

export default {
  data() {
    return {
      animationFrameId: null, // ID of the animation frame
      renderer: null, // Renderer for the scene
    };
  },
  mounted() {
    this.initThree(); // Initialize the Three.js scene when the component is mounted
  },
  beforeDestroy() {
    this.cleanupThree(); // Cleanup the Three.js scene before the component is destroyed
  },
  methods: {
    initThree() {
      try {
        const instance = getCurrentInstance(); // Get the current Vue instance
        instance.proxy.$scene = new THREE.Scene(); // Create a new Three.js scene
        const camera = new THREE.PerspectiveCamera(75, 300 / 300, 0.1, 1000); // Create a new camera
        this.renderer = new THREE.WebGLRenderer({ antialias: true }); // Create a new renderer with antialiasing
        this.renderer.shadowMap.enabled = true; // Enable shadow mapping
        this.renderer.setClearColor(0xffffff); // Set the clear color to white
        this.renderer.setSize(300, 300); // Set the size of the renderer
        this.renderer.setPixelRatio(window.devicePixelRatio * 2); // Set the pixel ratio of the renderer
        this.$refs.mount.appendChild(this.renderer.domElement); // Append the renderer to the mount element
        const geometry = new THREE.BoxGeometry(3, 3, 3); // Create a new box geometry
        const material = new THREE.MeshStandardMaterial({ color: 0xffffff }); // Create a new standard material
        const cube = new THREE.Mesh(geometry, material); // Create a new mesh with the geometry and material
        cube.castShadow = true; // Enable shadow casting for the cube
        cube.receiveShadow = true; // Enable shadow receiving for the cube
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5); // Create a new directional light
        directionalLight.position.set(1, 0, 1); // Set the position of the light
        instance.proxy.$scene.add(directionalLight); // Add the light to the scene
        instance.proxy.$scene.add(cube); // Add the cube to the scene
        camera.position.z = 5.5; // Set the z position of the camera
        const animate = () => {
          this.animationFrameId = requestAnimationFrame(animate); // Request a new animation frame
          cube.rotation.x += 0.01; // Rotate the cube around the x-axis
          cube.rotation.y += 0.01; // Rotate the cube around the y-axis
          this.renderer.render(instance.proxy.$scene, camera); // Render the scene with the camera
        };
        animate(); // Start the animation
      } catch (error) {
        console.error(
          "An error occurred while initializing the Three.js scene:",
          error
        );
      }
    },
    cleanupThree() {
      cancelAnimationFrame(this.animationFrameId); // Cancel the animation frame
      const instance = getCurrentInstance(); // Get the current Vue instance
      while (instance.proxy.$scene.children.length > 0) {
        instance.proxy.$scene.remove(instance.proxy.$scene.children[0]); // Remove all children from the scene
      }
      this.renderer.dispose(); // Dispose the renderer
    },
  },
};
</script>

<style lang="scss">
@import "./style.scss";
</style>
