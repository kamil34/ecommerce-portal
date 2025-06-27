<template>
  <v-app-bar :elevation="5" class="bg-light-blue lighten-3">
    <template v-slot:prepend>
      <v-app-bar-nav-icon @click="navdrawer = !navdrawer"></v-app-bar-nav-icon>
    </template>
    <v-app-bar-title>Stal-Trade S.A.</v-app-bar-title>
  </v-app-bar>
  <v-navigation-drawer class="bg-light-blue lighten-3" v-model="navdrawer">
    <v-list>
      <v-list-item prepend-avatar="https://randomuser.me/api/portraits/women/85.jpg">
        <v-list-item-title>Alicja Kowalska</v-list-item-title>
        <v-list-item-subtitle>Konto B2B</v-list-item-subtitle>
        <v-list-item-subtitle>a.kowalska@company.net</v-list-item-subtitle>
      </v-list-item>
    </v-list>
    <v-divider></v-divider>
    <v-list density="compact" nav>
      <v-list-item v-for="(item, i) in items" :key="i" :to="item.link" :prepend-icon="item.icon" :title="item.text"
        :value="item.value" link></v-list-item>
    </v-list>
    <template v-slot:append>
      <div class="pa-2">
        <v-btn block>
          Wyloguj
        </v-btn>
      </div>
    </template>
  </v-navigation-drawer>
</template>

<script setup>
import { ref } from 'vue';

const navdrawer = ref(true);
const items = ref([
  { icon: 'mdi-home', text: 'Strona główna', link: '/', value: 'home' },
  { icon: 'mdi-account', text: 'Twoje zamówienia', link: '/customer', value: 'customer' },
  { icon: 'mdi-cart', text: 'Koszyk', link: '/cart', value: 'cart' },
  { icon: 'mdi-storefront', text: 'Panel dostawcy', link: '/supplier', value: 'supplier' },
]);
</script>