<script setup>
import OneGear from './components/OneGear.vue'
import Swal from 'sweetalert2'
import { ref, computed, onMounted, onBeforeUnmount, watch } from 'vue'

const angleDiff = (a, b) => Math.abs(((a - b + 180 + 360) % 360) - 180)
const playPauseBtn = ref(null);

function handleKeydown(event) {
  if (event.code === 'Space') {
    event.preventDefault();
    toggleAnimation();
  }
  if (event.key.toLowerCase() === 'r') {
    resetAnimation();
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
});

onMounted(() => {
  Swal.fire({
    title: 'Objetivo del Juego',
    html: `
      <p>Descifrar cuántas vueltas tiene que dar el engranaje para obtener nuevamente el triángulo.</p>
      <ul style="text-align: left;">
        <li>Hay 3 engranajes, cada uno con un número estipulado de dientes.</li>
        <li>Cada vez que se hace clic, el engranaje rota un diente.</li>
      </ul>
      <p><strong>Estrategia:</strong> Lograr que el triángulo se forme nuevamente dando clic.</p>
      <p><strong>¡Obtén el triángulo!</strong><br>¡Jugar!</p>
    `,
    icon: 'info',
    confirmButtonText: 'Entendido',
    allowOutsideClick: false,
    allowEscapeKey: false
  })
})
onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeydown);
});

const isPlaying = ref(false)
const rotationCount = ref(0)
const initialAngles = ref({})
const initialLineAngles = ref([])
const haDadoVuelta = ref(false)

const restartEverything = () => {
  isPlaying.value = false
  rotationCount.value = 0
  haDadoVuelta.value = false
  initialAngles.value = {}
  initialLineAngles.value = []
  finalAngles.value = {}
  gearCounts.value = {}

  // También puedes forzar el reinicio visual si es necesario:
  // window.location.reload()
}


const captureInitialTriangleAngles = () => {
  if (lineAngles.value.length === 3) {
    initialLineAngles.value = lineAngles.value.map(l => l.line2Angle)
    console.log('📐 Ángulos iniciales del triángulo:', initialLineAngles.value)
  } else {
    console.warn('⚠️ No se pudo capturar el triángulo aún.')
  }
}

const captureInitialAngles = () => {
  if ([1, 2, 3].every(index => finalAngles.value[index] !== undefined)) {
    initialAngles.value = {
      1: Math.round(finalAngles.value[1]) % 360,
      2: Math.round(finalAngles.value[2]) % 360,
      3: Math.round(finalAngles.value[3]) % 360
    }
    console.log('📌 Posiciones iniciales exactas guardadas:', initialAngles.value)
  } else {
    console.warn('⚠️ No se pudieron capturar todas las posiciones.')
  }
}

const finalAngles = ref({})
const gearCounts = ref({})

// const toggleAnimation = () => {
//   if (!isPlaying.value) {
//     captureInitialTriangleAngles()
//     rotationCount.value = 0
//     haDadoVuelta.value = false
//   }
//   isPlaying.value = !isPlaying.value
// }
const toggleAnimation = () => {
  if (!mcmValido.value) {
    console.warn("🚫 No se puede iniciar la animación: MCM inválido.")
    return
  }

  if (!isPlaying.value) {
    captureInitialTriangleAngles()
    rotationCount.value = 0
    haDadoVuelta.value = false
  }

  isPlaying.value = !isPlaying.value
}


let alignmentCheckTimeout

const handleFinalAngle = ({ gearIndex, currentAngle, count }) => {
  finalAngles.value[gearIndex] = currentAngle
  gearCounts.value[gearIndex] = count
  // rotationCount.value = count

  // handleRotationComplete(count);

  if (gearIndex === 3) {
    rotationCount.value = count
    console.log(count);
      if (count >= 3) {
        haDadoVuelta.value = true
      }
  }

  clearTimeout(alignmentCheckTimeout)
  alignmentCheckTimeout = setTimeout(() => {
    checkIfTriangleAligned()
  }, 100)
}

const checkIfTriangleAligned = () => {
  if (!haDadoVuelta.value) return

  if (initialLineAngles.value.length !== 3 || lineAngles.value.length !== 3) return

  const allAligned = lineAngles.value.every((line, i) => {
    const actual = Math.round(line.line2Angle) % 360
    const inicial = Math.round(initialLineAngles.value[i]) % 360
    console.log(`🔺 L${i + 1} → actual: ${actual} | inicial: ${inicial}`)

    return angleDiff(actual, inicial) < 2

  })

  if (allAligned) {
    isPlaying.value = false
    console.log('✅ Triángulo regresó a su forma original.')
    setTimeout(() => {
    restartEverything()
  }, 500)
  }
}

const resetAnimation = () => {
  isPlaying.value = false
  rotationCount.value = 0
  window.location.reload()
}

const engranajes = [
  { top: -10000, left: -40 },   // 0: Medio (ángulo ~30°)
  { top: -170, left: 170 },  // 1: Pequeño (ángulo ~100°)
  { top: 100, left: 320 },    // 2: Grande (ángulo ~45°)
]

const offsetLine1 = [6, 6, 6]
const offsetLine2 = [7, 10, -45] 

const angulosInternos = [48, 110, 22]

const calcularAngulo = (x1, y1, x2, y2) => {
  return (Math.atan2(y2 - y1, x2 - x1) * 180 / Math.PI + 360) % 360
}

const sentidos = [true, false, true] 

const lineAngles = computed(() => {
  return engranajes.map((engranaje, i) => {
    const siguiente = engranajes[(i + 1) % engranajes.length]
    const line1 = calcularAngulo(
      engranaje.left,
      engranaje.top,
      siguiente.left,
      siguiente.top
    )

    const angulo = angulosInternos[i]
    const baseLine2 = sentidos[i]
      ? (line1 + angulo) % 360
      : (line1 - angulo + 360) % 360

    // Ajustes con offset
    const ajustadoLine1 = (line1 + offsetLine1[i] + 360) % 360
    const ajustadoLine2 = (baseLine2 + offsetLine2[i] + 360) % 360

    return {
      line1Angle: ajustadoLine1,
      line2Angle: ajustadoLine2
    }
  })
})

watch(lineAngles, () => {
  checkIfTriangleAligned()
}, { deep: true })

const startHolding = () => {
  if (!isPlaying.value) {
    captureInitialTriangleAngles()
    rotationCount.value = 0
    haDadoVuelta.value = false
  }
  isPlaying.value = true
}

const stopHolding = () => {
  isPlaying.value = false
  restartEverything()
}

const gearTeeth = ref([15, 9, 45])

const mcmValido = ref(true)

// Función auxiliar para calcular el MCD
function gcd(a, b) {
  return b === 0 ? a : gcd(b, a % b)
}

// Función auxiliar para calcular el MCM de dos números
function lcm(a, b) {
  return (a * b) / gcd(a, b)
}

// MCM de un array de números
function lcmArray(arr) {
  return arr.reduce((acc, val) => lcm(acc, val))
}

// Verifica si hay un MCM válido cada vez que cambian los dientes
watch(gearTeeth, (newTeeth) => {
  console.log("Cantidad de dientes actualizada:", newTeeth)

  const todosValidos = newTeeth.every(n => n > 0)
  if (todosValidos) {
    const mcm = lcmArray(newTeeth)
    console.log("MCM:", mcm)

    // Validación: el MCM debe ser mayor que el más grande para ser útil
    mcmValido.value = mcm > Math.max(...newTeeth)

    if (mcmValido.value) {
      console.log("MCM válido entre los engranajes.")
    } else {
      console.warn("Los engranajes no tienen un MCM útil. Considera ajustar los dientes.")
    }
  } else {
    mcmValido.value = false
    console.warn("Todos los engranajes deben tener más de 0 dientes.")
  }
})

const velocidad = ref(1) // 1x por defecto

</script>

<template>
  <div class="main">
    <div class="wrapper">
      <OneGear 
      size="normal" 
      :teeth="gearTeeth[0]" 
      :duration="3 / velocidad" 
      :clockwise="true" 
      :top="20" 
      :left="47"
      :line1Angle="lineAngles[0].line1Angle" 
      :line2Angle="lineAngles[0].line2Angle" 
      :isPlaying="isPlaying"
      :rotationCount="rotationCount.value" 
      :manualOffset="2" 
      :gearIndex="1" 
      counterClass="normal-gear"
      idName="normalID" 
      @final-angle="handleFinalAngle" />

      <OneGear 
      size="small" 
      :teeth="gearTeeth[1]" 
      :duration="2 / velocidad" 
      :top="125" 
      :left="145"
      :line1Angle="lineAngles[1].line1Angle" 
      :line2Angle="lineAngles[1].line2Angle" 
      :isPlaying="isPlaying"
      :rotationCount="rotationCount.value" 
      :manualOffset="10" 
      :gearIndex="2" 
      counterClass="small-gear"
      idName="smallID" 
      @final-angle="handleFinalAngle" />

      <OneGear
      size="big" 
      :teeth="gearTeeth[2]" 
      :duration="4 / velocidad" 
      :top="0" 
      :left="220" 
      :clockwise="true"
      :line1Angle="lineAngles[2].line1Angle" 
      :line2Angle="lineAngles[2].line2Angle" 
      :isPlaying="isPlaying"
      :rotationCount="rotationCount.value" 
      :manualOffset="50" 
      :gearIndex="3" 
      counterClass="big-gear" 
      idName="bigID"
      @final-angle="handleFinalAngle" />
    </div>

    <!-- Controles -->
    <div class="controls">
      <!-- Botón presionado (mantener pulsado para reproducir, soltar para reiniciar) -->
      <button class="play-pause-btn" @mousedown="startHolding" @mouseup="stopHolding" @mouseleave="stopHolding"
        @touchstart.prevent="startHolding" @touchend="stopHolding" title="Mantener pulsado" ref="playPauseBtn">
        ⏭️
      </button>

      <!-- Botón alternativo manual -->
      <button class="play-pause-btn" @click="toggleAnimation" title="Iniciar animación">
        ▶️
      </button>

      <!-- Reiniciar -->
      <button class="reset-btn" @click="resetAnimation" title="Reiniciar todo">
        🔄
      </button>
      <div class="velocidad-control">
        <label for="velocidadSlider">Velocidad:</label>
        <input type="range" id="velocidadSlider" min="0.5" max="5" step="0.1" v-model.number="velocidad" title="Ajustar la velocidad" />
        <span>{{ velocidad.toFixed(1) }}x</span>
      </div>

    </div>

    <!-- Dientes personalizados -->
    <div class="dientes-personalizados">
      <div v-for="(gear, index) in engranajes" :key="index" class="diente-input">
        <label class="eng" :for="'dientes-' + index">Engranaje {{ index + 1 }}:</label>
        <input type="number" :id="'dientes-' + index" v-model.number="gearTeeth[index]" min="1" step="1" />
      </div>
    </div>

    <!-- Alerta por MCM inválido -->
    <div v-if="!mcmValido" class="mcm-alerta">
      Los engranajes deben tener un mínimo común múltiplo útil. Ajusta los dientes.
    </div>
  </div>
</template>

<style scoped>

.dientes-personalizados {
  display: flex;
  gap: 1rem;
}
.velocidad-control {
  color: aliceblue;
  margin-top: 25px;
  /* position: absolute; */
  /* margin-left: 20%; */
}
.eng {
  color: wheat;
}
.diente-input {
  display: flex;
  flex-direction: column;
  font-size: 14px;
}

/* Estilos principales para centrar todo en pantalla y aplicar fondo oscuro */
.main {
  min-height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: #111;
  gap: 20px;
}

/* ✅ Todas estas clases comparten los mismos estilos */
.big-gear,
.small-gear,
.normal-gear {
  position: absolute;
  top: 70px;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  white-space: nowrap;
  z-index: 6;
}


/* Contenedor donde se posicionan los engranajes */
.wrapper {
  width: 460px;
  height: 240px;
  position: relative;
  background: transparent;
}

/* Estilo del botón de play/pausa */
.play-pause-btn {
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  padding: 10px;
  border-radius: 50%;
  transition: transform 0.2s;
}

/* Animación al pasar el cursor por el botón */
.play-pause-btn:hover {
  transform: scale(1.1);
}

/* Animación al hacer clic en el botón */
.play-pause-btn:active {
  transform: scale(0.95);
}

.controls {
  display: flex;
  gap: 10px;
}

.reset-btn {
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  padding: 10px;
  border-radius: 50%;
  transition: transform 0.2s;
}

.reset-btn:hover {
  transform: scale(1.1);
}

.reset-btn:active {
  transform: scale(0.95);
}

</style>
