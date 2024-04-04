<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const tasks = ref([]);
const taskName = ref("");
const taskDescription = ref("");
const taskStatus = ref("");
let authUser = null;

// Fetch tasks from the server
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
  } catch (error) {
    console.error("Error fetching tasks:", error);
    tasks.value = [];
  }
};

// Fetch user data to get user_id
const fetchUserData = async () => {
  try {
    const token = localStorage.getItem("token");
    if (!token) {
      return;
    }
    const response = await axios.get("/api/user", {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    authUser = response.data.user;
  } catch (error) {
    console.error("Error fetching user data:", error);
  }
};

// Add a new task
const addTask = async () => {
  try {
    const token = localStorage.getItem("token");
    if (!token) {
      return;
    }
    await fetchUserData(); // Fetch user data before adding the task
    const requestData = {
      name: taskName.value,
      description: taskDescription.value,
      status: taskStatus.value,
      user_id: authUser.id, // Set the user_id to the authenticated user's ID
    };
    const response = await axios.post("/api/tasks/store", requestData, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    tasks.value.push(response.data); // Add the new task to the list
    clearTaskInputs(); // Clear task input fields
  } catch (error) {
    console.error("Error adding task:", error);
  }
};

// Clear task input fields
const clearTaskInputs = () => {
  taskName.value = "";
  taskDescription.value = "";
  taskStatus.value = "";
};

onMounted(() => {
  fetchTasks();
});

// Edit an existing task
const editTask = async (task) => {
  const newTaskName = prompt("Enter new task name:", task.name);
  const newTaskDescription = prompt(
    "Enter new task description:",
    task.description
  );
  const newTaskStatus = prompt("Enter new task status:", task.status);
  if (newTaskName && newTaskDescription && newTaskStatus) {
    try {
      const token = localStorage.getItem("token");
      if (!token) {
        return;
      }

      // Fetch user data to get user ID
      await fetchUserData();

      // Ensure authUser is not null before proceeding
      if (!authUser) {
        console.error("Error: User data not available.");
        return;
      }

      // Create the updated task object
      const updatedTask = {
        ...task,
        name: newTaskName, // Update the name field
        description: newTaskDescription, // Update the description field
        status: newTaskStatus, // Update the status field
        user_id: authUser.id, // Include the user_id
      };

      const response = await axios.put(
        `/api/tasks/${task.id}`,
        updatedTask, // Send the entire updated task object
        {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        }
      );

      // Update the task in the list with the response data
      const index = tasks.value.findIndex((t) => t.id === task.id);
      tasks.value[index] = response.data;
    } catch (error) {
      console.error("Error editing task:", error);
    }
  }
};

// Delete a task
const deleteTask = async (taskId) => {
  if (confirm("Are you sure you want to delete this task?")) {
    try {
      const token = localStorage.getItem("token");
      if (!token) {
        return;
      }
      await axios.delete(`/api/tasks/${taskId}`, {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });
      tasks.value = tasks.value.filter((task) => task.id !== taskId); // Remove the deleted task from the list
    } catch (error) {
      console.error("Error deleting task:", error);
    }
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
        <!-- Task input form -->
        <form @submit.prevent="addTask">
          <input
            v-model="taskName"
            type="text"
            placeholder="Enter task name"
            class="border-b border-gray-300 p-2 w-full"
          />
          <textarea
            v-model="taskDescription"
            cols="30"
            rows="10"
            class="border-b border-gray-300 p-2 w-full"
          ></textarea>
          <input
            v-model="taskStatus"
            type="text"
            placeholder="Enter task status"
            class="border-b border-gray-300 p-2 w-full"
          />
          <button type="submit" class="bg-blue-500 text-white px-4 py-2 mt-2">
            Add Task
          </button>
        </form>

        <!-- Task list -->
        <ul>
          <li
            v-for="task in tasks"
            :key="task.id"
            class="flex justify-between items-center border-b border-gray-300 py-2"
          >
            <span>{{ task.name }}</span>
            <div>
              <button @click="editTask(task)" class="text-blue-500 mr-2">
                Edit
              </button>
              <button @click="deleteTask(task.id)" class="text-red-500">
                Delete
              </button>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </section>
</template>
