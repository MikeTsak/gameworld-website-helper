<template>
  <div id="app">
          <!-- 1. Personal Articles Section -->
    <div class="container">
      <a href="https://www.gameworld.gr/jreviews/my-listings?user=54184" class="articles-button">
        <img src="https://www.gameworld.gr/images/avatar/5763925d123ca66c2e226227.jpg" alt="Avatar" class="button-avatar" />
        Read articles by me
      </a>

      <hr> <!-- Horizontal line to separate sections -->

            <!-- 2. Game Rating Section -->
      <input class="input-style" type="text" v-model="prefix" placeholder="Enter game name for images" />
      <div class="rating-section">
        <div v-for="category in categories" :key="category.name" class="slider-container">
          <label :for="category.name">{{ category.label }}</label>
          <input v-model.number="category.value" type="range" min="0" max="10" :id="category.name" />
          <input v-model.number="category.value" type="number" min="0" max="10" class="rating-input" />
          <!-- <span>{{ category.value }}</span> -->
        </div>
        <div class="slider-container">
          <label for="average">Γενικά</label>
          <!-- <input type="range" id="average" disabled :value="average.toFixed(1)" /> -->
          <!-- <input type="number" disabled :value="average.toFixed(1)" class="rating-input" /> -->
          <span>{{ average.toFixed(1) }}</span>
        </div>
      </div>
      <button @click="openMetacriticSearch" class="metacritic-button">
        <img src="https://pbs.twimg.com/profile_images/527528131171590144/EQXs3lpX_400x400.png" alt="Metacritic logo" />
        Metacritic Score
      </button>

      <hr> <!-- Horizontal line to separate sections -->

          <!-- 3. Screenshot Tool -->
      <h3>Screenshot tool for reviews</h3>
      <input class="input-style" type="file" multiple @change="handleFiles" />
      (use Ctrl + Click to select multiple files)
      
      <button class="button-style" @click="processImages">Resize and Add Watermark</button>

      <hr> <!-- Horizontal line to separate sections -->

           <!-- 4. News Photo Editor -->     
      <h3>News Photo Editor</h3>
      <input type="file" class="input-style" @change="onImageChange" ref="fileInput">
      <div class="cropper-container">
      <div v-if="showCropper">

        <vue-cropper 
          ref="cropper"
          :src="imgSrc"
          :aspect-ratio="16/9"
          :view-mode="2"
          :guides="true">
        </vue-cropper>
        <div class="button-crop">
        <button  class="button-style" @click="cropImage">Crop</button>
        <button  class="button-style" @click="cancelCrop">Cancel</button>
        </div>
      </div>
      </div>
      
      <img v-if="croppedImage" :src="croppedImage" alt="Cropped Image">
      <a v-if="croppedImage" :href="croppedImage" download="cropped-image.jpg" class="download-button">Download Cropped Image</a>

      <footer>
        <a href="https://www.miketsak.gr" class="footer-link">
          Made by miketsak.gr
        </a>
        <br />
        <a href="https://www.gameworld.gr/" class="footer-link">
          The GameWorld logo is a © of gameworld.gr 
        </a>
        ||
        <a href="https://www.metacritic.com/" class="footer-link">
          The Metacritic logo is a © of metacritic.com
        </a>
      </footer>

    </div>
  </div>
</template>

<script>
import VueCropper from 'vue-cropperjs';
import 'cropperjs/dist/cropper.css';

export default {
  data() {
    return {
      files: [],
      prefix: "",
      newsImage: null,
      imgSrc: null,
      showCropper: false,
      croppedImage: null,
    showCropPopup: false,
      categories: [
        { label: 'Γραφικά', name: 'graphics', value: 5 },
        { label: 'Ήχος', name: 'sound', value: 5 },
        { label: 'Gameplay', name: 'gameplay', value: 5 },
        { label: 'Σενάριο', name: 'scenario', value: 5 },
        { label: 'Αντοχή', name: 'endurance', value: 5 },
      ]
    };
  },
  watch: {
    prefix: {
      handler: "sanitizeGameName",
      immediate: true
    }
  },
  components: {
    VueCropper
  },
  computed: {
    average() {
      const sum = this.categories.reduce((acc, cat) => acc + cat.value, 0);
      return sum / this.categories.length;
    }
  },
  methods: {
    handleFiles(event) {
      this.files = [...event.target.files];
    },
    async processImage(file, index) {
      const originalImage = await this.createImageFromFile(file);
      await this.addWatermark(originalImage, index);
    },
    createImageFromFile(file) {
      return new Promise((resolve) => {
        const img = new Image();
        img.onload = () => resolve(img);
        img.src = URL.createObjectURL(file);
      });
    },
    async addWatermark(img, index) {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");

      const aspectRatio = img.width / img.height;
      let newWidth = 1920;
      let newHeight = newWidth / aspectRatio;
      if (newHeight > 1080) {
        newHeight = 1080;
        newWidth = newHeight * aspectRatio;
      }
      canvas.width = newWidth;
      canvas.height = newHeight;

      ctx.drawImage(img, 0, 0, newWidth, newHeight);

      const watermarkSrc = "/_Watermark 2015 - small.png";
      const watermark = await this.createImageFromFile(
        await fetch(watermarkSrc).then((res) => res.blob())
      );

      const watermarkWidth = 230 * 1.5;
      const watermarkHeight = 40 * 1.5;
      ctx.drawImage(watermark, newWidth - watermarkWidth, newHeight - watermarkHeight, watermarkWidth, watermarkHeight);

      const filename = `${this.prefix}-${index + 1}.jpg`;
      this.downloadImage(canvas.toDataURL("image/jpeg"), filename);
    },
    downloadImage(dataUrl, filename) {
      const a = document.createElement("a");
      a.href = dataUrl;
      a.download = filename;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    },
    processImages() {
      this.files.forEach((file, index) => {
        this.processImage(file, index);
      });
    },
    sanitizeGameName() {
      // Replace spaces with dashes
      this.prefix = this.prefix.replace(/\s+/g, '-');

      // Remove or replace characters that are not allowed in filenames
      // Disallowed characters: \ / : * ? " < > |
      this.prefix = this.prefix.replace(/[\\/:*?"<>|]/g, '-');
    },
    openMetacriticSearch() {
      const baseUrl = 'https://www.metacritic.com/search/';
      const searchUrl = baseUrl + encodeURIComponent(this.prefix) + '/';
      window.open(searchUrl, '_blank');
    },
    onImageChange(e) {
      const file = e.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (event) => {
          const img = new Image();
          img.onload = () => {
            if (img.width !== 1024 || img.height !== 576) {
              this.imgSrc = event.target.result;
              this.showCropper = true;
            } else {
              this.croppedImage = event.target.result;
            }
          };
          img.src = event.target.result;
        };
        reader.readAsDataURL(file);
      }
    },
    cropImage() {
      const canvas = this.$refs.cropper.getCroppedCanvas({ width: 1024, height: 576 });
      this.croppedImage = canvas.toDataURL();
      this.showCropper = false;
      this.$refs.fileInput.value = null; // Clear the file input
    },
    cancelCrop() {
      this.imgSrc = null;
      this.showCropper = false;
      this.$refs.fileInput.value = null; // Clear the file input
    }
  },
};
</script>


<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #f4f4f4;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.top-link {
  margin-bottom: 20px;
  color: blue;
  text-decoration: none;
  font-size: 18px;
}

.rating-section {
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 300px;
  margin-bottom: 20px;
}

.slider-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 10px 0;
}
hr {
  border: 0;
  height: 1px;
  width: 25%;
  background-color: #274c99; /* Light gray color */
  margin: 10px 0; /* Add space above and below the line */
}
.slider-container input[type="range"] {
  flex: 1;
  margin: 0 10px;
}

/* Styling for the rating number input */
.slider-container .rating-input {
  width: 50px;
  margin-right: 10px;
  padding: 5px;
  border: none;
  border-bottom: 2px solid #3163bb;
  background-color: transparent;
  outline: none;
  transition: border-color 0.3s ease;
  text-align: center;
  color: #3163bb;
  font-weight: bold;
}

.slider-container .rating-input:focus {
  border-bottom-color: darkblue;
  box-shadow: 0 2px 4px rgba(0, 0, 255, 0.2);
}

/* Styling for other input fields */
.input-style {
  margin: 10px 0;
  padding: 10px;
  border: 2px solid transparent;
  border-radius: 5px;
  background-color: #fff;
  outline: none;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
  width: 100%;
  max-width: 300px;
  font-size: 16px;
}

.input-style:focus {
  border-color: #3163bb;
  box-shadow: 0 4px 6px rgba(0, 0, 255, 0.2);
}

.button-style {
  margin: 10px 0;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #3163bb;
  color: #fff;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.button-style:hover {
  background-color: #274c99;
}
.articles-button {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  padding: 10px 20px;
  background-color: #3163bb;
  color: white;
  text-decoration: none;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}

.articles-button:hover {
  background-color: #274c99;  /* Slightly darker shade for hover effect */
}


.button-crop {
  display: flex;
  justify-content: center;
  margin: 10px 0;
}

.button-style + .button-style {
  margin-left: 10px;
}

.button-avatar {
  width: 40px;   /* You can adjust this based on your preference */
  height: 40px;
  border-radius: 50%;  /* Makes the image circular */
  margin-right: 10px;  /* Space between the image and the text */
  object-fit: cover;   /* Makes sure the image covers the entire space without being distorted */
}
footer {
  margin-top: 40px;  /* Adds spacing between the footer and the other elements */
  padding: 10px;
  border-top: 1px solid #ddd;  /* Optional: Adds a subtle line above the footer for visual separation */
  width: 100%;
  text-align: center;  /* Centers the text/link inside the footer */
}

.footer-link {
  color: #3163bb;
  text-decoration: none;
  transition: color 0.3s ease;
  position: relative;
  display: inline-block; /* Needed to position the pseudo-element correctly */
}

.footer-link::before {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0; /* Initial width */
  height: 2px; /* Thickness of underline */
  background: #274c99; /* Color of underline */
  transition: width 0.3s ease; /* Transition for the underline */
}

.footer-link:hover {
  color: #274c99;
}

.footer-link:hover::before {
  width: 100%; /* Full width on hover */
}
.metacritic-button {
  background: #3163bb;
  color: white;
  border: none;
  cursor: pointer;
  margin-left: 10px;
  padding: 10px 15px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  font-size: 16px;
  transition: background-color 0.3s ease;
}

.metacritic-button:hover {
  background-color: #274c99;
}

.metacritic-button img {
  width: 24px; /* Adjust size as needed */
  height: 24px; /* Adjust size as needed */
  margin-right: 10px;
}

.cropper-container {
  max-width: 50%; 
  margin: 0 auto; 
  overflow: hidden; 
}
.cropper-container img {
  max-width: 100%; 
}
</style>
