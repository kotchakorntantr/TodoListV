<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import Swal from "sweetalert2";

const user = ref({});
const name = ref("");
const router = useRouter();

const newTask = ref("");
const tasks = ref([]);
const apiUrl = "http://localhost:3000/tasks";
const userId = JSON.parse(localStorage.getItem("user")).id;

const handleLogout = () => {
//   ลบแค่ status ของ user in localstorage
  localStorage.removeItem("user");
  router.push("/");
};

const getTasks = async () => {
  try {
    const response = await axios.get(
      `http://localhost:3000/tasks?userId=${userId}`
    );
    tasks.value = response.data;
  } catch (error) {
    console.error("Error fetching tasks:", error);
  }
};

const addTask = async () => {
  if (newTask.value.trim() !== "") {
    try {
      const newTaskData = {
        text: newTask.value.trim(),
        done: false,
        userId: userId, // ผูก task กับ user id
      };
      const response = await axios.post(
        "http://localhost:3000/tasks",
        newTaskData
      );
      tasks.value.push(response.data);
      newTask.value = "";
    } catch (error) {
      console.error("Failed to add task:", error);
    }
  }
};
const toggleTask = async (task) => {
  try {
    await axios.patch(`${apiUrl}/${task.id}`, {
      done: !task.done,
    });
    task.done = !task.done;
  } catch (error) {
    console.error("Failed to toggle task:", error);
  }
};

const deleteTask = async (id) => {
  try {
    const result = await Swal.fire({
      title: "Are you sure?",
      text: "Do you want to delete this task?",
      icon: "warning",
      showCancelButton: true,
      confirmButtonColor: "#3085d6",
      cancelButtonColor: "#d33",
      confirmButtonText: "Yes, delete it!",
      cancelButtonText: "Cancel",
    });
    if (result.isConfirmed) {
      // กดยืนยันแล้วค่อยลบจริง
      await axios.delete(`${apiUrl}/${id}`);
      tasks.value = tasks.value.filter((task) => task.id !== id);

      await Swal.fire({
        icon: "success",
        title: "Deleted!",
        text: "Your task has been deleted.",
        timer: 2000,
        showConfirmButton: false,
      });
    }
  } catch (error) {
    console.error("Failed to delete task:", error);
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Failed to delete the task!",
    });
  }
};

onMounted(async () => {
  if (!userId) {
    // ถ้าไม่มี userId -> logout กลับไปหน้า login
    router.push("/");
  } else {
    await getTasks();
  }
  const storedUser = localStorage.getItem("user");
  if (storedUser) {
    user.value = JSON.parse(storedUser);
    name.value = user.value.name;
  } else {
    console.error("No user found in local storage");
  }
});
</script>

<template>
  <div class="text-3xl font-bold text-center mt-10">Hello, {{ name }}</div>
  <div class="flex flex-col items-center justify-center mt-8">
    <ul class="menu menu-horizontal bg-base-200 rounded-box">
      <li class="hover:bg-blue-300 bg-amber-200" @click="handleLogout">
        <a>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"
            />
          </svg>
        </a>
      </li>
      <li class="hover:bg-blue-300 bg-amber-300">
        <a>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
            />
          </svg>
        </a>
      </li>
      <li class="hover:bg-blue-300 bg-amber-400">
        <a>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"
            />
          </svg>
        </a>
      </li>
    </ul>
  </div>

  <div
    class="max-w-md mx-auto mt-10 p-6 bg-white rounded-xl shadow-lg space-y-4"
  >
    <h2 class="text-2xl font-bold text-center">My Todo List</h2>

    <div class="flex space-x-2">
      <input
        v-model="newTask"
        @keyup.enter="addTask"
        type="text"
        placeholder="Add new task..."
        class="flex-grow border rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-300"
      />
      <button
        @click="addTask"
        class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600"
      >
        Add
      </button>
    </div>

    <ul class="space-y-2">
      <li
        v-for="task in tasks"
        :key="task.id"
        class="flex justify-between items-center p-2 border rounded-md bg-gray-50"
      >
        <div
          :class="{ 'line-through text-gray-400': task.done }"
          class="cursor-pointer"
          @click="toggleTask(task)"
        >
          {{ task.text }}
        </div>
        <button @click="deleteTask(task.id)" class="text-red-500 hover:text-red-700"> ✕
        </button>
      </li>
    </ul>
  </div>
</template>
