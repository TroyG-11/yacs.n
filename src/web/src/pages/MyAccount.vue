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

          <!-- Profile Section -->
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
                      <b-icon icon="school" class="mr-2"></b-icon>
                      <strong>Major:</strong> {{ user.major || "Not specified" }}
                    </b-list-group-item>
                    <b-list-group-item class="info-item">
                      <b-icon icon="calendar" class="mr-2"></b-icon>
                      <strong>Year:</strong> {{ user.year || "N/A" }}
                    </b-list-group-item>
                  </b-list-group>
                </b-col>
              </b-row>
              <div class="text-center mt-4">
                <b-button variant="primary" @click="openEditModal">Edit Profile</b-button>
                <b-button variant="danger" class="ml-2" @click="logOut">Log Out</b-button>
              </div>
            </b-card>
          </div>

          <!-- Student Information Section -->
          <div v-if="activeSection === 'Student Information'">
            <b-card class="profile-card">
              <b-list-group class="info-list">
                <b-list-group-item class="info-item">
                  <b-icon icon="mortarboard" class="mr-2"></b-icon>
                  <strong>Degree:</strong> {{ user.degree || "N/A" }}
                </b-list-group-item>
              </b-list-group>
            </b-card>
          </div>

          <!-- Settings Section -->
          <div v-if="activeSection === 'Settings'">
            <b-card class="profile-card">
              <b-list-group class="info-list">
                <b-list-group-item class="info-item">
                  <b-icon icon="envelope" class="mr-2"></b-icon>
                  <strong>Email:</strong> {{ user.email }}
                </b-list-group-item>
              </b-list-group>
            </b-card>
          </div>
        </b-col>
      </b-row>
    </b-container>
    
    <!-- Edit Profile Modal -->
    <b-modal v-model="showEditModal" title="Edit Profile" hide-footer>
      <b-form @submit.prevent="saveProfile">
        <b-form-group label="Major:" label-for="major">
          <b-form-input id="major" v-model="editableMajor"></b-form-input>
        </b-form-group>
        <b-form-group label="Year:" label-for="year">
          <b-form-input
            id="year"
            type="number"
            v-model.number="editableYear"
            :min="currentYear"
          ></b-form-input>
           </b-form-group>
        <div class="text-right">
          <b-button variant="secondary" @click="showEditModal = false">Cancel</b-button>
          <b-button variant="primary" type="submit" class="ml-2">Save</b-button>
        </div>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
import { mapGetters, mapActions, mapMutations } from "vuex";
import { userTypes } from "../store/modules/user";

export default {
  name: "MyAccount",
  data() {
    return {
      activeSection: "Profile",
      showEditModal: false,
      editableMajor: "",
      editableYear: "",
      currentYear: new Date().getFullYear(),
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
    ...mapMutations({
      updateUserInfo: userTypes.mutations.SET_USER_INFO,
    }),
    async logOut() {
    try {
      await this.$store.dispatch(userTypes.actions.LOGOUT);
      this.$router.replace("/");
    } catch (err) {
      console.error("Logout error:", err);
    }
  },
    loadUserFromStorage() {
    const storedUser = localStorage.getItem("userProfile");
    if (storedUser) {
      this.updateUserInfo(JSON.parse(storedUser));
    }
  },
    openEditModal() {
      this.editableMajor = this.user.major || "";
      this.editableYear = this.user.year || this.currentYear;
      this.showEditModal = true;
    },
    async saveProfile() {
  if (this.editableYear < this.currentYear) {
    alert("Year must be the current year or later.");
    return;
  }

  const updatedUser = {
    name: this.user.name || "",
    sessionID: this.$store.state.sessionID || "",  // Make sure sessionID exists
    email: this.user.email || "",
    phone: this.user.phone || "",  
    newPassword: "", // Provide an empty password if not changing it
    major: this.editableMajor,
    degree: this.user.degree || "",
  };

  console.log("Sending API request with data:", updatedUser); // Debugging

  try {
    const response = await fetch("/api/user", {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(updatedUser),
    });

    const responseData = await response.json();
    console.log("API response:", responseData); // Debugging

    if (!response.ok) {
      alert("Failed to update profile: " + (responseData.message || "Unknown error"));
      return;
    }

    this.updateUserInfo(updatedUser);
    localStorage.setItem("userProfile", JSON.stringify(updatedUser));
    this.showEditModal = false;
  } catch (error) {
    console.error("Error updating profile:", error);
    alert("Something went wrong.");
  }
}
  },
  async mounted() {
    this.loadUserFromStorage();

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
