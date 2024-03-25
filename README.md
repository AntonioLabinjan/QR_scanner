# Programski kod

```Python
import tkinter as tk
from tkinter import filedialog, messagebox
import qrcode
import barcode
from barcode.writer import ImageWriter


def generiraj_barkod():
    tekst = unos_polje.get()
    tip_barkoda = vrsta_barkoda.get()

    if tip_barkoda.lower() not in ("ean13", "code128"):
        status_label.config(text="Nepodržani tip barkoda")
        return

    if not tekst:
        status_label.config(text="Unesite tekst za generiranje barkoda")
        return

    try:
        kod = barcode.get_barcode_class(tip_barkoda.lower())
    except barcode.errors.BarcodeNotFoundError:
        status_label.config(text="Greška pri dohvaćanju klase barkoda")
        return

    kod_instance = kod(tekst, writer=ImageWriter())
    img = kod_instance.render()

    save_path = filedialog.asksaveasfilename(defaultextension=".png")
    if save_path:
        img.save(save_path)
        status_label.config(
            text="Barkod je uspješno spremljen u datoteku {}".format(save_path))


def generiraj_qr_kod():
    tekst = tekst_unos.get()

    if not tekst:
        status_label.config(text="Unesite tekst ili URL za generiranje QR koda")
        return

    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=10,
        border=4,
    )
    qr.add_data(tekst)
    qr.make(fit=True)

    img = qr.make_image(fill_color="black", back_color="white")
    save_path = filedialog.asksaveasfilename(defaultextension=".png")
    if save_path:
        img.save(save_path)
        status_label.config(
            text=f"QR kod je uspješno spremljen u datoteku {save_path}")


root = tk.Tk()
root.title("Generiranje barkoda i QR kodova")

barkod_label = tk.Label(root, text="Unesite tekst za generiranje barkoda:")
barkod_label.pack()
unos_polje = tk.Entry(root)
unos_polje.pack()

vrsta_barkoda_label = tk.Label(root, text="Odaberite vrstu barkoda:")
vrsta_barkoda_label.pack()
vrsta_barkoda = tk.StringVar()
vrsta_barkoda.set("ean13")
vrste_barkoda_optionmenu = tk.OptionMenu(root, vrsta_barkoda, "ean13",
                                          "code128")
vrste_barkoda_optionmenu.pack()

generiraj_barkod_gumb = tk.Button(root,
                                  text="Generiraj barkod",
                                  command=generiraj_barkod)
generiraj_barkod_gumb.pack()

qr_label = tk.Label(root, text="Unesite tekst ili URL za generiranje QR koda:")
qr_label.pack()
tekst_unos = tk.Entry(root)
tekst_unos.pack()

generiraj_qr_gumb = tk.Button(root,
                               text="Generiraj QR kod",
                               command=generiraj_qr_kod)
generiraj_qr_gumb.pack()

status_label = tk.Label(root, text="")
status_label.pack()

root.mainloop()
```
```JavaScript
<template>
  <div id="app">
    <h1>Skener</h1>
    <div v-if="!cameraAvailable">
      <p>Kamera nije dostupna na ovom uređaju.</p>
    </div>
    <div v-else>
      <video ref="video" width="300" height="200" autoplay></video>
      <canvas ref="canvas" width="300" height="200" style="display: none;"></canvas>
      <div v-if="scannedData" class="result">
        <p>Skenirani sadržaj:</p>
        <p>{{ scannedData }}</p>
        <button @click="openScannedLink">Otvori link</button>
      </div>
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
        const stream = await navigator.mediaDevices.getUserMedia({
          video: { facingMode: 'environment' } // Postavljanje facingMode na 'environment' za zadnju kameru
        });
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

    const openScannedLink = () => {
      if (scannedData.value.startsWith('http')) {
        window.open(scannedData.value, '_blank');
      } else {
        alert('Skenirani sadržaj nije valjan URL.');
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

    return { video, canvas, scannedData, cameraAvailable, openScannedLink };
  },
};
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  text-align: center;
}

.container {
  margin: 20px auto;
}

video {
  border: 2px solid #333;
  border-radius: 5px;
}

.result {
  margin-top: 20px;
}

button {
  padding: 10px 20px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #0056b3;
}
</style>
```

