<template>
    <div :class="['account-container', darkMode ? 'dark-mode' : 'light-mode']">
      <b-card class="account-card">
        <h2 class="text-center">My Account</h2>
        <b-row class="justify-content-center">
          <b-col md="4" class="text-center">
            <b-avatar size="100px" variant="primary">{{ userInitials }}</b-avatar>
            <p class="username mt-2">{{ user.name }}</p>
            <b-badge variant="info">{{ user.role || "Student" }}</b-badge>
          </b-col>
          <b-col md="8">
            <b-list-group class="info-list">
              <b-list-group-item class="info-item">
                <b-icon icon="envelope" class="mr-2"></b-icon>
                <strong>Email:</strong> {{ user.email }}
              </b-list-group-item>
              <b-list-group-item class="info-item">
                <b-icon icon="phone" class="mr-2"></b-icon>
                <strong>Phone:</strong> {{ user.phone || "Not provided" }}
              </b-list-group-item>
              <b-list-group-item class="info-item">
                <b-icon icon="calendar" class="mr-2"></b-icon>
                <strong>Joined:</strong> {{ user.joinedDate || "N/A" }}
              </b-list-group-item>
              <b-list-group-item class="info-item">
                <b-icon icon="briefcase" class="mr-2"></b-icon>
                <strong>Role:</strong> {{ user.role || "Student" }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
        </b-row>
        <div class="text-center mt-4">
          <b-button variant="primary" @click="editProfile">Edit Profile</b-button>
          <b-button variant="danger" class="ml-2" @click="logOut">Log Out</b-button>
        </div>
      </b-card>
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
        darkMode: "darkModeState", 
      }),
      userInitials() {
        if (this.user && this.user.name) {
          return this.user.name
            .split(" ")
            .map((n) => n[0])
            .join("")
            .toUpperCase();
        }
        return "?";
      },
    },
    methods: {
      ...mapActions([userTypes.actions.LOAD_SESSION_COOKIE]),
      async logOut() {
        try {
          await this.$store.dispatch(userTypes.actions.LOGOUT);
          this.$router.push("/");
        } catch (err) {
          console.error("Logout error:", err);
        }
      },
      editProfile() {
        alert("Edit profile feature coming soon!");
      },
    },
    async mounted() {
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
  /* Default Light Mode Styles */
  .account-container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: var(--background-light);
    padding: 20px;
    transition: background 0.3s ease-in-out;
  }
  
  .account-card {
    width: 600px;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    background: var(--card-light);
    color: var(--text-light);
    transition: background 0.3s ease-in-out, color 0.3s ease-in-out;
  }
  
  .username {
    font-size: 1.4rem;
    font-weight: bold;
  }
  
  .text-center {
    text-align: center;
  }
  
  /* Light Mode Variables */
  .light-mode {
    --background-light: #f8f9fa;
    --card-light: #ffffff;
    --text-light: #333;
    --info-background: #f1f3f5;
    --info-border: #d1d1d1;
  }
  
  /* Dark Mode Variables */
  .dark-mode {
    --background-light: hsl(225, 15%, 16%);
    --card-light: hsl(225, 15%, 16%);
    --text-light: #ffffff;
    --info-background: hsl(225, 25%, 20%);
    --info-border: hsl(225, 30%, 25%);
  }
  
  /* Card theme to columns */
  .info-list {
    border-radius: 8px;
    overflow: hidden;
  }
  
  .info-item {
    background: var(--info-background);
    color: var(--text-light);
    border-color: var(--info-border);
    transition: background 0.3s ease-in-out, color 0.3s ease-in-out;
  }
  </style>
  