<template>
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
import { ref } from 'vue';
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
</script>
>>>>>>> Tabnine >>>>>>>// {"conversationId":"a3ae4d01-4286-48c3-b58c-c59514c5747d","source":"instruct"}