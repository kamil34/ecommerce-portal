<template>
  <v-container class="fill-height" fluid>
    <v-row justify="center">
      <v-col cols="12" md="9">
        <h1 class="text-h2 font-weight-bold text-left mb-4">
          {{ title }}
        </h1>
        <v-row>
          <v-col v-for="item in ordersData" :key="item.orderID" cols="12">
            <v-card class="mx-auto" color="surface-variant" rounded="lg" variant="tonal">
              <v-card-item>
                <v-card-title>Status zamówienia:</v-card-title>
                <v-card-text>Otwarte, w trakcie realizacji</v-card-text>
                <v-card-text class="text-green font-weight-bold">Proces wykończenia zamówienia: {{ progress
                }}%</v-card-text>
              </v-card-item>
              <v-card-item>
                <v-progress-linear color="green" rounded :model-value="progress" :height="10"></v-progress-linear>
              </v-card-item>
              <v-card-item>
                <v-table density="compact">
                  <thead>
                    <tr>
                      <th class="text-left">
                        Lp
                      </th>
                      <th class="text-left">
                        Produkt
                      </th>
                      <th class="text-left">
                        Jednostka
                      </th>
                      <th class="text-left">
                        Ilość
                      </th>
                      <th class="text-left">
                        Suma
                      </th>
                      <th class="text-left">
                        Ilość partii (domyślnie 1)
                      </th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(product, index) in item.orderedItems" :key="product.id">
                      <td>{{ index + 1 }}</td>
                      <td>{{ product.title }}</td>
                      <td>{{ product.unit }}</td>
                      <td>{{ product.quantity }}</td>
                      <td>90000 zł</td>
                      <td>
                        <!-- todo -->
                        <v-row align="center" no-gutters>
                          <v-col cols="12" md="3">
                            <v-select :items="shipmentsNum" v-model="selectedShipmentsNum" item-title="text"
                              item-value="value" variant="solo-filled"></v-select>
                          </v-col>
                          <v-col cols="12" md="3">
                            <v-checkbox v-for="num in selectedShipmentsNum" :key="num" density="compact"
                              v-model="selectedCheckboxesArr" :label="`Partia ${num}`" :value="num"
                              hide-details></v-checkbox>
                          </v-col>
                        </v-row>
                      </td>
                    </tr>
                  </tbody>
                </v-table>
              </v-card-item>
              <v-card-item>
                <v-row class="flex-wrap justify-center pa-0 ma-0">
                  <v-btn prepend-icon="mdi-check-outline" variant="outlined">Oznacz jako zrealizowane</v-btn>
                </v-row>
              </v-card-item>
              <v-card-actions class="flex-column">
                <v-row class="w-100 mb-2" no-gutters align="center" justify="space-between">
                  <v-col cols="auto">
                    <v-card-text>
                      <div class="font-weight-bold">Zamówienie nr</div> {{ item.orderID }}
                    </v-card-text>
                  </v-col>
                  <v-col cols="auto" class="mx-4">
                    <v-card-text>
                      <div class="font-weight-bold">Data zamówienia</div> {{ item.orderDate }}
                    </v-card-text>
                  </v-col>
                  <v-col cols="auto" class="ml-auto">
                    <v-btn :prepend-icon="expandedOrderID === item.orderID ? 'mdi-chevron-down' : 'mdi-cart-outline'"
                      class="text-grey-darken-4" @click="toggleDetails(item.orderID)"
                      :disabled="item.orderChanges.length === 0">Szczegóły zamówienia</v-btn>
                  </v-col>
                  <v-row class="w-100" no-gutters justify="end">
                    <v-col cols="auto">
                      <v-btn prepend-icon="mdi-cart-outline" disabled>Anuluj zamówienie</v-btn>
                    </v-col>
                  </v-row>
                </v-row>
              </v-card-actions>
              <v-expand-transition>
                <div v-show="expandedOrderID === item.orderID">
                  <v-card-text>
                    <v-col cols="12" md="5" class="d-flex align-center">
                      <v-text-field density="comfortable" label="Wprowadź zmiany dla wybranej partii" variant="solo"
                        hide-details single-line class="flex-grow-1"></v-text-field>
                      <v-btn class="ml-4" aria-label="Refresh" icon="mdi-check"></v-btn>
                    </v-col>
                  </v-card-text>
                  <v-divider></v-divider>
                  <v-timeline align="start" density="compact">
                    <v-timeline-item v-for="(update, index) in item.orderChanges" :key="`${item.orderID}-${index}`"
                      dot-color="red" size="x-small">
                      <div class="mb-4">
                        <div class="font-weight-normal">
                          {{ update.changeText }}
                        </div>
                        <div class="font-weight-normal">
                          <strong>Jan Kowalski (To be updated)</strong> @{{ update.changeDate }}
                        </div>
                      </div>
                    </v-timeline-item>
                  </v-timeline>
                </div>
              </v-expand-transition>
            </v-card>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { useLoadingStore } from '@/stores/loading';
import axios from 'axios';

const progress = ref(0);
const props = defineProps({
  title: String,
});
const ordersData = ref([]);
const loading = useLoadingStore();
const router = useRouter();
const expandedOrderID = ref(null);
const shipmentsNum = Array.from({ length: 5 }, (_, i) => ({
  text: i === 0 ? null : (i + 1).toString(),
  value: i === 0 ? null : i + 1
}));
const selectedShipmentsNum = ref(null)
const selectedCheckboxesArr = ref([])
const toggleDetails = (orderID) => {
  expandedOrderID.value = expandedOrderID.value === orderID ? null : orderID;
};
const getOrders = async () => {
  loading.start();
  try {
    const response = await axios.get('http://172.16.0.10:3001/endpoints/get_orders')
    ordersData.value = response.data['orders'];
  } catch (error) {
    console.error('Błąd przy pobieraniu zamówień:', error);
    router.push('/error');
  } finally {
    loading.stop();
    // console.log(ordersData.value);
  };
};

watch(selectedShipmentsNum, (newVal) => {
  console.log(selectedCheckboxesArr.value);
  selectedCheckboxesArr.value = selectedCheckboxesArr.value.filter(item => item <= newVal);
});

watch(selectedCheckboxesArr, (shipsCalculate) => {
  setTimeout(() => {
    const max = selectedShipmentsNum.value;
    if (!max) {
      progress.value = 0;
      return;
    }
    const count = shipsCalculate.length > max ? max : shipsCalculate.length;
    progress.value = Math.round((count / max) * 100);
  }, 250);
}, { deep: true });

onMounted(() => {
  getOrders();
  setTimeout(() => {
    progress.value = selectedCheckboxesArr.value * 20;
  }, 500);
});
</script>
