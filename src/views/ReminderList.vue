<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Erinnerungen</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>

      <!-- Reminder Modal -->
      <ReminderModal
        v-if="showModal"
        :reminder="currentReminder"
        :isEditing="isEditing"
        :showModal="showModal"
        :existingIds="existingIds"
        @save="handleSaveReminder"
        @dismiss="dismissModal"
      />
      
      <!-- Reminder List -->
      <ion-list>
        <ion-item v-for="reminder in reminders" :key="reminder.id">
          <ion-label>
            <h2>{{ reminder.text }}</h2>
            <p>{{ formatDateTime(reminder.date, reminder.time) }}</p>
          </ion-label>
          <ion-button @click="openEditReminderModal(reminder)" fill="outline" slot="end">Bearbeiten</ion-button>
          <ion-button @click="deleteReminder(reminder.id)" color="danger" slot="end">Löschen</ion-button>
        </ion-item>
      </ion-list>

      <!-- Create new Reminder -->
      <ion-fab vertical="bottom" horizontal="center" slot="fixed">
        <ion-fab-button @click="openAddReminderModal">
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
import ReminderModal from './ReminderModal.vue';

interface Reminder {
  id: number;
  text: string;
  date: string;
  time: string;
}

const reminders = ref<Reminder[]>([]);
const showModal = ref(false);
const currentReminder = ref<Reminder | undefined>(undefined);
const isEditing = ref(false);
const existingIds = ref<number[]>([]);

onMounted(() => {
  requestNotificationPermissons();
  loadReminders();
});

const loadReminders = async () => {
  try {
    const { value } = await Preferences.get({ key: 'reminders' });
    if (value) {
      reminders.value = JSON.parse(value);
      existingIds.value = reminders.value.map(reminder => reminder.id);
    } 
  } catch (error) {
    console.error('Fehler beim Laden der Erinnerungen: ', error);
  }
};

const saveReminders = async () => {
  try {
    await Preferences.set({ key: 'reminders', value: JSON.stringify(reminders.value) });
  } catch (error) {
    console.error('Fehler bei der Speicherung der Erinnerungen: ', error);
  }
};

const deleteReminder = async (id: number) => {
  reminders.value = reminders.value.filter(reminder => reminder.id !== id);
  existingIds.value = reminders.value.map(reminder => reminder.id);
  await saveReminders();
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

const scheduleNotification = async (reminder: Reminder) => {
  const { display } = await LocalNotifications.checkPermissions();
  if (display !== 'granted') {
    const alert = await alertController.create({
      header: 'Berechtigung erforderlich!',
      message: 'Um eine Benachrichtigung zu erhalten, muss die Berechtigung vergeben werden.',
      buttons: ['OK']
    });
    await alert.present();
    return;
  }

  try {
    //check following date
    const [year, month, day] = reminder.date.split('-').map(Number);
    const [hours, minutes] = reminder.time.split(':').map(Number);
    const reminderDate = new Date(year, month - 1, day, hours, minutes);
    
    //TestAlert
    const alert = await alertController.create({
      header: 'Test of Date',
      message: 'Date' + reminderDate,
      buttons: ['OK']
      });
      await alert.present();
    
    if (reminderDate < new Date()) {
      const alert = await alertController.create({
      header: 'Datum liegt in der Vergangenheit!',
      message: 'Es können nur Benachrichtigungen für die Zukunft erstellt werden.',
      buttons: ['OK']
      });
      await alert.present();
      return;
    }

    await LocalNotifications.schedule({
      notifications: [
        {
          title: reminder.text,
          body: `Erinnerung: ${reminder.text} ist fällig.`,
          id: reminder.id,
          schedule: { at: reminderDate },
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

const openAddReminderModal = () => {
  currentReminder.value = undefined;
  isEditing.value = false;
  showModal.value = true;
}

const openEditReminderModal = (reminder: Reminder) => {
  currentReminder.value = { ...reminder};
  isEditing.value = true;
  showModal.value = true;
}

const dismissModal = () => {
  showModal.value = false;
}

const formatDateTime = (date: string, time: string) => {
  return `${date} ${time}`;
};

const handleSaveReminder = (reminder: Reminder) => {
  if (isEditing.value) {
    const index = reminders.value.findIndex(r => r.id === reminder.id);
    if (index !== -1) {
      reminders.value[index] = reminder;
    }
  } else {
    reminders.value.push(reminder);
    existingIds.value.push(reminder.id);
    if (reminder.date && reminder.time) {
      scheduleNotification(reminder);
    } 
  } 
  saveReminders();
  dismissModal(); 
};

</script>

<style scoped>
</style>
