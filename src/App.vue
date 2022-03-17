<template>
  <ion-app>
    <ion-router-outlet />
  </ion-app>
</template>

<script>
import { IonApp, IonRouterOutlet } from '@ionic/vue';
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'App',
  components: {
    IonApp,
    IonRouterOutlet
  }
});
</script>

<style>
/* -------------------------------- */
/* Global Button ------------------ */
/* -------------------------------- */
button.button {
  background-color: var(--ion-color-primary);
  color: var(--ion-color-white);
  height: 40px;
  font-size: 14px;
  font-weight: 700;
  padding-right: 20px;
  padding-left: 20px;
  border-radius: 10px;
  box-shadow: var(--button-shadow);
  transition: all 0.2s ease-in-out;
}
button.button:hover {
  background-color: var(--ion-color-primary-shade);
  box-shadow: var(--button-shadow-hover);
}
button.button.button-block {
  width: 100%;
}
button.button ion-icon {
  font-size: 1.5em;
  fill: var(--ion-color-white);
  vertical-align: middle;
  margin-right: 5px;
  position: relative;
  top: -2px;
}
button.button ion-icon:last-child {
  margin-right: 0;
  /* margin-left: 5px; */
}
button.button svg {
  vertical-align: middle;
  margin-right: 5px;
  margin-top: -3px;
  fill: #ffffff;
}
button.button.button-solo svg {
  margin: 0px;
}
</style>