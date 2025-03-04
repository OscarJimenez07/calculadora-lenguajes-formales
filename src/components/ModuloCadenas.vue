<template>
  <div>
    <div>
      <h2>Operaciones con Cadenas</h2>      
      <div>
        <button 
          v-for="op in operaciones" 
          :key="op.id"
          @click="prepararOperacion(op.id)" 
          :class="operacionActiva === op.id"
        >
          <div>{{ op.nombre }}</div>
        </button>
      </div>
    </div>  

    <hr>
    <h3>Añadir Cadena</h3>
    <div class="input-container">
      <input 
        v-model="valorCadena" 
        type="text" 
        placeholder="Cadena (ej: abc, xyz, 123)"
        class="input-large"
      />
      <button @click="agregarCadena">Añadir Cadena</button>
    </div>
     
    <hr>
    <h3>Cadenas creadas</h3>
    <div v-if="Object.keys(cadenas).length === 0">
       No hay cadenas creadas. Crea una para comenzar.
    </div>
    <div class="cadenas-seleccion">
      <div 
        v-for="(cadena, nombre) in cadenas" 
        :key="nombre" 
        class="cadena-item"
        :class="{ 'cadena-seleccionada': cadenasSeleccionadas.includes(nombre) }"
        @click="toggleSeleccionCadena(nombre)"
      >
        <span>{{ nombre }}:</span> 
        <span>"{{ cadena }}"</span>
        <button @click.stop="eliminarCadena(nombre)">×</button>
      </div>
    </div>
  
    <div v-if="operacionActiva">
      <h3>{{ obtenerNombreOperacion(operacionActiva) }}</h3>
      <div v-if="Object.keys(cadenas).length > 0">
        <div v-if="operacionActiva === 'subcadena'">
          <label>Subcadena a buscar</label>
          <input 
            v-model="subcadenaEntrada" 
            type="text" 
            placeholder="Subcadena a buscar" 
            class="input-large"
          />
        </div>   
        
        <div v-if="operacionActiva === 'potencia'">
          <label>Exponente</label>
          <input 
            v-model="potenciaEntrada" 
            type="number" 
            min="0"
            placeholder="Exponente" 
            class="input-large"
          />
        </div>
        
        <div v-if="operacionActiva === 'apendizar'">
          <label>Símbolo a agregar</label>
          <input 
            v-model="apendizarEntrada" 
            type="text" 
            placeholder="Símbolo a agregar" 
            class="input-large"
          />
        </div>

        <button 
          @click="ejecutarOperacion" 
          :disabled="!(operacionActiva === 'concatenacion' || cadenasSeleccionadas.length > 0)"
        >
          Ejecutar Operación
        </button>
      </div>
      <div v-else>
        Primero debes añadir al menos una cadena para realizar operaciones.
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

// Definimos las cadenas de manera reactiva
const cadenas = reactive<Record<string, string>>({});

// Variables para añadir nuevas cadenas
const valorCadena = ref('');
const contadorCadenas = ref(1);

// Cadenas seleccionadas
const cadenasSeleccionadas = ref<string[]>([]);

// Función para agregar una nueva cadena
const agregarCadena = () => {
  const valor = valorCadena.value.trim();

  if (!valor) {
    error.value = 'Error: Debes proporcionar un valor para la cadena.';
    return;
  }

  const nombre = `Cadena ${contadorCadenas.value}`;
  cadenas[nombre] = valor;
  valorCadena.value = '';
  contadorCadenas.value++;
  error.value = '';
};

// Función para eliminar una cadena
const eliminarCadena = (nombre: string) => {
  delete cadenas[nombre];
  const index = cadenasSeleccionadas.value.indexOf(nombre);
  if (index > -1) {
    cadenasSeleccionadas.value.splice(index, 1);
  }
};

// Función para seleccionar/deseleccionar cadenas
const toggleSeleccionCadena = (nombre: string) => {
  const index = cadenasSeleccionadas.value.indexOf(nombre);
  if (index > -1) {
    cadenasSeleccionadas.value.splice(index, 1);
  } else {
    cadenasSeleccionadas.value.push(nombre);
  }
};

// Definimos las operaciones disponibles
const operaciones = [
  { 
    id: 'concatenacion', 
    nombre: 'Concatenación', 
  },
  { 
    id: 'longitud', 
    nombre: 'Longitud', 
  },
  { 
    id: 'subcadena', 
    nombre: 'Búsqueda de Subcadena', 
  },
  { 
    id: 'inverso', 
    nombre: 'Inverso', 
  },
  { 
    id: 'potencia', 
    nombre: 'Potencia', 
  },
  { 
    id: 'apendizar', 
    nombre: 'Apendizar', 
  }
];

// Obtener nombre de operación
const obtenerNombreOperacion = (id: string): string => {
  const nombres: Record<string, string> = {
    'concatenacion': 'Concatenación',
    'longitud': 'Longitud',
    'subcadena': 'Búsqueda de Subcadena',
    'inverso': 'Inverso',
    'potencia': 'Potencia',
    'apendizar': 'Apendizar'
  };
  return nombres[id] || id;
};

// Variables reactivas
const operacionActiva = ref('');
const subcadenaEntrada = ref('');
const potenciaEntrada = ref<number | null>(null);
const apendizarEntrada = ref('');
const resultado = ref<string | string[]>('');
const error = ref('');

// Prepare operation function
const prepararOperacion = (operacionId: string) => {
  operacionActiva.value = operacionId;
  cadenasSeleccionadas.value = [];
  subcadenaEntrada.value = '';
  potenciaEntrada.value = null;
  apendizarEntrada.value = '';
  resultado.value = '';
  error.value = '';
};

// Seleccionar una operación por defecto al cargar la página
onMounted(() => {
  prepararOperacion('concatenacion');
});

// Execute operation function
const ejecutarOperacion = () => {
  // Reset previous results and errors
  resultado.value = '';
  error.value = '';
    
  // Para concatenación, necesitamos todos los conjuntos
  const nombresCadenas = operacionActiva.value === 'concatenacion' 
    ? Object.keys(cadenas) 
    : cadenasSeleccionadas.value;
  
  // Validate input cadenas
  const cadenasValidas = nombresCadenas.filter(nombre => cadenas.hasOwnProperty(nombre));
  const cadenasInvalidas = nombresCadenas.filter(nombre => !cadenas.hasOwnProperty(nombre));
  
  if (cadenasInvalidas.length > 0) {
    error.value = `Error: Las cadenas ${cadenasInvalidas.join(', ')} no son válidas.`;
    return;
  }
  
  // Validación de entrada específica para algunas operaciones
  if (operacionActiva.value === 'subcadena' && !subcadenaEntrada.value) {
    error.value = 'Error: Debes ingresar una subcadena para buscar.';
    return;
  }
  
  if (operacionActiva.value === 'potencia' && potenciaEntrada.value === null) {
    error.value = 'Error: Debes ingresar un valor de exponente.';
    return;
  }
  
  if (operacionActiva.value === 'apendizar' && !apendizarEntrada.value) {
    error.value = 'Error: Debes ingresar un símbolo para apendizar.';
    return;
  }
  
  // Execute the selected operation
  switch (operacionActiva.value) {
    case 'concatenacion':
      resultado.value = calcularConcatenacion(cadenasValidas);
      break;
    case 'longitud':
      resultado.value = calcularLongitud(cadenasValidas);
      break;
    case 'subcadena':
      resultado.value = buscarSubcadena(cadenasValidas, subcadenaEntrada.value);
      break;
    case 'inverso':
      resultado.value = calcularInverso(cadenasValidas);
      break;
    case 'potencia':
      resultado.value = calcularPotencia(cadenasValidas, potenciaEntrada.value!);
      break;
    case 'apendizar':
      resultado.value = calcularApendizar(cadenasValidas, apendizarEntrada.value);
      break;
  }
};

// Operation implementations
const calcularConcatenacion = (nombresCadenas: string[]): string => {
  let resultado = '';
  nombresCadenas.forEach(nombre => {
    resultado += cadenas[nombre];
  });
  return resultado;
};

const calcularLongitud = (nombresCadenas: string[]): string[] => {
  return nombresCadenas.map(nombre => {
    const cadena = cadenas[nombre];
    return `${nombre}: ${cadena.length} caracteres`;
  });
};

const buscarSubcadena = (nombresCadenas: string[], subcadena: string): string => {
  const cadenasConcidencias = nombresCadenas.filter(nombre => 
    cadenas[nombre].includes(subcadena)
  );
  
  if (cadenasConcidencias.length > 0) {
    return `La subcadena "${subcadena}" fue encontrada en: ${cadenasConcidencias.join(', ')}`;
  } else {
    return `La subcadena "${subcadena}" no fue encontrada en ninguna cadena seleccionada.`;
  }
};

const calcularInverso = (nombresCadenas: string[]): string[] => {
  return nombresCadenas.map(nombre => {
    const cadena = cadenas[nombre];
    const inverso = cadena.split('').reverse().join('');
    return `${nombre} inverso: "${inverso}"`;
  });
};

const calcularPotencia = (nombresCadenas: string[], exponente: number): string[] => {
  return nombresCadenas.map(nombre => {
    const cadena = cadenas[nombre];
    let resultado = '';
    
    for (let i = 0; i < exponente; i++) {
      resultado += cadena;
    }
    
    return `${nombre}^${exponente}: "${resultado}"`;
  });
};

const calcularApendizar = (nombresCadenas: string[], simbolo: string): string[] => {
  return nombresCadenas.map(nombre => {
    const cadena = cadenas[nombre];
    return `${nombre} + "${simbolo}": "${cadena + simbolo}"`;
  });
};
</script>

<style scoped>
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

.cadenas-seleccion {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.cadena-item {
  margin-bottom: 10px;
  padding: 8px;
  border: 1px solid #ccc;
  cursor: pointer;
  transition: background-color 0.3s;
  display: flex;
  align-items: center;
  gap: 10px;
}

.cadena-item:hover {
  background-color: gray;
}

.cadena-seleccionada {
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