<template>
    <v-container class="mx-auto" max-width="100vw" fluid>
        <div>
            <v-row justify="center">
                <v-col cols="12" md="12">
                    <h1 class="text-h2 font-weight-bold text-center mt-10">
                        {{ title }}
                    </h1>
                </v-col>
            </v-row>
        </div>
    </v-container>
</template>
<script setup>
const props = defineProps({
    title: String,
    required: true,
});
</script>