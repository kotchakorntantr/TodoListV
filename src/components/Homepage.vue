<script setup>
import { ref } from "vue";
import Swal from "sweetalert2";
import { useRouter } from "vue-router";
import { supabase } from "../supabaseClient";

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
    const { data, error: signupError } = await supabase.auth.signUp({
      email: email.value,
      password: password.value,
      options: {
        data: {
          name: name.value, // Store additional user data
        },
      },
    });
    if (signupError) {
      console.error("Signup error:", signupError.message);
      throw signupError;
    }
    if (data.user) {
      const { error: profileError } = await supabase
        .from("profiles")
        .insert([
          { id: data.user.id, name: name.value }, // id คือ ID ของผู้ใช้ที่ Supabase Auth สร้าง
        ]);
      if (profileError) {
        console.error("Error inserting profile data:", profileError.message);
        Swal.fire({
          icon: "error",
          title: "Signup Failed!",
          text: "Error saving user profile. Please try again or contact support.",
        });
        return; // หยุดการทำงานถ้าบันทึก profile ไม่สำเร็จ
      }
    } else {
      successMessage =
        "Signup Success! Please check your email to confirm your account before logging in.";
    }

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

    const { data, error: signInError } = await supabase.auth.signInWithPassword(
      {
        email: email.value,
        password: password.value,
      }
    );

    if (signInError) {
      console.error("Supabase login error:", signInError.message);
      throw signInError;
    }
    if (data.user) {
      // ถ้า data.user มีค่า = Login สำเร็จ
      // ดึงข้อมูล Profile จากตาราง profiles
      const { data: profileData, error: profileError } = await supabase
        .from("profiles")
        .select("name")
        .eq("id", data.user.id)
        .single();
      if (profileError) {
        console.error("Error fetching profile data:", profileError.message);
        Swal.fire({
          icon: "warning",
          title: "Profile Error!",
          text: "Could not load user profile. Displaying default name.",
          timer: 2000,
          showConfirmButton: false,
        });
      }
      localStorage.setItem(
        "user",
        JSON.stringify({
          id: data.user.id,
          name: profileData ? profileData.name : "User",
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
    } else {
      Swal.fire({
        icon: "error",
        title: "Login Failed!",
        text: "An unexpected error occurred. Please try again.",
      });
    }
  } catch (error) {
    console.error(error);
    Swal.fire({
      icon: "error",
      title: "Login Failed!",
      text: error.message || "Invalid credentials. Please try again.",
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
