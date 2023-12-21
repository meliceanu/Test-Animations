<template>
  <div ref="mount" class="physics-animation">
    <p class="physics-text-animation">Physics animation with Cannon-es</p>
  </div>
</template>

<script>
import * as THREE from "three";
import * as CANNON from "cannon-es";

export default {
  data() {
    return {
      world: null, // The physics world
      pingPongMaterial: null, // The material for the ping pong balls
      animationFrameId: null, // ID of the animation frame
      camera: null, // The camera
      renderer: null, // The renderer
    };
  },
  mounted() {
    this.init(); // Initialize the scene
    window.addEventListener("resize", this.onWindowResize); // Listen for window resize events
  },
  beforeDestroy() {
    cancelAnimationFrame(this.animationFrameId); // Cancel the animation frame
    window.removeEventListener("resize", this.onWindowResize); // Stop listening for window resize events
  },
  methods: {
    init() {
      const scene = this.createScene(); // Create the scene
      const camera = this.createCamera(); // Create the camera
      const renderer = this.createRenderer(); // Create the renderer

      this.$refs.mount.appendChild(renderer.domElement); // Append the renderer to the mount element

      const light = this.createLight(); // Create the light
      scene.add(light); // Add the light to the scene

      this.world = this.createWorld(); // Create the physics world
      this.pingPongMaterial = this.createMaterial(); // Create the material for the ping pong balls

      const groundBody = this.createGround(); // Create the ground
      this.world.addBody(groundBody); // Add the ground to the world

      const groundMesh = this.createGroundMesh(); // Create the ground mesh
      scene.add(groundMesh); // Add the ground mesh to the scene

      const spheres = this.createSpheres(scene); // Create the spheres

      this.animate(spheres, renderer, scene, camera); // Start the animation
    },
    onWindowResize() {
      this.camera.aspect =
        this.$refs.mount.offsetWidth / this.$refs.mount.offsetHeight; // Update the camera aspect ratio
      this.camera.updateProjectionMatrix(); // Update the camera projection matrix
      this.renderer.setSize(
        this.$refs.mount.offsetWidth,
        this.$refs.mount.offsetHeight
      ); // Update the size of the renderer
    },
    createScene() {
      return new THREE.Scene(); // Create a new scene
    },
    createCamera() {
      const aspectRatio =
        this.$refs.mount.offsetWidth / this.$refs.mount.offsetHeight; // Calculate the aspect ratio
      const camera = new THREE.PerspectiveCamera(90, aspectRatio, 0.1, 1000); // Create a new camera
      camera.position.set(0, 10, 20); // Set the position of the camera
      camera.lookAt(0, 0, 0); // Set the camera to look at the origin
      return camera;
    },
    createRenderer() {
      const renderer = new THREE.WebGLRenderer(); // Create a new renderer
      renderer.setSize(
        this.$refs.mount.offsetWidth,
        this.$refs.mount.offsetHeight
      ); // Set the size of the renderer
      renderer.setPixelRatio(window.devicePixelRatio * 2); // Set the pixel ratio of the renderer
      return renderer;
    },
    createLight() {
      const light = new THREE.DirectionalLight(0xffffff, 1); // Create a new directional light
      light.position.set(0, 20, 10); // Set the position of the light
      return light;
    },
    createWorld() {
      const world = new CANNON.World(); // Create a new physics world
      world.gravity.set(0, -70, 0); // Set the gravity of the world
      return world;
    },
    createMaterial() {
      const pingPongMaterial = new CANNON.Material("pingPongMaterial"); // Create a new material
      pingPongMaterial.restitution = 0.9; // Set the restitution of the material
      pingPongMaterial.friction = 0.1; // Set the friction of the material
      return pingPongMaterial;
    },
    createGround() {
      const groundShape = new CANNON.Box(new CANNON.Vec3(15, 0.1, 15)); // Create a new box shape for the ground
      const groundBody = new CANNON.Body({
        mass: 0,
        material: this.pingPongMaterial,
      }); // Create a new body for the ground
      groundBody.addShape(groundShape); // Add the shape to the body
      groundBody.position.y = -0.1; // Set the position of the ground body
      return groundBody;
    },
    createGroundMesh() {
      const groundMaterial = new THREE.MeshBasicMaterial({ color: 0x888888 }); // Create a new basic material for the ground mesh
      const groundMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(30, 30),
        groundMaterial
      ); // Create a new mesh for the ground
      groundMesh.rotation.x = -Math.PI / 2; // Rotate the ground mesh
      return groundMesh;
    },
    createSpheres(scene) {
      const sphereShape = new CANNON.Sphere(1); // Create a new sphere shape
      const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 }); // Create a new basic material for the spheres
      const spheres = [];
      for (let i = 0; i < 10; i++) {
        const sphereBody = new CANNON.Body({
          mass: 0.1,
          material: this.pingPongMaterial,
        }); // Create a new body for the sphere
        sphereBody.addShape(sphereShape); // Add the shape to the body
        sphereBody.position.set(
          Math.random() * 10 - 5,
          Math.random() * 5 + 10,
          Math.random() * 10 - 5
        ); // Set the position of the sphere body
        this.world.addBody(sphereBody); // Add the body to the world

        const sphereMesh = new THREE.Mesh(
          new THREE.SphereGeometry(sphereShape.radius),
          sphereMaterial
        ); // Create a new mesh for the sphere
        scene.add(sphereMesh); // Add the mesh to the scene
        spheres.push({ body: sphereBody, mesh: sphereMesh }); // Add the sphere to the spheres array
      }
      return spheres;
    },
    animate(spheres, renderer, scene, camera) {
      this.animationFrameId = requestAnimationFrame(() =>
        this.animate(spheres, renderer, scene, camera)
      ); // Request a new animation frame

      this.world.step(1 / 60); // Step the physics world

      spheres.forEach(({ body, mesh }) => {
        mesh.position.copy(body.position); // Copy the position of the body to the mesh
        mesh.quaternion.copy(body.quaternion); // Copy the quaternion of the body to the mesh

        if (body.position.y < -10) {
          body.position.set(
            Math.random() * 10 - 5,
            Math.random() * 5 + 10,
            Math.random() * 10 - 5
          ); // Reset the position of the body if it falls below -10
          body.velocity.set(0, 0, 0); // Reset the velocity of the body
        }
      });

      renderer.render(scene, camera); // Render the scene with the camera
    },
  },
};
</script>

<style lang="scss">
@import "./style.scss";
</style>
