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
                <v-card-text>
                  {{ item.completed ? 'Zamknięte' : 'Otwarte, w trakcie realizacji' }}
                </v-card-text>
                <v-card-text class="text-green font-weight-bold">
                  Proces realizacji zamówienia: {{ Math.floor(orderProgress(item)) === 100 ? 'Ukończone' :
                    Math.floor(orderProgress(item)) + '%' }}
                </v-card-text>
              </v-card-item>
              <v-card-item>
                <v-progress-linear :model-value="animatedProgressMap[item.orderID] || 0" color="green" height="10"
                  rounded></v-progress-linear>
              </v-card-item>
              <v-card-item>
                <v-table density="compact">
                  <thead>
                    <tr>
                      <th>Lp</th>
                      <th>Postęp</th>
                      <th>Produkt</th>
                      <th>Jednostka</th>
                      <th>Ilość</th>
                      <th>Suma</th>
                      <th>Ilość partii (domyślnie 1)</th>
                      <th></th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(product, index) in item.orderedItems" :key="product.id">
                      <td>{{ index + 1 }}</td>
                      <td>
                        Potwierdzono: {{ product.confirmed_quantity || 0 }} /
                        {{ product.quantity }} <br>
                        Wysłano, ale niepotwierdzone: {{ product.unconfirmed_quantity || 0 }}
                      </td>
                      <td>{{ product.title }}</td>
                      <td>{{ product.unit }}</td>
                      <td>{{ product.quantity }}</td>
                      <td>90000 zł</td>
                      <td>
                        <v-row align="center" justify="start" no-gutters>
                          <v-col class="my-1" cols="auto" md="auto">
                            <v-text-field :model-value="shipmentInputs[item.orderID]?.[product.id] || null"
                              @update:model-value="
                                val => {
                                  if (!shipmentInputs[item.orderID]) shipmentInputs[item.orderID] = {};
                                  shipmentInputs[item.orderID][product.id] = val;
                                }" @focus="() => {
                                  expandedOrderID = item.orderID;
                                  textFieldOrderID = item.orderID;
                                }" label="Do wysyłki" type="number" density="comfortable" hide-details
                              :max="remainingToShip(item.orderID, product)" />
                          </v-col>
                        </v-row>
                      </td>
                      <td>
                        <v-btn class="ml-md-4" color="primary" @click="() => {
                          textFieldOrderID = 0;
                          sendShipment(item.orderID, product, 'Jan Kowalski');
                        }
                        " :disabled="!canShip(item.orderID, product)" hide-details>Wyślij </v-btn>
                      </td>
                    </tr>
                  </tbody>
                </v-table>
              </v-card-item>
              <v-card-item>
                <v-row class="flex-wrap justify-center pa-0 ma-0">
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
                      class="text-grey-darken-4" @click="() => {
                        toggleDetails(item.orderID)
                        if (expandedOrderID !== 0) { textFieldOrderID = 0; };
                      }" :disabled="item.orderChanges.length === 0">Szczegóły zamówienia</v-btn>
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
                  <v-card-text v-if="textFieldOrderID === item.orderID">
                    <v-col cols="12" md="5" class="d-flex align-center">
                      <v-text-field density="comfortable" label="Wprowadź zmiany dla wybranej partii" variant="solo"
                        v-model="changeText" hide-details single-line class="flex-grow-1" />
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
//
const props = defineProps({
  title: {
    type: String,
    default: "Panel dostawcy",
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
    const response = await axios.get('http://localhost:3001/endpoints/get_orders')
    ordersData.value = response.data['orders'];
    // console.log(ordersData.value)
    syncAnimatedProgress();
  } catch (error) {
    console.error('Błąd przy pobieraniu zamówień:', error);
    router.push('/error');
  } finally {
    loading.stop();
    // console.log(ordersData.value);
  };
};

const shipmentInputs = ref({});

const sendShipment = async (orderID, product, changeUser) => {
  const qty = shipmentInputs.value[orderID]?.[product.id] || 0;

  if (qty <= 0) return;

  try {
    await axios.post('http://localhost:3001/endpoints/report_shipment', {
      orderID,
      productID: product.id,
      quantity: qty,
      changeUser: changeUser,
      changeText: changeText.value,
    });
    await getOrders();
    shipmentInputs.value[orderID][product.id] = 0;
  } catch (error) {
    // console.log(changeUser, changeText.value);
    console.error('Błąd przy wysyłaniu partii:', error);
    router.push('/error')
  };
};

const remainingToShip = (orderID, product) => {
  const ordered = product.quantity;
  const confirmed = product.confirmed_quantity || 0;
  const unconfirmed = product.unconfirmed_quantity || 0;
  return ordered - confirmed - unconfirmed;
};

const canShip = (orderID, product) => {
  const qty = shipmentInputs.value[orderID]?.[product.id] || 0;
  return qty > 0 && qty <= remainingToShip(orderID, product);
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
  }, 350);
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
