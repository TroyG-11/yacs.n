<template>
  <div :class="['account-container', darkMode ? 'dark-mode' : 'light-mode']">
    <b-container fluid>
      <b-row>
        <!-- Schedules Section -->
        <div v-if="activeSection === 'Schedules'">
          <b-card class="profile-card">
            <h4 class="mb-3">Schedules</h4>

            <!-- Search Bar -->
            <b-form-input
              v-model="searchQuery"
              placeholder="Search schedules..."
              class="mb-3"
            />

            <b-list-group>
              <b-list-group-item
                v-for="(schedule, index) in filteredSchedules"
                :key="index"
                class="d-flex justify-content-between align-items-center"
              >
                <div>
                  <strong>{{ schedule.name }}</strong><br />
                  <small>{{ schedule.courses.join(', ') }}</small><br />
                  <small class="text-muted">{{ schedule.notes || 'No notes' }}</small>
                </div>
                <div>
                  <b-button size="sm" variant="primary" @click="applySchedule(index)">Load</b-button>
                  <b-button size="sm" variant="danger" class="ml-1" @click="deleteSchedule(index)">Delete</b-button>
                  <b-button size="sm" variant="warning" class="ml-1" @click="startRenaming(index)">Rename</b-button>
                  <b-button size="sm" variant="secondary" class="ml-1" @click="moveSchedule(index, -1)" :disabled="index === 0">↑</b-button>
                  <b-button size="sm" variant="secondary" class="ml-1" @click="moveSchedule(index, 1)" :disabled="index === savedSchedules.length - 1">↓</b-button>
                </div>
              </b-list-group-item>
            </b-list-group>

            <div class="mt-4">
              <b-form @submit.prevent="saveNewSchedule">
                <b-form-group label="New Schedule Name">
                  <b-form-input v-model="newScheduleName" placeholder="e.g. Fall 2025 Draft"></b-form-input>
                </b-form-group>
                <b-form-group label="Notes (optional)">
                  <b-form-textarea v-model="newScheduleNotes" rows="2" placeholder="e.g. Avoid Friday classes, focus on CS core."></b-form-textarea>
                </b-form-group>
                <b-button type="submit" variant="success">Save Current Schedule</b-button>
              </b-form>
            </div>

            <div class="mt-4">
              <b-button variant="info" size="sm" class="mr-2" @click="exportSchedules">Export</b-button>
              <b-form-file v-model="importFile" @change="importSchedules" accept=".json" browse-text="Import" size="sm" />
            </div>
          </b-card>
        </div>

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
                    <b-icon icon="person-fill" class="mr-2"></b-icon>
                    <strong>Full Name:</strong> {{ user.name || "N/A" }}
                  </b-list-group-item>
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
              <div class="text-center mt-4">
                <b-button variant="primary" @click="openEditModal">Edit</b-button>
              </div>
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
              <div class="text-center mt-4">
                <b-button variant="primary" @click="openEditModal">Edit</b-button>
              </div>
            </b-card>
          </div>
          <!-- Website Theme Section -->
          <div v-if="activeSection === 'Website Theme'">
            <b-container fluid>
              <b-row class="justify-content-center">
                <b-col md="6">
                  <b-card class="profile-card text-center">
                    <h5>Choose a Theme:</h5>
                    <b-button-group class="mt-3">
                      <b-button
                        variant="outline-secondary"
                        :class="{ active: darkMode === false }"
                        @click="toggleTheme(false)"
                      >
                        Light Mode
                      </b-button>
                      <b-button
                        variant="outline-secondary"
                        :class="{ active: darkMode === true }"
                        @click="toggleTheme(true)"
                      >
                        Dark Mode
                      </b-button>
                      <b-button
                        variant="outline-secondary"
                        :class="{ active: darkMode === null }"
                        @click="followDeviceTheme"
                      >
                        Follow Device
                      </b-button>
                    </b-button-group>
                  </b-card>
                </b-col>
              </b-row>
            </b-container>
          </div>

        </b-col>
      </b-row>
    </b-container>
  
    <!-- Edit Profile Modal -->
    <b-modal v-model="showEditModal" title="Edit Profile" hide-footer>
      <b-form @submit.prevent="saveProfile">
        <b-form-group label="Full Name:" label-for="full-name">
        <b-form-input id="full-name" v-model="editableName"></b-form-input>
      </b-form-group>
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
        <b-form-group label="Degree:" label-for="degree">
          <b-form-input id="degree" v-model="editableDegree"></b-form-input>
        </b-form-group>
        <b-form-group label="Email:" label-for="email">
          <b-form-input id="email" v-model="editableEmail"></b-form-input>
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
import {
  TOGGLE_DARK_MODE,
  SAVE_DARK_MODE,
  RESET_DARK_MODE,
} from "@/store";

export default {
  name: "MyAccount",
  data() {
    return {
      savedSchedules: JSON.parse(localStorage.getItem('savedSchedules') || '[]'),
      newScheduleName: "",
      activeSection: "Profile",
      showEditModal: false,
      editableName: "",
      editableMajor: "",
      editableYear: "",
      editableDegree: "",
      editableEmail: "",
      currentYear: new Date().getFullYear(),
      sections: [
        { name: "Schedules", label: "Schedules" },
        { name: "Profile", label: "Profile" },
        { name: "Student Information", label: "Student Information" },
        { name: "Settings", label: "Settings" },
        { name: "Website Theme", label: "Website Theme" },
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
      saveNewSchedule() {
    if (!this.newScheduleName) {
      alert("Please enter a name for your schedule.");
      return;
    }

    const currentCourses = this.getCurrentSchedule();
    const newSchedule = {
      name: this.newScheduleName,
      courses: currentCourses,
    };

    this.savedSchedules.push(newSchedule);
    localStorage.setItem("savedSchedules", JSON.stringify(this.savedSchedules));
    this.newScheduleName = "";
  },

  applySchedule(index) {
    const schedule = this.savedSchedules[index];
    this.setCurrentSchedule(schedule.courses); // <- Replace this
    alert(`Schedule "${schedule.name}" loaded.`);
  },

  deleteSchedule(index) {
    this.savedSchedules.splice(index, 1);
    localStorage.setItem("savedSchedules", JSON.stringify(this.savedSchedules));
  },

    async logOut() {
      try {
        await this.$store.dispatch(userTypes.actions.LOGOUT);
        this.$router.replace("/");
      } catch (err) {
        console.error("Logout error:", err);
      }
    },
        toggleTheme(mode) {
      const deviceTheme = window.matchMedia("(prefers-color-scheme: dark)").matches;

      if (
        (mode === false && this.darkMode === true) ||
        (mode === true && this.darkMode === false) ||
        (this.darkMode == null && mode !== deviceTheme)
      ) {
        this.$store.commit(TOGGLE_DARK_MODE);
        this.$store.commit(SAVE_DARK_MODE);
      } else {
        this.$store.commit(SAVE_DARK_MODE);
      }

      this.darkMode = this.$store.getters.darkModeState;
    },

    followDeviceTheme() {
      this.$store.commit(RESET_DARK_MODE);
      this.$store.commit(TOGGLE_DARK_MODE);
      this.darkMode = null;
    },
    loadUserFromStorage() {
      const storedUser = localStorage.getItem("userProfile");
      if (storedUser) {
        this.updateUserInfo(JSON.parse(storedUser));
      }
    },
    openEditModal() {
      this.editableName = this.user.name || "";
      this.editableMajor = this.user.major || "";
      this.editableYear = this.user.year || this.currentYear;
      this.editableDegree = this.user.degree || "";
      this.editableEmail = this.user.email || "";
      this.showEditModal = true;
    },
    async saveProfile() {
  if (this.editableYear < this.currentYear) {
    alert("Year must be the current year or later.");
    return;
  }

  const originalEmail = this.user.email; 

  const updatedUser = {
    name: this.editableName || "",
    sessionID: this.$store.state.sessionID || "",
    email: this.editableEmail || "",
    phone: this.user.phone || "",
    newPassword: "",
    major: this.editableMajor,
    degree: this.editableDegree,
    year: this.editableYear,
  };

  try {
    const response = await fetch("/api/user", {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(updatedUser),
    });

    const responseData = await response.json();

    if (!response.ok) {
      alert("Failed to update profile: " + (responseData.message || "Unknown error"));
      return;
    }

    this.updateUserInfo(updatedUser);
    localStorage.setItem("userProfile", JSON.stringify(updatedUser));

    if (originalEmail !== this.editableEmail) {
      alert("Email changed! Please log in with your new email.");
      await this.$store.dispatch(userTypes.actions.LOGOUT);
      this.$router.replace("/"); 
    } else {
      this.showEditModal = false;
    }
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