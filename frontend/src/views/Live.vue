<template>
    <v-container fluid class="d-flex flex-column align-center">
        <!-- Page Header -->
        <h2>Live Sensor Data</h2>

        <!-- Main Grid Layout -->
        <v-row class="d-flex justify-start" style="max-width: 1200px;">
            <!-- Left Column: Graphs -->
            <v-col cols="9">
                <!-- Temperature Graph -->
                <v-card>
                    <v-card-title>Temperature Analysis (Live)</v-card-title>
                    <v-card-text>
                        <figure class="highcharts-figure">
                            <div id="container"></div>
                        </figure>
                    </v-card-text>
                </v-card>

                <!-- Humidity Graph -->
                <v-card class="mt-5">
                    <v-card-title>Humidity Analysis (Live)</v-card-title>
                    <v-card-text>
                        <figure class="highcharts-figure">
                            <div id="container1"></div>
                        </figure>
                    </v-card-text>
                </v-card>
            </v-col>

            <!-- Right Column: Data Cards -->
            <v-col cols="3">
                <!-- Temperature Card -->
                <v-card class="mb-5" max-width="500" color="primaryContainer">
                    <v-card-item>
                        <v-card-title>Temperature</v-card-title>
                        <span class="text-h3 text-onPrimaryContainer">{{ temperature }}</span>
                    </v-card-item>
                </v-card>
                <!-- Heat Index Card -->
                <v-card class="mb-5" max-width="500" color="tertiaryContainer">
                    <v-card-item>
                        <v-card-title>Heat Index (Feels like)</v-card-title>
                        <span class="text-h3 text-onTertiaryContainer">{{ heatindex }}</span>
                    </v-card-item>
                </v-card>

                <!-- Humidity Card -->
                <v-card max-width="500" color="secondaryContainer">
                    <v-card-item>
                        <v-card-title>Humidity</v-card-title>
                        <span class="text-h3 text-onSecondaryContainer">{{ humidity }}</span>
                    </v-card-item>
                </v-card>
            </v-col>
        </v-row>
    </v-container>
</template>


<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import { useMqttStore } from "@/store/mqttStore"; 
import {storeToRefs } from "pinia";


// VARIABLES
const router      = useRouter();
const route       = useRoute();  
const mqttStore = useMqttStore();
const payload = ref({ temperature: 0, heatindex: 0, humidity: 0 });

// COMPUTED PROPERTIES
const temperature = computed(() => {
    return payload.value.temperature ? `${parseFloat(payload.value.temperature).toFixed(2)} °C` : "--";
});

const heatindex = computed(() => {
    return payload.value.heatindex ? `${parseFloat(payload.value.heatindex).toFixed(2)} °C` : "--";
});

const humidity = computed(() => {
    return payload.value.humidity ? `${parseFloat(payload.value.humidity).toFixed(2)} %` : "--";
});

// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    Mqtt.connect();
    setTimeout( ()=>{ Mqtt.subscribe("620164419")}, 3000);
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    Mqtt.unsubcribeAll();
});


</script>


<style scoped>
/** CSS STYLE HERE */


</style>
  