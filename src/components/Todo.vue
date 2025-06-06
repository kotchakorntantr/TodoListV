<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import Swal from "sweetalert2";
import { supabase } from "../supabaseClient.js";

const user = ref({});
const name = ref("");
const router = useRouter();

const newTask = ref("");
const tasks = ref([]);
const userId = ref(null); //use ref for reactivity

onMounted(async () => {
  const storedUser = localStorage.getItem("user");
  if (storedUser) {
    user.value = JSON.parse(storedUser);
    name.value = user.value.name;
    userId.value = user.value.id;
    if (!userId.value) {
      router.push("/"); //if no userId, redirect to login
    } else {
      await getTasks(); // fetch tasks for the user
    }
  } else {
    console.error("No user found in local storage");
    router.push("/");
  }
});
const handleLogout = () => {
  // ลบแค่ status ของ user in localstorage
  localStorage.removeItem("user");
  router.push("/");
};

const getTasks = async () => {
  if (!userId.value) return;
  try {
    const { data, error } = await supabase
      .from("todos") //table name
      .select("*")
      .eq("userId", userId.value) // filter by userId
      .order("created_at", { ascending: false });

    if (error) throw error;
    tasks.value = data;
  } catch (error) {
    console.error("Failed to fetch tasks:", error.message);
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Failed to load tasks!",
    });
  }
};

const addTask = async () => {
  if (!newTask.value.trim() || !userId.value) return;
  try {
    const newTaskData = {
      task: newTask.value.trim(),
      is_complete: false,
      userId: userId.value, // ผูก task กับ userId
    };
    const { data, error } = await supabase
      .from("todos")
      .insert(newTaskData)
      .select();

    if (error) throw error;
    tasks.value.unshift(data[0]); // เพิ่มข้อมูลที่ Supabase คืนกลับมา
    newTask.value = "";
  } catch (error) {
    console.error("Failed to add task:", error.message);
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Failed to add task!",
    });
  }
};
// status task
const toggleTask = async (task) => {
  try {
    const { error } = await supabase
      .from("todos")
      .update({ is_complete: !task.is_complete }) // อัปเดตคอลัมน์ 'is_complete'
      .eq("id", task.id); // อัปเดตตาม id ของ task

    if (error) throw error;
    task.is_complete = !task.is_complete;
  } catch (error) {
    console.error("Failed to toggle task:", error.message);
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Failed to toggle task status!",
    });
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
      const { error } = await supabase.from("todos").delete().eq("id", id); // ลบตาม id ของ task
      if (error) throw error;
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
    console.error("Failed to delete task:", error.message);
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Failed to delete the task!",
    });
  }
};
</script>

<template>
  <div class="pt-10">
    <div class="text-3xl font-bold text-center">Hello, {{ name }}</div>
    <div class="flex flex-col items-center justify-center mt-8">
      <ul class="menu menu-horizontal bg-base-200 rounded-box">
        <li class="hover:bg-[#C4E1F6] bg-[#FEEE91]" @click="handleLogout">
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
        <li class="hover:bg-[#C4E1F6] bg-[#FFBD73]">
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
        <li class="hover:bg-[#C4E1F6] bg-[#FF9D3D]">
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

    <div class="flex justify-center items-center mt-6 px-4"> 
    <div class="w-full md:w-3/4 lg:w-1/2 xl:w-2/3 2xl:w-[500px] mx-auto mt-10 p-6 bg-[#C4E1F6] rounded-xl shadow-lg space-y-4">
      <h2 class="text-2xl font-bold text-center">My Todo Lists</h2>

      <div class="flex space-x-2">
        <input
          v-model="newTask"
          @keyup.enter="addTask"
          type="text"
          placeholder="Add new task..."
          class="flex-grow border border-white bg-white rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-300"
        />
        <button
          @click="addTask"
          class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600"
        >
          Add
        </button>
      </div>

      <ul class="space-y-2 max-h-60 overflow-y-auto hide-scrollbar">
        <li
          v-for="task in tasks"
          :key="task.id"
          class="flex justify-between items-center p-2 rounded-md bg-gray-50"
        >
          <div
            :class="{ 'line-through text-gray-400': task.is_complete }"
            class="cursor-pointer"
            @click="toggleTask(task)"
          >
            {{ task.task }}
          </div>
          
          <button
            @click="deleteTask(task.id)"
            class="text-red-500 hover:text-red-700"
          >
            ✕
          </button>
        </li>
      </ul>
    </div>
    </div>
  </div>
</template>

<style scoped>
.hide-scrollbar::-webkit-scrollbar {
  display: none; 
}

.hide-scrollbar {
  scrollbar-width: none;  
}
</style>