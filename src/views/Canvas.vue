<template>
  <div>
    <div class="panel">
      <div style="display: flex; align-items: center">
        <label style="margin-right: 4px">Select Color : </label>
        <input type="color" name="Color Picker" v-model="color" />
      </div>
    </div>
    <canvas ref="webGl" class="webGl"> </canvas>
  </div>
</template>

<script lang="ts">
import {
  Scene,
  MeshPhysicalMaterial,
  Mesh,
  PointLight,
  PerspectiveCamera,
  WebGLRenderer,
  Color,
  AmbientLight,
} from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { watch, onMounted, ref, computed, reactive } from "vue";
import { useWindowSize } from "@vueuse/core";
export default {
  setup() {
    const webGl = ref();
    const color = ref("#ff0000");
    const { width, height } = useWindowSize();
    const aspectRatio = computed(() => {
      return width.value / height.value;
    });
    const carMaterialDetails = reactive({
      color: 0xff0000,
      metalness: 0.9,
      roughness: 0.5,
      clearcoat: 1,
      clearcoatRoughness: 0.004,
      sheen: 0.5,
    });
    let camera: PerspectiveCamera;
    let renderer: WebGLRenderer;
    let scene: Scene;
    let mesh: Mesh;
    let controls: OrbitControls;
    let light: PointLight;
    let light1: PointLight;
    let Alight: AmbientLight;
    let loader: GLTFLoader;
    let carMaterial: MeshPhysicalMaterial;
    const setCanvas = () => {
      // Create Scene
      scene = new Scene();
      scene.background = new Color(0xdddddd);

      // Lights
      Alight = new AmbientLight(0x404040, 100);
      scene.add(Alight);
      light = new PointLight(0xffffff, 1);
      light.position.set(200, 100, 0);
      scene.add(light);

      light1 = new PointLight(0xffffff, 1);
      light1.position.set(50, 0, 500);
      scene.add(light1);

      // Camera
      camera = new PerspectiveCamera(40, aspectRatio.value, 1, 5000);
      camera.rotation.y = (45 / 180) * Math.PI;
      camera.position.x = 60;
      camera.position.y = 20;
      camera.position.z = 80;
      scene.add(camera);

      // Renderer
      const canvas = webGl.value;
      renderer = new WebGLRenderer({ canvas, antialias: true });
      renderer.setSize(width.value, height.value);

      // Loader
      loader = new GLTFLoader();
      loader.load("/images/datsun.glb", (gltf) => {
        gltf.scene.position.y = -3;
        gltf.scene.scale.set(20, 20, 20);
        const n = gltf.scene;
        carMaterial = new MeshPhysicalMaterial({ ...carMaterialDetails });
        n.traverse((node) => {
          if (node.name == "Object_3") {
            node.material = carMaterial;
          }
        });
        scene.add(n);

        renderer.render(scene, camera);
      });

      // Controls
      controls = new OrbitControls(camera, canvas);

      controls.enableDamping = true;
    };

    const updateCamera = () => {
      camera.aspect = aspectRatio.value;
      camera.updateProjectionMatrix();
    };

    const updateRenderer = () => {
      renderer.setSize(width.value, height.value);
      renderer.render(scene, camera);
    };

    const updateColor = () => {
      carMaterial.color.setHex(color.value.toString().replace("#", "0x"));
      renderer.render(scene, camera);
    };

    watch(aspectRatio, (val) => {
      if (val) {
        updateCamera();
        updateRenderer();
      }
    });

    watch(color, (val) => {
      if (val) {
        updateColor();
      }
    });

    const animate = () => {
      // mesh.rotation.y += 0.01;
      controls.update();
      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    };

    onMounted(() => {
      setCanvas();
      animate();
    });

    return { webGl, updateColor, color };
  },
};
</script>

<style lang="css" scoped>
.panel {
  position: absolute;
  z-index: 1;
  right: 50%;
  top: 3rem;
}
</style>
