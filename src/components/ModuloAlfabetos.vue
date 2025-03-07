<template>
  <div>
    <div>
      <h2>Operaciones con Alfabetos</h2>      
      <div>
        <button @click="prepararOperacion('pertenencia')" :class="operacionActiva === 'pertenencia'"><div>Pertenencia</div></button>
        <button @click="prepararOperacion('union')" :class="operacionActiva === 'union'"><div>Unión</div></button>
        <button @click="prepararOperacion('interseccion')" :class="operacionActiva === 'interseccion'"><div>Intersección</div></button>
        <button @click="prepararOperacion('complemento')" :class="operacionActiva === 'complemento'"><div>Complemento</div></button>
        <button @click="prepararOperacion('diferencia_absoluta')" :class="operacionActiva === 'diferencia_absoluta'"><div>Diferencia Absoluta</div></button>
        <button @click="prepararOperacion('diferencia_simetrica')" :class="operacionActiva === 'diferencia_simetrica'"><div>Diferencia Simétrica</div></button>
      </div>
    </div> 
  <hr>
    <h3>Añadir alfabeto</h3>
    <div class="input-container">
      <input v-model="nuevoAlfabetoSimbolos" type="text" placeholder="Símbolos separados por espacio (ej: a b c)"/>
      
      <button @click="agregarAlfabeto">Añadir Alfabeto</button>
    </div>  
  <hr>
    <h3>Alfabetos creados</h3>
    <div v-if="Object.keys(alfabetos).length === 0">
       No hay alfabetos creados. Crea uno para comenzar.
    </div>

    <div class="alfabetos-seleccion">
      <div 
        v-for="(alfabeto, nombre) in alfabetos" :key="nombre" class="alfabeto-item"
        :class="{ 'alfabeto-seleccionado': conjuntosSeleccionados.includes(nombre) }"
        @click="ConjuntoSeleccionado(nombre)">
        <span>{{ nombre }}:</span> <span>{{ alfabeto.join(', ') }}</span>
        <button @click.stop="eliminarAlfabeto(nombre)">×</button>
      </div>
    </div>
  
    <div v-if="operacionActiva">
      <h3>{{ obtenerNombreOperacion(operacionActiva) }}</h3>
      <div v-if="Object.keys(alfabetos).length > 0" class="input-container">

        <div v-if="operacionActiva === 'complemento'">
          <label>Conjunto Universo: </label>
          <select v-model="universoSeleccionado">
            <option v-for="(_, nombre) in alfabetos" :key="nombre" :value="nombre">{{ nombre }}</option>
          </select>
        </div>

        <button @click="ejecutarOperacion" :disabled="!puedeEjecutarOperacion">
          Ejecutar Operación
        </button>
      </div>

      <div v-else>
        Primero debes añadir al menos un alfabeto para realizar operaciones.
      </div>
    </div>

    <!-- Resultado -->
    <div v-if="resultadoA || resultadoB">
      <h3>Resultado:</h3>
      <div v-if="operacionActiva === 'pertenencia'">
        <div>Elementos que pertenecen: {{ resultadoA.join(', ') }}</div>
        <div>Elementos que no pertenecen: {{ resultadoB.join(', ') }}</div>
      </div>
      <div v-else-if="operacionActiva === 'diferencia_absoluta'">
        <div>Diferencia Absoluta A-B: {{ resultadoA.join(', ') }}</div>
        <div>Diferencia Absoluta B-A: {{ resultadoB.join(', ') }}</div>
      </div>
      <div v-else>
        <div>{{ Array.isArray(resultadoA) ? resultadoA.join(', ') : resultadoA }}</div>
      </div>
    </div>
    
    <!-- Mensajes de error -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from 'vue';

// Definición de Tipos
type AlfabetosType = Record<string, string[]>;
type OperacionesType = Record<string, string>;

// Constantes
const NOMBRES_OPERACIONES: OperacionesType = {
  'pertenencia': 'Pertenencia',
  'union': 'Unión',
  'interseccion': 'Intersección',
  'diferencia_simetrica': 'Diferencia Simétrica',
  'complemento': 'Complemento',
  'diferencia_absoluta': 'Diferencia Absoluta'
};

// Variables para añadir nuevos alfabetos
const nuevoAlfabetoSimbolos = ref('');
const alfabetos = reactive<AlfabetosType>({});
const contadorConjuntos = ref(1);
const conjuntosSeleccionados = ref<string[]>([]);
const universoSeleccionado = ref('');

// Variables reactivas para operaciones
const operacionActiva = ref('pertenencia');
const resultadoA = ref<string[]>([]);
const resultadoB = ref<string[]>([]);
const error = ref('');

// Computed properties
const puedeEjecutarOperacion = computed(() => {
  if (operacionActiva.value === 'complemento') {
    return universoSeleccionado.value !== '' && conjuntosSeleccionados.value.length === 1;
  } else {
    return conjuntosSeleccionados.value.length > 0;
  }
});

// Funciones de Gestión de Alfabetos
const ConjuntoSeleccionado = (nombre: string) => {
  const index = conjuntosSeleccionados.value.indexOf(nombre);
  
  if (index > -1) {
    conjuntosSeleccionados.value.splice(index, 1);
  } else {
    // Si la operación solo necesita 2 conjuntos, limitar a 2 selecciones
    if ((operacionActiva.value === 'pertenencia' || 
         operacionActiva.value === 'diferencia_absoluta') && 
        conjuntosSeleccionados.value.length >= 2) {
      conjuntosSeleccionados.value = [];
    }
    
    // Para complemento, solo permitir seleccionar uno
    if (operacionActiva.value === 'complemento' && conjuntosSeleccionados.value.length >= 1) {
      conjuntosSeleccionados.value = [];
    }
    
    conjuntosSeleccionados.value.push(nombre);
  }
};

const agregarAlfabeto = () => {
  let simbolosTexto = nuevoAlfabetoSimbolos.value.trim();
  const nombre = `Conjunto ${contadorConjuntos.value}`;

  // Manejar caso de conjunto vacío
  if (!simbolosTexto) {
    alfabetos[nombre] = ["λ"]; // Asignar 'λ' si no hay entrada
  } else {
    // Dividir por espacios como en Python
    const simbolos = Array.from(
      new Set(
        simbolosTexto
          .split(/\s+/)
          .map(s => s.trim())
          .filter(s => s !== '')
      )
    );
    
    alfabetos[nombre] = simbolos.length ? simbolos : ["λ"];
  }

  nuevoAlfabetoSimbolos.value = '';
  error.value = '';
  contadorConjuntos.value++;
};

const eliminarAlfabeto = (nombre: string) => {
  delete alfabetos[nombre];
  const index = conjuntosSeleccionados.value.indexOf(nombre);
  if (index > -1) {
    conjuntosSeleccionados.value.splice(index, 1);
  }
  
  if (universoSeleccionado.value === nombre) {
    universoSeleccionado.value = '';
  }
};

// Funciones de Utilidad
const obtenerNombreOperacion = (id: string): string => NOMBRES_OPERACIONES[id];

const prepararOperacion = (operacionId: string) => {
  operacionActiva.value = operacionId;
  conjuntosSeleccionados.value = [];
  universoSeleccionado.value = '';
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
        if (conjuntosSeleccionados.value.length < 2) {
          error.value = "Se necesitan dos conjuntos para la operación de pertenencia.";
          return;
        }
        
        const [conjA, conjB] = conjuntosSeleccionados.value;
        const conjuntoA = new Set(alfabetos[conjA]);
        const conjuntoB = new Set(alfabetos[conjB]);
        
        // Calcular elementos que pertenecen (intersección)
        const pertenecen = [...conjuntoB].filter(elem => conjuntoA.has(elem));
        
        // Calcular elementos que no pertenecen (diferencia)
        const noPertenecen = [...conjuntoB].filter(elem => !conjuntoA.has(elem));
        
        resultadoA.value = pertenecen.length ? pertenecen.sort() : ["λ"];
        resultadoB.value = noPertenecen.length ? noPertenecen.sort() : ["λ"];
        break;
        
      case 'union':
        if (conjuntosSeleccionados.value.length < 2) {
          error.value = "Se necesitan al menos dos conjuntos para la unión.";
          return;
        }
        
        const union = new Set<string>();
        conjuntosSeleccionados.value.forEach(nombre => {
          alfabetos[nombre].forEach(simbolo => {
            if (simbolo !== "λ") union.add(simbolo);
          });
        });
        
        resultadoA.value = union.size ? [...union].sort() : ["λ"];
        break;
        
      case 'interseccion':
        if (conjuntosSeleccionados.value.length < 2) {
          error.value = "Se necesitan al menos dos conjuntos para la intersección.";
          return;
        }
        
        let interseccion: string[] = [];
        const primerConjunto = alfabetos[conjuntosSeleccionados.value[0]].filter(s => s !== "λ");
        
        if (primerConjunto.length) {
          interseccion = [...primerConjunto];
          
          for (let i = 1; i < conjuntosSeleccionados.value.length; i++) {
            const conjuntoActual = alfabetos[conjuntosSeleccionados.value[i]].filter(s => s !== "λ");
            interseccion = interseccion.filter(simbolo => conjuntoActual.includes(simbolo));
          }
        }
        
        resultadoA.value = interseccion.length ? interseccion.sort() : ["λ"];
        break;
        
      case 'complemento':
        if (conjuntosSeleccionados.value.length !== 1 || !universoSeleccionado.value) {
          error.value = "Para el complemento, se necesita un conjunto y el universo.";
          return;
        }
        
        const conjunto = new Set(alfabetos[conjuntosSeleccionados.value[0]].filter(s => s !== "λ"));
        const universo = new Set(alfabetos[universoSeleccionado.value].filter(s => s !== "λ"));
        
        const complemento = [...universo].filter(simbolo => !conjunto.has(simbolo));
        resultadoA.value = complemento.length ? complemento.sort() : ["λ"];
        break;
        
      case 'diferencia_absoluta':
        if (conjuntosSeleccionados.value.length < 2) {
          error.value = "Se necesitan dos conjuntos para la diferencia absoluta.";
          return;
        }
        
        const [conjuntoNombreA, conjuntoNombreB] = conjuntosSeleccionados.value;
        const conjA_elementos = alfabetos[conjuntoNombreA].filter(s => s !== "λ");
        const conjB_elementos = alfabetos[conjuntoNombreB].filter(s => s !== "λ");
        
        const conjuntoASet = new Set(conjA_elementos);
        const conjuntoBSet = new Set(conjB_elementos);
        
        // A - B
        const diferenciaAB = [...conjuntoASet].filter(simbolo => !conjuntoBSet.has(simbolo));
        
        // B - A
        const diferenciaBA = [...conjuntoBSet].filter(simbolo => !conjuntoASet.has(simbolo));
        
        resultadoA.value = diferenciaAB.length ? diferenciaAB.sort() : ["λ"];
        resultadoB.value = diferenciaBA.length ? diferenciaBA.sort() : ["λ"];
        break;
        
      case 'diferencia_simetrica':
        if (conjuntosSeleccionados.value.length < 2) {
          error.value = "Se necesitan al menos dos conjuntos para la diferencia simétrica.";
          return;
        }
        
        const conjuntos = conjuntosSeleccionados.value.map(nombre => 
          new Set(alfabetos[nombre].filter(s => s !== "λ"))
        );
        
        // Unión de todos los conjuntos
        const unionTotal = new Set<string>();
        conjuntos.forEach(conj => conj.forEach(simbolo => unionTotal.add(simbolo)));
        
        // Intersección de todos los conjuntos
        let interseccionTotal: string[] = [];
        if (conjuntos[0].size) {
          interseccionTotal = [...conjuntos[0]];
          for (let i = 1; i < conjuntos.length; i++) {
            interseccionTotal = interseccionTotal.filter(simbolo => conjuntos[i].has(simbolo));
          }
        }
        
        // Diferencia simétrica
        const diferenciaSimetrica = [...unionTotal].filter(simbolo => !interseccionTotal.includes(simbolo));
        resultadoA.value = diferenciaSimetrica.length ? diferenciaSimetrica.sort() : ["λ"];
        break;
    }
  } catch (e) {
    error.value = `Error en la operación: ${e}`;
  }
};
</script>