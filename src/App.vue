<template>
  <div id="app">
    <h1>Skener</h1>
    <div v-if="!cameraAvailable">
      <p>Kamera nije dostupna na ovom uređaju.</p>
    </div>
    <div v-else>
      <video ref="video" width="300" height="200" autoplay></video>
      <canvas ref="canvas" width="300" height="200" style="display: none;"></canvas>
      <div v-if="scannedData">Skenirani sadržaj: {{ scannedData }}</div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import jsQR from 'jsqr';

export default {
  setup() {
    const video = ref(null);
    const canvas = ref(null);
    const scannedData = ref(null);
    const cameraAvailable = ref(true);

    const startCamera = async () => {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        video.value.srcObject = stream;
      } catch (err) {
        console.error('Nije moguće pristupiti kameri:', err);
        cameraAvailable.value = false;
      }
    };

    const scanQRCode = () => {
      const context = canvas.value.getContext('2d');
      context.drawImage(video.value, 0, 0, 300, 200);
      const imageData = context.getImageData(0, 0, 300, 200);
      const code = jsQR(imageData.data, imageData.width, imageData.height);
      if (code) {
        scannedData.value = code.data;
      } else {
        requestAnimationFrame(scanQRCode);
      }
    };

    onMounted(() => {
      startCamera();
      if (video.value) {
        video.value.addEventListener('loadedmetadata', () => {
          canvas.value.width = video.value.videoWidth;
          canvas.value.height = video.value.videoHeight;
          requestAnimationFrame(scanQRCode);
        });
      }
    });

    return { video, canvas, scannedData, cameraAvailable };
  },
};
</script>


<style>
/* Dodajte CSS stilove ovdje ako je potrebno */
</style>
