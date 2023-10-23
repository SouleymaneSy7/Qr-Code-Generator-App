<template>
  <header class="header">
    <nav class="navbar container">
      <a href="#" class="navbar__logo">
        <h1><strong>Qr.</strong>Generator</h1>
      </a>

      <button
        type="button"
        class="navbar--btn"
        title="Show Information Modal"
        @click="toggleModal"
      >
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 192 512">
          <path
            d="M48 80a48 48 0 1 1 96 0A48 48 0 1 1 48 80zM0 224c0-17.7 14.3-32 32-32H96c17.7 0 32 14.3 32 32V448h32c17.7 0 32 14.3 32 32s-14.3 32-32 32H32c-17.7 0-32-14.3-32-32s14.3-32 32-32H64V256H32c-17.7 0-32-14.3-32-32z"
          />
        </svg>
      </button>

      <BaseModal :showModal="showModal" @hide-modal="toggleModal">
        <h2 class="modal__title">About Us:</h2>
        <p class="modal__text">
          Qr Code Generator allow you to create Qr Code from any of your URL,
          And to be able to download the generated Qr Code Image.
        </p>

        <h3 class="modal__title">How to use it:</h3>
        <ol class="modal__list">
          <li>Add or Paste a Url, Text or Coordinates in the Url inputs.</li>
          <li>
            Choose a color for your Qr Code by clicking on the color inputs or
            let it by defaults.
          </li>
          <li>Choose the size of your Qr Code.</li>
          <li>
            If you want to download the Qr Code Image, you have to just click on
            the "Download" button.
          </li>
        </ol>
      </BaseModal>
    </nav>
  </header>

  <main class="main container">
    <section class="main--form">
      <div class="main__img">
        <img src="../assets/qr-code-icons.svg" alt="Qr Code Icons" />
      </div>

      <form class="form" @submit.prevent="QrCodeSubmit">
        <div class="form__url">
          <label for="url"> Enter your URL:</label>
          <input
            type="text"
            name="url"
            id="url"
            placeholder="https://www.google.com"
            :class="urlInputErrors ? 'errors' : ''"
            v-model="QrURL"
          />
          <span class="url--errors" v-if="urlInputErrors">
            {{ urlInputErrorMessage }}
          </span>
        </div>

        <div class="form__colors">
          <label for="color">Qr Code Colors:</label>
          <div class="colors">
            <input type="color" name="color" id="color" v-model="QrLineColor" />
            <p class="colors--outputs">{{ QrLineColor.toUpperCase() }}</p>
          </div>
        </div>

        <div class="form__size">
          <label for="size">Qr Code Size:</label>
          <select name="size" id="size" v-model="QrSize">
            <option value="100">100 x 100</option>
            <option value="200">200 x 200</option>
            <option value="300" selected>300 x 300</option>
            <option value="400">400 x 400</option>
            <option value="500">500 x 500</option>
          </select>
        </div>

        <button class="form--btn" type="submit">Generate QR Code</button>
      </form>
    </section>

    <section class="results">
      <div class="spinner" v-if="isLoading"></div>
      <div id="qrcode"></div>
    </section>
  </main>
</template>

<script setup>
import { ref } from "vue";
import BaseModal from "./BaseModal.vue";

// Variable
const isLoading = ref(false);
const showModal = ref(false);
const showGeneratedQrCode = ref(false);

const QrURL = ref("");
const urlInputErrors = ref(false);
const urlInputRegExp =
  /^(?:http(s)?:\/\/)?[\w.-]+(?:\.[\w\.-]+)+[\w\-\._~:/?#[\]@!\$&'\(\)\*\+,;=.]+$/;
const urlInputErrorMessage = ref(null);

const QrLineColor = ref("#000000");
const QrSize = ref(300);

// Functions

// Show Modal
const toggleModal = () => {
  showModal.value = !showModal.value;
};

// Generate QR Code
const generateQRCode = (url, size, colors) => {
  const qrCodeElement = document.getElementById("qrcode");

  qrCodeElement.classList.add("qrcode--style");

  const qrcode = new QRCode(qrCodeElement, {
    text: url.value,
    width: size.value,
    height: size.value,
    colorDark: colors.value,
  });
};

// Clear the Qr Code element Content
const clearQrCodeUI = () => {
  document.getElementById("qrcode").innerHTML = "";
  document.getElementById("qrcode").classList.remove("qrcode--style");
  const Link = document.querySelector(".download--btn");

  if (Link) Link.remove();
};

// Download QR Code Image
const downloadQrCodeImage = (url, name) => {
  let link = document.createElement("a");
  link.setAttribute("download", name);
  link.classList = "download--btn";
  link.id = "download-btn";
  link.href = url;
  link.innerHTML = `Download
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
    <path
      d="M288 32c0-17.7-14.3-32-32-32s-32 14.3-32 32V274.7l-73.4-73.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3l128 128c12.5 12.5 32.8 12.5 45.3 0l128-128c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L288 274.7V32zM64 352c-35.3 0-64 28.7-64 64v32c0 35.3 28.7 64 64 64H448c35.3 0 64-28.7 64-64V416c0-35.3-28.7-64-64-64H346.5l-45.3 45.3c-25 25-65.5 25-90.5 0L165.5 352H64zm368 56a24 24 0 1 1 0 48 24 24 0 1 1 0-48z"
    />
  </svg>`;

  document.querySelector(".results").appendChild(link);
};

// Submit Form
const QrCodeSubmit = () => {
  clearQrCodeUI();

  if (QrURL.value.length < 1) {
    urlInputErrors.value = true;
    urlInputErrorMessage.value = "URL Field Cannot be Empty";
    isLoading.value = false;
    showGeneratedQrCode.value = false;
    return;
  } else if (urlInputRegExp.test(QrURL.value)) {
    urlInputErrors.value = false;
    urlInputErrorMessage.value = "";

    isLoading.value = true;

    setTimeout(() => {
      isLoading.value = false;

      generateQRCode(QrURL, QrSize, QrLineColor);

      setTimeout(() => {
        const qrCode = document.getElementById("qrcode");
        const SaveIMG = qrCode.querySelector("img").src;
        let ancientImgName = qrCode.title.replace("https://www.", "");
        let newImgName = ancientImgName.replace(".com", "");
        downloadQrCodeImage(SaveIMG, `qrcode-${newImgName}`);
      }, 10);
    }, 1000);
  } else {
    urlInputErrors.value = true;
    urlInputErrorMessage.value = "Please Enter A Valid URL";
    isLoading.value = false;
    showGeneratedQrCode.value = false;
    return;
  }
};
</script>
