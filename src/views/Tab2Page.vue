<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Post</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
     

       <ion-list>
       <ion-item v-for="post in usuariospost" :key="post.id">
          <ion-label>
            {{ post.title }}
            <small>{{ post.body }}</small>
          </ion-label>
        </ion-item>
        
      </ion-list>
     
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import type { Posts } from '@/interfaces/Posts';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem, IonLabel } from '@ionic/vue';
import { ref } from 'vue';

const usuariospost = ref<Posts[]>([]);

function cargarPost(){

  fetch('https://jsonplaceholder.typicode.com/posts')
  .then(response => response.json())
  .then(json => {
    console.log(json);
    usuariospost.value = json;

  })

}
cargarPost();

</script>
