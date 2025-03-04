<template>
  <div>
    <h2>Operaciones con Lenguajes</h2>
    
    <div>
      <button 
        v-for="(op, index) in operations" 
        :key="index" 
        @click="prepararOperacion(op.id)"
        :class="operacionActiva === op.id"
      >
        <div>{{ op.name }}</div>
      </button>
    </div>

    <hr>
    <div v-if="operacionActiva">
      <h3>{{ obtenerNombreOperacion(operacionActiva) }}</h3>
      
      <div class="input-container">
        <input 
          v-if="['concatenation', 'union', 'intercession', 'subtraction'].includes(operacionActiva)"
          v-model="conjunto1" 
          type="text" 
          placeholder="Símbolos del primer conjunto separados por coma (ej: a, b, c)"
          class="input-large"
        />
        <input 
          v-if="['concatenation', 'union', 'intercession', 'subtraction'].includes(operacionActiva)"
          v-model="conjunto2" 
          type="text" 
          placeholder="Símbolos del segundo conjunto separados por coma (ej: x, y, z)"
          class="input-large"
        />
        
        <input 
          v-if="['empowerment', 'reflection'].includes(operacionActiva)"
          v-model="conjunto1" 
          type="text" 
          placeholder="Símbolos del conjunto separados por coma (ej: a, b, c)"
          class="input-large"
        />
        
        <input 
          v-if="['closureKlenne', 'closurePositive'].includes(operacionActiva)"
          v-model="conjunto1" 
          type="text" 
          placeholder="Símbolos del conjunto separados por coma (ej: a, b, c)"
          class="input-large"
        />
        <input 
          v-if="['closureKlenne', 'closurePositive'].includes(operacionActiva)"
          v-model="profundidad" 
          type="number" 
          placeholder="Profundidad"
          class="input-large"
        />
        
        <button 
          @click="ejecutarOperacion" 
          :disabled="!validarEntrada()"
        >
          Ejecutar Operación
        </button>
      </div>
    </div>
    
    <!-- Resultado -->
    <div v-if="resultado">
      <h3>Resultado:</h3>
      <span>{{ resultado }}</span>
    </div>
    
    <!-- Mensajes de error -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

// Definición de operaciones
const operations = [
  { id: 'concatenation', name: 'Concatenación' },
  { id: 'empowerment', name: 'Potenciamiento' },
  { id: 'reflection', name: 'Reflexión' },
  { id: 'union', name: 'Unión' },
  { id: 'intercession', name: 'Intersección' },
  { id: 'subtraction', name: 'Sustracción' },
  { id: 'closureKlenne', name: 'Cierre de Kleene' },
  { id: 'closurePositive', name: 'Cierre Positivo' }
];

// Variables reactivas
const operacionActiva = ref('');
const conjunto1 = ref('');
const conjunto2 = ref('');
const profundidad = ref(2);
const resultado = ref('');
const error = ref('');

// Función para preparar la operación
const prepararOperacion = (operacionId: string) => {
  operacionActiva.value = operacionId;
  conjunto1.value = '';
  conjunto2.value = '';
  profundidad.value = 2;
  resultado.value = '';
  error.value = '';
};

// Obtener nombre de operación
const obtenerNombreOperacion = (id: string): string => {
  const operacion = operations.find(op => op.id === id);
  return operacion ? operacion.name : id;
};

// Validar entrada antes de ejecutar operación
const validarEntrada = (): boolean => {
  if (!operacionActiva.value) return false;
  
  const procesarConjunto = (conjuntoStr: string) => 
    conjuntoStr.split(',').map(s => s.trim()).filter(s => s);
  
  const conjuntoA = procesarConjunto(conjunto1.value);
  
  switch (operacionActiva.value) {
    case 'concatenation':
    case 'union':
    case 'intercession':
    case 'subtraction':
      const conjuntoB = procesarConjunto(conjunto2.value);
      return conjuntoA.length > 0 && conjuntoB.length > 0;
    case 'empowerment':
    case 'reflection':
    case 'closureKlenne':
    case 'closurePositive':
      return conjuntoA.length > 0;
    default:
      return false;
  }
};

// Ejecutar operación
const ejecutarOperacion = () => {
  if (!validarEntrada()) {
    error.value = 'Por favor, complete los campos requeridos.';
    return;
  }
  
  const procesarConjunto = (conjuntoStr: string) => 
    conjuntoStr.split(',').map(s => s.trim()).filter(s => s);
  
  const conjuntoA = procesarConjunto(conjunto1.value);
  const conjuntoB = procesarConjunto(conjunto2.value);
  
  try {
    switch (operacionActiva.value) {
      case 'concatenation':
        resultado.value = operacionesDeLenguaje.concatenacion(conjuntoA, conjuntoB);
        break;
      case 'empowerment':
        resultado.value = operacionesDeLenguaje.potenciamiento(conjuntoA);
        break;
      case 'reflection':
        resultado.value = operacionesDeLenguaje.reflexion(conjuntoA);
        break;
      case 'union':
        resultado.value = operacionesDeLenguaje.union(conjuntoA, conjuntoB);
        break;
      case 'intercession':
        resultado.value = operacionesDeLenguaje.interseccion(conjuntoA, conjuntoB);
        break;
      case 'subtraction':
        resultado.value = operacionesDeLenguaje.sustraccion(conjuntoA, conjuntoB);
        break;
      case 'closureKlenne':
        resultado.value = operacionesDeLenguaje.cierreKleene(conjuntoA, profundidad.value);
        break;
      case 'closurePositive':
        resultado.value = operacionesDeLenguaje.cierrePositivo(conjuntoA, profundidad.value);
        break;
    }
  } catch (err) {
    error.value = `Error al ejecutar la operación: ${err instanceof Error ? err.message : err}`;
  }
};

// Clase de operaciones de lenguaje
class OperacionesDeLenguaje {
  concatenacion(set1: string[], set2: string[]): string {
    const set3 = set1.flatMap(x => set2.map(y => x + y));
    return `La concatenación de los conjuntos es {${['g', ...set3].join(', ')}}`;
  }

  potenciamiento(set1: string[]): string {
    if (set1.length === 0) return "[g]";
    const newSet = set1.flatMap(x => set1.map(y => x + y));
    return `El potenciamiento del lenguaje es {${['g', ...newSet].join(', ')}}`;
  }

  reflexion(set1: string[]): string {
    const set2 = ['g', ...set1.map(x => x.split('').reverse().join(''))];
    return `La reflexión de los elementos es {${set2.join(', ')}}`;
  }

  union(set1: string[], set2: string[]): string {
    const set3 = Array.from(new Set(['g', ...set1, ...set2]));
    return `La unión de los conjuntos es {${set3.join(', ')}}`;
  }

  interseccion(set1: string[], set2: string[]): string {
    const set3 = set1.filter(x => set2.includes(x));
    return `La intersección de los conjuntos es {${['g', ...set3].join(', ')}}`;
  }

  sustraccion(set1: string[], set2: string[]): string {
    const set3 = set1.filter(x => !set2.includes(x));
    return `La sustracción de los conjuntos es {${['g', ...set3].join(', ')}}`;
  }

  cierreKleene(set: string[], maxDepth: number): string {
    let result = [''];
    for (let power = 1; power <= maxDepth; power++) {
      const newWords = result.flatMap(word => 
        set.map(symbol => word + symbol)
      );
      result = [...result, ...newWords];
    }
    return `El cierre de Kleene es {${result.join(', ')}}`;
  }

  cierrePositivo(set: string[], maxDepth: number): string {
    let result = [...set];
    for (let power = 2; power <= maxDepth; power++) {
      const newWords = result.flatMap(word => 
        set.map(symbol => word + symbol)
      );
      result = [...result, ...newWords];
    }
    return `El cierre positivo es {${result.join(', ')}}`;
  }
}

const operacionesDeLenguaje = new OperacionesDeLenguaje();

// Seleccionar una operación por defecto al cargar la página
onMounted(() => {
  prepararOperacion('concatenation');
});
</script>

<style>


.input-large {
  flex-grow: 1;
  padding: 8px;
  font-size: 16px;
}

.error-message {
  color: red;
  margin-top: 10px;
}

button {
  padding: 8px 15px;
}
</style>