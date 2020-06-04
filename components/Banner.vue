<template>
  <div class="banner">
    <div class="banner-inner">
      <div class="banner-logo-holder">
        <img
          src="@/static/sbclogo.png"
          class="banner-logo"
          alt="Sailboat Companion logo"
        />
      </div>
    </div>
    <div ref="animated" class="animated"></div>
  </div>
</template>

<script>
import { Water } from './Water.js'
import { Sky } from './Sky.js'

export default {
  name: 'Banner',
  mounted() {
    const THREE = require('three')
    const _this = this

    let container
    let camera, scene, renderer, light
    let water, sphere
    //
    init()
    animate()
    this.$refs.animated.classList.add('animating')

    //
    function init() {
      container = _this.$refs.animated

      //
      renderer = new THREE.WebGLRenderer()
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(container.offsetWidth, container.offsetHeight)
      container.appendChild(renderer.domElement)

      //

      scene = new THREE.Scene()

      //

      camera = new THREE.PerspectiveCamera(
        55,
        window.innerWidth / window.innerHeight,
        1,
        20000
      )
      onWindowResize()
      camera.position.set(30, 30, 100)
      camera.rotateY(1)

      //

      light = new THREE.DirectionalLight('#ffffff', 0.8)
      scene.add(light)

      // Water

      const waterGeometry = new THREE.PlaneBufferGeometry(10000, 10000)

      water = new Water(waterGeometry, {
        textureWidth: 512 * 2,
        textureHeight: 512 * 2,
        waterNormals: new THREE.TextureLoader().load(
          './waternormals.jpg',
          function(texture) {
            texture.wrapS = texture.wrapT = THREE.RepeatWrapping
          }
        ),
        alpha: 1.0,
        sunDirection: light.position.clone().normalize(),
        sunColor: '#ffffff',
        waterColor: '#001e0f',
        distortionScale: 8,
        fog: scene.fog !== undefined
      })

      water.rotation.x = -Math.PI / 2

      scene.add(water)

      // Skybox

      const sky = new Sky()

      const uniforms = sky.material.uniforms

      uniforms.turbidity.value = 10
      uniforms.rayleigh.value = 2
      uniforms.luminance.value = 1
      uniforms.mieCoefficient.value = 0.005
      uniforms.mieDirectionalG.value = 0.8

      const parameters = {
        distance: 400,
        inclination: 0.49,
        azimuth: 0.205
      }

      const cubeRenderTarget = new THREE.WebGLCubeRenderTarget(512, {
        format: THREE.RGBFormat,
        generateMipmaps: true,
        minFilter: THREE.LinearMipmapLinearFilter
      })
      const cubeCamera = new THREE.CubeCamera(0.1, 1, cubeRenderTarget)

      scene.background = cubeRenderTarget

      function updateSun() {
        const theta = Math.PI * (parameters.inclination - 0.5)
        const phi = 2 * Math.PI * (parameters.azimuth - 0.5)

        light.position.x = parameters.distance * Math.cos(phi)
        light.position.y = parameters.distance * Math.sin(phi) * Math.sin(theta)
        light.position.z = parameters.distance * Math.sin(phi) * Math.cos(theta)

        sky.material.uniforms.sunPosition.value = light.position.copy(
          light.position
        )
        water.material.uniforms.sunDirection.value
          .copy(light.position)
          .normalize()

        cubeCamera.update(renderer, sky)
      }

      updateSun()

      const geometry = new THREE.IcosahedronBufferGeometry(20, 1)
      const count = geometry.attributes.position.count

      const colors = []
      const color = new THREE.Color()

      for (let i = 0; i < count; i += 3) {
        color.setHex(Math.random() * '#ffffff')

        colors.push(color.r, color.g, color.b)
        colors.push(color.r, color.g, color.b)
        colors.push(color.r, color.g, color.b)
      }

      geometry.setAttribute(
        'color',
        new THREE.Float32BufferAttribute(colors, 3)
      )

      const material = new THREE.MeshStandardMaterial({
        vertexColors: true,
        roughness: 0.0,
        flatShading: true,
        envMap: cubeRenderTarget.texture,
        side: THREE.DoubleSide
      })

      sphere = new THREE.Mesh(geometry, material)
      scene.add(sphere)

      window.addEventListener('resize', onWindowResize, false)
    }

    function onWindowResize() {
      camera.aspect = container.offsetWidth / container.offsetHeight
      camera.updateProjectionMatrix()

      renderer.setSize(container.offsetWidth, container.offsetHeight)
    }

    function animate() {
      requestAnimationFrame(animate)
      render()
    }

    function render() {
      const time = performance.now() * 0.001

      sphere.position.y = Math.sin(time) * 20 + 5
      sphere.rotation.x = time * 0.5
      sphere.rotation.z = time * 0.51

      water.material.uniforms.time.value += 1.0 / 60.0

      renderer.render(scene, camera)
    }
  }
}
</script>

<style lang="scss">
@import '~@/sass/variables';

.banner {
  position: relative;
  height: 400px;
  z-index: 1;
  background: $myblue;

  &:after {
    content: '';
    width: 100%;
    height: 100%;
    position: absolute;
    z-index: -1;
    left: 0;
    top: 0;
    background-image: radial-gradient(
        circle at $padding-side-desktop 100%,
        darken($myblue, 20),
        transparent 66%
      ),
      radial-gradient(circle at 0 120%, darken($myblue, 20), transparent 75%);
  }

  .banner-inner {
    height: 100%;
    z-index: 1;
    padding: 52px $padding-side-desktop;
    @media (max-width: $tablet-breakpoint) {
      padding: 52px $padding-side-tablet;
      @media (max-width: $tablet-breakpoint) {
        padding: 52px $padding-side-mobile;
      }
    }
    display: flex;

    .banner-logo-holder {
      max-width: 100%;
      flex: 0 0 50%;
      @media (max-width: $mobile-breakpoint) {
        flex: 0 0 100%;
      }
      display: flex;
      justify-content: center;
    }

    img {
      max-height: 100%;
      @media (max-height: $mobile-breakpoint) {
        max-width: 300px;
        width: 75%;
        top: 50%;
        transform: translateY(-50%);
        position: absolute;
        max-height: unset;
        height: auto;
      }
    }
  }
}

.animated {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  z-index: -1;
  opacity: 0;
  transition: opacity 0.5s ease-in-out;

  &.animating {
    opacity: 1;
  }
}
</style>
