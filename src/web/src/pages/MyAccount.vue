<template>
    <div class="account-page">
      <h1>My Account</h1>
      <p>Name: {{ user.name }}</p>
      <p>Email: {{ user.email }}</p>
      <!-- more to come.. -->
    </div>
  </template>
  
  <script>
  import { mapGetters } from "vuex";
  
  export default {
    name: "MyAccount",
    computed: {
      ...mapGetters({
        user: "CURRENT_USER_INFO",
      }),
    },
  };
  </script>
  
  <style scoped>
  .account-page {
    max-width: 600px;
    margin: auto;
    padding: 20px;
    text-align: center;
  }
  </style>
  