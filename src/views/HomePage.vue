<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>CPS Counter</ion-title>
        <ion-buttons slot="primary">
          <ion-button color="primary" fill="solid" @click="InfoPopup">
            <ion-icon :icon="infoIcon"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header class="ion-no-border ion-padding-top ion-padding-horizontal">
        <ion-grid>
          <ion-row>
            <ion-col>
              <div class="ion-text-start" id="score">
                Clicks per Second: {{ cpsResult.toFixed(2) }}
              </div>
            </ion-col>
            <ion-col>
              <div class="ion-text-end">
                Time Left: {{ temporitzador }}
                Counter: {{ comptador }}
              </div>
            </ion-col>
          </ion-row>
        </ion-grid>
      </ion-header>

      <div id="container">
        <ion-button @click="Boto" :disabled="buttonDisabled">
          {{ funciona ? '¡CLICK!' : 'Start Game' }}
        </ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import { alertController, IonButton, IonContent, IonGrid, IonHeader, IonIcon, IonPage, IonTitle, IonToolbar, toastController } from '@ionic/vue';
import { defineComponent } from 'vue';
import { informationCircleOutline } from "ionicons/icons";

const INITIAL_TIME = 5;
const RESET_DELAY = 2000;

export default defineComponent({
  name: 'CPSCounter',
  components: { IonButton, IonContent, IonHeader, IonIcon, IonPage, IonTitle, IonToolbar, IonGrid },
  setup() {
    return {
      infoIcon: informationCircleOutline,
      buttonDisabled: false,
    }
  },
  data() {
    return {
      comptador: 0,
      temporitzador: INITIAL_TIME,
      funciona: false,
      cpsResult: 0,
      countdownTimer: null,
      clickCountTimer: null,
      updateUITimer: null
    }
  },
  watch: {
    temporitzador(newTime) {
      if (newTime <= 0) {
        this.funciona = false;
        clearInterval(this.countdownTimer);
        clearInterval(this.clickCountTimer);
        this.Resultat();
        this.buttonDisabled = true;
        setTimeout(() => { this.reset(); this.buttonDisabled = false; }, RESET_DELAY);
      }
    }
  },
  methods: {
    updateCPS() {
      if (this.temporitzador <= 0) {
        this.cpsResult = this.comptador / INITIAL_TIME;
        this.cpsResult = parseFloat(this.cpsResult.toFixed(2));
      }
    },
    async InfoPopup() {
      const alert = await alertController.create({
        header: 'App Comptador',
        subHeader: 'Creado por Eric Ortega Gisbert',
        message: 'Puedes encontrar el código fuente en: <a href="https://github.com/EricOrtegaGi/Ionic">GitHub</a>',
        buttons: ['OK'],
      });
      await alert.present();
    },
    Boto() {
      if (!this.funciona && !this.buttonDisabled) this.startGame();
      else if (this.funciona && !this.buttonDisabled) this.increment();
    },
    startGame() {
      this.reset();
      this.funciona = true;
      this.countdownTimer = setInterval(() => { if (this.temporitzador > 0) this.temporitzador--; else this.endGame(); }, 1000);

      this.clickCountTimer = setInterval(() => { /* Add anything needed every second */ }, 1000);

      this.updateUITimer = setInterval(() => { if (this.temporitzador <= 0) clearInterval(this.updateUITimer); }, 100);
    },
    increment() {
      this.comptador++;
      this.updateCPS();
    },
    endGame() {
      clearInterval(this.countdownTimer);

      if (this.temporitzador <= 0) this.Resultat();

      this.buttonDisabled = true;
      setTimeout(() => { this.reset(); this.buttonDisabled = false; }, RESET_DELAY);
    },
    reset() {
      clearInterval(this.countdownTimer);
      clearInterval(this.clickCountTimer);
      clearInterval(this.updateUITimer);

      this.comptador = 0;
      this.temporitzador = INITIAL_TIME;
      this.funciona = false;
    },
    init() {
      this.clickCountTimer = setInterval(() => { /* Add anything needed every second */ }, 1000);
    },
    async Resultat() {
      const message = `Game Over! Counter: ${this.comptador}`;
      console.log(message);
      const toast = await toastController.create({ color: 'dark', duration: 2000, message, showCloseButton: true });
      await toast.present();
    }
  }
});
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
</style>