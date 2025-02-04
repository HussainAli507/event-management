<!-- This view allows a user to update a specific service's information. -->
<template>
  <main>
    <!--Header-->
    <h1 class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10">
      Service Details
    </h1>
    <div class="px-10 py-20">
        <!-- grid container -->
        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10">
          <h2 class="text-2xl font-bold">Service Details</h2>
          <!-- Service Name input field -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Service Name</span>
              <span style="color: #ff0000">*</span>
              <input type="text"
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="service.name" />
            </label>
            <span v-if="v$.service.name.$error" class="text-red-500">
              Service Name is required
            </span>
          </div>

          <!-- Description input field -->
          <div class="flex flex-col">
            <label>
              <span class="text-gray-700">Description:</span>
              <textarea
                class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                v-model="service.description"></textarea>
            </label>
          </div>
        </div>

        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10">
          <div></div>
          <!-- form field: update service description -->
          <div class="flex flex-col">
            <label class="block">
              <span class="text-gray-700">Service Status</span>
              <span style="color: #ff0000">*</span>
              <div class="mt-1">
                <label class="inline-flex items-center">
                  <input type="radio" class="form-radio" value="Active" v-model="service.status">
                  <span class="ml-2">Active</span>
                </label>
                <label class="inline-flex items-center ml-6">
                  <input type="radio" class="form-radio" value="Inactive" v-model="service.status">
                  <span class="ml-2">Inactive</span>
                </label>
              </div>
            </label>
          </div>
        </div>

        <!-- grid container -->
        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10">
          <div></div>
          <!-- Update Service button -->
          <div class="flex justify-between mt-10 mr-20">
            <div class="flex justify-between mt-10 mr-25">
              <button @click="submitServiceUpdate" type="submit" class="bg-green-700 disabled:opacity-50 text-white rounded"
                :disabled="user.role === 'viewer'">
                Update Service
              </button>
            </div>
          </div>
          <!--Delete Service button-->
          <div class="flex justify-between mt-20 mr-20">
            <button @click="submitDeleteService" type="submit" class="bg-red-700 disabled:opacity-50 text-white rounded"
              :disabled="user.role === 'viewer'">
              Delete Service
            </button>
          </div>
          <!--Go back button-->
          <div class="flex justify-between mt-20 mr-20">
            <button type="reset" class="border border-red-700 bg-white text-red-700 rounded" @click="$router.back()">
              Go back
            </button>
          </div>
        </div>
    </div>

    <hr class="mt-10 mb-10" />
    <!-- Display Found Data -->
    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10">
      <!--List of Events table-->
      <div class="ml-10">
        <h2 class="text-2xl font-bold">List of Events</h2>
        <h3 class="italic">Click table row to view event details</h3>
      </div>
      <div class="flex flex-col col-span-2">
        <table class="min-w-full shadow-md rounded">
          <thead class="bg-gray-50 text-xl">
            <tr>
              <th class="p-4 text-left">Event Name</th>
              <th class="p-4 text-left">Event Date</th>
              <th class="p-4 text-left">Event Address</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-300">
            <tr @click="$router.push({name: 'eventdetails', params: { id: event._id } })" 
            v-for="event in events" :key="event._id" class="cursor-pointer"
              :class="{ 'hoverRow': hoverId === event._id }" @mouseenter="hoverId = event._id"
              @mouseleave="hoverId = null">
              <td class="p-2 text-left">{{ event.name }}</td>
              <td class="p-2 text-left">{{ formatDate(event.date) }}</td>
              <td class="p-2 text-left">{{ event.address.line1 }}</td>
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
3. chatgpt
4. https://stackoverflow.com/questions/66844015/vue-3-composition-api-setting-date-input-using-the-valueasdate-property -->

<script setup>
import { ref, onMounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { useVuelidate, required } from '@vuelidate/core'
import { useLoggedInUserStore } from "../store/loggedInUser";
import { getServiceById, getEventsByServiceId, updateService, deleteService } from '../api/api';
import { useToast } from 'vue-toastification';

const router = useRouter();
const route = useRoute();
const toast = useToast();
const user = useLoggedInUserStore();

// variable to store service info
const service = ref({
  name: '',
  description: '',
  status: ''
});

// all events that host the service
const events = ref([]);
// var stores the ID of the row that the mouse is currently on
const hoverId = ref(null);

const v$ = useVuelidate({
  service: {
    name: { required },
  }
}, service);

onMounted(async () => {
  try {
    const [serviceResponse, eventsResponse] = await Promise.all([
      getServiceById(route.params.id),
      getEventsByServiceId(route.params.id),
    ]);

    service.value = serviceResponse;
    events.value = eventsResponse;
  } catch (error) {
    toast.error(`Error loading data: ${error}`);
  }
});

// method to format the event dates
const formatDate = (date) => {
  const isoDate = new Date(date);
  return `${isoDate.getUTCFullYear()}-${String(isoDate.getUTCMonth() + 1).padStart(2, '0')}-${String(isoDate.getUTCDate()).padStart(2, '0')}`;
};

const submitServiceUpdate = async () => {
  await v$.value.$validate();

  if (v$.value.$error) {
    return; // Form is invalid, do not proceed
  }

  try {
    await updateService(route.params.id, service.value);
    toast.success('Service updated successfully!');
    router.push('/findservice');
  } catch (error) {
    toast.error(`Error updating service: ${error}`);
  }
};

const submitDeleteService = async () => {
  if (events.value.length !== 0) {
    toast.info('Service cannot be deleted since it is being used by events.');
    return;
  }

  try {
    await deleteService(route.params.id);
    toast.success('Service deleted successfully');
    router.push('/findservice');
  } catch (error) {
    toast.error(`Error deleting service: ${error}`);
  }
};
</script>
