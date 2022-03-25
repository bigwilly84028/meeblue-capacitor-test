<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>My BLE App</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Blank</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-grid>
        <ion-row>
          <ion-col>
            {{ ble_status_ui_message }}
          </ion-col>
        </ion-row>
        <ion-row>
          <ion-col>
            <button class="button" @click="scan()">Begin Scan</button>
          </ion-col>
        </ion-row>
        <ion-row>
          <ion-col>
            <button class="button" @click="requestDevice()">Request the Device</button>
          </ion-col>
        </ion-row>
        <ion-row>
          <ion-col>
            <button class="button" @click="connect()">Connect</button>
          </ion-col>
        </ion-row>
        <!-- <ion-row>
          <ion-col>
            <button class="button" @click="authenticate()">Authenticate</button>
          </ion-col>
        </ion-row> -->
        <!-- <ion-row>
          <ion-col>
            <button class="button" @click="getServices()">Get Services</button>
          </ion-col>
        </ion-row> -->
        <ion-row>
          <ion-col>
            <button class="button" @click="read()">Read</button>
          </ion-col>
        </ion-row>
        <ion-row>
          <ion-col>
            <button class="button" @click="notify()">Notify</button>
          </ion-col>
        </ion-row>
        <ion-row v-if="device">
          <ion-col>
            {{ device }}
          </ion-col>
        </ion-row>
        <ion-row v-if="sensorData">
          <ion-col>
            {{ sensorData }}
          </ion-col>
        </ion-row>
      </ion-grid>
    </ion-content>
  </ion-page>
</template>

<script>
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
} from "@ionic/vue";
import { defineComponent } from "vue";
// import { BleClient, numbersToDataView, numberToUUID } from "@capacitor-community/bluetooth-le";
import { BleClient, numberToUUID } from "@capacitor-community/bluetooth-le";

export default defineComponent({
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
  },
  data() {
    return {
      DEFAULT_PASSCODE: "imeeble",
      MEEBLUE_MAIN_SERVICE: "D35B1000-E01C-9FAC-BA8D-7CE20BDBA0C6",
      AUTH_CHARACTERISTIC: "D35B1001-E01C-9FAC-BA8D-7CE20BDBA0C6",

      TEMP_SENSOR_SERVICE: "D35B3000-E01C-9FAC-BA8D-7CE20BDBA0C6",
      TEMP_SENSOR_CHARACTERISTIC: "D35B3001-E01C-9FAC-BA8D-7CE20BDBA0C6",
      // TEMPERATURE_CHARACTERISTIC: numberToUUID(0x8531),
      
      device: undefined,
      ble_status_ui_message: undefined,
      sensorData: undefined,
    };
  },
  async created() {
    this.init();
  },
  methods: {
    async init(){
      try {
        await BleClient.initialize();

        await BleClient.startEnabledNotifications((enabled) => {
          this.ble_status_ui_message = "Bluetooth connection is " + enabled;
        });

      } catch (err) {
        alert(err.message);
      }
    },
    async requestDevice() {
      try{

        this.device = await BleClient.requestDevice({
          name: "meeblue"
        });

      } catch (err) {
        alert(err.message);
      }
    },
    async scan() {
      try {
        await BleClient.requestLEScan(
          {
            name: "meeblue"
          },
          (result) => {
            this.device = result.device;
            console.log("received new scan result @@@@ ", result);
          }
        );

        setTimeout(async () => {
          await BleClient.stopLEScan();
          console.log("stopped scanning");
        }, 15000);
      } catch (error) {
        console.error(error);
      }
    },
    async connect() {
      try {
        await BleClient.connect(this.device.deviceId, (deviceId) =>
          // this.onDisconnect(deviceId)
          this.ble_status_ui_message = "Connection Successful"
        );
      } catch (error) {
        this.ble_status_ui_message = "Connection Unsuccessful";
        console.log(error);
      }
    },
    async authenticate(){
      try {
        await BleClient.writeWithoutResponse(this.device.deviceId, this.MEEBLUE_MAIN_SERVICE, this.AUTH_CHARACTERISTIC, this.DEFAULT_PASSCODE);
        this.ble_status_ui_message = "Authentication Successful"
      } catch (error) {
        console.log(error);
      }
    },
    async getServices() {
      try {

        await BleClient.getServices(
          this.device.deviceId
        );

      } catch (error) {
        console.log(error);
      }
    },
    async read(){
      const result = await BleClient.read(
        this.device.deviceId,
        this.TEMP_SENSOR_SERVICE,
        this.TEMP_SENSOR_CHARACTERISTIC
      );

      // try to parse sensor data...
      const flags = result.getUint8(0);
        const rate16Bits = flags & 0x1;
        let parsedVal;
        if (rate16Bits > 0) {
          parsedVal = result.getUint16(1, true);
        } else {
          parsedVal = result.getUint8(1);
        }

        console.log("T&H Read: " + parsedVal);
    },
    async notify() {
      try {
        await BleClient.startNotifications(
          this.device.deviceId,
          this.TEMP_SENSOR_SERVICE,
          this.TEMP_SENSOR_CHARACTERISTIC,
          (result) => {
            // try to parse sensor data...
            const flags = result.getUint8(0);
            const rate16Bits = flags & 0x1;
            let parsedVal;
            if (rate16Bits > 0) {
              parsedVal = result.getUint16(1, true);
            } else {
              parsedVal = result.getUint8(1);
            }

            console.log("T&H startNotifications: " + parsedVal);
          }
        );
      } catch (error) {
        console.log(error);
      }
    },
    parseValue(value){
      const flags = value.getUint8(0);
      const rate16Bits = flags & 0x1;
      let parsedVal;
      if (rate16Bits > 0) {
        parsedVal = value.getUint16(1, true);
      } else {
        parsedVal = value.getUint8(1);
      }
      return parsedVal
    },
    onDisconnect(deviceId) {
      console.log(`device ${deviceId} disconnected`);
    },
  },
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
