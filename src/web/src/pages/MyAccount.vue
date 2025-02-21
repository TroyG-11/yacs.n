<template>
    <div class="account-container">
      <b-card class="account-card">
        <h2 class="text-center">My Account</h2>
        <b-row class="justify-content-center">
          <b-col md="4" class="text-center">
            <b-avatar size="100px" variant="primary">{{ userInitials }}</b-avatar>
            <p class="username mt-2">{{ user.name }}</p>
            <b-badge variant="info">{{ user.role || "Student" }}</b-badge>
          </b-col>
          <b-col md="8">
            <b-list-group>
              <b-list-group-item>
                <b-icon icon="envelope" class="mr-2"></b-icon>
                <strong>Email:</strong> {{ user.email }}
              </b-list-group-item>
              <b-list-group-item>
                <b-icon icon="phone" class="mr-2"></b-icon>
                <strong>Phone:</strong> {{ user.phone || "Not provided" }}
              </b-list-group-item>
              <b-list-group-item>
                <b-icon icon="calendar" class="mr-2"></b-icon>
                <strong>Joined:</strong> {{ user.joinedDate || "N/A" }}
              </b-list-group-item>
              <b-list-group-item>
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
        // edit profile for later..
        alert("Edit profile feature soon..");
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
  .account-container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: #f8f9fa;
    padding: 20px;
  }
  
  .account-card {
    width: 600px;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    background: white;
  }
  
  .username {
    font-size: 1.4rem;
    font-weight: bold;
  }
  
  .text-center {
    text-align: center;
  }
  </style>
  