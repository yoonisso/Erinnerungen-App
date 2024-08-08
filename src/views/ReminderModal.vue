<template>
    <ion-modal ref="modal" :is-open="showModal" @ionModalDidDismiss="dismissModal">
      <ion-header>
        <ion-toolbar>
          <ion-title>{{ isEditing ? 'Erinnerung bearbeiten' : 'Erinnerung hinzufügen' }}</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="dismissModal">Schließen</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
  
      <ion-content>
        <ion-item>
          <ion-label position="stacked">Text</ion-label>
          <ion-input v-model="reminder.text" placeholder="Erinnerungstext eingeben"></ion-input>
        </ion-item>
        <ion-item>
          <ion-label position="stacked">Datum</ion-label>
          <ion-button expand="full" @click="selectDate">Datum auswählen</ion-button>
          <ion-label>{{ formattedDate }}</ion-label>
        </ion-item>
        <ion-item>
          <ion-label position="stacked">Uhrzeit</ion-label>
          <ion-button expand="full" @click="selectTime">Uhrzeit auswählen</ion-button>
          <ion-label>{{ formattedTime }}</ion-label>
        </ion-item>
        <ion-button expand="full" @click="saveReminder">{{ isEditing ? 'Speichern' : 'Hinzufügen' }}</ion-button>
      </ion-content>
    </ion-modal>
  </template>
  
  <script setup lang="ts">
  import { ref, watch, defineProps, defineEmits } from 'vue';
  import { IonInput, IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonLabel, IonButton} from '@ionic/vue';
  import { DatetimePicker } from '@capawesome-team/capacitor-datetime-picker';
  
  interface Reminder {
    id: number;
    text: string;
    date: string;
    time: string;
  }
  
  const props = defineProps<{
    reminder?: Reminder | undefined;
    isEditing: boolean;
    showModal: boolean;
  }>();
  
  const emit = defineEmits<{
    (e: 'save', reminder: Reminder): void;
    (e: 'dismiss'): void;
  }>();
  
  const modal = ref<HTMLIonModalElement | null>(null);
  const reminder = ref<Reminder>({
    id: 1, //TODO add Check that not two reminders with same value
    text: '',
    date: '',
    time: ''
  });
  
  watch(() => props.reminder, (newVal) => {
    if (newVal) {
      reminder.value = { ...newVal };
    }
  }, { immediate: true });
  
  const formattedDate = ref('');
  const formattedTime = ref('');
  
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
  
  const saveReminder = () => {
    emit('save', reminder.value);
    dismissModal();
  };
  </script>
  
  <style scoped>
  </style>
  