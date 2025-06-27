<template>
  <v-container class="fill-height" fluid>
    <v-row justify="center">
      <v-col cols="12" md="9">
        <h1 class="text-h2 font-weight-bold text-left mb-4">
          {{ title }}
        </h1>
        <v-row>
          <v-col v-for="order in ordersData" :key="order.orderID" cols="12">
            <v-card class="mx-auto" color="surface-variant" rounded="lg" variant="tonal">
              <v-card-item>
                <v-card-title>Status zamówienia:</v-card-title>
                <v-card-text>
                  {{ order.completed ? 'Zamknięte' : 'Otwarte, w trakcie realizacji' }}
                </v-card-text>
                <v-card-text class="text-blue font-weight-bold">
                  Proces realizacji zamówienia: {{ Math.floor(orderProgress(order)) === 100 ? 'Ukończone' :
                    Math.floor(orderProgress(order)) + '%' }}
                </v-card-text>
              </v-card-item>
              <v-card-item>
                <v-progress-linear :model-value="animatedProgressMap[order.orderID] || 0" color="blue" height="10"
                  rounded></v-progress-linear>
              </v-card-item>
              <v-card-item>
                <v-table density="compact">
                  <thead>
                    <tr>
                      <th>Lp</th>
                      <th>Produkt</th>
                      <th>Jednostka</th>
                      <th>Dostarczono (niepotwierdzone)</th>
                      <th>Potwierdzona ilość</th>
                      <th>Suma</th>
                      <th>Do potwierdzenia</th>
                      <th></th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(product, index) in order.orderedItems" :key="product.id">
                      <td>{{ index + 1 }}</td>
                      <td>{{ product.title }}</td>
                      <td>{{ product.unit }}</td>
                      <td>{{ product.unconfirmed_quantity || 0 }}</td>
                      <td>{{ product.confirmed_quantity || 0 }}</td>
                      <td>90000 zł</td>
                      <td>
                        <v-row align="center" justify="start" no-gutters>
                          <v-col class="my-1" cols="auto" md="auto">
                            <v-text-field :model-value="confirmInputs[order.orderID]?.[product.id] || null"
                              @update:model-value="
                                val => {
                                  if (!confirmInputs[order.orderID]) confirmInputs[order.orderID] = {};
                                  confirmInputs[order.orderID][product.id] = val;
                                }" @focus="() => {
                                  expandedOrderID = order.orderID;
                                  textFieldOrderID = order.orderID;
                                }" type="number" density="comfortable" hide-details
                              :max="product.unconfirmed_quantity || 0" label="Ilość" />
                          </v-col>
                        </v-row>
                      </td>
                      <td>
                        <v-btn v-if="product.shipments.some(s => !s.confirmed)" type="button" @click="() => {
                          textFieldOrderID = 0;
                          confirmShipment(order.orderID, product, product.shipments.find(s => !s.confirmed).shipmentID, 'Alicja Kowalska');
                        }" :disabled="!canConfirm(order.orderID, product)" color="primary">
                          Potwierdź
                        </v-btn>
                      </td>
                    </tr>
                  </tbody>
                </v-table>
              </v-card-item>
              <v-card-actions class="flex-column">
                <v-row class="w-100 mb-2" no-gutters align="center" justify="space-between">
                  <v-col cols="auto">
                    <v-card-text>
                      <div class="font-weight-bold">Zamówienie nr</div> {{ order.orderID }}
                    </v-card-text>
                  </v-col>
                  <v-col cols="auto" class="mx-4">
                    <v-card-text>
                      <div class="font-weight-bold">Data zamówienia</div> {{ order.orderDate }}
                    </v-card-text>
                  </v-col>
                  <v-col cols="auto" class="ml-auto">
                    <v-btn :prepend-icon="expandedOrderID === order.orderID ? 'mdi-chevron-down' : 'mdi-cart-outline'"
                      class="text-grey-darken-4" @click="() => {
                        toggleDetails(order.orderID)
                        if (expandedOrderID !== 0) { textFieldOrderID = 0; };
                      }" :disabled="order.orderChanges.length === 0">Szczegóły zamówienia</v-btn>
                  </v-col>
                  <v-row class="w-100" no-gutters justify="end">
                    <v-col cols="auto">
                      <v-btn prepend-icon="mdi-cart-outline" disabled>Anuluj zamówienie</v-btn>
                    </v-col>
                  </v-row>
                </v-row>
              </v-card-actions>
              <v-expand-transition>
                <div v-show="expandedOrderID === order.orderID">
                  <v-card-text v-if="textFieldOrderID === order.orderID">
                    <v-col cols="12" md="5" class="d-flex align-center">
                      <v-text-field density="comfortable" label="Wprowadź zmiany dla wybranej partii" variant="solo"
                        v-model="changeText" hide-details single-line class="flex-grow-1" />
                    </v-col>
                  </v-card-text>
                  <v-divider></v-divider>
                  <v-timeline align="start" density="compact">
                    <v-timeline-item v-for="(update, index) in order.orderChanges" :key="`${order.orderID}-${index}`"
                      dot-color="green" size="x-small">
                      <div class="mb-4">
                        <div class="font-weight-normal">
                          {{ update.changeText }}
                        </div>
                        <div class="font-weight-normal">
                          <strong>{{ update.changeUser }}</strong> @{{ update.changeDate }}
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
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { useLoadingStore } from '@/stores/loading';
import axios from 'axios';

const props = defineProps({
  title: {
    type: String,
    default: "Twoje zamówienia",
  }
});
const ordersData = ref([]);
const loading = useLoadingStore();
const router = useRouter();
const expandedOrderID = ref(null);
const textFieldOrderID = ref(null);
const animatedProgressMap = ref({});
const changeText = ref('');
const toggleDetails = (orderID) => {
  expandedOrderID.value = expandedOrderID.value === orderID ? null : orderID;
};

const getOrders = async () => {
  loading.start();
  try {
    const response = await axios.get('http://localhost:3001/endpoints/get_orders');
    ordersData.value = response.data['orders'];
    // console.log(ordersData.value);
    syncAnimatedProgress();
  } catch (error) {
    console.error('Błąd przy pobieraniu zamówień:', error);
    router.push('/error');
  } finally {
    loading.stop();
  }
};

const confirmInputs = ref({});

const confirmShipment = async (orderID, product, shipmentID, changeUser) => {
  const qty = Number(confirmInputs.value[orderID]?.[product.id] || 0);

  if (qty <= 0) return;
  if (!shipmentID) {
    router.push('/error');
    console.error("Brakuje shipmentID dla produktu", product);
    return;
  };

  try {
    await axios.post('http://localhost:3001/endpoints/confirm_shipment', {
      orderID,
      shipmentID: shipmentID,
      productID: product.id,
      quantity: qty,
      changeUser: changeUser,
      changeText: changeText.value,
    });
    await getOrders();
    // console.log({
    //   orderID,
    //   shipmentID: shipmentID,
    //   productID: product.id,
    //   quantity: qty,
    //   changeUser: changeUser,
    //   changeText: changeText.value,
    // })
    confirmInputs.value[orderID][product.id] = 0;
    // console.log(changeText.value);
  } catch (error) {
    console.error('Błąd podczas potwierdzania dostawy:', error);
    router.push('/error');
  };
};

const canConfirm = (orderID, product) => {
  const qty = Number(confirmInputs.value[orderID]?.[product.id] || 0);
  return qty > 0 && qty <= (product.unconfirmed_quantity || 0);
};

const syncAnimatedProgress = () => {
  ordersData.value.forEach(order => {
    animatedProgressMap.value[order.orderID] = 0;
  });

  setTimeout(() => {
    ordersData.value.forEach(order => {
      const progress = orderProgress(order);
      animatedProgressMap.value[order.orderID] = progress;
    });
  }, 300);
};

const orderProgress = (order) => {
  let totalQty = 0;
  let confirmedQty = 0;
  let shippedQty = 0;

  order.orderedItems.forEach(product => {
    const quantity = product.quantity || 0;
    const confirmed = product.confirmed_quantity || 0;
    const unconfirmed = product.unconfirmed_quantity || 0;

    totalQty += quantity;
    confirmedQty += confirmed;
    shippedQty += confirmed + unconfirmed;
  });

  const customerProgress = confirmedQty / totalQty;
  const supplierProgress = shippedQty / totalQty;

  return ((customerProgress + supplierProgress) / 2) * 100;
};

onMounted(() => {
  getOrders();
});
</script>
