<script setup>

import {ref} from 'vue'
const isLogin = ref(true)

const handleLogin = () => {
  console.log('Login clicked')

}

const handleSignup = () => {
  console.log('Signup clicked')
}
</script>

<template>
  <div class="hero bg-base-200 min-h-screen">
      <div class="card bg-base-100 w-full max-w-sm shrink-0 shadow-2xl">
        <div class="card-body ">
        <div>
            <h2 class="text-center font-bold text-2xl"> Todo List</h2>
        </div>
          <div v-if="isLogin">
            <label class="label font-bold ">Email</label>
            <input type="email" class="input border mt-2 rounded-xl" placeholder="Email" />
            <label class="label font-bold mt-2">Password</label>
            <input type="password" class="input border mt-2 rounded-xl" placeholder="Password" />
            <div class="mt-2">
                <a @click="isLogin = false" class="link link-hover ">Don't have an account?</a>
            </div>
            <div class="flex justify-center">
                <button @click="handleLogin" class="bg-black text-white w-full mt-4 py-1 hover:bg-blue-200 hover:text-black item-center rounded-xl ">Login</button>
            </div>
          </div>

          <div v-else>
            <label class="label font-bold">Name or Username</label>
            <input type="text" class="input border mt-2 rounded-xl" placeholder="Name or Username" />
            <label class="label font-bold mt-2">Email</label>
            <input type="email" class="input border mt-2 rounded-xl" placeholder="Email" />
            <label class="label font-bold mt-2">Password</label>
            <input type="password" class="input border mt-2 rounded-xl" placeholder="Password" />
            <div class="mt-2">
                <a @click="isLogin = true" class="link link-hover ">Already have an account?</a>
            </div>
            <div class="flex justify-center">
                <button @click="handleSignup" class="bg-black text-white w-full mt-4 py-1 hover:bg-blue-200 hover:text-black item-center  rounded-xl ">Signup</button>
            </div>
            
          </div>
        </div>
      </div>
  </div>

</template>
