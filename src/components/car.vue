<template>
  <div class="home">
    <div class="canvas-container" ref="canvasDemo">
    </div>

    <div class="home-content">
        <div class="home-content-title">
            <h1>汽车展示与选配</h1>
        </div>
        <div>
            <h2>选择车身颜色</h2>
        </div>
       
        <div class="select">
            <div class="select-item" v-for="[index, item] of colors.entries()" :key="index" @click="selectColor(index)">
                <div class="select-item-color" :style="{backgroundColor: item}"></div>
            </div>  
        </div>
        <div><h2>选择贴膜材质</h2></div>
        <div class="select">
            <div class="select-item" v-for="(item, index) in materials" :key="index" @click="selectMaterials(index)">
                <div class="select-item-text"> {{ item.name }} </div>
            </div>  
        </div>

    </div>

  </div>
</template>

<script setup>
import * as THREE from 'three';
import {ref, reactive, onMounted} from 'vue';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";
import MinMaxGUIHelper from "@/utils/MinMaxGUIHelper";
let canvasDemo = ref(null);
let colors = ref([]);
colors.value = [
    "red",
    "blue",
    "green",
    "gray",
    "orange",
    "purple"
];
let materials = ref([]);
materials = [
    {name: "磨砂", value: 1},
    {name: "冰晶", value: 0}
]
// 创建场景
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.set(2, 2, 5);
// const helper = new THREE.CameraHelper( camera );
// scene.add( helper );
function updateCamera() {
  camera.updateProjectionMatrix();
}
const gui = new GUI();
gui.add(camera, 'fov', 1, 180).onChange(updateCamera);
const minMaxGUIHelper = new MinMaxGUIHelper(camera, 'near', 'far', 0.1);
gui.add(minMaxGUIHelper, 'min', 0.1, 50, 0.1).name('near').onChange(updateCamera);
gui.add(minMaxGUIHelper, 'max', 0.1, 50, 0.1).name('far').onChange(updateCamera);
const axesHelper = new THREE.AxesHelper(5);
// size 表示代表轴的线段长度，默认为1
scene.add(axesHelper);

const renderer = new THREE.WebGLRenderer({
    antialias: true,
})
renderer.setSize(window.innerWidth, window.innerHeight);

// 添加控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(0, 0, 0); // 轨道控制器可以使得相机围绕目标进行轨道运动
controls.enableDamping = true
controls.update();
// scene.add(controls);


const render = () => {
    requestAnimationFrame(render);
    controls.update();
    renderer.render(scene, camera);
    
}
// 解码器的版本一般去本地复制，不要版本对应不上
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("./draco/"); // 先把 Draco 解码器文件放置在指定路径./draco/下
dracoLoader.preload();

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);
const bodyMaterial = new THREE.MeshStandardMaterial({
        color: 0xff0000,
        metalness: 1,
        roughness: 0.5,
        clearcoat: 1,
        clearcoatRoughness: 0,
    })

    const frontMaterial = new THREE.MeshPhysicalMaterial({
        color: 0xff0000,
        metalness: 1,
        roughness: 0.5,
        clearcoat: 1,
        clearcoatRoughness: 0,
    })

    const hoodMaterial = new THREE.MeshPhysicalMaterial({
        color: 0xff0000,
        metalness: 1,
        roughness: 0.5,
        clearcoat: 1,
        clearcoatRoughness: 0,
    })

    const wheelsMaterial = new THREE.MeshPhysicalMaterial({
        color: 0xff0000,
        metalness: 1,
        roughtness: 0.1,
    })
    const glassMaterial = new THREE.MeshPhysicalMaterial({
        color: 0xffffff,
        metalness: 0.01,
        roughness: 0,
        transmission: 1,
        transparent: true,
    })
let carBody, frontCar, hoodCar, glassCar;
let wheels = [];

let selectColor = (index) => {
    console.log(index, "11");
    bodyMaterial.color.set(colors.value[index]);
    frontMaterial.color.set(colors.value[index]);
    hoodMaterial.color.set(colors.value[index]);
    wheelsMaterial.color.set(colors.value[index]);
    glassMaterial.color.set(colors.value[index]);
   
}

let selectMaterials = (index) => {
    bodyMaterial.clearcoatRoughness = materials[index].value;
    frontMaterial.clearcoatRoughness = materials[index].value;
    hoodMaterial.clearcoatRoughness = materials[index].value;
}

function loadGLTFModel(url) {
    
    return new Promise((resolve, reject) => {
  
    // 创建材质
        gltfLoader.load(url, (gltf) => {
            const bmw = gltf.scene;
            // console.log(gltf);
            // 缩放模型大小
            // gltf.scene.scale.set(0.1, 0.1, 0.1);
            // model.position.set(-4, 0, 0);

            bmw.traverse((child) => {
                if (child.isMesh) {
                    console.log(child.name);
                }
                // 判断是否是轮毂
                if (child.isMesh && child.name.includes("轮毂")) {
                    child.material = wheelsMaterial;
                    wheels.push(child);
                    
                }
                // 判断是否是车身
                if(child.isMesh && child.name.includes("Mesh002")) {
                    carBody = child;
                    carBody.material = bodyMaterial;
                }
                // 判断是否是前脸
                if(child.isMesh && child.name.includes("前脸")) {
                    frontCar = child;
                    frontCar.material = frontMaterial;
                }
                // 判断是否是引擎盖
                if(child.isMesh && child.name.includes("引擎盖_1")) {
                    hoodCar = child;
                    hoodCar.material = hoodMaterial;
                }
                // 判断是否是挡风玻璃
                if(child.isMesh && child.name.includes("挡风玻璃")) {
                    glassCar = child;
                    glassCar.material = glassMaterial;
                }
            })
            scene.add(bmw);
            resolve(); // onLoad callback
        }),
            undefined,  //onProcess callback 不需要可以传undefined
            (error) => { // onError callback
                reject(error);
            }
    })
}

function addLight() {
    const light1 = new THREE.DirectionalLight(0xffffff, 1);
    light1.position.set(0, 0, 10);
    scene.add(light1);

    const light2 = new THREE.DirectionalLight(0xffffff, 1);
    light2.position.set(0, 0, -10);
    scene.add(light2);

    const light3 = new THREE.DirectionalLight(0xffffff, 1);
    light2.position.set(10, 0, 0);
    scene.add(light3);

    const light4 = new THREE.DirectionalLight(0xffffff, 1);
    light2.position.set(-10, 0, 0);
    scene.add(light4);

    const light5 = new THREE.DirectionalLight(0xffffff, 1);
    light2.position.set(0, 10, 0);
    scene.add(light5);

    

    const light6 = new THREE.DirectionalLight(0xffffff, 1);
    light2.position.set(5, 10, 0);
    scene.add(light6);

    const light7 = new THREE.DirectionalLight(0xffffff, 1);
    light2.position.set(5, -10, 0);
    scene.add(light7);

    const light8 = new THREE.DirectionalLight(0xffffff, 1);
    light2.position.set(0, -10, 5);
    scene.add(light8);


    const light9 = new THREE.DirectionalLight(0xffffff, 1);
    light2.position.set(0, -10, 0);
    scene.add(light9);


}

onMounted(async () => {
    console.log(canvasDemo.value);
    canvasDemo.value.appendChild(renderer.domElement);

    renderer.setClearColor("#000");
    scene.background = new THREE.Color("#ccc");
    scene.environment = new THREE.Color("#ccc");

    // 添加网格地面
    const gridHelper = new THREE.GridHelper(10, 10);
 
    gridHelper.material.opacity = 0.2;
    gridHelper.material.transparent = true;
    scene.add(gridHelper);
    // gridHelper.position.x = -4;
    render();
    // 添加汽车模型
    let gltfUrl = './model/bmw01.glb';
    await loadGLTFModel(gltfUrl);
    addLight();
})









// 一般异步代码会出现黑屏情况，所以加了await
// (async function () {
//     // 添加模型
//     let gltfUrl = './model/scene.glb';
//     await loadGLTFModel(gltfUrl);
// }());

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.home-content {
    position: fixed;
    top: 0px;
    right: 20px;
    /* display: flex; */
}
.select {
    display: flex;
}
.select-item-color {
    width: 50px;
    height: 50px;
    border: 1px solid #ccc;
    margin: 10px;
    display: inline-block;
    cursor: pointer;
    border-radius: 10px;
}
.select-item-text{
    width: 50px;
    height: 50px;
    border: 1px solid #ccc;
    margin: 10px;
    display: inline-block;
    cursor: pointer;
    border-radius: 10px;
    background: rgb(1, 1, 1, 0.1);
    display: flex;
    justify-content: center;
    align-items: center;
}
</style>
