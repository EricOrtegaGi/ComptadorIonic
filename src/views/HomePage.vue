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
              <div class="ion-text-end">
                Time Left: {{ temporitzador }}
                Counter: {{ comptador }}
              </div>
              <div class="ion-text-center">
                CPS: {{ clicksPerSecond }}<br>
                Peak CPS: {{ peakCPS }}
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
      buttonDisabled: false
    }
  },
  data() {
    return {
      comptador: 0,
      temporitzador: INITIAL_TIME,
      funciona: false,
      clicksPerSecond: 0,
      peakCPS: 0,
      countdownTimer: null,
      clickCountTimer: null,
      lastSecondClics: 0,
      currentSecondClics: 0
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
      this.countdownTimer = setInterval(() => {
        if (this.temporitzador > 0) this.temporitzador--; else this.endGame();
      }, 1000);

      // Reiniciamos los contadores de CPS y Peak CPS
      this.clicksPerSecond = 0;
      this.peakCPS = 0;

      // Configuramos el temporizador para calcular CPS y Peak CPS
      this.clickCountTimer = setInterval(() => {
        this.clicksPerSecond = this.currentSecondClics;
        if (this.currentSecondClics > this.peakCPS) {
          this.peakCPS = this.currentSecondClics;
        }
        this.currentSecondClics = 0; // Reiniciamos los clics por segundo para el próximo segundo
      }, 1000);
    },
    increment() {
      this.comptador++;
      this.currentSecondClics++;
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

      this.comptador = 0;
      this.temporitzador = INITIAL_TIME;
      this.funciona = false;
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
