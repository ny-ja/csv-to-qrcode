<template>
  <div>
    <div>
      <label class="block mb-2 text-lg font-medium text-gray-900"
        >Upload image</label
      >
      <input
        class="block w-1/8 text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50"
        type="file"
        @change="onImageChange"
        accept="image/*"
      />
    </div>

    <div v-if="image">
      <label class="block mb-2 text-lg font-medium text-gray-900 mt-4"
        >Upload file</label
      >
      <input
        class="block w-1/8 text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50"
        type="file"
        @change="onFileChange"
        accept=".csv"
      />
    </div>

    <div class="grid grid-cols-3 gap-2 mt-4">
      <div v-for="(item, index) in csvData" :key="index" class="...">
        <p class="...">{{ item.name }}</p>
        <div v-if="!item.mergedImage" class="flex items-center">
          <div>
            <svg
              class="w-4 h-4 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600"
              viewBox="0 0 100 101"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
                fill="currentColor"
              />
              <path
                d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
                fill="currentFill"
              />
            </svg>
          </div>
          Preparing your file
        </div>
        <img v-else :src="item.mergedImage" class="w-full h-auto" />
      </div>
    </div>
    <button
      v-if="csvData.some((item) => item.mergedImage)"
      @click="downloadQRCodes"
      class="text-gray-900 bg-[#F7BE38] hover:bg-[#F7BE38]/90 focus:ring-4 focus:outline-none focus:ring-[#F7BE38]/50 font-medium rounded-lg text-sm px-5 py-2.5 text-center inline-flex items-center mt-4"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="w-6 h-6"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M9 13.5l3 3m0 0l3-3m-3 3v-6m1.06-4.19l-2.12-2.12a1.5 1.5 0 00-1.061-.44H4.5A2.25 2.25 0 002.25 6v12a2.25 2.25 0 002.25 2.25h15A2.25 2.25 0 0021.75 18V9a2.25 2.25 0 00-2.25-2.25h-5.379a1.5 1.5 0 01-1.06-.44z"
        />
      </svg>
      Download images
    </button>
  </div>
</template>

<script setup>
import { ref } from "vue";
import Papa from "papaparse";
import QRCode from "qrcode";
import JSZip from "jszip";
import { saveAs } from "file-saver";
import { createCanvas, loadImage } from "canvas";

const csvData = ref([]);
const image = ref(null);
const csvFileName = ref("");

const onFileChange = (e) => {
  const file = e.target.files[0];
  if (!file.type.match("text/csv")) {
    alert("Please upload a CSV file");
    return;
  }
  csvFileName.value = file.name.split(".").slice(0, -1).join(".");
  Papa.parse(file, {
    header: true,
    complete: function (results) {
      results.data.forEach(async (item) => {
        const qrcode = await QRCode.toDataURL(item.content, { scale: 8 });
        const img = await loadImage(image.value);
        const canvas = createCanvas(img.width, img.height);
        const ctx = canvas.getContext("2d");
        ctx.drawImage(img, 0, 0, img.width, img.height);
        const qrImg = await loadImage(qrcode);
        ctx.drawImage(
          qrImg,
          img.width - qrImg.width - 10,
          10,
          qrImg.width,
          qrImg.height
        ); // Adjust these values to position the QR code
        const mergedImage = canvas.toDataURL();
        csvData.value.push({
          ...item,
          qrcode,
          mergedImage,
        });
      });
    },
  });
};

const onImageChange = (e) => {
  const file = e.target.files[0];
  if (!file.type.startsWith("image/")) {
    alert("Please upload an image file");
    return;
  }
  const reader = new FileReader();
  reader.onload = (e) => {
    image.value = e.target.result;
  };
  reader.readAsDataURL(e.target.files[0]);
};

const downloadQRCodes = async () => {
  const zip = new JSZip();
  for (let item of csvData.value) {
    zip.file(`${item.name}.png`, item.mergedImage.split("base64,")[1], {
      base64: true,
    });
  }
  const content = await zip.generateAsync({ type: "blob" });
  saveAs(content, `${csvFileName.value}.zip`);
};
</script>
