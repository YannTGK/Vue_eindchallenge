<template>
  <!-- Header with logo and buttons -->
  <div class="header">
    <img src="/images/logo.png" alt="Dyangoo Logo" class="logo">
    <div class="header-buttons">
      <button class="login-btn">Login</button>
      <button class="cart-btn">Shopping Cart ({{ quantity }})</button>
    </div>
  </div>

  <div class="site">
    <div class="three-container">
      <canvas id="threeCanvas"></canvas>
    </div>
    <div class="right-panel">
      
      <div class="product-images">
        <img src="/images/logo.png" alt="Dyangoo Boots" class="product-thumbnail">
      </div>
      
      <p class="product-description">
        Everybody knows that the best way to make a statement is with a pair of Dyangoo boots.
        <br><br>
        We are know for the most Customizable boots in the market. Click on a part of the boot and edit the color.
      </p>

      <div class="colorButtons">
        <button v-for="(color, name) in colors" :key="name" @click="changeColor(name)" :style="{ backgroundColor: color }">
          {{ name }}
        </button>
      </div>

      <p class="product-description">
        Not only the color should be unique, but by editing the fabric you can make your boots even more special.
        <br><br>
        Nobody in the world will have the same boots as you.
      </p>

      <div class="fabricButtons">
        <button @click="changeFabric('fabric1')">Standaard</button>
        <button @click="changeFabric('fabric2')">Rubber</button>
        <button @click="changeFabric('fabric3')">Wool</button>
      </div>

      <div class="quantity-selector">
        <label for="quantity">Quantity:</label>
        <button @click="quantity = Math.max(1, quantity - 1)">-</button>
        <span>{{ quantity }}</span>
        <button @click="quantity = Math.min(10, quantity + 1)">+</button>
      </div>

      <p class="price">$149.99</p>

      <div class="reviews">
        <span>★★★★★</span> <span>(50 Reviews)</span>
      </div>

      <button class="buy-now-button">Add to Cart</button>

      <div class="cart">
        <p>Items in Cart: {{ quantity }}</p>
        <button class="view-cart">View Cart</button>
      </div>

      <div class="shipping-info">
        <p>Free shipping on all orders over $100!</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
// Import functions from the Three.js configurator
import { initializeCanvas, updateShoeColor, updateShoeFabric } from '../three-configurator';

// Color setup
const currentColor = ref('none');
const colors = {
  red: '#ff4b5c',
  blue: '#3b82f6',
  yellow: '#fbbf24',
  purple: '#8b5cf6',
  orange: '#fb923c',
  pink: '#ec4899',
  teal: '#14b8a6',
  gray: '#888888',
  Black: '#000000',
};

// Fabric setup
const currentFabric = ref('fabric1');
const fabrics = {
  fabric1: '/textures/normal_t_white.jpg',
  fabric2: '/textures/soft.png',
  fabric3: '/textures/cuchon.png',
};

// Quantity setup
const quantity = ref(1);

function changeColor(color) {
  currentColor.value = color;
  updateShoeColor(color);  // Update shoe color
}

function changeFabric(fabric) {
  currentFabric.value = fabric;
  updateShoeFabric(fabrics[fabric]);  // Update shoe fabric texture
}

onMounted(() => {
  const canvas = document.querySelector('#threeCanvas');
  initializeCanvas(canvas); // Initialize the 3D scene
});
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
}
.header {
  display: flex;
  position: absolute;
  justify-content: space-between;
  align-items: center;
  padding: 16px 40px;
  width: 100%;
  background-color: #333;
  color: white;
}

.site {
  display: flex;
  height: 100vh; /* Full viewport height */
}

.logo {
  height: 50px; /* Adjust the logo size */
}

.header-buttons button {
  margin-left: 20px;
  padding: 10px 15px;
  font-size: 14px;
  font-weight: bold;
  background-color: #ff4b5c;
  color: white;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.header-buttons button:hover {
  background-color: #0056b3; /* Darker shade on hover */
}

.three-container {
  flex: 1; /* Take up available space */
  margin-top: 82px; /* Match the header height */
  height: calc(100vh - 82px); /* Subtract header height */
  overflow: hidden;
  position: relative; /* Prevent overlapping issues */
}

#threeCanvas {
  width: 100%;
  height: 100%;
  display: block; /* Avoid unwanted scrollbars */
  position: relative;
}

.product-description {
  font-size: 24px;
  text-align: center;
}

.right-panel {
  width: 50%;
  background-color: #f9f9f9;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 400px 50px 50px 50px;
  gap: 30px;
  overflow-y: auto; /* Allow scrolling for content in the right panel if it overflows */
}

.product-thumbnail {
  width: 400px; /* Adjust the image size */
  object-fit: contain; /* Prevent image distortion */
}
.colorButtons {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  justify-content: center;
}

.fabricButtons {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  justify-content: center;
}

.fabricButtons > button {
  background-color: #007bff;
}

button {
  padding: 18px 40px;
  font-size: 16px;
  font-weight: bold;
  text-transform: uppercase;
  border-radius: 50px;
  border: none;
  cursor: pointer;
  transition: 0.3s ease-in-out;
  color: white;
}

.quantity-selector {
  display: flex;
  align-items: center;
  gap: 10px;
}

.quantity-btn {
  padding: 5px 10px; /* Smaller padding */
  font-size: 14px; /* Smaller font size */
  background-color: #007bff; /* Change background color if needed */
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  transition: background-color 0.3s;
}

.quantity-btn:hover {
  background-color: #0056b3; /* Darker shade on hover */
}

.quantity-selector span {
  font-size: 16px; /* Adjust the font size of the quantity number */
}

.buy-now-button {
  padding: 18px 60px;
  font-size: 18px;
  font-weight: bold;
  background: linear-gradient(135deg, #ff4b5c, #d03f48);
  border-radius: 50px;
  color: white;
  border: none;
  cursor: pointer;
}
</style>