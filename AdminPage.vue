<template>
  <div class="orders-page">
    <!-- Header -->
    <div class="header">
      <img src="/images/logo.png" alt="Dyangoo Logo"  @click="home" class="logo">
      <div class="header-buttons">
        <button class="login-btn" @click="redirectToLogin">Logout</button>
        <button class="cart-btn" @click="home">Home</button>
      </div>
    </div>

    <!-- Main Content -->
    <div class="orders-container">
      <h2 class="orders-title">All Orders ({{ orders.length }})</h2>

      <div v-if="orders.length === 0" class="no-orders-message">
        No orders yet.
      </div>

      <div v-for="order in orders" :key="order._id" class="order-card">
        <div class="order-header">
          <h3>Order ID: {{ order._id }}</h3>
          <span 
            class="order-status" 
            :style="{ backgroundColor: order.send === 'On its way' ? '#4caf50' : '#ff4b5c' }">
            {{ order.send || 'Pending' }}
          </span>
        </div>

        <div class="order-details">
          <div class="order-info">
            <strong>Customer:</strong> {{ order.customerName }} ({{ order.customerEmail }})
          </div>
          <div class="order-info">
            <strong>Shipping Address:</strong> {{ order.shippingAddress }}
          </div>
          <div class="order-info">
            <strong>Quantity:</strong> {{ order.quantity }}
          </div>
          <div class="order-info">
            <strong>Total Price:</strong> €{{ order.totalPrice }}
          </div>
          <div class="order-info">
            <strong>Fabric:</strong> ({{ order.fabric }})
          </div>
          <h4>Shoe Colors:</h4>
        </div>

        <!-- Shoe Colors Section -->
        <div class="order-colors">
          <div class="color-item" v-for="(color, key) in order.color" :key="key">
            <strong>{{ formatColorKey(key) }} — {{ color }}:</strong>
            <div class="color-swatch" :style="{ backgroundColor: formatColor(color) }"></div>
          </div>
        </div>

        <!-- Buttons Section -->
        <div class="order-actions">
          <button class="confirm-btn" @click="confirmOrder(order._id)">Mark as Sent</button>
          <button class="delete-btn" @click="deleteOrder(order._id)">Delete Order</button>
        </div>
      </div>
    </div>

    <!-- Delete Confirmation Banner -->
    <div v-if="showDeleteBanner" class="delete-banner">
      Order deleted successfully.
    </div>
     <!-- Delete Confirmation Banner -->
     <div v-if="showConfirmBanner" class="confirm-banner">
      Order deleted successfully.
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const orders = ref([]);
const showDeleteBanner = ref(false);
const showConfirmBanner = ref(false);

// On mounted, establish WebSocket connection and listen for order updates
onMounted(() => {
  const token = localStorage.getItem('authToken');
  if (!token) {
    window.location.href = '/login'; // Redirect to login if no token found
  }

  // Fetch orders
  axios.get('http://localhost:5001/api/orders', {
    headers: { Authorization: `Bearer ${token}` }
  })
  .then(response => {
    orders.value = response.data;
  })
  .catch(error => {
    console.error('Error fetching orders:', error.response?.data?.message);
  });

  // Listen for real-time updates from the WebSocket server
  const socket = inject('$socket');
  socket.on('orderStatusUpdated', (updatedOrder) => {
    const index = orders.value.findIndex(o => o._id === updatedOrder._id);
    if (index !== -1) {
      orders.value[index] = updatedOrder; // Update the order with new status
    }
  });
});

// Helper function to format the color (add '#' if missing)
const formatColor = (color) => {
  return color.startsWith('#') ? color : `#${color}`;
};

// Mark order as "On its way"
const confirmOrder = (orderId) => {
  axios.patch(`http://localhost:5001/api/orders/${orderId}`, { send: 'On its way' })
    .then(() => {
      const socket = inject('$socket');
      socket.emit('orderStatusUpdated', { orderId, status: 'On its way' });

      // Update order locally
      const order = orders.value.find(o => o._id === orderId);
      if (order) order.send = 'On its way'; // Update order status locally

      showConfirmBanner.value = true;
      setTimeout(() => {
        showConfirmBanner.value = false;
      }, 3000);
    })
    .catch(error => {
      console.error('Error marking order as "On its way":', error.response?.data?.message || error.message);
    });
};

// Delete order function
const deleteOrder = (orderId) => {
  axios.delete(`http://localhost:5001/api/orders/${orderId}`)
    .then(() => {
      orders.value = orders.value.filter(order => order._id !== orderId);

      showDeleteBanner.value = true;
      setTimeout(() => {
        showDeleteBanner.value = false;
      }, 3000);
    })
    .catch(error => {
      console.error('Error deleting order:', error.response?.data?.message || error.message);
    });
};
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
}

body {
  font-family: 'Arial', sans-serif;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 40px;
  background-color: #333;
  color: white;
}

.logo {
  height: 50px;
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
  background: #E42528;
}

.orders-container {
  padding: 40px;
  background-color: #f4f4f4;
}

.orders-title {
  font-size: 26px;
  font-weight: bold;
  color: #333;
  margin-bottom: 30px;
}

.no-orders-message {
  font-size: 18px;
  color: #888;
  text-align: center;
}

.order-card {
  background-color: white;
  padding: 20px;
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s, box-shadow 0.3s;
}

.order-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.order-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.order-header h3 {
  font-size: 20px;
  color: #333;
}

.order-status {
  background-color: #ff4b5c;
  color: white;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 14px;
  text-transform: capitalize;
}

.order-details {
  margin-top: 20px;
}

.order-info {
  margin-bottom: 12px;
  font-size: 16px;
}

.order-info strong {
  color: #444;
}

.order-colors {
  margin-top: 20px;
  padding-top: 10px;
  border-top: 1px solid #ddd;
  display: flex;
  flex-wrap: wrap;
  justify-content: space;
  align-items: center;
  gap: 12px;
}

.color-item {
  font-size: 16px;
  display: flex;
  align-items: center;
}

.color-swatch {
  width: 40px;
  height: 40px;
  display: inline-block;
  margin-left: 8px;
  border-radius: 50%;
  border: 2px solid #fff;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}

.order-actions {
  margin-top: 20px;
  display: flex;
  gap: 12px;
}

.confirm-btn, .delete-btn {
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.confirm-btn {
  background-color: #4caf50;
  color: white;
}

.confirm-btn:hover {
  background-color: #45a049;
}

.delete-btn {
  background-color: #f44336;
  color: white;
}

.delete-btn:hover {
  background-color: #e53935;
}

/* Delete Banner Style */
.delete-banner {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  background-color: #f44336;
  color: white;
  text-align: center;
  padding: 10px;
  font-size: 16px;
  z-index: 1000;
}
</style>