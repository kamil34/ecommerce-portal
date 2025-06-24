<template>
  <v-container fluid>
    <v-row justify="center">
      <v-col cols="12" md="9" class="text-left">
        <h1 class="text-h2 font-weight-bold mb-4">
          {{ title }}
        </h1>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
const props = defineProps({
  title: String,
});
</script>
