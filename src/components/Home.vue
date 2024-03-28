<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

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
    console.log(user.value);
  } catch (error) {
    console.error("Unauthenticated:", error.response.data);
    router.push("/login");
  }
});
</script>

<template>
  <div>
    <h1>{{ user?.name }}</h1>
    <h3>{{ user?.email }}</h3>
  </div>
</template>
