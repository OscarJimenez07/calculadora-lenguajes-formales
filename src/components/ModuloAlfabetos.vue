<template>
  <div>
    <div>
      <h2>Operaciones con Alfabetos</h2>      
      <div>
        <button @click="prepararOperacion('pertenencia')" :class="operacionActiva === 'pertenencia'"><div>Pertenencia</div></button>
        <button @click="prepararOperacion('union')":class="operacionActiva === 'union'"><div>Unión</div></button>
        <button @click="prepararOperacion('interseccion')":class="operacionActiva === 'interseccion'"><div>Intersección</div></button>
        <button @click="prepararOperacion('diferencia_simetrica')":class="operacionActiva === 'diferencia_simetrica'"><div>Diferencia Simétrica</div></button>
        <button @click="prepararOperacion('complemento')":class="operacionActiva === 'complemento'"><div>Complemento</div></button>
        <button @click="prepararOperacion('diferencia_absoluta')":class="operacionActiva === 'diferencia_absoluta'"><div>Diferencia Absoluta</div></button>
      </div>
    </div>  

    <hr>
    <h3>Añadir alfabeto</h3>
    <div class="input-container">
      <input 
        v-model="nuevoAlfabetoSimbolos" 
        type="text" 
        placeholder="Símbolos separados por coma (ej: a, b, c)"
        class="input-large"
      />
      <button @click="agregarAlfabeto">Añadir Alfabeto</button>
    </div>
     
    <hr>
    <h3>Alfabetos creados</h3>
    <div v-if="Object.keys(alfabetos).length === 0">
       No hay alfabetos creados. Crea uno para comenzar.
    </div>
    <div class="alfabetos-seleccion">
      <div 
        v-for="(alfabeto, nombre) in alfabetos" 
        :key="nombre" 
        class="alfabeto-item"
        :class="{ 'alfabeto-seleccionado': conjuntosSeleccionados.includes(nombre) }"
        @click="toggleSeleccionConjunto(nombre)"
      >
        <span>{{ nombre }}:</span> 
        <span>{{ alfabeto.join(', ') }}</span>
        <button @click.stop="eliminarAlfabeto(nombre)">×</button>
      </div>
    </div>
  
    <div v-if="operacionActiva">
      <h3>{{ obtenerNombreOperacion(operacionActiva) }}</h3>
      <div v-if="Object.keys(alfabetos).length > 0">
        <div v-if="operacionActiva === 'pertenencia'">
          <label>Símbolo a verificar</label>
          <input 
            v-model="simboloEntrada" 
            type="text" 
            placeholder="Símbolo a verificar" 
            class="input-large"
          />
        </div>   
        <button 
          @click="ejecutarOperacion" 
          :disabled="!(operacionActiva === 'complemento' || conjuntosSeleccionados.length > 0)"
        >
          Ejecutar Operación
        </button>
      </div>
      <div v-else>
        Primero debes añadir al menos un alfabeto para realizar operaciones.
      </div>
    </div>
    
    <!-- Resultado -->
    <div v-if="resultado">
      <h3>Resultado:</h3>
      <span v-if="Array.isArray(resultado)">{{ resultado.join(', ') }}</span>
      <span v-else>{{ resultado }}</span>
    </div>
    
    <!-- Mensajes de error -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';

// Variables para añadir nuevos alfabetos
const nuevoAlfabetoSimbolos = ref('');

// Definimos los alfabetos como un objeto reactivo vacío
const alfabetos = reactive<Record<string, string[]>>({});

// Variable para controlar el contador de conjuntos
const contadorConjuntos = ref(1);

// Conjuntos seleccionados para operaciones
const conjuntosSeleccionados = ref<string[]>([]);

// Función para añadir un conjunto a la selección o quitarlo
const toggleSeleccionConjunto = (nombre: string) => {
  const index = conjuntosSeleccionados.value.indexOf(nombre);
  if (index > -1) {
    // Si ya está seleccionado, lo quito
    conjuntosSeleccionados.value.splice(index, 1);
  } else {
    // Si no está seleccionado, lo añado
    conjuntosSeleccionados.value.push(nombre);
  }
};

// Función para agregar un nuevo alfabeto
const agregarAlfabeto = () => {
  const simbolosTexto = nuevoAlfabetoSimbolos.value.trim();
  const nombre = `Conjunto ${contadorConjuntos.value}`;
  
  if (!simbolosTexto) {
    error.value = 'Error: Debes proporcionar al menos un símbolo para el alfabeto.';
    return;
  }
  
  const simbolos = Array.from(new Set(
    simbolosTexto.split(',').map(s => s.trim()).filter(s => s !== '')
  ));
  
  if (simbolos.length === 0) {
    error.value = 'Error: No se pudieron procesar los símbolos ingresados.';
    return;
  }
  
  alfabetos[nombre] = simbolos;
  
  nuevoAlfabetoSimbolos.value = '';
  error.value = '';
  contadorConjuntos.value++;
};

// Función para eliminar un alfabeto
const eliminarAlfabeto = (nombre: string) => {
  delete alfabetos[nombre];
  // Quitar de conjuntos seleccionados si estaba seleccionado
  const index = conjuntosSeleccionados.value.indexOf(nombre);
  if (index > -1) {
    conjuntosSeleccionados.value.splice(index, 1);
  }
};

// Obtener nombre de operación
const obtenerNombreOperacion = (id: string): string => {
  const nombres: Record<string, string> = {
    'pertenencia': 'Pertenencia',
    'union': 'Unión',
    'interseccion': 'Intersección',
    'diferencia_simetrica': 'Diferencia Simétrica',
    'complemento': 'Complemento',
    'diferencia_absoluta': 'Diferencia Absoluta'
  };
  return nombres[id] || id;
};

// Variables reactivas
const operacionActiva = ref('');
const simboloEntrada = ref('');
const resultado = ref<string | string[]>('');
const error = ref('');

// Prepare operation function
const prepararOperacion = (operacionId: string) => {
  operacionActiva.value = operacionId;
  conjuntosSeleccionados.value = [];
  simboloEntrada.value = '';
  resultado.value = '';
  error.value = '';
};

// Seleccionar una operación por defecto al cargar la página
onMounted(() => {
  prepararOperacion('pertenencia');
});

// Execute operation function
const ejecutarOperacion = () => {
  resultado.value = '';

  // Para complemento, necesitamos todos los conjuntos
  const nombresAlfabetos = operacionActiva.value === 'complemento' 
    ? Object.keys(alfabetos) 
    : conjuntosSeleccionados.value;
  
  // Validate input alphabets
  const alfabetosValidos = nombresAlfabetos.filter(nombre => alfabetos.hasOwnProperty(nombre));
  const alfabetosInvalidos = nombresAlfabetos.filter(nombre => !alfabetos.hasOwnProperty(nombre));
  
  if (alfabetosInvalidos.length > 0) {
    error.value = `Error: Los alfabetos ${alfabetosInvalidos.join(', ')} no son válidos.`;
    return;
  }
  
  // Validación de símbolos para pertenencia
  if (operacionActiva.value === 'pertenencia' && !simboloEntrada.value) {
    error.value = 'Error: Debes ingresar un símbolo para verificar su pertenencia.';
    return;
  }
  
  switch (operacionActiva.value) {
    case 'pertenencia':
      resultado.value = verificarPertenencia(alfabetosValidos, simboloEntrada.value);
      break;
    case 'union':
      resultado.value = calcularUnion(alfabetosValidos);
      break;
    case 'interseccion':
      resultado.value = calcularInterseccion(alfabetosValidos);
      break;
    case 'diferencia_simetrica':
      resultado.value = calcularDiferenciaSimetrica(alfabetosValidos);
      break;
    case 'complemento':
      resultado.value = calcularComplemento(alfabetosValidos);
      break;
    case 'diferencia_absoluta':
      resultado.value = calcularDiferenciaAbsoluta(alfabetosValidos);
      break;
  }
};

const verificarPertenencia = (nombresAlfabetos: string[], simbolo: string): string => {
  const alfabetosQueContienen = nombresAlfabetos.filter(nombre => 
    alfabetos[nombre].includes(simbolo)
  );
  if (alfabetosQueContienen.length === 0) {
    return `El símbolo '${simbolo}' no pertenece a ninguno de los alfabetos seleccionados.`;
  } else if (alfabetosQueContienen.length === nombresAlfabetos.length) {
    return `El símbolo '${simbolo}' pertenece a todos los alfabetos seleccionados.`;
  } else {
    return `El símbolo '${simbolo}' pertenece a los alfabetos: ${alfabetosQueContienen.join(', ')}.`;
  }
};

const calcularUnion = (nombresAlfabetos: string[]): string[] => {
  const union = new Set<string>();
  nombresAlfabetos.forEach(nombre => {
    alfabetos[nombre].forEach(simbolo => union.add(simbolo));
  });
  return Array.from(union).sort();
};

const calcularInterseccion = (nombresAlfabetos: string[]): string[] => {
  if (nombresAlfabetos.length === 0) return [];
  if (nombresAlfabetos.length === 1) return [...alfabetos[nombresAlfabetos[0]]];
  let interseccion = [...alfabetos[nombresAlfabetos[0]]];
  for (let i = 1; i < nombresAlfabetos.length; i++) {
    const currentAlfabeto = alfabetos[nombresAlfabetos[i]];
    interseccion = interseccion.filter(simbolo => currentAlfabeto.includes(simbolo));
  }
  return interseccion;
};

const calcularDiferenciaSimetrica = (nombresAlfabetos: string[]): string[] => {
  if (nombresAlfabetos.length === 0) return [];
  if (nombresAlfabetos.length === 1) return [...alfabetos[nombresAlfabetos[0]]];
  
  const union = calcularUnion(nombresAlfabetos);
  const interseccion = calcularInterseccion(nombresAlfabetos);
  return union.filter(simbolo => !interseccion.includes(simbolo));
};

const calcularComplemento = (nombresAlfabetos: string[]): string[] => {
  const universo = new Set<string>();
  Object.values(alfabetos).forEach(alfabeto => {
    alfabeto.forEach(simbolo => universo.add(simbolo));
  });
  const union = calcularUnion(nombresAlfabetos);
  return Array.from(universo).filter(simbolo => !union.includes(simbolo));
};

const calcularDiferenciaAbsoluta = (nombresAlfabetos: string[]): string[] => {
  if (nombresAlfabetos.length < 2) {
    return [];
  }

  const primerAlfabeto = new Set(alfabetos[nombresAlfabetos[0]]);

  const restoUnidos = new Set<string>();
  for (let i = 1; i < nombresAlfabetos.length; i++) {
    alfabetos[nombresAlfabetos[i]].forEach(simbolo => restoUnidos.add(simbolo));
  }

  return Array.from(primerAlfabeto).filter(simbolo => !restoUnidos.has(simbolo));
};
</script>

<style>
.input-container {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.input-large {
  flex-grow: 1;
  padding: 8px;
  font-size: 16px;
}

.alfabetos-seleccion {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.alfabeto-item {
  margin-bottom: 10px;
  padding: 8px;
  border: 1px solid #ccc;
  cursor: pointer;
  transition: background-color 0.3s;
  display: flex;
  align-items: center;
  gap: 10px;
}

.alfabeto-item:hover {
  background-color: gray;
}

.alfabeto-seleccionado {
  background-color: gray;
  border-color: blue;
}

.error-message {
  color: red;
  margin-top: 10px;
}

button {
  padding: 8px 15px;
}
</style>