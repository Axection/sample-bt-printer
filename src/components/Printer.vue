<template>
  <div>
    <input class="border-2 mr-2" type="text" :value="message">
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
    return { message, error };
  },
  methods: {
    async sendToPrinter() {
      try {
        const device = await navigator.bluetooth.requestDevice({
          acceptAllDevices: true,
        });
        const server = await device.gatt?.connect();
        const service = await server?.getPrimaryService('000018f0-0000-1000-8000-00805f9b34fb');
        const chars = await service?.getCharacteristic('00002af1-0000-1000-8000-00805f9b34fb');
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
