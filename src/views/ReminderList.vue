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
            <p>{{ formatDateTime(reminder.date, reminder.time) }}</p>
          </ion-label>
          <ion-button @click="editReminder()" fill="outline" slot="end">Bearbeiten</ion-button>
          <ion-button @click="deleteReminder(reminder.id)" color="danger" slot="end">Löschen</ion-button>
          <ion-button @click="scheduleNotification(reminder.id)" fill="outline" slot="end">SheduleTestButton</ion-button>
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
import { ref, onMounted } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem, IonLabel, IonButton, IonFab, IonFabButton, IonIcon, alertController } from '@ionic/vue';
import { add } from 'ionicons/icons';
import { Preferences } from '@capacitor/preferences';
import { LocalNotifications } from '@capacitor/local-notifications';

const reminders = ref([
  // Beispiel-Daten
  { id: 1, text: 'Erinnerung 1', date: '2024-08-05', time: '10:00' },
  { id: 2, text: 'Erinnerung 2', date: '2024-08-06', time: '14:00' }
]);

const scheduleNotification = async (id: number) => {
  const { display } = await LocalNotifications.checkPermissions();
  if (display !== 'granted') {
    const alert = await alertController.create({
      header: 'Berechtigung erforderlich!',
      message: 'Mitteilung kann nicht erstellt werden, da die Berechtigung fehlt.',
      buttons: ['OK']
    });
    await alert.present();
    return;
  }

  try {
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
  } catch (error) {
    console.error('Fehler bei der Planung der Benachrichtigung: ' , error);
  }
};

const deleteReminder = (id: number) => {
  reminders.value = reminders.value.filter(reminder => reminder.id !== id);
  saveReminders();
};

const addReminder = () => {
  const newId = reminders.value.length ? Math.max(...reminders.value.map(r => r.id)) + 1 : 1;
  reminders.value.push({ id: newId, text: `Erinnerung ${newId}`, date: '2024-08-07', time: '12:00' });
  saveReminders();
};

const editReminder = () => {
  //Modal view open
};

const formatDateTime = (date: string, time: string) => {
  return `${date} ${time}`;
};

const loadReminders = async () => {
  const { value } = await Preferences.get({ key: 'reminders' });
  if (value) {
    reminders.value = JSON.parse(value);
  }
};

const saveReminders = async () => {
  await Preferences.set({ key: 'reminders', value: JSON.stringify(reminders.value) });
};

const requestNotificationPermissons = async () => {
  try {
    const { display } = await LocalNotifications.checkPermissions();
    if (display !== 'granted') {
      await LocalNotifications.requestPermissions();
    }
  } catch (error) {
    console.error('Fehler beim Anfordern der Berechtigungen: ', error);
  }
}

onMounted(() => {
  requestNotificationPermissons();
  loadReminders();
});

</script>

<style scoped>
</style>
