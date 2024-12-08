<template>
  <!-- Header with logo and buttons -->
  <div class="header">
    <img src="/images/logo.png" alt="Dyangoo Logo" class="logo">
    <div class="header-buttons">
      <button class="login-btn" @click="redirectToLogin">Login</button>
      <button class="cart-btn" @click="goCart">Shopping Cart</button>
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
        We are known for the most customizable boots in the market. Click on a part of the boot and edit the color.
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
        <button @click="changeFabric('standaard')">Standaard</button>
        <button @click="changeFabric('rubber')">Rubber</button>
        <button @click="changeFabric('wool')">Wool</button>
      </div>

      <p class="product-description">
        Want to match with your friends? Order more than one pair!
      </p>

      <div class="quantity-selector">
        <label for="quantity">Quantity:</label>
        <button @click="quantity = Math.max(1, quantity - 1)">-</button>
        <span>{{ quantity }}</span>
        <button @click="quantity = Math.min(10, quantity + 1)">+</button>
      </div>

      <h3 class="price">{{price}} x {{ quantity }} = <p>{{price * quantity }}</p></h3>

      <button class="buy-now-button" @click="submitOrder">Add to Cart</button>

      <div class="shipping-info">
        <p>Free shipping on all orders over $100!</p>
      </div>
      <div class="reviews">
        <span>★★★★★</span> <span>(50 Reviews)</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, inject } from 'vue';
import { initializeCanvas, updateShoeColor, updateShoeFabric } from '../configurator/three-configurator';
import axios from 'axios';

const price = 149.99;
const quantity = ref(1);

// Color setup
const shoeColors = ref({
  laces: '#ffffff',
  inside: '#ff0000',
  outside_1: '#000000',
  outside_2: '#ff0000',
  outside_3: '#ff0000',
  sole_top: '#888888',
  sole_bottom: '#000000',
});

// The selected color to update the shoe
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
  black: '#000000',
};

// Fabric setup
const currentFabric = ref('standaard');
const fabrics = {
  standaard: '/textures/normal_t_white.jpg',
  rubber: '/textures/soft.png',
  wool: '/textures/cuchon.png',
};

function changeColor(color) {
  updateShoeColor(color);  // Update shoe color in Three.js
}

//go to cart page
function goCart(){
  window.location.href = '/cart';
}

// Fabric change handler
function changeFabric(fabric) {
  currentFabric.value = fabric;
  updateShoeFabric(fabrics[fabric]);  // Update shoe fabric in Three.js
  
  // Log the fabric update (if needed)
  console.log('Updated fabric:', fabric);
}

// Mount Three.js canvas
onMounted(() => {
  const canvas = document.querySelector('#threeCanvas');
  initializeCanvas(canvas); // Initialize the 3D scene

  // Listen for the colors from Three.js
  window.updateColorsFromThree = (colors) => {
    shoeColors.value = colors;
    //console.log('Updated shoe colors from Three.js:', shoeColors.value);
  };

  // Inject and set up WebSocket
  const socket = inject('$socket');

  // Listen for real-time updates (e.g., new offers)
  socket.on('newOffer', (offer) => {
    console.log('New offer received:', offer);
    // Display the new offer (optional: update UI with offer details)
  });

  // Listen for cart updates (if applicable)
  socket.on('cartUpdated', (updatedCart) => {
    console.log('Cart updated:', updatedCart);
    // Update cart UI with the updated cart
  });
});

// Submit order handler
async function submitOrder() {
  try {
    const order = {
      productId: 'dyangoo-boots',
      color: {
        inside: shoeColors.value.inside || '#ffffff',
        laces: shoeColors.value.laces || '#000000',
        outside_1: shoeColors.value.outside_1 || '#888888',
        outside_2: shoeColors.value.outside_2 || '#888888',
        outside_3: shoeColors.value.outside_3 || '#888888',
        sole_top: shoeColors.value.sole_top || '#000000',
        sole_bottom: shoeColors.value.sole_bottom || '#000000',
      },
      fabric: currentFabric.value,
      quantity: quantity.value,
      totalPrice: price * quantity.value,  // Make sure price is defined somewhere
      customerName: 'Yann Tagakou', // Replace with actual input
      customerEmail: 'Yan@thomasmore.be', // Replace with actual input
      shippingAddress: 'Leuvense steenweg 206, 3000 Leuven', // Replace with actual input
      send: 'new',
    };

    const response = await axios.post('http://localhost:5001/api/orders', order);
    console.log('Order created:', response.data);

    // Emit cart update event after order is placed (for real-time sync)
    const socket = inject('$socket');
    socket.emit('cartUpdated', { action: 'add', updatedCart: response.data });

  } catch (error) {
    console.error('Error submitting order:', error); // Log error details for better debugging
    alert('Failed to submit order. Please try again.');
  }
}

function redirectToLogin() {
  window.location.href = '/login'; // Replace with your login page route
}
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
  background: #E42528; /* Darker shade on hover */
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
  font-size: 16px;
  text-align: left;
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

.price {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
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
  background: #E42528; /* Darker shade on hover */
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  transition: background-color 0.3s;
}

.quantity-btn:hover {
  background: #E42528; /* Darker shade on hover */
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