<template>
  <div class="gear" :class="sizeClass" :style="positionStyle">
    <div class="gear-inner" 
    :class="{
      'rotate-clockwise': clockwise && isPlaying,
      'rotate-counter-clockwise': !clockwise && isPlaying,
      'paused': !isPlaying
    }" :style="{
      animationDuration: duration + 's',
      transform: !isPlaying ? 'rotate(' + (angleComputed + manualOffset) + 'deg)' : undefined
    }">
      <div v-for="n in teeth" :key="n" class="bar"
        :style="{ transform: 'rotate(' + ((n - 1) * (360 / teeth)) + 'deg)' }"></div>

      <div class="center-circle"></div>

      <div class="red-line line1" :style="{ transform: 'rotate(' + line1Angle + 'deg)' }"></div>
      <div class="red-line line2" :style="{ transform: 'rotate(' + line2Angle + 'deg)' }"></div>
    </div>

    <div class="rotation-counter" :class="counterClass" :style="counterStyle">
      Vueltas: {{ localRotationCount }}
    </div>
    <div class="gear-id" :class="idName" :style="counterStyle">
      E{{ gearIndex }}
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'

const angle = ref(0)
const localRotationCount = ref(0)

const props = defineProps({
  size: { type: String, default: 'small' },
  teeth: { type: Number, default: 3 },
  duration: { type: Number, default: 3 },
  clockwise: { type: Boolean, default: false },
  top: { type: [String, Number], default: 'auto' },
  left: { type: [String, Number], default: 'auto' },
  right: { type: [String, Number], default: 'auto' },
  bottom: { type: [String, Number], default: 'auto' },
  line1Angle: { type: Number, default: 0 },
  line2Angle: { type: Number, default: 180 },
  isPlaying: { type: Boolean, default: true },
  rotationCount: { type: Number, default: 0 },
  manualOffset: { type: Number, default: 0 },
  gearIndex: { Number },
  counterClass: String,
  counterStyle: Object,
  idName: String

  // angle: { type: Number, default: 0 }
})

const emit = defineEmits(['rotation-complete', 'final-angle'])


let animationStartTime = null
let lastRotationTime = null

const updateRotationCount = () => {
  if (!props.isPlaying) return

  const now = performance.now()

  if (!animationStartTime) {
    animationStartTime = now
    lastRotationTime = now
    return
  }

  const elapsedTime = now - lastRotationTime
  const rotationDuration = props.duration * 1000
  const deltaAngle = (props.clockwise ? 1 : -1) * (360 * (elapsedTime / rotationDuration))
  angle.value = (angle.value + deltaAngle) % 360
    console.log(`Engranaje ${props.size}:`, {
    elapsedTime,
    angle: angle.value,
    rotationCount: props.rotationCount,
  });

  if (elapsedTime >= rotationDuration) {
    lastRotationTime = now
    localRotationCount.value += 1
    emit('rotation-complete', {
      count: localRotationCount,
      currentAngle: angle.value,
      gearIndex: props.gearIndex
    })
    const normalizedangle = ((angle.value % 360) + 360) % 360
    emit('final-angle', {
      count: localRotationCount.value,
      currentAngle: normalizedangle,
      gearIndex: props.gearIndex
    })
  }
}
  watch(() => props.isPlaying, (newVal) => {
    if (!newVal) {
      angle.value = (Math.round(angle.value / 360) * 360) + props.manualOffset;
    }
  });
let animationFrameId

const normalizeAngle = (angle) => ((angle % 360) + 360) % 360;

// Example Usage:
angle.value = normalizeAngle(angle.value);

onMounted(() => {
  const animate = () => {
    updateRotationCount()
    animationFrameId = requestAnimationFrame(animate)
  }
  animate()
})

onUnmounted(() => {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
  }
})

const sizeClass = computed(() => {
  if (props.size === 'big') return 'big'
  if (props.size === 'normal') return 'normal'
  return 'small'
})

const positionStyle = computed(() => ({
  position: 'absolute',
  top: typeof props.top === 'number' ? props.top + 'px' : props.top,
  left: typeof props.left === 'number' ? props.left + 'px' : props.left,
  right: typeof props.right === 'number' ? props.right + 'px' : props.right,
  bottom: typeof props.bottom === 'number' ? props.bottom + 'px' : props.bottom
}))

const angleComputed = computed(() => {
  return (angle.value - props.manualOffset) % 360
})

watch(() => props.isPlaying, (isPlaying) => {
  if (!isPlaying) {
    angle.value = normalizeAngle(angle.value);
    console.log(`Engranaje pausado:`, {
      angle: angle.value,
      rotationCount: props.rotationCount,
    });
  }
});

let lastUpdateTime = 0;

const animate = () => {
  const now = performance.now();
  if (now - lastUpdateTime > 16) { // Roughly 60fps
    updateRotationCount();
    lastUpdateTime = now;
  }
  animationFrameId = requestAnimationFrame(animate);
};

</script>

<style scoped>
.gear {
  position: absolute;
  border-radius: 60px;
}

.gear.small {
  width: 60px;
  height: 60px;
  margin-left: -5px;
  margin-top: -30px;
  transform-origin: center 30px;
}

.gear.small .center-circle {
  height: 36px;
  width: 36px;
  border-radius: 36px;
  margin-left: -18px;
  margin-top: -18px;
}

.gear.small .gear-inner .bar {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 14px;
  height: 30px;
  margin-left: -7px;
  margin-top: -40px;
  transform-origin: center 40px;
  background: #3cf0ed;
  rotate: -5deg;

  clip-path: polygon(0% 100%,
      /* abajo izquierda */
      0% 70%,
      /* inicio lateral izquierdo inclinado */
      30% 0%,
      /* esquina superior izquierda */
      70% 0%,
      /* esquina superior derecha */
      100% 70%,
      /* inicio lateral derecho inclinado */
      100% 100%
      /* abajo derecha */
    );
}

.gear.small .red-line {
  height: 30px;
  margin-top: -15px;
}

.gear.normal {
  width: 120px;
  height: 120px;
  margin-left: -5px;
  margin-top: -46px;
  transform-origin: center 50px;
  /* transform: rotate(6deg); */
}

.gear.normal .gear-inner .bar {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 40px;
  margin-left: -10px;
  margin-top: -81px;
  transform-origin: center 80px;
  background: #3cf0ed;
  rotate: 4deg;

  clip-path: polygon(0% 100%,
      /* abajo izquierda */
      0% 70%,
      /* inicio lateral izquierdo inclinado */
      30% 0%,
      /* esquina superior izquierda */
      70% 0%,
      /* esquina superior derecha */
      100% 70%,
      /* inicio lateral derecho inclinado */
      100% 100%
      /* abajo derecha */
    );
}

.gear.normal .center-circle {
  height: 96px;
  width: 96px;
  border-radius: 48px;
  margin-left: -48px;
  margin-top: -48px;
}

.gear.normal .red-line {
  height: 60px;
  margin-top: -30px;
}

.gear.big {
  width: 240px;
  height: 240px;
  margin-left: -10px;
  margin-top: -70px;
  transform-origin: center 70px;
  /* transform: rotate(-7deg); */
}

.gear.big .center-circle {
  height: 192px;
  width: 192px;
  border-radius: 96px;
  margin-left: -96px;
  margin-top: -96px;
}

.gear.big .gear-inner .bar {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 70px;
  margin-left: -11px;
  margin-top: -140px;
  transform-origin: center 140px;
  background: #3cf0ed;
  /* background: #0c10f7; */

  clip-path: polygon(0% 100%,
      /* abajo izquierda */
      0% 70%,
      /* inicio lateral izquierdo inclinado */
      30% 0%,
      /* esquina superior izquierda */
      70% 0%,
      /* esquina superior derecha */
      100% 70%,
      /* inicio lateral derecho inclinado */
      100% 100%
      /* abajo derecha */
    );
}

.gear.big .red-line {
  height: 120px;
  margin-top: -60px;
}

.center-circle {
  position: absolute;
  content: '';
  background-color: #111;
  top: 50%;
  left: 50%;
  z-index: 3;
  box-shadow: 0 0 10px rgb(255, 255, 255, 0.1), inset 0 0 10px rgb(0, 0, 0, 0.1), inset 0 2px 0 0 #090909, inset 0 -1px 0 0 #3cf0ed;
}

.gear-inner {
  position: relative;
  height: 100%;
  width: 100%;
  background: #3cf0ed;
  border-radius: 30px;
  animation: rotate-counter-clockwise 3s linear infinite;
}

.gear-inner.paused {
  animation-play-state: paused;
}

.gear.normal .gear-inner {
  border-radius: 60px;
}

.gear.big .gear-inner {
  border-radius: 120px;
}

.red-line {
  position: absolute;
  width: 3px;
  background-color: #ff0000;
  top: 25%;
  left: 50%;
  transform-origin: bottom center;
  margin-left: -1.5px;
  z-index: 5;
  box-shadow: 0 0 5px rgba(255, 0, 0, 0.5);
  pointer-events: none;
}

.rotation-counter {
  position: absolute;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  white-space: nowrap;
  z-index: 6;
}

.normal-gear {
  background-color: rgba(0, 0, 255, 0.7);
  /* azul */
  top: 200%;
  left: 50%;
  transform: translateX(-50%);
}

.small-gear {
  background-color: rgba(0, 255, 0, 0.7);
  /* verde */
  top: 199%;
  left: 125%;
  transform: translateX(-50%);
}

.big-gear {
  background-color: rgba(255, 0, 0, 0.7);
  /* rojo */
  top: calc(110% + 20px);
  left: 50%;
  transform: translateX(-50%);
}

.gear-id {
  white-space: nowrap;
  border-radius: 4px;
  color: white;
  font-weight: 800;
  padding:  2px 2px 2px 4px;
  width: 27px;
}

.normalID{
  background-color: rgba(0, 0, 255, 0.7);
  margin-top: 15%;
}

.smallID{
  background-color: rgba(0, 255, 0, 0.7);
  margin-top: 25%;
}

.bigID{
  background-color: rgba(255, 0, 0, 0.7);
  margin-top: -1%;
  margin-left: 5%;
}

@keyframes rotate-counter-clockwise {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(-360deg);
  }
}

@keyframes rotate-clockwise {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

.rotate-clockwise {
  animation-name: rotate-clockwise !important;
}

.rotate-counter-clockwise {
  animation-name: rotate-counter-clockwise !important;
}
</style>