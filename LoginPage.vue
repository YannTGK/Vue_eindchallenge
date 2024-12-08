<template>
  <!-- Header with logo and buttons -->
  <div class="header">
    <img src="/images/logo.png" alt="Dyangoo Logo" class="logo" @click="redirectToHome">
    <div class="header-buttons">
      <button class="back-btn" @click="goBack">Go Back</button>
    </div>
  </div>

  <!-- Login Form Section -->
  <div class="site">
    <div class="form-container">
      <h2>Welcome Back!</h2>
      <p>Please log in to your account</p>

      <form @submit.prevent="handleLogin">
        <div class="input-group">
          <label for="email">Email:</label>
          <input type="email" id="email" v-model="email" placeholder="email" required />
        </div>

        <div class="input-group">
          <label for="password">Password:</label>
          <input type="password" id="password" v-model="password" placeholder="password" required />
        </div>

        <button type="submit" class="submit-btn">Login</button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const email = ref('');
const password = ref('');

// Check if the user is already logged in when the component is mounted
onMounted(() => {
  const token = localStorage.getItem('authToken');
  if (token) {
    // If a token exists, redirect to the admin page
    window.location.href = '/admin'; // Or use Vue Router to navigate
  }
});

function handleLogin() {
  console.log('Email:', email.value); // Check if email is filled
  console.log('Password:', password.value); // Check if password is filled
  
  // Check if email and password are not empty
  if (!email.value || !password.value) {
    console.error('Both email and password are required!');
    return;
  }

  axios.post('http://localhost:5001/api/auth/login', { email: email.value, password: password.value })
    .then(response => {
      const token = response.data.token;
      localStorage.setItem('authToken', token); // Save the token in localStorage
      console.log('Logged in successfully');
      // Redirect to the admin page
      window.location.href = '/admin'; // Or use Vue Router to navigate
    })
    .catch(error => {
      if (error.response) {
        // Server responded with an error
        console.error('Login error:', error.response.data.message);
      } else if (error.request) {
        // Request was made but no response was received
        console.error('Login error: No response from the server');
      } else {
        // Something went wrong in setting up the request
        console.error('Login error:', error.message);
      }
    });
}

function goBack() {
  window.history.back(); // Go back to the previous page
}

function redirectToHome() {
  window.location.href = '/'; // Replace with your login page route
}
</script>
<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
}

button {
  text-transform: uppercase;
}

.header {
  display: flex;
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
  justify-content: center;
  align-items: center;
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

.form-container {
  width: 100%;
  max-width: 500px; /* Make the form wider */
  background-color: white;
  padding: 40px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  text-align: center;
}

h2 {
  margin-bottom: 15px;
  font-size: 24px;
  color: #333;
}

p {
  font-size: 16px;
  margin-bottom: 20px;
  color: #666;
}

form {
  display: flex;
  flex-direction: column;
}

.input-group {
  margin-bottom: 20px;
}

label {
  margin-bottom: 5px;
  font-size: 14px;
  text-align: left;
  color: #333;
}

input {
  padding: 12px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 100%;
}

.submit-btn {
  padding: 12px 40px;
  font-size: 16px;
  font-weight: bold;
  text-transform: uppercase;
  background: linear-gradient(135deg, #ff4b5c, #d03f48);
  color: white;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: background-color 0.3s ease-in-out;
}

.submit-btn:hover {
  background: #E42528; /* Darker shade on hover */
}

.back-btn {
  background: none;
  border: none;
  color: #fff;
  font-size: 16px;
  cursor: pointer;
  font-weight: bold;
  transition: color 0.3s ease-in-out;
}

.back-btn:hover {
  background-color: #ff4b5c;
}
</style>