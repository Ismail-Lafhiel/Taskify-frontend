<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import Nav from "./components/Nav.vue";

const user = ref(null);
const router = useRouter();

onMounted(async () => {
  try {
    const token = localStorage.getItem("token");
    if (!token) {
      router.push("/login");
      return;
    }
    const response = await axios.get("/api/user", {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    user.value = response.data.user;
    // console.log(user.value);
  } catch (error) {
    console.error("Unauthenticated:", error.response.data);
    router.push("/login");
  }
});

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
    user.value = null; // Clear user data
    router.push("/login");
  } catch (error) {
    console.error("Logout failed:", error);
  }
};
</script>

<script>
import Nav from "./components/Nav.vue";

export default {
  components: {
    Nav,
  },
};
</script>
<template>
  <div>
    <Nav :user="user" :logout="logout" />
    <router-view :user="user" />
  </div>
</template>

