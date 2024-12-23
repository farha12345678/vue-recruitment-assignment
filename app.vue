<template>
  <div class="wrapper">
    <h2>Submit a Complaint</h2>

    <!-- Form to add a new complaint -->
    <div class="complain-form">
      <input
        v-model="title"
        type="text"
        placeholder="Title (e.g., Product Issue)"
        @input="resetFieldStyles"
        :class="{ invalid: !title.trim() && formError }"
      />
      <textarea
        v-model="body"
        placeholder="Enter your complaint details here..."
        @input="resetFieldStyles"
        :class="{ invalid: !body.trim() && formError }"
      ></textarea>

      <button :disabled="isSaving" @click="saveComplain">
        {{ isSaving ? "Submitting..." : "Submit Complaint" }}
      </button>

      <!-- Display form-specific error messages -->
      <div v-if="formError" class="error-message">{{ formError }}</div>
    </div>

    <h2>Complaints List</h2>

    <!-- Placeholder while loading -->
    <div v-if="isLoading" class="placeholder">
      <div class="placeholder-item"></div>
      <div class="placeholder-item"></div>
      <div class="placeholder-item"></div>
    </div>

    <!-- Display complaints list -->
    <template v-else-if="complains.length">
      <div class="complain-list">
        <div
          v-for="complain in complains"
          :key="complain.Id"
          class="complain-item"
        >
          <div class="complain-header">
            <h3>{{ complain.Title }}</h3>
            <span class="complain-id">#{{ complain.Id }}</span>
          </div>
          <p class="complain-body">{{ complain.Body }}</p>
        </div>
      </div>
    </template>

    <!-- Display fetch-specific error -->
    <div v-else>
      <p v-if="fetchError" class="error-message">{{ fetchError }}</p>
      <p v-else>No complaints available.</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const baseUrl = "https://sugarytestapi.azurewebsites.net/";
const listPath = "TestApi/GetComplains";
const savePath = "TestApi/SaveComplain";

const complains = ref([]);
const title = ref("");
const body = ref("");
const isSaving = ref(false);
const formError = ref("");
const fetchError = ref("");
const isLoading = ref(false);

// Fetch complaints from the API
const fetchComplains = async () => {
  try {
    isLoading.value = true;
    fetchError.value = "";
    const response = await fetch(`${baseUrl}${listPath}`);
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || "Failed to fetch complaints.");
    }
    const data = await response.json();
    // Sort complaints by ID in descending order
    complains.value = data.sort((a, b) => b.Id - a.Id);
  } catch (error) {
    console.error(error);
    fetchError.value = "Unable to load complaints. Please try again.";
  } finally {
    isLoading.value = false;
  }
};

// Validate the form fields
const validateForm = () => {
  if (!title.value.trim() || !body.value.trim()) {
    formError.value = "Both title and body are required.";
    return false;
  }
  return true;
};

// Save a new complaint
const saveComplain = async () => {
  if (!validateForm()) return;

  try {
    isSaving.value = true;
    formError.value = "";
    const response = await fetch(`${baseUrl}${savePath}`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        Title: title.value,
        Body: body.value,
      }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || "Failed to save complaint.");
    }

    const data = await response.json();
    if (!data.Success) throw new Error(data.Message || "Failed to save complaint.");

    // Add the new complaint to the list
    complains.value.unshift({
      Id: data.Id, // Use the ID returned by the API
      Title: title.value,
      Body: body.value,
    });

    // Clear form fields
    title.value = "";
    body.value = "";
  } catch (error) {
    console.error(error);
    formError.value = error.message || "An error occurred while saving the complaint.";
  } finally {
    isSaving.value = false;
  }
};

// Reset styles for invalid fields
const resetFieldStyles = () => {
  formError.value = "";
};

onMounted(() => {
  fetchComplains();
});
</script>

<style>
.wrapper {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  background-color: #f9f9f9;
}

.complain-form {
  margin-bottom: 20px;
}

.complain-form input,
.complain-form textarea {
  width: 100%;
  margin-bottom: 10px;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 16px;
}

.complain-form input.invalid,
.complain-form textarea.invalid {
  border-color: #e74c3c;
}

.complain-form button {
  padding: 10px 16px;
  font-size: 16px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.complain-form button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.complain-form button:hover:not(:disabled) {
  background-color: #0056b3;
}

.error-message {
  color: red;
  margin-top: 10px;
  font-size: 14px;
}

.complain-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.complain-item {
  padding: 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.complain-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.complain-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.complain-header h3 {
  margin: 0;
  font-size: 18px;
  color: #333;
}

.complain-header .complain-id {
  font-size: 14px;
  color: #888;
}

.complain-body {
  margin: 0;
  font-size: 16px;
  color: #555;
}

.placeholder {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.placeholder-item {
  height: 20px;
  background-color: #e0e0e0;
  border-radius: 4px;
}
</style>
