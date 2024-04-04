<template>
  <div class="mainContainer">
    <h1>Introducing ThreeJS</h1>
        <div class="text2">click &amp; drag to rotate the dragon 👇</div>
        <!-- <div class="text">koo design studio</div> -->

        <div 
          v-show="progressBarContainer"
          class="progress-bar-container">
          <label for="progress-bar">LOADING...</label>
          <progress id="progress-bar" :value=progressBar max="100"></progress>
        </div>

        <!-- threejs container -->
        <div class="canvasRef">
        <canvas ref="canvasRef"></canvas>
      </div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from "vue"
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
// FBX LOADER
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader'

let scene = new THREE.Scene();
let renderer;
let controls;

let canvasRef = ref();

// Red cube for test
let redCubeGeometry = new THREE.BoxGeometry(0.001, 0.001, 0.001)
redCubeGeometry.translate(0,0,0)
let redCubeMaterial = new THREE.MeshPhongMaterial({color: "#000000" , shininess: 1})
let redCube = new THREE.Mesh(redCubeGeometry, redCubeMaterial)
redCube.position.set(0, 0, 0)
scene.add(redCube)

let ambientLight = new THREE.AmbientLight("#404040", 0.8);
ambientLight.position.set(0, 0, 0)
// scene.add(ambientLight);

// directinalLight
let directinalLight = new THREE.DirectionalLight("#ffffff", 6)
directinalLight.position.set(-2, 3, 10)
scene.add(directinalLight)

// directinalLight2
let directinalLight2 = new THREE.DirectionalLight("#aaaaaa", 4)
directinalLight2.position.set(2, -3, -10)
scene.add(directinalLight2)


let camera = new THREE.PerspectiveCamera(
  35,
  window.innerWidth / window.innerHeight,
  0.1,
  100
);

camera.position.set(-8, 1.5, 10)
scene.add(camera);


// loading manager to show loading bar
const loadingManager = new THREE.LoadingManager()

// -- onStart
// loadingManager.onStart = (url, item, total) => {
//   console.log(`started loading: ${url}`)
// }

// -- onProgress
const progressBar = ref(0)
loadingManager.onProgress = (url, loaded, total) => {
  // console.log(`started loading: ${url, loaded, total}`)
  progressBar.value = (loaded / total ) * 100
}

// -- onProgress
const progressBarContainer = ref(true)
loadingManager.onLoad = () => {
  progressBarContainer.value = false
  // console.log(`just finished loading`)
}

// -- onError
loadingManager.onError = (url) => {
  console.error(`got a problem loading: ${url}`)
}

// load FBX dragon animated
let loader = new FBXLoader(loadingManager)
loader.load(
  // import fbx file from /public folder
    'dragonHi_anim06.fbx',
    (fbx) => {
        // fbx.scene.traverse(child => {
        //     if(child.material) child.material.metalness = 0
        // })
        let dragonModel = fbx
        dragonModel.position.set(1.8, -2.5, -2)
        dragonModel.rotation.set(0, -0.9, 0)
        
        let mixer = new THREE.AnimationMixer(dragonModel)
        let action = mixer.clipAction(dragonModel.animations[0])
        action.play()

        let clock = new THREE.Clock()
        let animate = () => {
            let delta = clock.getDelta()
            mixer.update(delta)
            renderer.render(scene, camera)
            requestAnimationFrame(animate)
        }
        animate()
        scene.add(fbx)
    }
)


// let loop = () => {
//   box.rotation.y += 0.02;
//   renderer.render(scene, camera);
//   requestAnimationFrame(loop);
// };




let loop = () => {
  controls.update();
  renderer.render(scene, camera);
};

let resizeCallback = () => {
  renderer.setSize(window.innerWidth, window.innerHeight);

  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
};

onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value,
    antialias: true,
    alpha: true,
  });

  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.render(scene, camera);

  renderer.setAnimationLoop(loop);

  controls = new OrbitControls(camera, canvasRef.value);
  controls.enableDamping = true;

  window.addEventListener("resize", resizeCallback);

  //   requestAnimationFrame(loop);

});

onUnmounted(() => {
  renderer.setAnimationLoop(null);
  window.removeEventListener("resize", resizeCallback);
});
</script>



<style lang="scss" scoped>
.progress-bar-container {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 100%;
  height: 100%;
  background-color: rgb(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 50;
}

#progress-bar {
  width: 30%;
  margin-top: 0.5%;
  height: 2%
}

label {
  color: white;
  font-size: 2rem;
}

canvas {
    position: absolute;
    top: 200;
    /* z-index: -1; */

}
.canvasRef {
  display: block;
  height: 870px;
  background-image: linear-gradient(#646464, #070707)
}
.text {
    position: absolute;
    top: 260px;
    right: 30px;
    width: 620px;
    height: 820px;
    padding-top: 18px;
    text-align: center;
    z-index: 10;
    color: rgb(171, 68, 222);
    font-size: 12rem;
    line-height: 11rem;
    background-color: rgb(171, 68, 222, 0.15);
    border-radius: 20px;
    // border: 10px solid rgb(171, 68, 222);
    @media (max-width: 1139px) {
        right: 40px !important;
        font-size: 11rem;
        width: 500px;
        line-height: 8.5rem;
    }
    @media(max-width:990px) {
        right: 20px !important;
        height: 600px;
        font-size: 7rem;
        width: 360px;
        line-height: 5.5rem;
    }
    @media(max-width:700px) {
        font-size: 6rem;
        width: 350px;
        height: 650px;
        top: 240px;
        right: 10px !important;
        line-height: 5rem;
    }
    @media(max-width:550px) {
        font-size: 4.5rem;
        width: 220px;
        height: 550px;
        top: 240px;
        right: 5px !important;
        line-height: 4rem;
    }
    @media(max-width:400px) {
        font-size: 3rem;
        width: 160px;
        height: 400px;
        top: 220px;
        right: 10px !important;
        line-height: 2.4rem;
    }
    small {
      font-size: 1.5rem;
    }
}


.text2 {
  position: relative;
  left: 20px
}

</style>
