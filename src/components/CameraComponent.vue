<template>
  <ion-card>
    <ion-card-header>
      <ion-card-title>Camera</ion-card-title>
    </ion-card-header>
    <ion-card-content>
      <ion-button expand="block" @click="takePicture">
        <ion-icon slot="start" :icon="cameraIcon" /> Take Picture
      </ion-button>
      <ion-text v-if="errorMessage" color="danger">
        <p>{{ errorMessage }}</p>
      </ion-text>
    </ion-card-content>
  </ion-card>
</template>

<script setup lang="ts">
import {
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonButton,
  IonIcon,
  IonText
} from "@ionic/vue";
import { camera as cameraIcon } from "ionicons/icons";
import { Camera, CameraResultType } from "@capacitor/camera";
import { ref } from "vue";

const errorMessage = ref("");

const emit = defineEmits<{ (event: "photoCaptured", photo: string): void }>();

const takePicture = async () => {
  errorMessage.value = "";
  try {
    const photo = await Camera.getPhoto({
      quality: 90,
      allowEditing: false,
      resultType: CameraResultType.Uri,
      saveToGallery: false
    });
    
    if (photo.webPath) {
      emit("photoCaptured", photo.webPath);
    }
  } catch (error: any) {
    console.error(error);
    errorMessage.value = error.message || "Failed to capture image";
  }
};
</script>