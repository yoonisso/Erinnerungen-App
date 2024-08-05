<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Erinnerungen</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- Reminder List -->
      <ion-list>
        <ion-item v-for="reminder in reminders" :key="reminder.id">
          <ion-label>
            <h2>{{ reminder.text }}</h2>
            <p>{{ reminder.date }} {{ reminder.time }}</p>
          </ion-label>
          <ion-button @click="scheduleNotification(reminder.id)" fill="outline" slot="end">Bearbeiten</ion-button>
          <ion-button @click="deleteReminder(reminder.id)" color="danger" slot="end">Löschen</ion-button>
        </ion-item>
      </ion-list>

      <!-- Create new Reminder -->
      <ion-fab vertical="bottom" horizontal="center" slot="fixed">
        <ion-fab-button @click="addReminder">
          <ion-icon :icon="add"></ion-icon>
        </ion-fab-button>
      </ion-fab>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem, IonLabel, IonButton, IonFab, IonFabButton, IonIcon } from '@ionic/vue';
import { add } from 'ionicons/icons';
import { Preferences } from '@capacitor/preferences';
import { LocalNotifications } from '@capacitor/local-notifications';

const reminders = ref([
  // Beispiel-Daten
  { id: 1, text: 'Erinnerung 1', date: '2024-08-05', time: '10:00' },
  { id: 2, text: 'Erinnerung 2', date: '2024-08-06', time: '14:00' }
]);

const scheduleNotification = async (id: number) => {
  await LocalNotifications.requestPermissions();
  await LocalNotifications.schedule({
    notifications: [
      {
        title: "Title",
        body: "Body",
        id,
        schedule: { at: new Date(Date.now() + 1000 * 5) }, // In 5 Sekunden
        sound: undefined,
        attachments: undefined,
        actionTypeId: "",
        extra: null
      }
    ]
  });
};

const deleteReminder = (id: number) => {
  reminders.value = reminders.value.filter(reminder => reminder.id !== id);
};

const addReminder = () => {
  const newId = reminders.value.length ? Math.max(...reminders.value.map(r => r.id)) + 1 : 1;
  reminders.value.push({ id: newId, text: `Erinnerung ${newId}`, date: '2024-08-07', time: '12:00' });
};
</script>

<style scoped>
</style>
