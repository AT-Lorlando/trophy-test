<template>
  <div id="gemWrapper" ref="gemWrapper" :style="{ opacity: ready ? 1 : 0 }" class="transition duration-1000">
    <canvas id="ThreeCanvas"/>
  </div>
</template>

<script setup>
const ready = ref()
const gemWrapper = ref(null)
let AnimationID

if (process.client) {
  async function loadGem() {
    const THREE = await import('three').then(m => m.default || m)
    const { OrbitControls } = await import('three/examples/jsm/controls/OrbitControls.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const { GLTFLoader } = await import('three/examples/jsm/loaders/GLTFLoader.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const { RGBELoader } = await import('three/examples/jsm/loaders/RGBELoader.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const { DRACOLoader } = await import('three/examples/jsm/loaders/DRACOLoader.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const { RoomEnvironment } = await import('three/examples/jsm/environments/RoomEnvironment.js' /* webpackChunkName: "trophy" */).then(m => m.default || m)
    const gltfLoader = new GLTFLoader()
    const dracoLoader = new DRACOLoader()

    const canvas = document.getElementById('ThreeCanvas')
    const renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true, canvas: canvas })
    const scene = new THREE.Scene()
    const pmremGenerator = new THREE.PMREMGenerator(renderer)
    const camera = new THREE.PerspectiveCamera(50, 1, 0.1, 20)
    const controls = new OrbitControls(camera, renderer.domElement)
    const clock = new THREE.Clock()
    let gem


    function init() {
      // Loaders
      dracoLoader.setDecoderPath('https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/jsm/libs/draco/')
      gltfLoader.setDRACOLoader(dracoLoader)

      // Renderer
      renderer.setPixelRatio(window.innerWidth / window.innerHeight)
      renderer.outputEncoding = THREE.sRGBEncoding
      renderer.toneMapping = THREE.ACESFilmicToneMapping
      renderer.toneMappingExposure = 1

      if (window.matchMedia('(min-width: 640px)').matches) {
        renderer.setSize(475, 475)
      } else if (window.matchMedia('(min-width: 400px)').matches) {
        renderer.setSize(240, 240)
      } else {
        renderer.setSize(180, 180)
      }

      // Scene
      scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture

      // Camera
      camera.position.set(0, 0, 14)

      // Controls
      controls.enableDamping = true
      controls.dampingFactor = 0.05
      controls.enableZoom = true
      controls.enablePan = false
      controls.maxPolarAngle = Math.PI * 0.5
      controls.minPolarAngle = Math.PI * 0.5

      // Gem
      gltfLoader.load('/assets/home/scene.glb', 
        (gltf) => {
          gem = gltf.scene
          gem.scale.set(1, 1, 1)
          gem.position.set(0, -5, 0)
          gem.rotation.z = 0
          scene.add(gem)
          ready.value = true
        },
        (xhr) => {
          // console.log((xhr.loaded / xhr.total * 100) + '% loaded')
        },
        (error) => {
          // console.log(error)
        }
      )
    }

    function onWindowResize() {
      if (window.matchMedia('(min-width: 640px)').matches) {
        renderer.setSize(475, 475)
      } else if (window.matchMedia('(min-width: 400px)').matches) {
        renderer.setSize(240, 240)
      } else {
        renderer.setSize(180, 180)
      }
    }

    function animate() {
      render()
      AnimationID = requestAnimationFrame(animate)
    }

    function render() {
      const delta = clock.getDelta()
      controls.update()
      if (gem) {
        gem.rotation.y -= delta * 0.5
      }
      renderer.render(scene, camera)
    }

    window.addEventListener('resize', onWindowResize, false)

    init()
    animate()
  }

  // onMounted(() => requestIdleCallback(() => {
  //   const observer = new IntersectionObserver((entries) => {
  //     entries.forEach((entry) => {
  //       if (entry.isIntersecting) {
  //         loadGem()
  //         observer.unobserve(entry.target)
  //       }
  //     })
  //   }, { threshold: 0.5 })
  //   observer.observe(gemAnim.value)
  // }))

  onMounted(() => {
    loadGem()
  })

  onUnmounted(() => {
    cancelAnimationFrame(AnimationID)
  })
}
</script>
