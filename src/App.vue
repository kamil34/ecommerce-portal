<template>
  <v-app>
    <v-main>
      <AppNavbars />
      <LoadingContent v-if="isLoading" />
      <router-view v-show="!isLoading" />
    </v-main>
  </v-app>
</template>

<script setup>
//
import { storeToRefs } from 'pinia';
import { useLoadingStore } from '@/stores/loading';

const { isLoading } = storeToRefs(useLoadingStore());

</script>
