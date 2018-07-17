<template>
  <div>
    <input type="file" @change="change">
    <button @click="getUrl">click</button>
    <div ref="div"></div>
    <keep-alive>
      <img v-if="url" :src="url"/>
    </keep-alive>
  </div>
</template>

<script>
import * as THREE from 'three'
import './FBXLoader'
import './OrbitControls'
export default {
  data () {
    return {
      camera: null,
      light: null,
      scene: null,
      loader: null,
      renderer: null,
      controls: null,
      url: '',
      mixers: [],
      clock: null
    }
  },
  mounted () {
    this.scene = new THREE.Scene()
    this.clock = new THREE.Clock()
    this.camera = new THREE.PerspectiveCamera(75, 400 / 400, 1, 2000)
    this.camera.position.set(0, 18, 28)

    this.controls = new THREE.OrbitControls(this.camera)
    this.controls.target.set(0, 10, 0)
    this.controls.update()

    this.light = new THREE.HemisphereLight(0xffffff, 0x444444)
    this.light.position.set(0, 1, 0)
    this.scene.add(this.light)

    this.light = new THREE.DirectionalLight(0xffffff)
    this.light.position.set(0, 1, 0)
    this.scene.add(this.light)

    // grid
    let gridHelper = new THREE.GridHelper(40, 40, 0x434242, 0x576456)
    this.scene.add(gridHelper)
    this.renderer = new THREE.WebGLRenderer({
      antialias: true,
      preserveDrawingBuffer: true
    })
    this.renderer.setPixelRatio(window.devicePixelRatio)
    this.renderer.setSize(400, 400)
    this.renderer.shadowMap.enabled = true
    this.$refs.div.appendChild(this.renderer.domElement)
    setTimeout(() => {
      this.animate()
    })
    window.addEventListener('resize', this.windowResize)
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.windowResize)
  },
  methods: {
    change (e) {
      let file = e.target.files[0]
      let readerFile = new FileReader()
      readerFile.onload = () => {
        this.loader = new THREE.FBXLoader()
        this.mixers = []
        this.loader.load(readerFile.result, (object) => {
          this.scene.children.forEach(item => {
            if (item.type === 'Group') {
              this.scene.remove(item)
            }
          })

          if (object.animations.length > 0) {
            object.mixer = new THREE.AnimationMixer(object)
            this.mixers.push(object.mixer)
            let action = object.mixer.clipAction(object.animations[0])
            action.play()
            object.traverse(child => {
              if (child.isMesh) {
                child.castShadow = true
                child.receiveShadow = true
              }
            })
          }

          this.scene.add(object)
          object.scale.multiplyScalar(0.3)
        })
      }
      readerFile.readAsDataURL(file)
    },
    windowResize () {
      this.camera.aspect = 400 / 400
      this.camera.updateProjectionMatrix()
      this.renderer.setSize(400, 400)
    },
    animate () {
      requestAnimationFrame(this.animate)
      this.$nextTick(() => {
        if (this.mixers.length > 0) {
          for (let i = 0; i < this.mixers.length; i++) {
            this.mixers[i].update(this.clock.getDelta())
          }
        }
        this.renderer.render(this.scene, this.camera)
      })
    },
    getUrl () {
      let canvas = document.getElementsByTagName('canvas')[0]
      this.url = canvas.toDataURL()
    }
  }
}
</script>

<style>

</style>
