<template>
    <div class="canvas-container" ref="screenDom"></div>
</template>
<script lang="ts" setup>
import * as THREE from "three";
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { Reflector } from "three/examples/jsm/objects/Reflector";
import { ref, onMounted } from "vue";
let screenDom = ref(null);
onMounted(() => {
    // 创建场景
    let scene = new THREE.Scene();
    // 创建相机
    let camera = new THREE.PerspectiveCamera(75, screenDom.value.clientWidth / screenDom.value.clientHeight, 0.1, 1000)
    camera.position.set(0, 1.5, 6);
    // 创建渲染器
    let renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(screenDom.value.clientWidth, screenDom.value.clientHeight);
    screenDom.value.appendChild(renderer.domElement);
    // 创建辅助坐标系
    // let axes = new THREE.AxesHelper(5)
    // scene.add(axes)
    // 添加控制器
    const controls = new OrbitControls(camera, renderer.domElement);
    // 创建rgbe加载器
    let hdrLoader = new RGBELoader();
    // 
    hdrLoader.load("/src/assets/gltf/studio_small_05_4k.hdr", (texture) => {
        texture.mapping = THREE.EquirectangularReflectionMapping;
        scene.background = texture;
        scene.environment = texture;
    });
    //添加gltf控制器
    let loader = new GLTFLoader();
    // 解压缩
    const dracoLoader = new DRACOLoader();
    dracoLoader.setDecoderPath('/src/assets/gltf/car/');
    dracoLoader.setDecoderConfig({ type: "js" });
    loader.setDRACOLoader(dracoLoader);
    // 加载模型
    loader.load('/src/assets/gltf/golden_knight.glb', (gltf) => {
        scene.add(gltf.scene);
    })

    // 添加灯光
    let light1 = new THREE.DirectionalLight(0xffffff, 0.3);
    light1.position.set(0, 10, 10);
    let light2 = new THREE.DirectionalLight(0xffffff, 0.9);
    light2.position.set(0, -10, 10);
    let light3 = new THREE.DirectionalLight(0xffffff, 0.8);
    light3.position.set(10, 10, 10);
    let light4 = new THREE.DirectionalLight(0xffffff, 0.8);
    light4.position.set(10, 0, 10);
    scene.add(light1, light2, light3, light4)

    // 添加光阵
    let video = document.createElement("video");
    video.src = "/src/assets/gltf/zp2.mp4";
    video.loop = true;
    video.muted = true;
    video.play();
    let videoTexture = new THREE.VideoTexture(video);
    const videoGeoPlane = new THREE.PlaneBufferGeometry(8, 4.5);
    const vide0Material = new THREE.MeshBasicMaterial({
        map: videoTexture,
        transparent: true,
        side: THREE.DoubleSide,
        alphaMap: videoTexture,
    })
    const videoMesh = new THREE.Mesh(videoGeoPlane, vide0Material);
    videoMesh.position.set(0, 0, 0);
    videoMesh.rotation.set(-Math.PI / 2, 0, 0);
    scene.add(videoMesh);

    // 添加镜面反射
    let reflectorGeometry = new THREE.PlaneBufferGeometry(100, 100);
    let reflectorPlane = new Reflector(reflectorGeometry, {
        textureWidth: window.innerWidth,
        textureHeight: window.innerHeight,
        color: 0x332222,
    })
    reflectorPlane.rotation.x = -Math.PI / 2;
    scene.add(reflectorPlane);
    //渲染
    function render() {
        requestAnimationFrame(render);
        controls.update();
        renderer.render(scene, camera);
    }
    render();
    
  // 监听画面变化，更新渲染画面
  window.addEventListener("resize", () => {
    //   console.log("画面变化了");
    // 更新摄像头
    camera.aspect = window.innerWidth / window.innerHeight;
    //   更新摄像机的投影矩阵
    camera.updateProjectionMatrix();

    //   更新渲染器
    renderer.setSize(window.innerWidth, window.innerHeight);
    //   设置渲染器的像素比
    renderer.setPixelRatio(window.devicePixelRatio);
  });
})



</script>


<style scoped>
* {
    margin: 0;
    padding: 0;
}

.canvas-container {
    width: 100vw;
    height: 100vh;
}
</style>