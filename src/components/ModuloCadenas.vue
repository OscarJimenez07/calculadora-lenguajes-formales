<template>
  <div>
    <div>
      <h2>Operaciones con Cadenas</h2>
      <div>
        <button @click="prepararOperacion('concatenacion')" :class="{ active: operacionActiva === 'concatenacion' }">Concatenación</button>
        <button @click="prepararOperacion('potenciacion')" :class="{ active: operacionActiva === 'potenciacion' }">Potenciación</button>
        <button @click="prepararOperacion('reflexion')" :class="{ active: operacionActiva === 'reflexion' }">Reflexión</button>
        <button @click="prepararOperacion('longitud')" :class="{ active: operacionActiva === 'longitud' }">Longitud</button>
      </div>
    </div>
    <hr>
    
    <div v-if="operacionActiva">
      <h3>{{ obtenerNombreOperacion(operacionActiva) }}</h3>
      
      <div class="input-container">
        <label>Primera cadena:</label>
        <input v-model="cadena1" type="text" placeholder="Ingrese la primera cadena"/>
        
        <div v-if="operacionActiva === 'concatenacion'">
          <label>Segunda cadena:</label>
          <input v-model="cadena2" type="text" placeholder="Ingrese la segunda cadena"/>
        </div>
        
        <div v-if="operacionActiva === 'potenciacion'">
          <label>Exponente:</label>
          <input v-model.number="exponente" type="number" min="0" placeholder="Ingrese un número entero no negativo"/>
        </div>
        
        <button @click="ejecutarOperacion" :disabled="!puedeEjecutarOperacion()">
          Ejecutar Operación
        </button>
      </div>
    </div>
    
    <div v-if="resultado !== null">
      <h3>Resultado:</h3>
      <div>{{ resultado }}</div>
    </div>
    
    <div v-if="error" class="error-message">
      {{ error }}
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

// Definición de tipos
type OperacionesType = Record<string, string>;

// Constantes
const NOMBRES_OPERACIONES: OperacionesType = {
  'concatenacion': 'Concatenación',
  'potenciacion': 'Potenciación',
  'reflexion': 'Reflexión',
  'longitud': 'Longitud'
};

// Variables reactivas
const operacionActiva = ref('concatenacion');
const cadena1 = ref('');
const cadena2 = ref('');
const exponente = ref(0);
const resultado = ref<string | number | null>(null);
const error = ref('');

// Utilidades
const obtenerNombreOperacion = (id: string): string => NOMBRES_OPERACIONES[id] || '';

const prepararOperacion = (operacionId: string) => {
  operacionActiva.value = operacionId;
  resultado.value = null;
  error.value = '';
};

const puedeEjecutarOperacion = () => {
  if (cadena1.value.trim() === '') return false;
  
  if (operacionActiva.value === 'concatenacion' && cadena2.value.trim() === '') {
    return false;
  }
  
  if (operacionActiva.value === 'potenciacion' && (isNaN(exponente.value) || exponente.value < 0)) {
    return false;
  }
  
  return true;
};

// Implementación de las operaciones según la lógica de Python
const operaciones = {
  concatenacion: (cadena1: string, cadena2: string): string => {
    if (cadena1 === 'λ') cadena1 = '';
    if (cadena2 === 'λ') cadena2 = '';
    
    const resultado = cadena1 + cadena2;
    return resultado === '' ? 'λ' : resultado;
  },
  
  potenciacion: (cadena: string, n: number): string => {
    if (cadena === 'λ') cadena = '';
    
    if (n === 0) return 'λ';
    
    let resultado = '';
    for (let i = 0; i < n; i++) {
      resultado += cadena;
    }
    
    return resultado === '' ? 'λ' : resultado;
  },
  
  reflexion: (cadena: string): string => {
    if (cadena === 'λ') return 'λ';
    
    return cadena.split('').reverse().join('');
  },
  
  longitud: (cadena: string): number => {
    if (cadena === 'λ') return 0;
    
    return cadena.length;
  }
};

// Ejecución de la operación
const ejecutarOperacion = () => {
  resultado.value = null;
  error.value = '';
  
  try {
    switch (operacionActiva.value) {
      case 'concatenacion':
        resultado.value = operaciones.concatenacion(cadena1.value, cadena2.value);
        break;
        
      case 'potenciacion':
        if (isNaN(exponente.value) || exponente.value < 0) {
          error.value = 'Error: El exponente debe ser un número entero no negativo.';
          return;
        }
        resultado.value = operaciones.potenciacion(cadena1.value, exponente.value);
        break;
        
      case 'reflexion':
        resultado.value = operaciones.reflexion(cadena1.value);
        break;
        
      case 'longitud':
        resultado.value = operaciones.longitud(cadena1.value);
        break;
        
      default:
        error.value = 'Error: Operación no reconocida.';
    }
  } catch (e) {
    error.value = `Error al ejecutar la operación: ${e}`;
  }
};
</script>