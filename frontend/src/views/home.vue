<!-- This is the home view - which shows a dashboard -->
<template>
  <main>
    <div>
      <h1
        class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10"
      >
        Dashboard
      </h1>
      <br />
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <div class="ml-10"></div>
        <div class="flex flex-col col-span-4 px-60">
          <h1
            class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10 mb-10"
          >
            Events Attendance
          </h1>

          <div v-if="!recentEvents.length" class="flex justify-center mt-10">
            No events found.
          </div>
          <table
            v-if="recentEvents.length"
            class="min-w-full shadow-md rounded"
          >
            <thead class="bg-gray-50 text-xl">
              <tr class="p-4 text-left">
                <th class="p-4 text-left">Event Name</th>
                <th class="p-4 text-left">Event Date</th>
                <th class="p-4 text-left">Number of Attendees</th>
                <!-- <th class="p-4 text-left">Actions</th> -->
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-300">
              <tr
                @click="editEvent(event._id)"
                v-for="event in recentEvents"
                :key="event._id"
              >
                <!-- <td class="p-2 text-left">{{ event.name }}</td> -->
                <td class="p-2 text-left">
                  <!-- Display editable field if event is being edited -->
                  <input v-if="event.editing" v-model="event.name" />
                  <!-- Otherwise, display plain text -->
                  <span v-else>{{ event.name }}</span>
                </td>

                <td class="p-2 text-left">
                  <!-- Display editable field if event is being edited -->
                  <input
                    v-if="event.editing"
                    v-model="event.date"
                    type="date"
                  />
                  <!-- Otherwise, display formatted date -->
                  <span v-else>{{ formatDate(event.date) }}</span>
                </td>

                <td class="p-2 text-center">
                  <!-- Display editable field if event is being edited -->
                  <input
                    v-if="event.editing"
                    v-model="event.attendees.length"
                  />
                  <!-- Otherwise, display plain text -->
                  <span v-else>{{ event.attendees.length }}</span>
                </td>
                <!-- <td class="p-2 text-center">
                  <button
                    v-if="event.editing"
                    class="border rounded-md bg-green-700 hover:bg-green-400 text-white"
                    @click="confirmUpdate(event)"
                  >
                    Confirm
                  </button>

                  <button
                    v-else
                    class="border rounded-md bg-red-700 hover:bg-red-400 text-white py-2 px-7 text-left"
                    @click="toggleEditing(event)"
                  >
                    Edit
                  </button>
                </td> -->
                <!-- <td class="p-2 text-center">
                  <button
                    class="border rounded-md bg-red-700 hover:bg-red-400 text-white"
                    @click="handleDelete(event._id)"
                  >
                    Delete
                  </button>
                </td> -->
              </tr>
            </tbody>
          </table>

          <div v-if="recentEvents.length">
            <AttendanceChart
              v-if="!loading && !error"
              :label="labels"
              :chart-data="chartData"
            ></AttendanceChart>

            <!-- Start of loading animation -->
            <div class="mt-40" v-if="loading">
              <p
                class="text-6xl font-bold text-center text-gray-500 animate-pulse"
              >
                Loading...
              </p>
            </div>
            <!-- End of loading animation -->

            <!-- Start of error alert -->
            <div class="mt-12 bg-red-50" v-if="error">
              <h3 class="px-4 py-1 text-4xl font-bold text-white bg-red-800">
                {{ error.title }}
              </h3>
              <p class="p-4 text-lg font-bold text-red-900">
                {{ error.message }}
              </p>
            </div>
            <!-- End of error alert -->
          </div>
        </div>
      </div>
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <div class="ml-10"></div>
        <div class="flex flex-col col-span-2">
          <h1
            class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10"
          >
            Clients by Zip Code
          </h1>

          <div v-if="!zips.length" class="flex justify-center mt-10">
            No clients found.
          </div>
          <table v-if="zips.length" class="min-w-full shadow-md rounded">
            <thead class="bg-gray-50 text-xl">
              <tr class="p-4 text-left">
                <th class="p-4 text-left">Zip Number</th>
                <th class="p-4 text-left">Client Count</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-300">
              <tr v-for="zip in zips" :key="zip._id">
                <td class="p-2 text-left">{{ zip._id }}</td>
                <td class="p-2 text-left">{{ zip.count }}</td>
              </tr>
            </tbody>
          </table>
          <div v-if="zips.length" class="flex justify-center mt-10">
            <ZipChart
              v-if="!zipLoading && !zipError"
              :label="zipLabels"
              :chart-data="zipChartData"
            ></ZipChart>

            <!-- Start of loading animation -->
            <div class="mt-40" v-if="zipLoading">
              <p
                class="text-6xl font-bold text-center text-gray-500 animate-pulse"
              >
                Loading ^.^ ...
              </p>
            </div>
            <!-- End of loading animation -->

            <!-- Start of error alert -->
            <div class="mt-12 bg-red-50" v-if="zipError">
              <h3 class="px-4 py-1 text-4xl font-bold text-white bg-red-800">
                {{ zipError.title }}
              </h3>
              <p class="p-4 text-lg font-bold text-red-900">
                {{ zipError.message }}
              </p>
            </div>
            <!-- End of error alert -->
          </div>
        </div>
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
import AttendanceChart from "../components/barChart.vue";
import ZipChart from "../components/donutZipChart.vue";
import { getAttendance, getClientsByZipCode } from "../api/api";
import { useRouter } from "vue-router"; // Import useRouter function
import { useToast } from "vue-toastification";

const router = useRouter(); // Access the router object

const toast = useToast();
const events = ref([]);
const recentEvents = ref([]);
const zips = ref([]);
const labels = ref([]);
const chartData = ref([]);
const zipLabels = ref([]);
const zipChartData = ref([]);
const loading = ref(false);
const error = ref(null);
const zipLoading = ref(false);
const zipError = ref(null);

const formatDate = (date) => {
  const isoDate = new Date(date);
  return `${String(isoDate.getUTCMonth() + 1).padStart(2, "0")}/${String(isoDate.getUTCDate()).padStart(2, "0")}/${isoDate.getUTCFullYear()}`;
};

const getAttendanceData = async () => {
  try {
    error.value = null;
    loading.value = true;
    const attendance = await getAttendance();
    recentEvents.value = attendance;
    labels.value = attendance.map(
      (item) => `${item.name} (${formatDate(item.date)})`
    );
    chartData.value = attendance.map((item) => item.attendees.length);
    // client received an error response (5xx, 4xx)
  } catch (err) {
    error.value = {
      // There's probably an error in your code
      title: "Error Loading Attendance Data",
      message: err.message || "An unknown error occurred.",
    };
  } finally {
    loading.value = false;
  }
};

const editEvent = async (eventId) => {
  try {
    // Assuming you want to navigate to the update page with the event ID as a parameter
    router.push({ name: "update-event", params: { id: eventId } });
    // Or you can execute any update logic here
  } catch (error) {
    console.error("Error updating event:", error);
  }
};

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
      toast.error("Network or server error occurred.");
    }
  }
}

const getZipData = async () => {
  try {
    zipError.value = null;
    zipLoading.value = true;
    const zipdata = await getClientsByZipCode();
    zips.value = zipdata;
    zipLabels.value = zipdata.map((item) => item._id);
    zipChartData.value = zipdata.map((item) => item.count);
    // client received an error response (5xx, 4xx)
  } catch (err) {
    zipError.value = {
      // client never received a response, or request never left
      title: "Error Loading Zip Data",
      message: err.message || "An unknown error occurred.",
    };
  } finally {
    zipLoading.value = false;
  }
};

onMounted(() => {
  // method called to format the event date
  getAttendanceData();
  getZipData();
});
</script>
