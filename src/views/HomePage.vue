<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonInput, IonButton, IonCard, IonCardHeader, IonCardTitle, IonCardContent } from '@ionic/vue';
import axios from 'axios';
import { Geolocation } from '@capacitor/geolocation';
import { ref } from 'vue';

const latitud = ref<number | null>(null);
const longitud = ref<number | null>(null);
const location = ref<string>('');
const weatherData = ref<any>(null);

const apiKey = '6QA8GJZ8C3ANC9MYZJ89HCYFY';

const viewUbicationByCountry = async () => {
  if (!location.value) return;
  const url = `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${location.value}?key=${apiKey}`;
  
  try {
    const response = await axios.get(url);
    weatherData.value = response.data;
  } catch (error) {
    console.error('Error fetching weather data:', error);
  }
};

const viewUbicationByCoordinates = async () => {
  getLocation();
  if (latitud.value === null || longitud.value === null) return;
  const url = `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${latitud.value},${longitud.value}?unitGroup=metric&key=${apiKey}`;
  
  try {
    const response = await axios.get(url);
    weatherData.value = response.data;
  } catch (error) {
    console.error('Error fetching weather data by coordinates:', error);
  }
};

const getLocation = async () => {
  try {
  
  
    const position = await Geolocation.getCurrentPosition();
    latitud.value = position.coords.latitude;
    longitud.value = position.coords.longitude;
    
  } catch (error) {
    console.error('Error getting location:', error);
  }
};
</script>

<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Aplicación para ver el clima</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content class="ion-padding">
      <ion-input 
        v-model="location" 
        label="Ubicación" 
        placeholder="Ingrese el nombre del país">
      </ion-input>
      <ion-button expand="block" @click="viewUbicationByCountry">Buscar por País</ion-button>
      <ion-button expand="block" @click="viewUbicationByCoordinates">Clima en  la ubicación actual </ion-button>

      <ion-card v-if="weatherData">
        <ion-card-header>
          <ion-card-title>Clima en {{ weatherData.resolvedAddress }}</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <p><strong>Temperatura:</strong> {{ weatherData.currentConditions.temp }}°C</p>
          <p><strong>Velocidad del viento:</strong> {{ weatherData.currentConditions.windspeed }} km/h</p>
          <p><strong>Probabilidad de lluvia:</strong> {{ weatherData.currentConditions.precipprob }}%</p>
          <p><strong>Clima general:</strong> {{ weatherData.currentConditions.conditions }}</p>
          <p><strong>Períodos anteriores y futuros:</strong></p>
          <ul>
            <li v-for="(day, index) in weatherData.days" :key="index">
              {{ day.datetime }} - {{ day.conditions }} - {{ day.temp }}°f
            </li>
          </ul>
        </ion-card-content>
      </ion-card>
    </ion-content>
  </ion-page>
</template>