<template>
    <v-container fluid background-color="surface" align="center">

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
            <v-col cols="3">
                <v-card class="mb-5" max-width="500px" color="primaryContainer">
                    <v-card-item>
                        <v-card-title>Temperature</v-card-title>
                        <span class="text-h3 text-onPrimaryContainer">{{ temperature }}</span>
                    </v-card-item>
                </v-card>
                <!-- Heat Index Card -->
                <v-card class="mb-5" max-width="500px" color="tertiaryContainer">
                    <v-card-item>
                        <v-card-title>Heat Index (Feels like)</v-card-title>
                        <span class="text-h3 text-onTertiaryContainer">{{ heatindex }}</span>
                    </v-card-item>
                </v-card>

                <!-- Humidity Card -->
                <v-card max-width="500px" color="secondaryContainer">
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
import Highcharts from 'highcharts'; 
import more from 'highcharts/highcharts-more'; 
import Exporting from 'highcharts/modules/exporting'; 
Exporting(Highcharts);  
more(Highcharts); 



// VARIABLES
const router      = useRouter();
const route       = useRoute();  
const Mqtt = useMqttStore();
const {payload, payloadTopic } = storeToRefs(Mqtt);
const points  = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously. 
const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph. 
//const payload = ref({ temperature: 0, heatindex: 0, humidity: 0 });
const tempHiChart = ref(null); // Chart object
const HumidChart = ref(null); // Chart object

const CreateCharts = async () => { 
// TEMPERATURE CHART 
tempHiChart.value = Highcharts.chart('container', { 
chart: { zoomType: 'x' }, 
title: { text: 'Air Temperature and Heat Index Analysis', align: 'left' }, 
yAxis: {  
title: { text: 'Air Temperature & Heat Index' , style:{color:'#000000'}}, 
labels: { format: '{value} °C' }         
}, 
xAxis: { 
type: 'datetime',  
title: { text: 'Time', style:{color:'#000000'} }, 
}, 
tooltip: { shared:true, }, 
series: [ 
{ 
name: 'Temperature', 
type: 'spline', 
data: [], 
turboThreshold: 0, 
color: Highcharts.getOptions().colors[0] 
},  
{ 
name: 'Heat Index', 
type: 'spline', 
data: [], 
turboThreshold: 0, 
color: Highcharts.getOptions().colors[1] 
} ], 
}); 
 

HumidChart.value = Highcharts.chart('container1', { 
chart: { zoomType: 'x' }, 
title: { text: 'Humidity Analysis', align: 'left' }, 
yAxis: {  
title: { text: 'Humidity' , style:{color:'#2F3301'}}, 
labels: { format: '{value} °C' }         
}, 
xAxis: { 
type: 'datetime',  
title: { text: 'Time', style:{color:'#000000'} }, 
}, 
tooltip: { shared:true, }, 
series: [ 
{ 
name: 'Humidity', 
type: 'spline', 
data: [], 
turboThreshold: 0, 
color: Highcharts.getOptions().colors[0] 
}, ], 
}); 
}; 

// COMPUTED PROPERTIES
const temperature = computed(()=>{
if(!!payload.value){
return `${payload.value.temperature.toFixed(2)} °C`;
}
});

const heatindex = computed(()=>{
if(!!payload.value){
return `${payload.value.heatindex.toFixed(2)} °C`;
}
});

const humidity = computed(()=>{
if(!!payload.value){
return `${payload.value.humidity.toFixed(2)} °C`;
}
});

// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    CreateCharts();
    Mqtt.connect();
    setTimeout( ()=>{ Mqtt.subscribe("620164419")}, 3000);
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    Mqtt.unsubcribeAll();
});

// WATCHERS 
watch(payload,(data)=> {   
if(points.value > 0){  points.value -- ; } 
else{ shift.value = true; }   
tempHiChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 }, 
true,  shift.value);  
tempHiChart.value.series[1].addPoint({y:parseFloat(data.heatindex.toFixed(2))
   ,x: data.timestamp * 1000 }, true,  shift.value);
HumidChart.value.series[0].addPoint({y:parseFloat(data.humidity.toFixed(2)) ,x: data.timestamp * 1000 }, 
true,  shift.value);  
});




</script>


<style scoped>
/** CSS STYLE HERE */
figure{
    margin: 2px;
}

</style>
  