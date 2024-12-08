<template>
    <div class="orders-page">
      <!-- Header -->
      <div class="header">
        <img src="/images/logo.png" alt="Dyangoo Logo" @click="home" class="logo">
        <div class="header-buttons">
          <button class="logout-btn" @click="redirectToLogin">Logout</button>
          <button class="home-btn" @click="home">Home</button>
        </div>
      </div>
  
      <!-- Main Content -->
      <div class="orders-container">
        <h2 class="orders-title">Your Orders</h2>
  
        <div v-if="orders.length === 0" class="no-orders-message">
          Your cart is empty.
        </div>
  
        <div v-for="order in orders" :key="order._id" class="order-card">
          <div class="order-header">
            <h3>Order #{{ order._id }}</h3>
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
            <button class="delete-btn" @click="deleteOrder(order._id)">Remove Order</button>
          </div>
        </div>
      </div>
  
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  import axios from 'axios';
  
  const orders = ref([]);
  const socket = ref(null); // WebSocket client
  
  // WebSocket Event Handlers
  const handleNewOrder = (newOrder) => {
    orders.value.push(newOrder);
  };
  
  const handleUpdatedOrder = (updatedOrder) => {
    const index = orders.value.findIndex(order => order._id === updatedOrder._id);
    if (index !== -1) {
      orders.value[index] = updatedOrder;
    }
  };
  
  onMounted(() => {
    const token = localStorage.getItem('authToken');
    if (!token) {
      window.location.href = '/login';
    }
  
    // Fetch existing orders on initial load
    axios.get('http://localhost:5001/api/orders', {
      headers: { Authorization: `Bearer ${token}` }
    })
    .then(response => {
      orders.value = response.data;
    })
    .catch(error => {
      console.error('Error fetching orders:', error.response?.data?.message);
    });
  
    // Initialize WebSocket
    socket.value = new WebSocket('ws://localhost:5001'); // Assuming WebSocket is running on this URL
  
    socket.value.onopen = () => {
      console.log('WebSocket connected');
    };
  
    socket.value.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.type === 'new-order') {
        handleNewOrder(data.order);
      } else if (data.type === 'update-order') {
        handleUpdatedOrder(data.order);
      }
    };
  
    socket.value.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  
    socket.value.onclose = () => {
      console.log('WebSocket connection closed');
    };
  });
  
  // Helper functions and button actions
  const home = () => window.location.href = '/';
  const redirectToLogin = () => {
    localStorage.removeItem('authToken');
    window.location.href = '/login';
  };
  
  const formatColor = (color) => color.startsWith('#') ? color : `#${color}`;
  
  const formatColorKey = (key) => key.charAt(0).toUpperCase() + key.slice(1).replace('_', ' ');
  
  const confirmOrder = (orderId) => {
    axios.patch(`http://localhost:5001/api/orders/${orderId}`, { send: 'On its way' })
      .then(() => {
        const order = orders.value.find(o => o._id === orderId);
        if (order) order.send = 'On its way';
      })
      .catch(error => console.error('Error updating order:', error.response?.data?.message));
  };
  
  const deleteOrder = (orderId) => {
    axios.delete(`http://localhost:5001/api/orders/${orderId}`)
      .then(() => {
        orders.value = orders.value.filter(order => order._id !== orderId);
      })
      .catch(error => console.error('Error deleting order:', error.response?.data?.message));
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
    background-color: #2a2a2a;
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
    background: #e42528;
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
  }
  
  .order-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 15px;
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
  </style>