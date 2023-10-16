<template>
  <div id="app">
    <div class="container">
      <a href="https://www.gameworld.gr/jreviews/my-listings?user=54184" class="articles-button">
        <img src="https://www.gameworld.gr/images/avatar/5763925d123ca66c2e226227.jpg" alt="Avatar" class="button-avatar" />
        Read articles by me
      </a>


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

      <h3>Screenshot tool for reviews</h3>
      <input class="input-style" type="text" v-model="prefix" placeholder="Enter Game Name" />
      <div class="metacritic-section">
        <div v-if="isLoading">Fetching Metascore...</div>
        <div v-else-if="metascore">{{ metascore }}</div>
      </div>
      <input class="input-style" type="file" multiple @change="handleFiles" />
      <button class="button-style" @click="processImages">Resize and Add Watermark</button>

      <footer>
        <a href="https://www.miketsak.gr" class="footer-link">
          Made by miketsak.gr
        </a>
      </footer>

    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      files: [],
      prefix: "",
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
  background-color: darkblue;
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
  color: blue;
  text-decoration: none;
  transition: color 0.3s ease;
}

.footer-link:hover {
  color: darkblue;  /* Darkens the link color on hover for a subtle effect */
}
.metacritic-section {
    margin-top: 20px;
    font-size: 18px;
    font-weight: bold;
  }

</style>
