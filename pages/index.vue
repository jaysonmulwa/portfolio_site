<script lang="ts" setup>
import { SRGBColorSpace, CineonToneMapping } from 'three'
import { ImprovedNoise } from 'three/examples/jsm/math/ImprovedNoise'
import { clamp } from 'three/src/math/MathUtils'
import { PALETTE } from '../components/content/synthwave-landscape/components/palette'

const gl = {
  clearColor: PALETTE[0],
  alpha: false,
  outputColorSpace: SRGBColorSpace,
  toneMapping: CineonToneMapping,
}

const PI2 = Math.PI * 2
const NOISE_PEAK = 4
const NOISE_SCALE = 6
const TERRAIN_SCALE = [120, 6, 360]
const SPEED = 0.0375
const noise = new ImprovedNoise().noise

const pathGen = (yy: number): [number, number, number] => {
  const path = Math.sin(PI2 * yy * 2) * 0.05
  const wave = Math.sin(PI2 * yy * 3) * 0.3
  return [path, wave, yy]
}

function fract(a: number) {
  return a - Math.floor(a)
}

function pingpong(a: number, b: number) {
  if (b == 0.0) {
    return 0.0
  }
  else {
    return Math.abs(fract((a - b) / (b * 2.0)) * b * 2.0 - b)
  }
}

const terrainGen = (planeX: number, planeY: number): [number, number, number] => {
  const [path, wave, z] = pathGen(planeY)
  const x = planeX
  const noiseV = Math.abs(
    noise(planeX * NOISE_SCALE, pingpong(planeY * 4, 0.5) * (NOISE_SCALE * (2 * Math.sin(PI2 * planeY))), 0),
  )
  const noisePeaks = -(NOISE_PEAK + Math.abs(Math.sin(PI2 * planeY))) * noiseV
  const y = (noisePeaks + noisePeaks * Math.abs(wave)) * Math.abs(Math.sin(PI2 * (planeX - path))) + wave
  return [-x, -y, z]
}
const x = shallowRef(0)
const y = shallowRef(0)
const z = shallowRef(0)
const dayProgress = shallowRef(0)

useRenderLoop().onLoop(({ elapsed }) => {
  const [xx, yy, _] = pathGen(elapsed * SPEED)
  x.value = xx * TERRAIN_SCALE[0]
  y.value = yy * TERRAIN_SCALE[1] + 3
  z.value = 0
  dayProgress.value = clamp(Math.cos(elapsed * SPEED * 2) + 0.5, 0, 1)
})
</script>

<template>
  <div class="synthwave-landscape">
    <div class="text-overlay">
      <div class="text-header">
        <h1>Hi! I'm Jayson Mulwa</h1>
        <br/>
        <div class="text-intro">
          <h6>&#x1F3A9; Software Engineer, &#x1F3A9; DevOps Engineer</h6>
        </div>
        
      </div>

      <div class="text-links">
        <a href="https://www.linkedin.com/in/jaysonmulwa">LinkedIn</a>
        <a href="https://gitlab.com/jayson_mulwa">GitLab</a>
        <a href="https://github.com/jaysonmulwa">GitHub</a>
        <a href="https://medium.com/@jaysonmulwa">Blog</a>
        <a href="mailto:jayson.mulwa@gmail.com">Mail</a>
        <a href="https://github.com/jaysonmulwa/jaysonmulwa/blob/master/CV.pdf?raw=true">CV</a>
      </div>
      

    </div>

  <MusicPlayer />
  <TresCanvas
    v-bind="gl"
    :disable-render="false"
  >
    <!-- <Postprocessing /> -->

    <TresPerspectiveCamera :position="[x, y, z]">
      <TresPointLight
        :intensity="1000"
        :position="[0, 3, 0]"
        :color="PALETTE[6]"
      />
      <!-- <Mountain
        :color="PALETTE[8]"
        :position="[0, 0, -500]"
        :scale="[24, 24, 12]"
      /> -->
<!-- 
      <Sun
        :scale="100"
        :color-a="PALETTE[3]"
        :color-b="PALETTE[7]"
        :position="[0, 133, -600]"
      /> -->

      <Grid
        :scale="2660"
        :position="[0, 71, -600]"
        :progress="dayProgress"
        :num-divisions="39"
        :color="PALETTE[1]"
        :fill="PALETTE[2]"
        :col-fills="[]"
      />

      <!-- :col-fills="[
          { color: PALETTE[6], colNum: -1 },
          { color: PALETTE[6], colNum: 0 },
          { color: PALETTE[6], colNum: 1 },
        ]" -->
      <Stars :scale="5" />
    </TresPerspectiveCamera>
    <TresAmbientLight
      :color="PALETTE[3]"
      :intensity="40 * (1 - dayProgress)"
    />
    <TresDirectionalLight
      :intensity="20 + 80 * dayProgress"
      :position="[0, 70, -400]"
      :color="PALETTE[3]"
    />
    <TresDirectionalLight
      :intensity="100 * dayProgress"
      :position="[200, 0, 100]"
      :color="PALETTE[1]"
    />

    <Terrain
      :terrain-gen-fn="terrainGen"
      :camera-z="z"
      :color-fills="PALETTE[2]"
      :color-lines="dayProgress > 0.5 ? PALETTE[4] : PALETTE[8]"
      :color-dust="PALETTE[3]"
      :scale="TERRAIN_SCALE"
      :speed="SPEED"
      :shininess="dayProgress"
    />
    <GradientSky
      :color-sky="PALETTE[3]"
      :color-ground="PALETTE[5]"
      :size="1000"
      :offset="0.3"
      :scale="2"
    />
  </TresCanvas>
  </div>
</template>


<style scoped>

/* Add Google Font link */
@import url('https://fonts.googleapis.com/css2?family=Playwrite+AU+QLD:wght@400;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Merriweather:wght@400;700&display=swap');

.synthwave-landscape {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
  display: flex; /* Use flexbox for centering */
  justify-content: center; /* Center horizontally */
  align-items: flex-start; /* Align text to the top, you can change this if needed */
}


.text-overlay {
  background: rgba(0, 0, 0, 0); /* Semi-transparent background */
  position: absolute;
  top: 10%;
  /* color: hsl(0, 0%, 100%); */
  color: #fff; /* White text color */
  left: 50%;
  transform: translateX(-50%);
  font-size: 2.5rem;
  font-family: 'Playwrite AU QLD', cursive;
  text-shadow: 0 0 10px #f0f, 0 0 20px #0ff;
  z-index: 100;
  pointer-events: none; /* Allows interaction with canvas underneath */
  width: 100%; /* Allow the text to scale with screen width */
  text-align: center; /* Center the text inside the div */
  padding: 0 20px; /* Add padding on sides to prevent text from hitting the edges */
}

.text-header {
  margin-bottom: 20px; /* Add space between header and links */
}

.text-intro {
  font-family: 'Merriweather', serif;
}

.text-links {
  pointer-events: auto; /* Allow interaction with links */
  margin-top: 20px; /* Add space above the links */
  padding-top: 40px; /* Add padding around the links */
  display: flex; /* Use flexbox layout */
  flex-direction: row; /* Stack links horizontally */
  flex-wrap: wrap; /* Allow the links to wrap onto the next line */
  align-items: center; /* Center links horizontally */
  gap: 20px; /* Space between links */
  text-align: center; /* Align text to the left */
  font-size: 1.5rem; /* Adjust font size for links */
  font-family: 'Merriweather', serif;
  justify-content: center; /* Center items horizontally */
}

/* Responsive font size for smaller devices */
@media (max-width: 600px) {
  .text-overlay {
    font-size: 1.8rem; /* Reduce font size on smaller screens */
  }
}
</style>
