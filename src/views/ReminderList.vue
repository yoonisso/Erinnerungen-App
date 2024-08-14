<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="gradient-toolbar">
        <ion-title>{{ getTitle }}</ion-title>
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
      <ion-list v-if="!showModal">
        <ion-item v-for="reminder in reminders" :key="reminder.id" class="reminder-item">
          <ion-label>
            <h2 class="reminder-text">{{ reminder.text }}</h2>
            <p class="reminder-datetime">{{ formatDateTime(formatDate(reminder.date), reminder.time) }}</p>
          </ion-label>
          <div class="button-container" slot="end">
            <ion-button @click="openEditReminderModal(reminder)" fill="outline" class="reminder-button">Bearbeiten</ion-button>
            <ion-button @click="deleteReminder(reminder.id)" color="danger" class="reminder-button">Löschen</ion-button>
          </div>
        </ion-item>
      </ion-list>

      <!-- Create new Reminder Button-->
      <ion-fab vertical="bottom" horizontal="center" slot="fixed" v-if="!showModal">
        <ion-fab-button @click="openAddReminderModal">
          <ion-icon :icon="add"></ion-icon>
        </ion-fab-button>
      </ion-fab>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted, computed} from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem, IonLabel, IonButton, IonFab, IonFabButton, IonIcon, alertController } from '@ionic/vue';
import { add } from 'ionicons/icons';
import { Preferences } from '@capacitor/preferences';
import { LocalNotifications } from '@capacitor/local-notifications';
import ReminderModal from './ReminderModal.vue';
import { formatDate } from '@/utils/dateUtils'; 

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

const getTitle = computed(() => {
  if (showModal.value) {
    return isEditing.value ? "Erinnerung bearbeiten" : "Erinnerung erstellen";
  }
  return "MindKeeper";
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
  await LocalNotifications.cancel({ notifications: [{ id }] });

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
    const [year, month, day] = reminder.date.split('-').map(Number);
    const [hours, minutes] = reminder.time.split(':').map(Number);
    const reminderDate = new Date(year, month - 1, day, hours, minutes);

    console.log('Reminder Date:', reminderDate);

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
          title: 'MindKeeper',
          body: `Erinnerung: ${reminder.text} ist fällig.`,
          id: reminder.id,
          schedule: { at: reminderDate },
          sound: 'beep.wav',
          attachments: undefined,
          actionTypeId: "",
          extra: null
        }
      ]
    });

    console.log('Notification scheduled:', reminder.id);

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
    LocalNotifications.cancel({ notifications: [{ id: reminder.id }] });

    const index = reminders.value.findIndex(r => r.id === reminder.id);
    if (index !== -1) {
      reminders.value[index] = reminder;
    }
  } else {
    reminders.value.push(reminder);
    existingIds.value.push(reminder.id);
  } 

  if (reminder.date && reminder.time) {
    scheduleNotification(reminder);
  }

  saveReminders();
  dismissModal(); 
};

</script>

<style scoped>
  .reminder-item {
    --padding-start: 16px;
    --padding-end: 16px;
    font-size: 18px;
    min-height: 70px;
    align-items: flex-start;
  }

  .reminder-text {
    font-size: 17px;
    font-weight: 600;
    margin-bottom: 4px;
  }

  .reminder-datetime {
    font-size: 13px;
    color: #6b6b6b;
  }

  .button-container {
    display: flex;
    gap: 1px;
  }

  .reminder-button {
    font-size: 16px;
    --padding-start: 12px;
    --padding-end: 12px;
    --padding-top: 8px;
    --padding-bottom: 8px;
    --border-radius: 6px;
  }

  ion-title {
    text-align: center;
    font-weight: 700;
  }

  .gradient-toolbar {
    --background: linear-gradient(135deg, #3A2FDF, #6146D9, #8B2FC0, #B52B9C, #D93875);
    color: white;
  }

  ion-fab-button {
    --background: linear-gradient(135deg, #3A2FDF, #6146D9, #8B2FC0, #B52B9C, #D93875);
  }

  ion-content {
    --background: linear-gradient(#FFFFFF, #ebebeb);
  }

</style>