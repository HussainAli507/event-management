<!-- This view displays a list of events. Users can search for events, and click on an event to be redirected to another component to view that event's details -->
<template>
  <main>
    <div>
      <!--Header-->
      <h1
        class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10"
      >
        List of Events
      </h1>
    </div>
    <div class="px-10 pt-20">
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <h2 class="text-2xl font-bold">Search Event By</h2>
        <!-- Displays Event Name or Event Date selection -->
        <div class="flex flex-col">
          <select
            class="rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            v-model="searchBy"
          >
            <option value="Event Name">Event Name</option>
            <option value="Event Date">Event Date</option>
          </select>
        </div>
        <!--Displays event name search field-->
        <div class="flex flex-col" v-if="searchBy === 'Event Name'">
          <label class="block">
            <input
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
              v-model="name"
              v-on:keyup.enter="handleSubmitForm"
              placeholder="Enter event name"
            />
          </label>
        </div>
        <!-- Displays event date search field -->
        <div class="flex flex-col" v-if="searchBy === 'Event Date'">
          <input
            class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            type="date"
            v-model="eventDate"
            v-on:keyup.enter="handleSubmitForm"
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
          <!--Search Event button-->
          <button
            class="bg-red-700 text-white rounded"
            @click="handleSubmitForm"
            type="submit"
          >
            Search Event
          </button>
        </div>
      </div>
    </div>

    <hr class="mt-10 mb-10" />
    <!-- Display Found Data -->
    <div
      class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
    >
      <!--Table showing List of Events-->
      <div class="ml-10">
        <h2 class="text-2xl font-bold">List of Events</h2>
        <h3 class="italic">Click table row to view event details</h3>
      </div>
      <div class="flex flex-col col-span-4 px-20 mb-20">
        <table class="min-w-full shadow-md rounded">
          <thead class="bg-gray-50 text-xl">
            <tr>
              <th class="p-4 text-left">Event Name</th>
              <th class="p-4 text-left">Event Date</th>
              <th class="p-4 text-left">Event Address</th>
              <th class="p-4 text-left">Attendance</th>
              <th v-if="userRole === 'editor'" class="p-4 text-center">
                Actions
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-300">
            <tr
              @click="
                $router.push({
                  name: 'eventdetails',
                  params: { id: event._id },
                })
              "
              v-for="event in events"
              :key="event._id"
              class="cursor-pointer"
              :class="{ hoverRow: hoverId === event._id }"
              @mouseenter="hoverId = event._id"
              @mouseleave="hoverId = null"
            >
              <!-- <td class="p-2 text-left">{{ event.name }}</td> -->
              <td class="p-2 text-left">
                <!-- Display editable field if event is being edited -->
                <input v-if="event.editing" v-model="event.name" />
                <!-- Otherwise, display plain text -->
                <span v-else>{{ event.name }}</span>
              </td>
              <!-- <td class="p-2 text-left">{{ formatDate(event.date) }}</td> -->

              <td class="p-2 text-left">
                <!-- Display editable field if event is being edited -->
                <input v-if="event.editing" v-model="event.date" type="date" />
                <!-- Otherwise, display formatted date -->
                <span v-else>{{ formatDate(event.date) }}</span>
              </td>
              <!-- <td class="p-2 text-left">{{ event.address.line1 }}</td> -->
              <td class="p-2 text-left">
                <!-- Display editable field if event is being edited -->
                <input v-if="event.editing" v-model="event.address.line1" />
                <!-- Otherwise, display plain text -->
                <span v-else>{{ event.address.line1 }}</span>
              </td>

              <td class="p-2 text-center pr-10">
                <!-- Display editable field if event is being edited -->
                <input v-if="event.editing" v-model="event.attendees.length" />
                <!-- Otherwise, display plain text -->
                <span v-else>{{ event.attendees.length }}</span>
              </td>
              <td class="p-2 text-left">
                <button
                  v-if="event.editing"
                  class="border rounded-md bg-green-700 hover:bg-green-400 text-white"
                  @click="confirmUpdate(event)"
                >
                  Confirm
                </button>

                <button
                  v-else
                  v-if="userRole === 'editor'"
                  class="border rounded-md bg-red-700 hover:bg-red-400 text-white py-2 px-7 text-left"
                  @click="toggleEditing(event)"
                >
                  Edit
                </button>
              </td>
              <td class="p-2 text-left">
                <button
                  v-if="userRole === 'editor'"
                  class="border rounded-md bg-red-700 hover:bg-red-400 text-white"
                  @click="handleDelete(event._id)"
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
import { getEvents, searchEvents, updateEvent, deleteEvent } from "../api/api";
import { useLoggedInUserStore } from "../store/loggedInUser"; // Import your user store
import { useToast } from "vue-toastification";

const toast = useToast();
// Inside your setup function
const user = useLoggedInUserStore(); // Access the user object from your store
const userRole = user.role; // Access the user's role
//variable to hold all events for the organization
const events = ref([]);
const searchBy = ref("");
const name = ref("");
const eventDate = ref("");
// variable stores the ID of the row that the mouse is currently hovering over (to highlight the row red)
const hoverId = ref(null);

const loadData = async () => {
  // Resets variables used for search
  searchBy.value = "";
  name.value = "";
  eventDate.value = "";

  try {
    const response = await getEvents();
    events.value = response;
  } catch (error) {
    toast.error("loadData error", error);
  }
};

onMounted(loadData);

const handleSubmitForm = async () => {
  // if user searches by event name
  if (searchBy.value === "Event Name" && name.value) {
    try {
      const query = {
        searchBy: "name",
        name: name.value,
      };
      const response = await searchEvents(query);
      events.value = response;
    } catch (error) {
      toast.error("Error searching events", error);
    }
  }
  // if user searches by event date
  else if (searchBy.value === "Event Date" && eventDate.value) {
    try {
      const eventDateObj = new Date(eventDate.value);
      const formattedDate = eventDateObj.toISOString().substring(0, 10);
      const query = {
        searchBy: "date",
        eventDate: formattedDate,
      };
      const response = await searchEvents(query);
      events.value = response;
    } catch (error) {
      toast.error("Error searching events:", error);
    }
  }
};

// Add a new property 'editing' to each event object to track editing state
events.value.forEach((event) => {
  event.editing = false;
});

// Function to toggle editing mode for an event
const toggleEditing = (event) => {
  event.editing = !event.editing;
};

// Function to confirm the update of an event
const confirmUpdate = async (event) => {
  try {
    await updateEvent(event._id, event);
    toast.success("Event updated successfully");
    loadData(); // Reload data to reflect changes
  } catch (error) {
    toast.error(`Error updating event: ${error.message}`);
  }
};

// In your handleDelete function
async function handleDelete(eventId) {
  try {
    const response = await deleteEvent(eventId);
    toast.success("Event deleted successfully");
    loadData(); // Reload data to reflect changes
  } catch (error) {
    if (error.response && error.response.data) {
      // More specific error information from server
      console.error("Detailed error information:", error.response.data);
      toast.error(`Error deleting event: ${error.response.data}`);
    } else {
      // Handling cases where error.response is undefined
      console.error("Error without response:", error);
      toast.warning("Cannot Delete service is selected.");
    }
  }
}

// method called for each event date so it will be formatted correctly on the table
const formatDate = (date) => {
  const isoDate = new Date(date);
  return `${isoDate.getUTCFullYear()}-${String(isoDate.getUTCMonth() + 1).padStart(2, "0")}-${String(isoDate.getUTCDate()).padStart(2, "0")}`;
};
</script>
