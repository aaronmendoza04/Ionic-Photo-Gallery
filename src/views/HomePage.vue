<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title> My Photo Gallery </ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <CameraComponent @photo-captured="addPhoto" />
      <PhotoGalleryComponent :photos="photos" />
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
} from "@ionic/vue";
import { ref, onMounted } from "vue";
import CameraComponent from "@/components/CameraComponent.vue";
import PhotoGalleryComponent from "@/components/PhotoGalleryComponent.vue";

// Firebase imports
import { db, storage } from "@/firebase";
import { ref as dbRef, push, onValue } from "firebase/database";
import { ref as storageRef, uploadBytes, getDownloadURL } from "firebase/storage";

const photos = ref<string[]>([]);

// 1. Fetch existing photos from Firebase when the app loads
onMounted(() => {
  const photosDatabaseRef = dbRef(db, 'gallery');
  
  onValue(photosDatabaseRef, (snapshot) => {
    const data = snapshot.val();
    if (data) {
      // Extract the URLs, put them in an array, and reverse it so newest is on top
      photos.value = Object.values(data)
        .map((item: any) => item.url)
        .reverse();
    } else {
      photos.value = [];
    }
  });
});

// 2. Handle taking a new photo and uploading it
const addPhoto = async (photoWebPath: string) => {
  try {
    // Convert the local webPath into a Blob (binary file) for Firebase
    const response = await fetch(photoWebPath);
    const blob = await response.blob();

    // Create a unique filename based on the current time
    const fileName = `photos/${Date.now()}.jpeg`;
    const sRef = storageRef(storage, fileName);

    // Upload the file to Firebase Storage
    await uploadBytes(sRef, blob);
    
    // Get the public download URL
    const downloadUrl = await getDownloadURL(sRef);

    // Save that URL into the Realtime Database
    const photosDatabaseRef = dbRef(db, 'gallery');
    await push(photosDatabaseRef, {
      url: downloadUrl,
      timestamp: Date.now()
    });
    
  } catch (error) {
    console.error("Error uploading photo:", error);
  }
};
</script>