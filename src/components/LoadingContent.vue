<template>
    <v-overlay :model-value="isLoading" absolute>
        <div class="d-flex justify-center align-center" style="height: 100vh; width: 100vw;">
            <v-progress-circular color="primary" indeterminate size="100" width="10" />
        </div>
    </v-overlay>
</template>
<script setup>
import { storeToRefs } from 'pinia';
import { useLoadingStore } from '@/stores/loading';

const { isLoading } = storeToRefs(useLoadingStore());
</script>