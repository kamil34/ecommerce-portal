<template>
  <v-container class="fill-height" fluid>
      <v-row justify="center">
        <v-col cols="12" md="7">
          <h1 class="text-h2 font-weight-bold text-left mb-4">
            {{ title }}
          </h1>
          <v-row>
            <v-col v-for="item in products" cols="12">
              <v-card v-if="item.available" class="mx-auto" color="surface-variant" rounded="lg" variant="tonal">
                <v-img :src="item.image" height="150px" cover />
                <v-card-title>
                  {{ item.title }}
                </v-card-title>
                <v-card-subtitle>
                  <div class="text-wrap">
                    {{ item.description }}
                  </div>
                </v-card-subtitle>
                <v-card-actions>
                  <v-number-input max-width="15vh" :reverse="false" control-variant="stacked" :label="item.unit_symbol"
                    :hideInput="false" inset></v-number-input>
                  <v-spacer></v-spacer>
                  <v-btn prepend-icon="mdi-cart-plus" class="text-light-blue lighten-3" text="Dodaj do koszyka"></v-btn>
                </v-card-actions>
              </v-card>
              <v-card v-else="item.available" class="mx-auto" color="surface-variant" rounded="lg" variant="tonal"
                disabled>
                <v-img :src="item.image" height="150px" cover />
                <v-card-title>
                  Aktualnie niedostępny
                </v-card-title>
                <v-card-subtitle>
                  <div class="text-wrap">
                    {{ item.description }}
                  </div>
                </v-card-subtitle>
                <v-card-actions>
                  <v-number-input max-width="15vh" :reverse="false" control-variant="stacked" :label="item.unit_symbol"
                    :hideInput="false" inset></v-number-input>
                  <v-spacer></v-spacer>
                  <v-btn prepend-icon="mdi-cart-plus" class="text-light-blue lighten-3" text="Dodaj do koszyka"></v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { useLoadingStore } from '@/stores/loading';
import axios from 'axios';

const props = defineProps({
  title: {
    type: String,
    default: "Nasze produkty",
  }
});
const products = ref([]);
const loading = useLoadingStore()
const router = useRouter();

const getProducts = async () => {
  loading.start();
  try {
    const response = await axios.get('http://localhost:3001/endpoints/get_items')
    products.value = response.data['products'];
  } catch (error) {
    console.error('Błąd przy pobieraniu zamówień:', error);
    router.push('/error')
  } finally {
    loading.stop();
  };
};

onMounted(() => {
  getProducts();
});
</script>