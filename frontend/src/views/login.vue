<!-- This is the login view. Users can login by entering a correct username and password -->
<template>
  <div>
    <!--Header-->
    <h1
      class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10"
    >
      Welcome
    </h1>
    <!--Form-->
    <form @submit.prevent="handleLogin" novalidate="true">
      <div class="flex justify-center mt-10">
        <label>
          <span class="text-gray-700">Username</span>
          <span style="color: #ff0000">*</span>
          <input
            type="text"
            class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            placeholder="Username"
            v-model="username"
          />
        </label>
      </div>
      <div class="flex justify-center mt-10">
        <label>
          <span class="text-gray-700">Password</span>
          <span style="color: #ff0000">*</span>
          <input
            type="password"
            class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            placeholder="Password"
            v-model="password"
          />
        </label>
      </div>
      <!--Login button-->
      <div class="flex justify-center mt-10">
        <button class="bg-red-700 text-white rounded" type="submit">
          Login
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import { useLoggedInUserStore } from "../store/loggedInUser";

const router = useRouter();
const store = useLoggedInUserStore();

const username = ref("");
const password = ref("");

// Define login method
const handleLogin = async () => {
  await store.login(username.value, password.value);
  // Since the store action itself handles navigation and toast notifications, there's no need to duplicate the logic.
};
</script>
