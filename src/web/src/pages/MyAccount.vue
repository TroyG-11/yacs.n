<template>
    <div :class="['account-container', darkMode ? 'dark-mode' : 'light-mode']">
      <b-container fluid>
        <b-row>
          <!-- Sidebar Navigation -->
          <b-col md="3" class="sidebar">
            <h3 class="sidebar-title">My Account</h3>
            <b-list-group class="sidebar-menu">
              <b-list-group-item
                v-for="(section, index) in sections"
                :key="index"
                :class="{ active: activeSection === section.name }"
                @click="activeSection = section.name"
              >
                {{ section.label }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
  
          <!-- Main Content -->
          <b-col md="9" class="content-area">
            <h2>{{ activeSection }}</h2>
            <div v-if="activeSection === 'Profile'">
              <b-card class="profile-card">
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
                        <b-icon icon="major" class="mr-2"></b-icon>
                        <strong>Major:</strong> {{ user.major}}
                      </b-list-group-item>
                      <b-list-group-item class="info-item">
                        <b-icon icon="degree" class="mr-2"></b-icon>
                        <strong>Degree:</strong> {{ user.degree}}
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
  
            <div v-if="activeSection === 'Student Information'">
              <b-card class="profile-card">
                <p>No information found.</p>
              </b-card>
            </div>
  
            <div v-if="activeSection === 'Settings'">
              <b-card class="profile-card">
                <p>No settings found.</p>
              </b-card>
            </div>
          </b-col>
        </b-row>
      </b-container>
    </div>
  </template>
  
  <script>
  import { mapGetters, mapActions } from "vuex";
  import { userTypes } from "../store/modules/user";
  
  export default {
    name: "MyAccount",
    data() {
      return {
        activeSection: "Profile",
        sections: [
          { name: "Profile", label: "Profile" },
          { name: "Student Information", label: "Student Information" },
          { name: "Settings", label: "Settings" },
        ],
      };
    },
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
        alert("Edit profile feature soon...");
      },
    },
    async mounted() {
      if (!this.user) {
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
  /* Light & Dark Mode Variables */
  .light-mode {
    --background-color: #f8f9fa;
    --sidebar-background: #ffffff;
    --sidebar-text: #333;
    --content-background: #ffffff;
    --card-background: #ffffff;
    --text-color: #333;
    --border-color: #e0e0e0;
  }
  
  .dark-mode {
    --background-color: #121212;
    --sidebar-background: #1e1e1e;
    --sidebar-text: #ffffff;
    --content-background: #1e1e1e;
    --card-background: #252525;
    --text-color: #ffffff;
    --border-color: #444;
  }
  
  /* Main Container */
  .account-container {
    display: flex;
    justify-content: center;
    min-height: 100vh;
    background: var(--background-color);
    padding: 20px;
    transition: background 0.3s ease-in-out;
  }
  
  /* Sidebar */
  .sidebar {
    background: var(--sidebar-background);
    padding: 20px;
    border-radius: 10px;
    height: fit-content;
  }
  
  .sidebar-title {
    font-size: 1.2rem;
    font-weight: bold;
    color: var(--text-color);
  }
  
  .sidebar-menu {
    margin-top: 10px;
  }
  
  .sidebar-menu .list-group-item {
    background: var(--sidebar-background);
    color: var(--sidebar-text);
    border: 1px solid var(--border-color);
    cursor: pointer;
    transition: all 0.3s ease-in-out;
  }
  
  .sidebar-menu .list-group-item:hover,
  .sidebar-menu .list-group-item.active {
    background: var(--card-background);
    color: var(--text-color);
  }
  
  /* Content Area */
  .content-area {
    background: var(--content-background);
    padding: 20px;
    border-radius: 10px;
    color: var(--text-color);
  }
  
  /* Profile Card */
  .profile-card {
    background: var(--card-background);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    color: var(--text-color);
  }
  
  /* Info List */
  .info-list {
    border-radius: 8px;
    overflow: hidden;
  }
  
  .info-item {
    background: var(--card-background);
    color: var(--text-color);
    border-color: var(--border-color);
    transition: background 0.3s ease-in-out, color 0.3s ease-in-out;
  }
  
  /* Username */
  .username {
    font-size: 1.4rem;
    font-weight: bold;
  }
  </style>
  