<template>
  <v-container class="fill-height" fluid>
    <div>
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
                  <v-card-text class="text-green font-weight-bold">Szacowana data dostarczenia: dzisiaj</v-card-text>
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
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="(product, index) in item.orderedItems" :key="product.id">
                        <td>{{ index + 1 }}</td>
                        <td>{{ product.title }}</td>
                        <td>{{ product.unit }}</td>
                        <td>{{ product.quantity }}</td>
                        <td>90000 zł</td>
                      </tr>
                    </tbody>
                  </v-table>
                </v-card-item>
                <v-card-actions class="flex-column">
                  <v-row class="w-100" no-gutters justify="start">
                    <v-col cols="auto">
                      <v-card-text class="pt-2">
                        <div class="text-caption text-grey">Dodatkowa informacja pod zamówieniem</div>
                      </v-card-text>
                    </v-col>
                  </v-row>
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
                        class="text-grey-darken-4" text="Szczegóły zamówienia" @click="toggleDetails(item.orderID)"
                        :disabled="item.orderChanges.length === 0"></v-btn>
                    </v-col>
                    <v-row class="w-100" no-gutters justify="end">
                      <v-col cols="auto">
                        <v-btn prepend-icon="mdi-cart-outline" text="Anuluj zamówienie" disabled></v-btn>
                      </v-col>
                    </v-row>
                  </v-row>
                </v-card-actions>
                <v-expand-transition>
                  <div v-show="expandedOrderID === item.orderID">
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
    </div>
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue';
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

function toggleDetails(orderID) {
  expandedOrderID.value = expandedOrderID.value === orderID ? null : orderID;
};

onMounted(() => {
  getOrders();
  setTimeout(() => {
    progress.value = 60;
  }, 500);
});

// const order_changes = [
//   {
//     orderID: 228105847812,
//     employee: 'Jan Kowalski',
//     changes: [
//       { changeText: 'Wysłano 30m² blachy', date: '2025-03-10', }
//     ],
//   },
// ]
// const orders = [
//   {
//     orderID: 235105847812,
//     orderDate: '2025-02-10',
//     orderedItems: [
//       { id: 1, title: "Rura stalowa", unit: '(t)', quantity: 21 },
//       { id: 2, title: "Teownik", unit: '(mb)', quantity: 83 },
//       { id: 3, title: "Blacha", unit: '(m²)', quantity: 43 },
//     ],
//   },
//   {
//     orderID: 212105236812,
//     orderDate: '2025-02-12',
//     orderedItems: [
//       { id: 1, title: "Rura stalowa", unit: '(t)', quantity: 22 },
//       { id: 2, title: "Teownik", unit: '(mb)', quantity: 33 },
//     ],
//   },
//   {
//     orderID: 234108102812,
//     orderDate: '2025-02-18',
//     orderedItems: [
//       { id: 1, title: "Rura stalowa", unit: '(t)', quantity: 22 },
//       { id: 3, title: "Blacha", unit: '(m²)', quantity: 43 },
//     ],
//   },
// ]
</script>
