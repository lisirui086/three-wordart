<script setup>
// 引入组合式API
import { onMounted } from 'vue'

// 引入threejs
import * as THREE from 'three'

// 载入glTF 2.0资源的加载器。
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'

// 用Draco库压缩的几何装载器。
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js'

// 引入轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

// 创建场景
const scene = new THREE.Scene()
scene.background = new THREE.Color(0xffffff)

// 创建相机
const camera = new THREE.PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
)
camera.position.set(0, 0, 3)
scene.add(camera)

// 创建渲染器
const renderer = new THREE.WebGLRenderer({antialias: true})
renderer.setSize(window.innerWidth, window.innerHeight)

document.body.appendChild(renderer.domElement)

// 创建控制器
const controls = new OrbitControls( camera, renderer.domElement )
controls.enableDamping = true
// controls.update()

// 创建
/* const light = new THREE.AmbientLight( 0x404040 ); // soft white light
scene.add( light ); */

// 加载模型
const gltfLoader = new GLTFLoader()
const dracoLoader = new DRACOLoader()

dracoLoader.setDecoderPath('/draco/gltf/')
gltfLoader.setDRACOLoader( dracoLoader )

gltfLoader.load('/model/newyear.glb', gltf => {
  const newyear = gltf.scene
  console.dir(newyear)
  scene.add( newyear )
})

// 渲染函数
function animate () {
  requestAnimationFrame( animate )
  controls.update()
  renderer.render(scene, camera)
}

animate()

</script>

<template>
  <div></div>
</template>

<style scoped>
  canvas {
    width: 100vw;
    height: 100vh;
  }
</style>
