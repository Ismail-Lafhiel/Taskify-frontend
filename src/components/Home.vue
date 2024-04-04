<!-- Home.vue -->
<script setup>
import { ref, onMounted, nextTick } from 'vue';
import axios from 'axios';

const tasks = ref([]);

const fetchTasks = async () => {
  try {
    const token = localStorage.getItem("token");
    if (!token) {
      return;
    }
    const response = await axios.get("/api/tasks", {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    tasks.value = response.data;
    await nextTick(); // Wait for the next tick to ensure tasks.value is updated
    // console.log("Tasks:", tasks.value);
  } catch (error) {
    console.error("Error fetching tasks:", error);
    tasks.value = [];
  }
};

onMounted(() => {
  fetchTasks();
});
</script>

<template>
  <section class="bg-gray-50 dark:bg-gray-900 p-3 sm:p-5">
    <div class="mx-auto max-w-screen-xl px-4 lg:px-12">
      <div
        class="bg-white dark:bg-gray-800 relative shadow-md sm:rounded-lg overflow-hidden"
      >
        <ul>
          <li v-for="task in tasks" :key="task.id">
            <div class="mb-4">
              <h3 class="text-lg font-semibold">{{ task.name }}</h3>
              <p class="text-gray-500">{{ task.description }}</p>
              <p>Status: {{ task.status }}</p>
              <p>User ID: {{ task.user_id }}</p>
            </div>
            <hr class="border-t border-gray-200 dark:border-gray-700">
          </li>
        </ul>
      </div>
    </div>
  </section>
</template>

