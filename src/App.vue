<!-- App.vue -->
<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import Nav from "./components/Nav.vue";

const user = ref(null);
const router = useRouter();
let isFetching = false;

// Function to fetch user data from the server
const fetchUserData = async () => {
  // If a request is already pending, do not send another one
  if (isFetching) return;

  try {
    isFetching = true;

    // Retrieve the authentication token from localStorage
    const token = localStorage.getItem("token");
    if (!token) {
      router.push("/login");
      return;
    }

    // Make the API call to fetch user data
    const response = await axios.get("/api/user", {
      headers: {
        Authorization: `Bearer ${token}`, // Include the token in the request headers
      },
    });

    // Update the user data with the response
    user.value = response.data.user;
    console.log("User data:", user.value); // Log user data here if needed
  } catch (error) {
    console.error("Error fetching user data:", error);
    if (error.response && error.response.status === 429) {
      console.error("Too many requests. Please try again later.");
    } else {
      // Handle other errors or redirect to login page
      router.push("/login");
    }
  } finally {
    // Reset the flag after the request is complete
    isFetching = false;
  }
};

// Call the fetchUserData function when the component is mounted
onMounted(() => {
  // Initial fetch
  fetchUserData();

  // Set up interval to fetch data every 5 seconds
  // setInterval(fetchUserData, 5000);
});

// Function to log out the user
const logout = async () => {
  try {
    const token = localStorage.getItem("token");
    if (!token) {
      router.push("/login");
      return;
    }
    await axios.post("/api/logout", null, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    localStorage.removeItem("token");
    user.value = null;
    router.push("/login");
  } catch (error) {
    console.error("Logout failed:", error);
  }
};
</script>

<template>
  <Nav :user="user" :logout="logout" />
  <router-view />
</template>
