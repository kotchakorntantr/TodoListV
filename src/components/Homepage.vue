<script setup>
import axios from "axios";
import { ref } from "vue";
import CryptoJS from "crypto-js";
import Swal from "sweetalert2";
import { useRouter } from "vue-router";

const router = useRouter();
const isLogin = ref(true);
const email = ref("");
const password = ref("");
const name = ref("");


// part signup + validate data
const handleSignup = async () => {
  if (name.value === "" || email.value === "" || password.value === "") {
    Swal.fire({
      icon: "warning",
      title: "Missing Fields!",
      text: "Please fill in all fields before signing up..",
    });
    return;
  }

  if (password.value.length < 6 || password.value.length > 10) {
    Swal.fire({
      icon: "warning",
      title: "Invalid Password!",
      text: "Password must be between 6 and 10 characters.",
    });
    return;
  }

  if (name.value.length < 6 || name.value.length > 15) {
    Swal.fire({
      icon: "warning",
      title: "Invalid Name!",
      text: "Name must be between 6 and 15 characters.",
    });
    return;
  }

  if (!email.value.includes("@") || !email.value.includes(".")) {
    Swal.fire({
      icon: "warning",
      title: "Invalid Email!",
      text: "Please enter a valid email address.",
    });
    return;
  }

  console.log("Signup clicked");
  try {
    const hashedPassword = CryptoJS.SHA256(password.value).toString();
    const newUser = {
      name: name.value,
      email: email.value,
      password: hashedPassword,
    };
    await axios.post("http://localhost:3000/users", newUser);
    Swal.fire({
      icon: "success",
      title: "Signup Success!",
      text: "Welcome to Login Page!",
      timer: 2000,
      showConfirmButton: false,
    });
    // clear form
    name.value = "";
    email.value = "";
    password.value = "";
    isLogin.value = true;
  } catch (error) {
    console.error(error);
    Swal.fire({
      icon: "error",
      title: "Signup Failed!",
      text: "Please try again.",
    });
  }
};


// part login
const handleLogin = async () => {
  if (!email.value.trim() || !password.value.trim()) {
    Swal.fire({
      icon: "warning",
      title: "Missing Fields!",
      text: "Please fill in all fields before logging in.",
    });
    return;
  }
  try {
    console.log("Login clicked");
    const hashedPassword = CryptoJS.SHA256(password.value).toString();
    const response = await axios.get(
      `http://localhost:3000/users?email=${email.value}&password=${hashedPassword}`
    );
    if (response.data.length > 0) {
      const user = response.data[0];
      localStorage.setItem(
        "user",
        JSON.stringify({
          id: user.id,
          name: user.name,
          email: user.email,
        })
      );
      Swal.fire({
        icon: "success",
        title: "Login Success!",
        text: "Welcome to your TodoList",
        timer: 2000,
        showConfirmButton: false,
      });
      router.push("/main"); 
    //   ถ้าlogin success จะลิ้งไปหน้า todolist
    } else {
      Swal.fire({
        icon: "error",
        title: "Login Failed!",
        text: "Please try again.",
      });
    }
  } catch (error) {
    console.error(error);
    Swal.fire({
      icon: "error",
      title: "Login Failed!",
      text: "Please try again.",
    });
  }
};


</script>

<template>
  <div class="hero bg-base-200 min-h-screen">
    <div class="card bg-base-100 w-full max-w-sm shrink-0 shadow-2xl">
      <div class="card-body">
        <div>
          <h2 class="text-center font-bold text-2xl">Todo List</h2>
        </div>
        <div v-if="isLogin">
          <label class="label font-bold">Email</label>
          <input
            v-model="email"
            type="email"
            class="input border mt-2 rounded-xl"
            placeholder="Email"
            required
            pattern=".+@.+"
          />
          <label class="label font-bold mt-2">Password</label>
          <input
            v-model="password"
            type="password"
            class="input border mt-2 rounded-xl"
            required
            minlength="6"
            maxlength="10"
            placeholder="Password"
          />
          <div class="mt-2">
            <a @click="isLogin = false" class="link link-hover"
              >Don't have an account?</a
            >
          </div>
          <div class="flex justify-center">
            <button
              @click="handleLogin"
              class="bg-black text-white w-full mt-4 py-1 hover:bg-blue-300 hover:text-black item-center rounded-xl"
            >
              Login
            </button>
          </div>
        </div>

        <div v-else>
          <label class="label font-bold">Name</label>
          <input
            v-model="name"
            type="text"
            class="input border mt-2 rounded-xl"
            placeholder="Name"
            required
            minlength="6"
            maxlength="15"
          />
          <label class="label font-bold mt-2">Email</label>
          <input
            v-model="email"
            type="email"
            class="input border mt-2 rounded-xl"
            placeholder="Email"
            required
            pattern=".+@.+"
            title="Please enter a valid email address with '@'"
          />
          <label class="label font-bold mt-2">Password</label>
          <input
            v-model="password"
            type="password"
            class="input border mt-2 rounded-xl"
            required
            minlength="6"
            maxlength="10"
            placeholder="Password"
          />
          <div class="mt-2">
            <a @click="isLogin = true" class="link link-hover"
              >Already have an account?</a
            >
          </div>
          <div class="flex justify-center">
            <button
              @click="handleSignup"
              class="bg-black text-white w-full mt-4 py-1 hover:bg-blue-300 hover:text-black item-center rounded-xl"
            >
              Signup
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>