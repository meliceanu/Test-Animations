<template>
  <!-- This div will serve as the mount point for our Three.js scene -->
  <div ref="mount" class="model-animation">
    <p class="model-text-animation">Loading 3D Models</p>
  </div>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"; // Import OrbitControls
import { onMounted, onBeforeUnmount, ref } from "vue";

export default {
  setup() {
    // Create a ref to the mount point in the DOM
    const mount = ref(null);
    let animateId = null;
    let renderer = null;
    let scene = null;
    let model = null;

    onMounted(() => {
      // Create a new Three.js scene
      scene = new THREE.Scene();

      // Create a new camera with a field of view of 75 degrees, aspect ratio of 1, and near and far clipping planes at 0.1 and 1000
      const camera = new THREE.PerspectiveCamera(75, 300 / 300, 0.1, 1000);

      // Create a new WebGL renderer
      renderer = new THREE.WebGLRenderer();
      renderer.setClearColor(0xffffff);
      renderer.setSize(300, 300);
      renderer.setPixelRatio(window.devicePixelRatio * 2);

      // Append the renderer's DOM element to the mount point
      mount.value.appendChild(renderer.domElement);

      // Create a new ambient light with a white color and an intensity of 0.5, and add it to the scene
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
      scene.add(ambientLight);

      // Create a new directional light with a white color and an intensity of 1, position it above the scene, and add it to the scene
      const directionalLight1 = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight1.position.set(0, 1, 0);
      scene.add(directionalLight1);

      // Create another directional light with a white color and an intensity of 1, position it below the scene, and add it to the scene
      const directionalLight2 = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight2.position.set(0, -1, 0);
      scene.add(directionalLight2);

      // Declare a variable to hold the 3D model

      // Create a new GLTFLoader to load the 3D model
      const loader = new GLTFLoader();

      // Load the 3D model
      loader.load(
        "/assets/2CylinderEngine.gltf",
        function (gltf) {
          // When the model has loaded, add it to the scene
          model = gltf.scene;
          scene.add(model);
        },
        undefined,
        function (error) {
          // If there's an error, log it to the console
          console.error("An error occurred while loading the model:", error);
        }
      );

      // Position the camera along the z-axis
      camera.position.z = 500;

      // Create new OrbitControls to allow the user to rotate the scene
      const controls = new OrbitControls(camera, renderer.domElement);

      // Define the animation loop
      const animate = function () {
        // Request the next frame of the animation
        animateId = requestAnimationFrame(animate);

        // If the model has loaded, rotate it
        if (model) {
          model.rotation.y = Math.sin(Date.now() * 0.001) * 0.5;
          model.rotation.x = Math.sin(Date.now() * 0.001) * 0.5;
        }

        // Update the controls
        controls.update();

        // Render the scene with the camera
        renderer.render(scene, camera);
      };

      // Start the animation loop
      animate();
    });

    onBeforeUnmount(() => {
      // Cancel the animation frame request
      cancelAnimationFrame(animateId);

      // Dispose of the Three.js resources
      renderer.dispose();
      scene.dispose();
      if (model) {
        model.geometry.dispose();
        model.material.dispose();
      }
    });

    // Return the mount ref so it can be used in the template
    return { mount };
  },
};
</script>

<style lang="scss">
@import "./style.scss";
</style>
