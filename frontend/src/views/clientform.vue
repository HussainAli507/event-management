<script>
import { createClient } from "../api/api";
import { useToast } from "vue-toastification";
import { reactive, toRefs } from "vue";
import {
  required,
  email,
  numeric,
  minLength,
  maxLength,
} from "@vuelidate/validators";

export default {
  setup() {
    const toast = useToast();
    // Use reactive to make the client object reactive
    const clientData = {
      firstName: "",
      middleName: "",
      lastName: "",
      email: "",
      phoneNumber: {
        primary: "",
        alternate: "",
      },
      address: {
        line1: "",
        line2: "",
        city: "",
        county: "",
        zip: "",
      },
    };
    const clientForm = reactive({ ...clientData });

    //   const v$ = useVuelidate({
    // firstName: { required },
    // lastName: { required },
    // email: { required, email },
    // phoneNumber: {
    //   primary: {
    //     required,
    //     numeric,
    //     minLength: minLength(10),
    //     maxLength: maxLength(10)
    //   }
    // },
    // address: {
    //   city: { required }
    // }
    // }, client);

    // Submit form handler
    // const submitForm = async () => {
    //   try {
    //     const response = await createClient(clientForm);
    //     toast.success("Client created successfully");
    //     Object.assign(clientForm, { ...clientData });
    //   } catch (error) {
    //     toast.error(`Error searching clients by phone number: ${error}`);
    //   }
    // };

    const submitForm = async () => {
      try {
        const response = await createClient(clientForm);
        toast.success("Client created successfully");
        // Reset each field to an empty string
        clientForm.firstName = "";
        clientForm.middleName = "";
        clientForm.lastName = "";
        clientForm.email = "";
        clientForm.phoneNumber.primary = "";
        clientForm.phoneNumber.alternate = "";
        clientForm.address.line1 = "";
        clientForm.address.line2 = "";
        clientForm.address.city = "";
        clientForm.address.county = "";
        clientForm.address.zip = "";
      } catch (error) {
        toast.error(`Error searching clients by phone number: ${error}`);
      }
    };

    // Return the reactive object and the submit function using toRefs
    // This ensures that each property of the client object can be used independently in the template
    return { clientForm, submitForm };
  },
};
</script>

<!-- This view allows a user to create client information. -->
<template>
  <main>
    <!--Header-->
    <h1
      class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10"
    >
      Client Intake Form
    </h1>
    <div class="px-10 py-20">
      <!-- form field -->
      <!-- @submit.prevent stops the submit event from reloading the page-->
      <form @submit.prevent="submitForm">
        <!-- grid container -->
        <div
          class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
        >
          <h2 class="text-2xl font-bold">Personal Details</h2>
          <!--First Name Input Field-->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">First Name</span>
              <span style="color: #ff0000">*</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="clientForm.firstName"
              />
            </label>
            <!-- Validation error messages -->
            <!-- <span v-if="v$.clientForm.firstName.$error" class="text-red-500">
              First Name is required
            </span> -->
          </div>

          <!--Middle Name Input Field-->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Middle Name</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                placeholder
                v-model="clientForm.middleName"
              />
            </label>
          </div>

          <!--Last Name Input Field-->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Last Name</span>
              <span style="color: #ff0000">*</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                placeholder
                v-model="clientForm.lastName"
              />
            </label>
            <!-- Validation error messages -->
            <!-- <span v-if="v$.clientForm.lastName.$error" class="text-red-500">
              Last Name is required
            </span> -->
          </div>

          <div></div>
          <!--Email Input Field-->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Email</span>
              <span style="color: #ff0000">*</span>
              <input
                type="email"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="clientForm.email"
              />
            </label>
            <!-- Validation error messages -->
            <!-- <span v-if="v$.clientForm.email.$error" class="text-red-500">
              Valid Email is required
            </span> -->
          </div>
          <!-- Phone Number Input Field -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Phone Number</span>
              <span style="color: #ff0000">*</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                pattern="[0-9]{3}[0-9]{3}[0-9]{4}"
                v-model="clientForm.phoneNumber.primary"
              />
            </label>
            <!-- Validation error messages -->
            <!-- <span v-if="v$.clientForm.phoneNumber.primary.$error" class="text-red-500">
              <span v-if="v$.clientForm.phoneNumber.primary.required.$invalid">
                Phone Number is required
              </span>
              <span
                v-else-if="!v$.clientForm.phoneNumber.primary.required.$invalid && v$.clientForm.phoneNumber.primary.numeric.$invalid">
                Phone Number must contain only digits
              </span>
              <span
                v-else-if="!v$.clientForm.phoneNumber.primary.required.$invalid && !v$.clientForm.phoneNumber.primary.numeric.$invalid && (v$.clientForm.phoneNumber.primary.minLength.$invalid || v$.clientForm.phoneNumber.primary.maxLength.$invalid)">
                Phone Number must be exactly 10 digits
              </span>
            </span> -->
          </div>

          <!-- Alternative Phone Number Input Field -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Alternative Phone Number</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                pattern="[0-9]{3}[0-9]{3}[0-9]{4}"
                v-model="clientForm.phoneNumber.alternate"
              />
            </label>
          </div>
        </div>

        <!-- grid container -->
        <div
          class="mt-10 grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
        >
          <h2 class="text-2xl font-bold">Address Details</h2>
          <!-- Address 1 Input Field -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Address Line 1</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="clientForm.address.line1"
              />
            </label>
          </div>

          <!-- Address 2 Input Field -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Address Line 2</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="clientForm.address.line2"
              />
            </label>
          </div>

          <!-- City Input Field -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">City</span>
              <span style="color: #ff0000">*</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="clientForm.address.city"
              />
            </label>
            <!-- Validation error messages -->
            <!-- <span v-if="v$.clientForm.address.city.$error" class="text-red-500">
              City is required
            </span> -->
          </div>
          <div></div>

          <!-- County Input Field -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">County</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="clientForm.address.county"
              />
            </label>
          </div>

          <!-- Zip Code Input Field -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Zip Code</span>
              <input
                type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="clientForm.address.zip"
              />
            </label>
          </div>
          <div></div>

          <!-- Add Client Submit Button -->
          <div class="flex justify-between mt-10 mr-20">
            <button class="bg-red-700 text-white rounded" type="submit">
              Add Client
            </button>
          </div>
        </div>
      </form>
    </div>
  </main>
</template>

<!-- references:
1. class code
2. https://vuejs.org/guide/extras/composition-api-faq.html
3. chatgpt -->
