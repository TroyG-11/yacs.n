<template>
    <div class="account-page">
      <h1>My Account</h1>
      <p v-if="user">Name: {{ user.name }}</p>
      <p v-if="user">Email: {{ user.email }}</p>
      <p v-else>Loading user information...</p> <!-- Debugging message -->
    </div>
  </template>
  
  <script>
  import { mapGetters, mapActions } from "vuex";
  import { userTypes } from "../store/modules/user";
  
  export default {
    name: "MyAccount",
    computed: {
      ...mapGetters({
        user: userTypes.getters.CURRENT_USER_INFO,
        isLoggedIn: userTypes.getters.IS_LOGGED_IN,
      }),
    },
    methods: {
    ...mapActions([userTypes.actions.LOAD_SESSION_COOKIE]),  
  },
    async mounted() {
      console.log("User in MyAccount.vue:", this.user); // Debugging
  
      // If user is not loaded, attempt to load it from the store
      if (!this.user) {
        console.log("Fetching user data...");
        try {
          await this.$store.dispatch(userTypes.actions.LOAD_SESSION_COOKIE);
        } catch (err) {
          console.error("Error loading user:", err);
        }
      }
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
  