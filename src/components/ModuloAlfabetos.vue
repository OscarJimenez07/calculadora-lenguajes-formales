<template>
  <div>
    <div>
      <h2>Operaciones con Alfabetos</h2>      
      <div>
        <button @click="prepararOperacion('pertenencia')" :class="{ active: operacionActiva === 'pertenencia' }">Pertenencia</button>
        <button @click="prepararOperacion('union')" :class="{ active: operacionActiva === 'union' }">Unión</button>
        <button @click="prepararOperacion('interseccion')" :class="{ active: operacionActiva === 'interseccion' }">Intersección</button>
        <button @click="prepararOperacion('complemento')" :class="{ active: operacionActiva === 'complemento' }">Complemento</button>
        <button @click="prepararOperacion('diferencia_absoluta')" :class="{ active: operacionActiva === 'diferencia_absoluta' }">Diferencia Absoluta</button>
        <button @click="prepararOperacion('diferencia_simetrica')" :class="{ active: operacionActiva === 'diferencia_simetrica' }">Diferencia Simétrica</button>
      </div>
    </div> 
  <hr>
    <h3>Conjunto A</h3>
    <div class="input-container">
      <input v-model="conjuntoAInput" type="text" placeholder="Símbolos separados por coma (ej: a,b,c)"/>
      <button @click="actualizarConjuntoA">Actualizar A</button>
    </div>  
  
    <h3>Conjunto B</h3>
    <div class="input-container">
      <input v-model="conjuntoBInput" type="text" placeholder="Símbolos separados por coma (ej: a,b,c)"/>
      <button @click="actualizarConjuntoB">Actualizar B</button>
    </div>
  
    <h3>Conjuntos:</h3>
    <div class="conjuntos-display">
      <div class="conjunto-item">
        <span>Conjunto A:</span> <span>{{ mostrarConjunto(conjuntoA) }}</span>
      </div>
      <div class="conjunto-item">
        <span>Conjunto B:</span> <span>{{ mostrarConjunto(conjuntoB) }}</span>
      </div>
    </div>
  
    <div v-if="operacionActiva">
      <h3>{{ obtenerNombreOperacion(operacionActiva) }}</h3>
      <div class="input-container">
        <button @click="ejecutarOperacion">
          Ejecutar Operación
        </button>
      </div>
    </div>

    <!-- Resultado -->
    <div v-if="resultadoA.length > 0 || resultadoB.length > 0">
      <h3>Resultado:</h3>
      <div v-if="operacionActiva === 'pertenencia'">
        <div>Elementos que pertenecen: {{ mostrarConjunto(resultadoA) }}</div>
        <div>Elementos que no pertenecen: {{ mostrarConjunto(resultadoB) }}</div>
      </div>
      <div v-else-if="operacionActiva === 'diferencia_absoluta'">
        <div>Diferencia Absoluta A-B: {{ mostrarConjunto(resultadoA) }}</div>
        <div>Diferencia Absoluta B-A: {{ mostrarConjunto(resultadoB) }}</div>
      </div>
      <div v-else>
        <div>{{ mostrarConjunto(resultadoA) }}</div>
      </div>
    </div>
    
    <!-- Mensajes de error -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// Constantes
const nombresOperaciones = {
  'pertenencia': 'Pertenencia',
  'union': 'Unión',
  'interseccion': 'Intersección',
  'diferencia_simetrica': 'Diferencia Simétrica',
  'complemento': 'Complemento',
  'diferencia_absoluta': 'Diferencia Absoluta'
};

// Variables para los conjuntos A y B
const conjuntoAInput = ref('');
const conjuntoBInput = ref('');
const conjuntoA = ref(new Set(['λ']));
const conjuntoB = ref(new Set(['λ']));

// Variables reactivas para operaciones
const operacionActiva = ref('pertenencia');
const resultadoA = ref([]);
const resultadoB = ref([]);
const error = ref('');

// Funciones para gestionar los conjuntos
const actualizarConjuntoA = () => {
  const simbolos = conjuntoAInput.value.trim() ? 
    Array.from(new Set(
      conjuntoAInput.value
        .trim()
        .split(',')
        .map(s => s.trim())
        .filter(s => s !== '')
    )) : ['λ'];
    
  conjuntoA.value = new Set(simbolos);
  resultadoA.value = [];
  resultadoB.value = [];
};

const actualizarConjuntoB = () => {
  const simbolos = conjuntoBInput.value.trim() ? 
    Array.from(new Set(
      conjuntoBInput.value
        .trim()
        .split(',')
        .map(s => s.trim())
        .filter(s => s !== '')
    )) : ['λ'];
    
  conjuntoB.value = new Set(simbolos);
  resultadoA.value = [];
  resultadoB.value = [];
};

// Utilidad para mostrar conjuntos
const mostrarConjunto = (conjunto) => {
  const elementos = Array.isArray(conjunto) ? conjunto : Array.from(conjunto);
  return elementos.length === 0 ? 'λ' : elementos.join(', ');
};

// Funciones de Utilidad
const obtenerNombreOperacion = (id) => nombresOperaciones[id] || id;

const prepararOperacion = (operacionId) => {
  operacionActiva.value = operacionId;
  resultadoA.value = [];
  resultadoB.value = [];
  error.value = '';
};

// Implementación de las operaciones según la lógica de Python
const ejecutarOperacion = () => {
  resultadoA.value = [];
  resultadoB.value = [];
  error.value = '';
  
  try {
    switch (operacionActiva.value) {
      case 'pertenencia':
        // Calcular elementos que pertenecen (intersección)
        const pertenecen = Array.from(conjuntoB.value).filter(elem => 
          elem !== 'λ' && conjuntoA.value.has(elem)
        );
        
        // Calcular elementos que no pertenecen (diferencia)
        const noPertenecen = Array.from(conjuntoB.value).filter(elem => 
          elem !== 'λ' && !conjuntoA.value.has(elem)
        );
        
        resultadoA.value = pertenecen.length ? pertenecen.sort() : ['λ'];
        resultadoB.value = noPertenecen.length ? noPertenecen.sort() : ['λ'];
        break;
        
      case 'union':
        const union = new Set();
        conjuntoA.value.forEach(simbolo => {
          if (simbolo !== 'λ') union.add(simbolo);
        });
        conjuntoB.value.forEach(simbolo => {
          if (simbolo !== 'λ') union.add(simbolo);
        });
        
        resultadoA.value = union.size ? Array.from(union).sort() : ['λ'];
        break;
        
      case 'interseccion':
        const conjA_elementos = Array.from(conjuntoA.value).filter(s => s !== 'λ');
        const conjB_elementos = Array.from(conjuntoB.value).filter(s => s !== 'λ');
        
        const interseccion = conjA_elementos.filter(simbolo => 
          conjB_elementos.includes(simbolo)
        );
        
        resultadoA.value = interseccion.length ? interseccion.sort() : ['λ'];
        break;
        
      case 'complemento':
        // En este caso, B es el universo
        const conjunto = new Set(Array.from(conjuntoA.value).filter(s => s !== 'λ'));
        const universo = new Set(Array.from(conjuntoB.value).filter(s => s !== 'λ'));
        
        const complemento = Array.from(universo).filter(simbolo => !conjunto.has(simbolo));
        resultadoA.value = complemento.length ? complemento.sort() : ['λ'];
        break;
        
      case 'diferencia_absoluta':
        const conjA_elems = Array.from(conjuntoA.value).filter(s => s !== 'λ');
        const conjB_elems = Array.from(conjuntoB.value).filter(s => s !== 'λ');
        
        // A - B
        const diferenciaAB = conjA_elems.filter(simbolo => !conjB_elems.includes(simbolo));
        
        // B - A
        const diferenciaBA = conjB_elems.filter(simbolo => !conjA_elems.includes(simbolo));
        
        resultadoA.value = diferenciaAB.length ? diferenciaAB.sort() : ['λ'];
        resultadoB.value = diferenciaBA.length ? diferenciaBA.sort() : ['λ'];
        break;
        
      case 'diferencia_simetrica':
        const conjuntoAElems = new Set(Array.from(conjuntoA.value).filter(s => s !== 'λ'));
        const conjuntoBElems = new Set(Array.from(conjuntoB.value).filter(s => s !== 'λ'));
        
        // Unión
        const unionTotal = new Set();
        conjuntoAElems.forEach(simbolo => unionTotal.add(simbolo));
        conjuntoBElems.forEach(simbolo => unionTotal.add(simbolo));
        
        // Intersección
        const interseccionTotal = Array.from(conjuntoAElems).filter(simbolo => 
          conjuntoBElems.has(simbolo)
        );
        
        // Diferencia simétrica (elementos en la unión pero no en la intersección)
        const diferenciaSimetrica = Array.from(unionTotal).filter(simbolo => 
          !interseccionTotal.includes(simbolo)
        );
        
        resultadoA.value = diferenciaSimetrica.length ? diferenciaSimetrica.sort() : ['λ'];
        break;
    }
  } catch (e) {
    error.value = `Error en la operación: ${e}`;
  }
};
</script>