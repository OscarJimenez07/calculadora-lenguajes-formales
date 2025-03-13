<template>
  <!-- Fondo animado -->
  <div id="particles-js"></div>

  <h2 class="">Calculadora Lenguajes Formales</h2>

  <header>
    <button v-for="(item, index) in menuItems" :key="index" @click="activeMenu = item.id">
      {{ item.name }}
    </button>
  </header>

  <main>
    <AlfabetosModule v-if="activeMenu === 'alfabetos'" />
    <CadenasModule v-if="activeMenu === 'cadenas'" />
    <LenguajesModule v-if="activeMenu === 'lenguajes'" />
  </main>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import AlfabetosModule from './components/ModuloAlfabetos.vue';
import CadenasModule from './components/ModuloCadenas.vue';
import LenguajesModule from './components/ModuloLenguajes.vue';

interface MenuItem {
  id: string;
  name: string;
}

const menuItems: MenuItem[] = [
  { id: 'alfabetos', name: 'Alfabetos' },
  { id: 'cadenas', name: 'Cadenas' },
  { id: 'lenguajes', name: 'Lenguajes' }
];

const activeMenu = ref('alfabetos');

// Inicializar Particles.js cuando el componente se monta
onMounted(() => {
  loadParticles();
});

const loadParticles = () => {
  // Verificar si ya está cargado
  if (window.particlesJS) {
    initParticles();
    return;
  }

  // Cargar script de particles.js
  const script = document.createElement('script');
  script.src = 'https://cdn.jsdelivr.net/particles.js/2.0.0/particles.min.js';
  script.onload = initParticles;
  document.body.appendChild(script);
};

// Función para inicializar partículas con interacción "push"
const initParticles = () => {
  particlesJS('particles-js', {
    particles: {
      number: { value: 80, density: { enable: true, value_area: 800 } },
      color: { value: "#ffffff" },
      shape: { type: "circle" },
      opacity: { value: 0.5, random: false },
      size: { value: 3, random: true },
      line_linked: { enable: true, distance: 120, color: "#ffffff", opacity: 0.4, width: 1 },
      move: { enable: true, speed: 2, direction: "none", random: false, out_mode: "out" }
    },
    interactivity: {
      detect_on: "window",
      events: {
        onhover: { enable: true, mode: "grab" },
        onclick: { enable: true, mode: "push" } // Click añade partículas
      },
      modes: {
        grab: { distance: 140, line_linked: { opacity: 1 } },
        push: { particles_nb: 5 } // Se añaden 5 partículas en cada clic
      }
    },
    retina_detect: true
  });

  // 🔥 Eliminar partículas después de cierto tiempo para evitar acumulación
  setInterval(() => {
    const particles = window.pJSDom[0]?.pJS?.particles?.array;
    if (particles && particles.length > 80) { 
      particles.splice(0, particles.length - 80); // Elimina las más antiguas
    }
  }, 5000); // Se limpia cada 5 segundos
};

loadParticles();

</script>

<style>
/* Fondo animado */
#particles-js {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: #2c2c2c;
  z-index: -1;
}

/* Estilos generales */
body {
  margin: 0;
  font-family: Arial, sans-serif;
  color: white;
  text-align: center;
  background: #2c2c2c;
  overflow: hidden; /* Evita el scroll por el fondo */
}

header {
  margin-top: 20px;
}

button {
  margin: 5px;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
  background: #000;
  color: white;
  border-radius: 5px;
}

button:hover {
  background: #333;
}
</style>
