<template>
  <ion-modal ref="modal" :is-open="showModal" @ionModalDidDismiss="dismissModal" class="custom-modal">
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="end">
          <ion-button @click="dismissModal" color="primary">Schließen</ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <!-- Erinnerungstext -->
      <ion-item lines="full" class="spaced-item">
        <ion-label position="stacked" class="label-large">Erinnerung</ion-label>
        <ion-input v-model="reminder.text" placeholder="Erinnerungstext eingeben" clear-input></ion-input>
      </ion-item>

      <!-- Datum -->
      <ion-item lines="none" class="spaced-item" detail-none>
        <ion-label position="stacked" class="label-large">Datum</ion-label>
        <div class="datetime-container">
          <ion-button expand="block" @click="selectDate" color="light">Datum auswählen</ion-button>
          <ion-label class="datetime-label">{{ formattedDate ? formatDate(formattedDate) : 'Kein Datum ausgewählt' }}</ion-label>
        </div>
      </ion-item>

      <!-- Uhrzeit -->
      <ion-item lines="none" class="spaced-item" detail-none>
        <ion-label position="stacked" class="label-large">Uhrzeit</ion-label>
        <div class="datetime-container">
          <ion-button expand="block" @click="selectTime" color="light">Uhrzeit auswählen</ion-button>
          <ion-label class="datetime-label">{{ formattedTime ? formattedTime : 'Keine Uhrzeit ausgewählt' }}</ion-label>
        </div>
      </ion-item>

      <!-- Speichern/Hinzufügen Button -->
      <ion-button expand="block" @click="saveReminder" shape="round">
        {{ isEditing ? 'Speichern' : 'Hinzufügen' }}
      </ion-button>
    </ion-content>
  </ion-modal>
</template>
  
  <script setup lang="ts">
  import { ref, watch, defineProps, defineEmits } from 'vue';
  import { IonInput, IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonLabel, IonButton, alertController} from '@ionic/vue';
  import { DatetimePicker } from '@capawesome-team/capacitor-datetime-picker';
  import { formatDate } from '@/utils/dateUtils'; 
  
  interface Reminder {
    id: number;
    text: string;
    date: string;
    time: string;
  }

  const formattedDate = ref('');
  const formattedTime = ref('');
  
  const props = defineProps<{
    reminder?: Reminder | undefined;
    isEditing: boolean;
    showModal: boolean;
    existingIds: number[];
  }>();
  
  const emit = defineEmits<{
    (e: 'save', reminder: Reminder): void;
    (e: 'dismiss'): void;
  }>();
  
  const modal = ref<HTMLIonModalElement | null>(null);
  const reminder = ref<Reminder>({
    id: 0,
    text: '',
    date: '',
    time: ''
  });
  
  watch(() => props.reminder, (newVal) => {
    if (newVal) {
      reminder.value = { ...newVal };
      formattedDate.value = reminder.value.date;
      formattedTime.value = reminder.value.time;
    }
  }, { immediate: true });

  const generateReminderId = (): number => {
    let id: number;
    do {
      id = Math.floor(Math.random() * 1000000);
    } while (props.existingIds.includes(id));
    return id;
  }
  
  const selectDate = async () => {
    const date = new Date(reminder.value.date || Date.now());
    const { value } = await DatetimePicker.present({
      mode: 'date',
      value: date.toISOString(),
      doneButtonText: 'Fertig',
      cancelButtonText: 'Abbrechen',
      theme: 'dark',
      locale: 'de-DE',
    });
    if (value) {
      reminder.value.date = value.split('T')[0];
      formattedDate.value = reminder.value.date;
    }
  };
  
  const selectTime = async () => {
    const time = new Date(`1970-01-01T${reminder.value.time || '00:00:00'}`);
    const { value } = await DatetimePicker.present({
      mode: 'time',
      value: time.toISOString(),
      doneButtonText: 'Fertig',
      cancelButtonText: 'Abbrechen',
      theme: 'dark',
      locale: 'de-DE',
    });
    if (value) {
      reminder.value.time = value.split('T')[1].substring(0, 5);
      formattedTime.value = reminder.value.time;
    }
  };
  
  const dismissModal = () => {
    emit('dismiss');
  };
  
  const saveReminder = async () => {
    if(!reminder.value.text.trim()) {
      const alert = await alertController.create({
        header: 'Beschreibung erforderlich',
        message: 'Bitte geben Sie einen Erinnerungstext ein, bevor Sie die Erinnerung speichern.',
        buttons: ['OK']
      });
      await alert.present();
      return;
    }

    if (!props.isEditing) {
      reminder.value.id = generateReminderId();
    }
    emit('save', reminder.value);
    dismissModal();
  };
  </script>
  
  <style scoped>
  .custom-modal {
    --height: 80%;
    --min-height: 80%;
    --border-radius: 15px;
  }

  .ion-padding {
    padding-bottom: 30px;
  }

  .spaced-item {
    margin-bottom: 20px;
  }

  ion-item {
    --inner-padding-end: 0;
    --padding-start: 20px;
    --border-color: #E5E5EA;
    font-size: 18px;
  }

  .label-large {
    font-size: 20px;
    font-weight: 700;
  }

  .datetime-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
  }

  .datetime-label {
    font-size: 16px;
    color: #6b6b6b;
  }

  ion-input {
    --padding-start: 0;
    --padding-end: 0;
    font-size: 19px;
    padding: 12px;
  }

  ion-button {
    --background: linear-gradient(135deg, #3A2FDF, #6146D9, #8B2FC0, #B52B9C, #D93875);
    --color: white;
    font-size: 18px;
    --padding: 16px 0;
  }
</style>