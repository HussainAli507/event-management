<!-- This view displays a list of clients. The user can search for clients, and click on a client to redirect to another component to view that client's information -->
<template>
  <main>
    <div>
      <!--Header-->
      <h1
        class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10"
      >
        Find Client
      </h1>
    </div>
    <div class="px-10 pt-20">
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <!--Search Client By selection-->
        <h2 class="text-2xl font-bold">Search Client By</h2>
        <!-- Displays Client Name search field -->
        <div class="flex flex-col">
          <select
            class="rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            v-model="searchBy"
          >
            <option value="Client Name">Client Name</option>
            <option value="Client Number">Client Number</option>
          </select>
        </div>
        <!--Input box for searching by Client First Name-->
        <div class="flex flex-col" v-if="searchBy === 'Client Name'">
          <label class="block">
            <input
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
              v-model="firstName"
              v-on:keyup.enter="handleSubmitForm"
              placeholder="Enter first name"
            />
          </label>
        </div>
        <!--Input box for searching by Client Last Name-->
        <div class="flex flex-col" v-if="searchBy === 'Client Name'">
          <label class="block">
            <input
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
              v-model="lastName"
              v-on:keyup.enter="handleSubmitForm"
              placeholder="Enter last name"
            />
          </label>
        </div>
        <!-- Displays Client Number search field -->
        <div class="flex flex-col" v-if="searchBy === 'Client Number'">
          <input
            class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            type="text"
            v-model="phoneNumber"
            v-on:keyup.enter="handleSubmitForm"
            placeholder="Enter Client Phone Number"
          />
        </div>
      </div>
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <div></div>
        <div></div>
        <!--Clear Search button-->
        <div class="mt-5 grid-cols-2">
          <button
            class="mr-10 border border-red-700 bg-white text-red-700 rounded"
            @click="loadData"
            type="submit"
          >
            Clear Search
          </button>
          <!--Search Client button-->
          <button
            class="bg-red-700 text-white rounded"
            @click="handleSubmitForm"
            type="submit"
          >
            Search Client
          </button>
        </div>
      </div>
    </div>

    <hr class="mt-10 mb-10" />
    <div
      class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
    >
      <div class="ml-10">
        <h2 class="text-2xl font-bold">List of Clients</h2>
        <h3 class="italic">Click table row to view client details</h3>
      </div>
      <!--Table showing list of Clients-->
      <div class="flex flex-col col-span-3 pr-60 mb-20">
        <table class="min-w-full shadow-md rounded">
          <thead class="bg-gray-50 text-xl">
            <tr>
              <th class="p-4 text-left">Name</th>
              <th class="p-4 text-left">Phone number</th>
              <th class="p-4 text-left">City</th>
              <th v-if="userRole === 'editor'" class="p-4 text-left">
                Actions
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-300">
            <tr
              @click="
                $router.push({
                  name: 'clientdetails',
                  params: { id: client._id },
                })
              "
              v-for="client in clients"
              :key="client._id"
              class="cursor-pointer"
              :class="{ hoverRow: hoverId === client._id }"
              @mouseenter="hoverId = client._id"
              @mouseleave="hoverId = null"
            >
              <td class="p-2 text-left">
                <input
                  v-if="client.editing"
                  v-model="client.firstName"
                  class="form-input"
                />
                <span v-else>{{ client.firstName }}</span>
              </td>

              <td class="p-2 text-left">
                <input
                  v-if="client.editing"
                  v-model="client.phoneNumber.primary"
                  class="form-input"
                />
                <span v-else>{{ client.phoneNumber.primary }}</span>
              </td>

              <td class="p-2 text-left">
                <input
                  v-if="client.editing"
                  v-model="client.address.city"
                  class="form-input"
                />
                <span v-else>{{ client.address.city }}</span>
              </td>

              <td class="p-2 text-left">
                <button
                  v-if="client.editing"
                  @click="confirmUpdate(client)"
                  class="border rounded-md bg-green-700 hover:bg-green-400 text-white"
                >
                  Confirm
                </button>
                <button
                  v-else
                  v-if="userRole === 'editor'"
                  @click="toggleEditing(client)"
                  class="border rounded-md bg-red-700 hover:bg-red-400 text-white py-2 px-7 text-left"
                >
                  Edit
                </button>
              </td>

              <td class="p-2 text-left">
                <button
                  v-if="userRole === 'editor'"
                  @click="handleDelete(client._id)"
                  class="border rounded-md bg-red-700 hover:bg-red-400 text-white"
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
  getClients,
  searchClients,
  updateClient,
  deleteClientbyId,
} from "../api/api";
import { useLoggedInUserStore } from "../store/loggedInUser"; // Import your user store
import { useToast } from "vue-toastification";

const toast = useToast();
const user = useLoggedInUserStore(); // Access the user object from your store
const userRole = user.role; // Access the user's role
//variable to store all clients and their information for the organization
// const clients = ref(null);
// Variable to store all clients and their information for the organization
const clients = ref([]);
const searchBy = ref(""); // Parameters for search to occur
const firstName = ref("");
const lastName = ref("");
const phoneNumber = ref("");
// variable stores the ID of the row that the mouse is currently hovering over (to highlight the row red)
const hoverId = ref(null);

// // Method to load data when component is mounted
// const loadData = async () => {
//   // Resets all the variables
//   searchBy.value = "";
//   firstName.value = "";
//   lastName.value = "";
//   phoneNumber.value = "";

//   // get list of clients
//   try {
//     const response = await getClients();
//     clients.value = response;
//   } catch (error) {
//     toast.error(`Error loading clients: ${error}`);
//   }
// };

const loadData = async () => {
  try {
    const response = await getClients();
    clients.value = response.map((client) => ({
      ...client,
      editing: false, // Ensure the editing property is added
      phoneNumber: client.phoneNumber || { primary: "" }, // Initialize if undefined
      address: client.address || { city: "" }, // Initialize if undefined
    }));
  } catch (error) {
    toast.error(`Error loading clients: ${error}`);
  }
};

// when component is mounted, load the data
onMounted(loadData);

// Method called when user searches for clients
const handleSubmitForm = async () => {
  // if user searched by client name
  if (searchBy.value === "Client Name") {
    if (firstName.value || lastName.value) {
      try {
        const query = {
          searchBy: "name",
          firstName: firstName.value,
          lastName: lastName.value,
        };
        const response = await searchClients(query);
        clients.value = response;
      } catch (error) {
        toast.error(`Error searching clients: ${error}`);
      }
    }
    // if user searches by client phone number
  } else if (searchBy.value === "Client Number") {
    if (phoneNumber.value) {
      try {
        const query = {
          searchBy: "number",
          phoneNumber: phoneNumber.value,
        };
        const response = await searchClients(query);
        clients.value = response;
      } catch (error) {
        toast.error(`Error searching clients by phone number: ${error}`);
      }
    }
  }
};

// Add a new property 'editing' to each client object to track editing state
clients.value.forEach((client) => {
  client.editing = false;
});

// Function to toggle editing mode for a client
const toggleEditing = (selectedClient) => {
  clients.value.forEach((client) => {
    // Toggle only for the selected client and reset others
    if (client._id === selectedClient._id) {
      client.editing = !client.editing;
    } else {
      client.editing = false;
    }
  });
};

const confirmUpdate = async (client) => {
  if (client.editing) {
    try {
      await updateClient(client._id, {
        firstName: client.firstName,
        lastName: client.lastName,
        phoneNumber: client.phoneNumber.primary,
        address: client.address.city,
      });
      toast.success("Client updated successfully");
      client.editing = false; // Turn off editing mode after update
    } catch (error) {
      toast.error(`Error updating client: ${error.message}`);
    }
  }
};

async function handleDelete(clientId) {
  try {
    await deleteClientbyId(clientId);
    toast.success("Event deleted successfully");
    loadData(); // Reload data to reflect changes
  } catch (error) {
    toast.error(`Error deleting event: ${error.message}`);
  }
}
</script>
