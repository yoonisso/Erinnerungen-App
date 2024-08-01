<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Erinnerungen</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Erinnerungen-App</ion-title>
        </ion-toolbar>
      </ion-header>
    
      <ion-list id="reminders">
        <ion-item id="reminder_idTODO">
          Erinnerung 1
        </ion-item>
        <ion-item>
          Erinnerung 2
        </ion-item>
        <ion-item>
          Erinnerung 3
        </ion-item>
      </ion-list>

      <div id="reminders">
        <h1><strong>Hier stehen die Erinnerungen</strong></h1>
        <div id="reminder">
          <h2>Hier steht eine Erinnerung</h2>
          <h2>Hier steht eine Erinnerung</h2>
          <h2>Hier steht eine Erinnerung</h2>
          <h2>Hier steht eine Erinnerung</h2>
          <h2>Hier steht eine Erinnerung</h2>
          <h2>Hier steht eine Erinnerung</h2>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonList} from '@ionic/vue';
</script>

<style scoped>
#container {
  text-align: center;
  
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
