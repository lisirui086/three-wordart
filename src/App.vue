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

// 定义着色器材质
let coloringMaterial

gltfLoader.load('/model/newyear.glb', gltf => {
  // const newyear = gltf.scene
  
  const text = gltf.scene.children[0]
  scene.add( text )

  // 获取当前的几何体
  const geometry = text.geometry
  // 获取所有顶点
  const position = geometry.attributes.position
  const vertextCount = position.count

  // 计算三角形数量
  const triangleCount = new Number( vertextCount /3 )

  // 生成随机方向、随机角度
  const randomDirections = []
  const randomStrengths = []

  for (let i = 0; i < triangleCount; i++ ) {
    // 方向
    const dir = new THREE.Vector3(
      Math.random() * 2 - 1,
      Math.random() * 2 - 1,
      Math.random() * 2 - 1
    ).normalize().toArray()

    // randomDirections.push(...dir)
    randomDirections.push(dir[0], dir[1], dir[2])

    // 强度
    const str = Math.random()

    randomStrengths.push(str, str, str)
  }

  const randomDirectionsAttribut = new THREE.Float32BufferAttribute(
    new Float32Array(randomDirections),
    3
  )
  geometry.setAttribute('randomDirection', randomDirectionsAttribut)

  const randomStrengthsAttribut = new THREE.Float32BufferAttribute(
    new Float32Array(randomStrengths),
    1
  )
  geometry.setAttribute('randomStrength', randomStrengthsAttribut)

  coloringMaterial = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 0 },
    },
    vertexShader: `
      attribute vec3 randomDirection;
      attribute float randomStrength;
      uniform float time;
      varying vec3 vPosition;

      void main() {
        vPosition = position;
        vec3 pos = position.xyz;
        pos += randomDirection * randomStrength * time;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(pos.xyz, 1.0);
      }
    `,
    fragmentShader: `
      varying vec3 vPosition;
      void main() {
        vec3 color = normalize(vPosition)*0.5+0.5;
        color.z = color.z*3.0;
        gl_FragColor = vec4(color,1.0);
      }
    `,
    // 材质常量 面
    side: THREE.DoubleSide,
    transparent: true
  })
  text.material = coloringMaterial
})

// 该对象用于跟踪时间
const clock = new THREE.Clock()

// 获取vaule
let value = 0

// 一看是时发散的
let showText = false


// 侦听showText的值，根据showTest的值决定是否发散
function handShowText () {
  if(coloringMaterial) {
    const delta = clock.getDelta()
    if(showText) {
      // 减
      value -= delta
      value = Math.max(value, 0)
    } else {
      // 加
      value += delta
      value = Math.min(value, 1)
    }
    coloringMaterial.uniforms.time.value = value
  }
}

// 侦听屏幕宽高，调整渲染器、相机宽高大小
function handOnsize () {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix ()
  renderer.setSize(window.innerWidth , window.innerHeight)
}

window.addEventListener('keyup', (event) => {
  event.code === 'Space' ? showText = !showText : showText
})

window.addEventListener('resize', handOnsize)


// 渲染函数
function animate () {
  handShowText()
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
