<template>
  <div>
    <h2>Operaciones con Lenguajes</h2>
    
    <div>
      <button 
        v-for="(op, index) in operations" 
        :key="index" 
        @click="prepararOperacion(op.id)"
        :class="{ active: operacionActiva === op.id }"
      >
        <div>{{ op.name }}</div>
      </button>
    </div>

    <hr>
    <div v-if="operacionActiva">
      <h3>{{ obtenerNombreOperacion(operacionActiva) }}</h3>
      
      <div class="input-container">
        <!-- Para operaciones que requieren dos conjuntos -->
        <div v-if="['concatenation', 'union', 'intercession', 'subtraction'].includes(operacionActiva)">
          <input 
            v-model="conjunto1" 
            type="text"
            placeholder="Lenguaje 1 (separado por comas)"
            class="input-large"
          />
          <input 
            v-model="conjunto2" 
            type="text" 
            placeholder="Lenguaje 2 (separado por comas)"
            class="input-large"
          />
        </div>
        
        <!-- Para operación de potenciamiento -->
        <div v-if="operacionActiva === 'empowerment'">
          <input 
            v-model="conjunto1" 
            type="text"
            placeholder="Lenguaje (separado por comas)"
            class="input-large"
          />
          <input 
            v-model="potencia" 
            type="number" 
            placeholder="Exponente"
            class="input-large"
          />
        </div>
        
        <!-- Para reflexión, que solo necesita un conjunto -->
        <div v-if="operacionActiva === 'reflection'">
          <input 
            v-model="conjunto1" 
            type="text"
            placeholder="Lenguaje (separado por comas)"
            class="input-large"
          />
        </div>
        
        <!-- Para operaciones de cierre -->
        <div v-if="['closureKlenne', 'closurePositive'].includes(operacionActiva)">
          <input 
            v-model="conjunto1" 
            type="text"
            placeholder="Lenguaje (separado por comas)"
            class="input-large"
          />
          <input 
            v-model="profundidad" 
            type="number" 
            placeholder="Profundidad"
            class="input-large"
          />
        </div>
        
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
      <pre>{{ resultado }}</pre>
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
const potencia = ref(2);
const profundidad = ref(2);
const resultado = ref('');
const error = ref('');

// Función para preparar la operación
const prepararOperacion = (operacionId: string) => {
  operacionActiva.value = operacionId;
  conjunto1.value = '';
  conjunto2.value = '';
  potencia.value = 2;
  profundidad.value = 2;
  resultado.value = '';
  error.value = '';
};

// Obtener nombre de operación
const obtenerNombreOperacion = (id: string): string => {
  const operacion = operations.find(op => op.id === id);
  return operacion ? operacion.name : id;
};

// Procesar conjunto de entrada
const procesarConjunto = (conjuntoStr: string): Set<string> => {
  return new Set(
    conjuntoStr
      .split(',')
      .map(s => s.trim())
      .filter(s => s)
  );
};

// Validar entrada antes de ejecutar operación
const validarEntrada = (): boolean => {
  if (!operacionActiva.value) return false;
  
  const conjuntoA = procesarConjunto(conjunto1.value);
  
  switch (operacionActiva.value) {
    case 'concatenation':
    case 'union':
    case 'intercession':
    case 'subtraction':
      const conjuntoB = procesarConjunto(conjunto2.value);
      return conjuntoA.size > 0 && conjuntoB.size > 0;
    case 'empowerment':
      return conjuntoA.size > 0 && potencia.value >= 0;
    case 'reflection':
      return conjuntoA.size > 0;
    case 'closureKlenne':
    case 'closurePositive':
      return conjuntoA.size > 0 && profundidad.value > 0;
    default:
      return false;
  }
};

// Clase de operaciones de lenguaje actualizada
class OperacionesDeLenguaje {
  private landa = "λ";

  concatenacion(L1: Set<string>, L2: Set<string>): string {
    // Si ambos conjuntos son lambda o están vacíos, devolver solo {λ}
    if ((L1.size === 0 || (L1.size === 1 && L1.has(this.landa))) && 
        (L2.size === 0 || (L2.size === 1 && L2.has(this.landa)))) {
      return `La concatenación de los conjuntos es {${this.landa}}`;
    }
    
    // Si uno de los conjuntos está vacío o es {λ}, devolver el otro conjunto
    if (L1.size === 0 || (L1.size === 1 && L1.has(this.landa))) {
      return `La concatenación de los conjuntos es {${Array.from(L2).join(', ')}}`;
    }
    
    if (L2.size === 0 || (L2.size === 1 && L2.has(this.landa))) {
      return `La concatenación de los conjuntos es {${Array.from(L1).join(', ')}}`;
    }
    
    // Caso general: concatenar todas las combinaciones
    const resultado = new Set<string>();
    L1.forEach(a => {
      L2.forEach(b => {
        resultado.add(a + b);
      });
    });
    
    return `La concatenación de los conjuntos es {${Array.from(resultado).sort().join(', ')}}`;
  }

  potenciamiento(L: Set<string>, n: number): string {
    // Si L es el conjunto vacío o solo contiene lambda (vacío)
    if (L.size === 0 || (L.size === 1 && L.has(this.landa))) {
      if (n === 0) {
        return `El potenciamiento del lenguaje es {${this.landa}}`; // Cualquier cosa elevada a 0 es {λ}
      }
      return `El potenciamiento del lenguaje es {${this.landa}}`; // λ elevado a cualquier n > 0 devuelve "λ"
    }
    
    // Caso estándar para n == 0
    if (n === 0) {
      return `El potenciamiento del lenguaje es {${this.landa}}`;
    }
    
    // Lógica de concatenación para potencias mayores a 0
    let resultado = new Set<string>(L);
    for (let i = 1; i < n; i++) {
      const nuevoResultado = new Set<string>();
      resultado.forEach(a => {
        L.forEach(b => {
          nuevoResultado.add(a + b);
        });
      });
      resultado = nuevoResultado;
    }
    
    return `El potenciamiento del lenguaje es {${Array.from(resultado).sort().join(', ')}}`;
  }

  reflexion(L: Set<string>): string {
    const resultado = new Set<string>();
    L.forEach(palabra => {
      resultado.add(palabra.split('').reverse().join(''));
    });
    
    return `La reflexión de los elementos es {${Array.from(resultado).sort().join(', ')}}`;
  }

  union(L1: Set<string>, L2: Set<string>): string {
  if (L1.size === 1 && L1.has("λ")) L1 = new Set();
  if (L2.size === 1 && L2.has("λ")) L2 = new Set();
  const resultado = new Set<string>([...L1, ...L2]);
  return `La unión de los conjuntos es {${Array.from(resultado).sort().join(', ')}}`;
}

  interseccion(L1: Set<string>, L2: Set<string>): string {
    const resultado = new Set<string>();
    L1.forEach(elem => {
      if (L2.has(elem)) {
        resultado.add(elem);
      }
    });

    if (resultado.size === 0) {
        return `La intersección de los conjuntos es {${this.landa}}`;
    } else {
        return `La intersección de los conjuntos es {${Array.from(resultado).sort().join(', ')}}`;
    }

  }

  sustraccion(L1: Set<string>, L2: Set<string>): string {
    const resultado = new Set<string>();
    L1.forEach(elem => {
      if (!L2.has(elem)) {
        resultado.add(elem);
      }
    });
    if (resultado.size === 0) {
      return `La sustracción de los conjuntos es {${this.landa}}`;
    } else {
      return `La sustracción de los conjuntos es {${Array.from(resultado).sort().join(', ')}}`;
    }
  }

  cierreKleene(L: Set<string>, maxDepth: number): string {
    let resultado = new Set<string>([this.landa]);
    let paso = new Set<string>(L);
    
    for (let i = 1; i <= maxDepth; i++) {
      resultado = new Set([...resultado, ...paso]);
      
      const nuevoPaso = new Set<string>();
      paso.forEach(a => {
        L.forEach(b => {
          nuevoPaso.add(a + b);
        });
      });
      paso = nuevoPaso;
    }
    
    return `El cierre de Kleene es {${this.landa}, ${Array.from(resultado).filter(x => x !== this.landa).sort().join(', ')}, ...}`;
  }

  cierrePositivo(L: Set<string>, maxDepth: number): string {
    let resultado = new Set<string>(L);
    let paso = new Set<string>(L);
    
    for (let i = 2; i <= maxDepth; i++) {
      const nuevoPaso = new Set<string>();
      paso.forEach(a => {
        L.forEach(b => {
          nuevoPaso.add(a + b);
        });
      });
      paso = nuevoPaso;
      resultado = new Set([...resultado, ...paso]);
    }
    
    return `El cierre positivo es {${Array.from(resultado).sort().join(', ')}, ...}`;
  }
}

const operacionesDeLenguaje = new OperacionesDeLenguaje();

// Ejecutar operación
const ejecutarOperacion = () => {
  if (!validarEntrada()) {
    error.value = 'Por favor, complete los campos requeridos.';
    return;
  }
  
  const conjuntoA = procesarConjunto(conjunto1.value);
  const conjuntoB = procesarConjunto(conjunto2.value);
  
  try {
    switch (operacionActiva.value) {
      case 'concatenation':
        resultado.value = operacionesDeLenguaje.concatenacion(conjuntoA, conjuntoB);
        break;
      case 'empowerment':
        resultado.value = operacionesDeLenguaje.potenciamiento(conjuntoA, potencia.value);
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
    error.value = `Error al ejecutar la operación: ${err instanceof Error ? err.message : String(err)}`;
  }
};

// Seleccionar una operación por defecto al cargar la página
onMounted(() => {
  prepararOperacion('concatenation');
});
</script>