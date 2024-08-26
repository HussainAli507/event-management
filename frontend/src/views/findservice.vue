<!-- This view shows a list of services. Users can search for services and click on a service to be redirected to another component that displays the service's details -->
<template>
  <main>
    <div>
      <!--Header-->
      <h1
        class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10"
      >
        Find Services
      </h1>
    </div>
    <div class="px-10 pt-20">
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <h2 class="text-2xl font-bold">Search Service By</h2>
        <!-- Displays Service Name and Description selection -->
        <div class="flex flex-col">
          <select
            class="rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            v-model="searchBy"
          >
            <option value="Service Name">Service Name</option>
            <option value="Service Description">Service Description</option>
          </select>
        </div>
        <!--Display service name search field-->
        <div class="flex flex-col" v-if="searchBy === 'Service Name'">
          <label class="block">
            <input
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
              v-model="name"
              v-on:keyup.enter="handleSubmitForm"
              placeholder="Enter service name"
            />
          </label>
        </div>
        <!--Display service description search field-->
        <div class="flex flex-col" v-if="searchBy === 'Service Description'">
          <label class="block">
            <input
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
              v-model="description"
              v-on:keyup.enter="handleSubmitForm"
              placeholder="Enter service description"
            />
          </label>
        </div>
      </div>
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <div></div>
        <div></div>
        <div class="mt-5 grid-cols-2">
          <!--Clear Search button-->
          <button
            class="mr-10 border border-red-700 bg-white text-red-700 rounded"
            @click="loadData"
            type="submit"
          >
            Clear Search
          </button>
          <!--Search Service button-->
          <button
            class="bg-red-700 text-white rounded"
            @click="handleSubmitForm"
            type="submit"
          >
            Search Service
          </button>
        </div>
      </div>
    </div>

    <hr class="mt-10 mb-10" />
    <!-- Display Found Data -->
    <div
      class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
    >
      <!--List of Services table-->
      <div class="ml-10">
        <h2 class="text-2xl font-bold">List of Services</h2>
        <h3 class="italic">Click table row to view service details</h3>
      </div>
      <div class="flex flex-col col-span-4 pl-60 mb-20 pr-60">
        <table class="min-w-full shadow-md rounded">
          <thead class="bg-gray-50 text-xl">
            <tr>
              <th class="p-4 text-left" style="width: 25%">Service Name</th>
              <th class="p-4 text-left" style="width: 75%">
                Service Description
              </th>
              <th class="p-4 text-left" style="width: 25%">Status</th>
              <th
                v-if="userRole === 'editor'"
                class="p-4 text-left"
                style="width: 75%"
              >
                Actions
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-300">
            <tr
              @click="
                $router.push({
                  name: 'servicedetails',
                  params: { id: service._id },
                })
              "
              v-for="service in services"
              :key="service._id"
              class="cursor-pointer"
              :class="{ hoverRow: hoverId === service._id }"
              @mouseenter="hoverId = service._id"
              @mouseleave="hoverId = null"
            >
              <td class="p-2 text-left- text-sm">
                <input
                  v-if="service.editing"
                  type="text"
                  v-model="service.name"
                  class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                />
                <span v-else>{{ service.name }}</span>
              </td>
              <td class="p-2 text-left text-sm">
                <input
                  v-if="service.editing"
                  type="text"
                  v-model="service.description"
                  class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                />
                <span v-else>{{ service.description }}</span>
              </td>
              <!-- 
              <td class="p-2 text-center text-sm">
                <input
                  v-if="service.editing"
                  type="text"
                  v-model="service.status"
                  class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                />
                <span v-else>{{ service.status }}</span>
              </td> -->

              <td class="p-2 text-center text-sm">
                <div v-if="service.editing" class="mt-1">
                  <label class="inline-flex items-center">
                    <input
                      type="radio"
                      class="form-radio"
                      value="Active"
                      v-model="service.status"
                    />
                    <span class="ml-2">Active</span>
                  </label>
                  <label class="inline-flex items-center ml-6">
                    <input
                      type="radio"
                      class="form-radio"
                      value="Inactive"
                      v-model="service.status"
                    />
                    <span class="ml-2">Inactive</span>
                  </label>
                </div>
                <span v-else>{{ service.status }}</span>
              </td>

              <!-- <td class="p-4 text-left">{{ service.name }}</td> -->
              <!-- <td class="p-4 text-left text-sm">{{ service.description }}</td> -->
              <td class="p-4 text-left text-sm">
                <!-- <button
                  class="border rounded-md bg-red-700 hover:bg-red-400 text-white"
                >
                  Update
                </button> -->
                <button
                  v-if="service.editing"
                  class="border rounded-md bg-green-700 hover:bg-green-400 text-white"
                  @click="confirmUpdate(service)"
                >
                  Confirm
                </button>

                <button
                  v-else
                  v-if="userRole === 'editor'"
                  class="border rounded-md bg-red-700 hover:bg-red-400 text-white py-2 px-7 text-left"
                  @click="toggleEditing(service)"
                >
                  Edit
                </button>
              </td>
              <td class="p-4 text-left text-sm">
                <button
                  v-if="userRole === 'editor'"
                  class="border rounded-md bg-red-700 hover:bg-red-400 text-white"
                  @click="handleDelete(service._id)"
                >
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </main>
</template>

<!-- references:
1. class code
2. https://vuejs.org/guide/extras/composition-api-faq.html
3. chatgpt -->
<script setup>
import { ref, onMounted } from "vue";
import {
  getServices,
  searchServices,
  updateService,
  deleteService,
} from "../api/api";
import { useToast } from "vue-toastification";
import { useLoggedInUserStore } from "../store/loggedInUser"; // Import your user store
import { required } from "@vuelidate/validators";

const toast = useToast();
const services = ref([]);
const searchBy = ref("");
const name = ref("");
const description = ref("");
const hoverId = ref(null);
const user = useLoggedInUserStore(); // Access the user object from your store
const userRole = user.role; // Access the user's role

const loadData = async () => {
  searchBy.value = "";
  name.value = "";
  description.value = "";

  try {
    const response = await getServices();
    services.value = response.map((service) => ({
      ...service,
      editing: false,
    }));
  } catch (error) {
    toast.error(`Error loading services: ${error}`);
  }
};

onMounted(loadData);

const handleSubmitForm = async () => {
  let query = {};

  if (searchBy.value === "Service Name" && name.value) {
    query = { searchBy: "name", name: name.value };
  } else if (searchBy.value === "Service Description" && description.value) {
    query = { searchBy: "description", description: description.value };
  }

  if (query.searchBy) {
    try {
      const response = await searchServices(query);
      services.value = response;
    } catch (error) {
      toast.error(`Error searching services: ${error}`);
    }
  }
};

const toggleEditing = (service) => {
  service.editing = !service.editing;
};

const confirmUpdate = async (service) => {
  try {
    await updateService(service._id, {
      name: service.name,
      description: service.description,
      status: service.status, // Include the status in the update
    });
    toast.success("Service updated successfully");
    service.editing = false; // Disable editing mode
  } catch (error) {
    toast.error(`Error updating service: ${error.message}`);
  }
};

async function handleDelete(serviceId) {
  try {
    await deleteService(serviceId);
    toast.success("Event deleted successfully");
    loadData(); // Reload data to reflect changes
  } catch (error) {
    toast.error(`Error deleting event: ${error.message}`);
  }
}
</script>
