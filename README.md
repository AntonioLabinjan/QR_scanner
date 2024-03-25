### Sustav za kreiranje i skeniranje QR kodova

Za potrebe projekta iz kolegija "Šifre i kodovi" vezanog uz temu "Barkodovi u modernoj komunikaciji s fokusom na QR kodove", kreirane su 2 mini-aplikacije. Jedna za kreiranje QR kodova i barkodova te druga za skeniranje i očitavanje barkodova.
Korišteni programski jezici za njihovu izradu su: Python sa uključenim bibliotekama tkinter, qrcode i barcode, te JavaScript sa Vue frameworkom. Ovdje će biti objašnjen princip rada obe aplikacije, sam programski kod te način njihovog pokretanja. Aplikacije su izrađene sa ciljem brzog i jednostavnog kreiranja i očitavanja kodova bilo kada i bilo gdje zato što su oni uvelike zastupljeni u našem svakodnevnom okruženju i često se koriste u svakodnevnim aktivnostima. Omogućuju nam da pretvorimo bilo kakve informacije u QR kod te isto tako očitamo bilo koji QR kod u bilo kojem trenutku. 

Preduvjeti: ukoliko želite lokalno preuzeti kod i pokrenuti ga, potrebno je instalirati određene pakete kako bi aplikacije imale sve uvjete za uspješno pokretanje.
Za generator kodova, potrebno je imati instaliran Python sa određenim bibliotekama.

<ol>1. tkinter</ol>
Iako bi tkinter trebao defaultno biti instaliran s Pythonom, najbolje je za svaki slučaj napisati naredbu:
```bash
pip install tkinter
```
Ta će naredba instalirati biblioteku tkinter ukoliko ona ne postoji na našem računalu. Ako već postoji, vratiti će nam obavijest o tome.
Tkinter je Pythonova biblioteka za izradu grafičkog korisničkog sučelja (GUI) koja je jednostavna za korištenje i može raditi na više operativnih sustava. Omogućuje kreiranje raznovrsnih grafičkih elemenata poput gumba, polja za unos teksta i dijaloških okvira.

<ol>2. qrcode[pil]</ol>
Zatim je potrebno instalirati biblioteku za kreiranje qr-kodova (qrcode) te dodatno i [pil] koji omogućuje rad sa slikama.
```Python
pip install qrcode[pil]
```
Biblioteka qrcode omogućuje generiranje QR kodova u Pythonu. Ona podržava različite konfiguracije QR kodova, uključujući veličinu, razinu ispravka grešaka, boju i pozadinu, omogućujući prilagodbu izgleda generiranih kodova. Osim toga, omogućuje jednostavno integriranje generiranih QR kodova u Python aplikacije, web stranice ili druge sustave. Ova biblioteka također podržava mogućnost generiranja QR kodova iz različitih formata podataka poput teksta, URL-ova, vCard kontaktnih podataka i drugih. 

Oznaka [pil] u naredbi za instalaciju biblioteke qrcode označava dodatnu neobaveznu komponentu "Python Imaging Library" (PIL), koja omogućuje generiranje slika QR kodova. Instalacija PIL-a uz qrcode omogućuje korisnicima da generiraju vizualne reprezentacije QR kodova. Bez PIL-a, qrcode bi generirao samo podatke QR koda, ali ne i sliku koja prikazuje QR kod.

<ol>3. barcode</ol>
Zadnja Python biblioteka koju moramo uključiti je barcode. To radimo sa sljedećom naredbom:
```Python
pip install python-barcode
```
Biblioteka python-barcode omogućuje generiranje različitih vrsta barkodova u programskom jeziku Python. Ova biblioteka pruža jednostavno sučelje za stvaranje linearnih i 2D barkodova, kao što su EAN-13, Code 39, Code 128 i QR kodovi. Korisnici mogu kontrolirati različite parametre generiranih barkodova, uključujući veličinu, boju i razinu ispravka grešaka. python-barcode podržava generiranje barkodova iz različitih formata podataka, što olakšava integraciju s različitim vrstama aplikacija. Ova biblioteka je korisna za razvoj aplikacija koje zahtijevaju generiranje barkodova, poput sustava za praćenje inventara ili trgovinskih aplikacija. Integracija python-barcode-a u Python aplikacije omogućuje brzo dodavanje funkcionalnosti generiranja barkodova.

Što se tiče čitača kodova, postoji easy path i hard path. U hard pathu, bilo bi potrebno instalirati Node.js, Vue.js, kreirati novi Vue projekt, dodavati određene zavisnosti u projekt i sl. No, Vue aplikacije je prilično jednostavno hostati na razne poslužitelje. U našem slučaju to je Vercel, pa je moguće bez ikakvih problema pristupiti čitaču preko linka.
qr-scanner-zeta.vercel.app
Jedini je preduvjet da uređaj na kojem radimo ima internetsku vezu, kameru i neki od browsera u kojem može otvoriti aplikaciju.
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
Kod objašnjen ukratko:

Uvozimo potrebne biblioteke. tkinter se koristi za stvaranje GUI-a, qrcode za generiranje QR kodova, a barcode za generiranje barkodova. Iz tkinter-a uvozimo filedialog i messagebox za rad s dijaloškim okvirima i prikazivanje poruka.
Definiramo funkciju generiraj_barkod() koja se poziva kad korisnik želi generirati barkod. Dohvaćamo tekst iz polja za unos i vrstu barkoda iz izbornika te provjeravamo jesu li uneseni podaci valjani.
Pokušavamo dohvatiti klasu barkoda na temelju odabrane vrste barkoda. Ako je odabrana vrsta barkoda nepodržana, prikazujemo odgovarajuću poruku.
Stvaramo instancu odabrane klase barkoda s tekstom i odabranim načinom pisanja kao parametrima te generiramo sliku barkoda.
Prikazujemo dijaloški okvir za odabir putanje i naziva datoteke u koju će se spremiti generirani barkod.
Ako je korisnik odabrao putanju za spremanje barkoda, sprema se generirani barkod kao slika na odabranoj putanji.
Postavljamo tekst poruke na statusnoj traci koja obavještava korisnika o uspješnom spremanju barkoda.
Funkcija generiraj_qr_kod() definira postupak generiranja QR koda. Dohvaćamo tekst unesen u polje za unos. Ako nije unesen tekst, prikazuje se odgovarajuća poruka na statusnoj traci. Nakon toga, stvaramo QR kod s postavkama za verziju, razinu korekcije grešaka, veličinu kutije i širinu granice. Dodajemo tekst QR kodu i generiramo ga prilagođavajući veličini teksta. Zatim stvaramo sliku QR koda s postavljenim bojama unutrašnjosti i pozadine. Korisniku se prikazuje dijaloški okvir za odabir putanje i naziva datoteke u koju će se spremiti generirani QR kod. Ako korisnik odabere putanju, QR kod se sprema kao slika na odabranoj putanji, a korisniku se prikazuje poruka na statusnoj traci o uspješnom spremanju. Na kraju, pokreće se glavna petlja programa za prikaz GUI-a.







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
Ovaj kod predstavlja Vue.js komponentu za skeniranje QR kodova putem kamere i njihovo otvaranje.
U HTML dijelu komponente, definira se struktura sučelja. Ako kamera nije dostupna na uređaju, prikazuje se poruka o nedostupnosti. Inače, prikazuje se video element koji prikazuje sliku s kamere te skriveni canvas element koji se koristi za snimanje slike s videa. Ako je skeniran QR kod, prikazuje se tekst skeniranog sadržaja i gumb za otvaranje skeniranog linka.
U JavaScript dijelu koristi se Vue-ov setup() hook za inicijalizaciju komponente. Definirane su ref varijable koje predstavljaju video i canvas elemente, skenirani podaci i indikator dostupnosti kamere.
Funkcija startCamera() pokreće kameru uređaja i postavlja sliku s kamere na video element. Ako kamera nije dostupna, postavlja se odgovarajuća poruka.
Funkcija scanQRCode() snima sliku s videa i pokušava pronaći QR kod na slici koristeći jsQR biblioteku. Ako QR kod bude pronađen, skenirani podaci se prikazuju, inače se funkcija ponovno poziva rekurzivno dok se ne nađe QR kod.
Funkcija openScannedLink() otvara skenirani URL u novom prozoru ako je pronađen URL u skeniranim podacima.
U Vue-ovom hooku onMounted() pokreće se inicijalizacija kamere i postavljanje veličine canvas elementa nakon što se komponenta učita.
U CSS dijelu definirani su stilovi za različite elemente komponente, uključujući stilove za video element, rezultat skeniranja i gumb za otvaranje linka.

#### Pokretanje:
Za pokretanje generatora QR kodova, moguće je koristiti većinu Python editora, iako je Replit možda najbolja opcija zato što automatski instalira sve potrebne biblioteke te se korisnici ne moraju time zamarati. Za pokretanje skenera, najlakše je samo otvoriti ovaj link: qr-scanner-zeta.vercel.app  , no moguća je i opcija preuzimanja cijele aplikacije, instalacije svih potrebnih ovisnosti (instalirati node.js, instalirati vue-cli, preuzeti cijeli folder i pokrenuti preko npm run serve preko terminala (iskreno preporučamo link)).

Napomena: ne dirati datoteke izvan src foldera!!

#### Screenshotovi: 

![Opis slike](QR_scanner/Screen1.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)
![Opis slike](slike/slika.png)

