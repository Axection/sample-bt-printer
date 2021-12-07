<template>
  <div>
    <h1>Configuration</h1>
    <div class="mb-2">
      <label for="device_id" class="mr-1">Device ID</label>
    <input class="border-2 w-full text-center" type="text" name="device_id" :value="deviceID">
    </div>
    <div class="mb-2">
      <label for="service_id" class="mr-1">Service ID</label>
      <input class="border-2 w-full text-center" type="text" name="service_id" :value="serviceID">
    </div>
    <div class="mb-2">
      <label for="char_id" class="mr-1">Characteristic ID</label>
      <input class="border-2 w-full text-center" type="text" name="char_id" :value="charID" >
    </div>
    <hr class="my-4">
    <label for="msg_text">Text to send:</label>
    <input class="border-2 mx-2" name="msg_text" type="text" :value="message">
    <input class="p-2 border-r-2 border-solid" type="button" value="Send" @click="sendToPrinter">
    <p class="text-red-400"> {{error}} </p>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';

export default defineComponent({
  setup() {
    const message = ref('');
    const error = ref('');
    const deviceID = ref('00001101-0000-1000-8000-00805f9b34fb');
    const serviceID = ref('000018f0-0000-1000-8000-00805f9b34fb');
    const charID = ref('00002af1-0000-1000-8000-00805f9b34fb');
    return {
      message, error, deviceID, serviceID, charID,
    };
  },
  methods: {
    async sendToPrinter() {
      try {
        const device = await navigator.bluetooth.requestDevice({
          acceptAllDevices: true,
          optionalServices: [this.deviceID],
        });
        const server = await device.gatt?.connect();
        console.log(device.uuids);
        const service = await server?.getPrimaryService(this.serviceID);
        const chars = await service?.getCharacteristic(this.charID);
        this.printData(chars);
      } catch (err) {
        this.error = err;
      }
    },
    // eslint-disable-next-line no-undef
    async printData(chars: BluetoothRemoteGATTCharacteristic | undefined) {
    // Get the bytes for the text
      const encoder = new TextEncoder();
      // Add line feed + carriage return chars to text
      const text = encoder.encode(`${this.message}\u000A\u000D`);
      await chars?.writeValue(text);
      console.log('Write done.');
    },
  },
  watch: {
    message() {
      this.error = '';
    },
  },
});
</script>

<style lang="scss" scoped>

</style>
