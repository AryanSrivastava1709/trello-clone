<script setup>
// imports
import { ref, reactive, computed, watchEffect } from "vue";
import { v4 as uuidv4 } from "uuid";

//data variables
const isModalOpen = ref(false);
let isEditing = ref(false);
let draggedTaskId = ref(null);
const notStartedCount = computed(
  () => storedTasks.filter((task) => task.status === "not started").length
);
const inProgressCount = computed(
  () => storedTasks.filter((task) => task.status === "in progress").length
);
const completedCount = computed(
  () => storedTasks.filter((task) => task.status === "completed").length
);

const task = reactive({
  id: "",
  title: "",
  description: "",
  status: "",
});

let storedTasks = reactive([]);
onMounted(() => {
  let tasks = JSON.parse(localStorage.getItem("tasks")) || [];
  storedTasks.push(...tasks);
});

//methods
function dragStart(id) {
  draggedTaskId.value = id;
}

function dragEnd() {
  draggedTaskId.value = null;
}

function drop(status) {
  if (draggedTaskId.value) {
    const task = storedTasks.find((task) => task.id === draggedTaskId.value);
    if (task) {
      task.status = status;
      let tasks = JSON.parse(localStorage.getItem("tasks")) || [];
      const localStorageIndex = tasks.findIndex(
        (task) => task.id === draggedTaskId.value
      );
      if (localStorageIndex !== -1) {
        tasks[localStorageIndex] = { ...task };
        localStorage.setItem("tasks", JSON.stringify(tasks));
      }
    }
  }
}

function openModal() {
  isModalOpen.value = true;
}

function createTask() {
  let tasks = JSON.parse(localStorage.getItem("tasks")) || [];
  task.id = uuidv4();
  let taskCopy = { ...task };

  tasks.push(taskCopy);

  localStorage.setItem("tasks", JSON.stringify(tasks));
  storedTasks.push(taskCopy);
  task.title = "";
  task.description = "";
  task.status = "";
  isModalOpen.value = false;
}

function deleteTask(taskId) {
  let tasks = JSON.parse(localStorage.getItem("tasks")) || [];

  const index = tasks.findIndex((task) => task.id === taskId);
  if (index !== -1) {
    tasks.splice(index, 1);
  }

  localStorage.setItem("tasks", JSON.stringify(tasks));

  const storedIndex = storedTasks.findIndex((task) => task.id === taskId);
  if (storedIndex !== -1) {
    storedTasks.splice(storedIndex, 1);
  }
}
function editTask(taskId) {
  const taskToEdit = storedTasks.find((task) => task.id === taskId);
  if (!taskToEdit) {
    console.error(`No task found with id ${taskId}`);
    return;
  }

  task.id = taskToEdit.id;
  task.title = taskToEdit.title;
  task.description = taskToEdit.description;
  task.status = taskToEdit.status;

  isModalOpen.value = true;
  isEditing.value = true;
}
function saveTask() {
  const index = storedTasks.findIndex(
    (storedTask) => storedTask.id === task.id
  );
  if (index === -1) {
    console.error(`No task found with id ${task.id}`);
    return;
  }

  storedTasks[index] = { ...task };

  let tasks = JSON.parse(localStorage.getItem("tasks")) || [];
  const localStorageIndex = tasks.findIndex(
    (storedTask) => storedTask.id === task.id
  );
  if (localStorageIndex !== -1) {
    tasks[localStorageIndex] = { ...task };
    localStorage.setItem("tasks", JSON.stringify(tasks));
  }

  task.id = "";
  task.title = "";
  task.description = "";
  task.status = "";
  isModalOpen.value = false;
  isEditing.value = false;
}
</script>

<template>
  <div class="font-sans text-gray-900 antialiased">
    <v-app>
      <v-app-bar app color="primary" class="shadow-md">
        <v-spacer></v-spacer>
        <v-toolbar-title
          class="text-center font-weight-bold text-sm sm:text-base md:text-lg lg:text-xl"
          >Trello Clone</v-toolbar-title
        >
        <v-spacer></v-spacer>
      </v-app-bar>
      <div class="flex items-center justify-center mt-36 gap-10">
        <v-btn
          color="primary"
          @click="openModal"
          class="px-5 py-2 font-semibold"
        >
          + Create New Task
        </v-btn>
      </div>

      <div class="mt-20 mx-6 flex flex-wrap justify-between">
        <div class="mb-10 w-full md:w-1/3 px-2">
          <h2 class="text-2xl font-bold mb-4 text-center text-red-500">
            Not Started
            <span class="ml-10 text-gray-500">
              {{ notStartedCount }}
            </span>
          </h2>
          <div
            class="border-b-2 border-gray-200 p-5"
            @dragover.prevent
            @drop="drop('not started')"
          >
            <div
              v-for="(storedTask, index) in storedTasks.filter(
                (task) => task.status === 'not started'
              )"
              draggable="true"
              @dragstart="dragStart(storedTask.id)"
              @dragend="dragEnd(storedTask.id)"
              :key="index"
              class="bg-white shadow-md rounded-lg p-6 mb-4 flex justify-between items-start"
            >
              <div>
                <h2 class="text-xl font-bold mb-2">{{ storedTask.title }}</h2>
                <p class="mb-2">{{ storedTask.description }}</p>
              </div>
              <div class="flex flex-col">
                <button
                  @click="editTask(storedTask.id)"
                  class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-1 px-2 rounded mb-2"
                >
                  Edit
                </button>
                <button
                  @click="deleteTask(storedTask.id)"
                  class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-2 rounded"
                >
                  Delete
                </button>
              </div>
            </div>
          </div>
        </div>

        <div class="mb-10 w-full md:w-1/3 px-2">
          <h2 class="text-2xl font-bold mb-4 text-center text-yellow-500">
            In Progress
            <span class="ml-10 text-gray-500">
              {{ inProgressCount }}
            </span>
          </h2>
          <div
            class="border-b-2 border-gray-200 p-5"
            @dragover.prevent
            @drop="drop('in progress')"
          >
            <div
              v-for="(storedTask, index) in storedTasks.filter(
                (task) => task.status === 'in progress'
              )"
              draggable="true"
              @dragstart="dragStart(storedTask.id)"
              @dragend="dragEnd(storedTask.id)"
              :key="index"
              class="bg-white shadow-md rounded-lg p-6 mb-4 flex justify-between items-start"
            >
              <div>
                <h2 class="text-xl font-bold mb-2">{{ storedTask.title }}</h2>
                <p class="mb-2">{{ storedTask.description }}</p>
              </div>
              <div class="flex flex-col">
                <button
                  @click="editTask(storedTask.id)"
                  class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-1 px-2 rounded mb-2"
                >
                  Edit
                </button>
                <button
                  @click="deleteTask(storedTask.id)"
                  class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-2 rounded"
                >
                  Delete
                </button>
              </div>
            </div>
          </div>
        </div>

        <div class="mb-10 w-full md:w-1/3 px-2">
          <h2 class="text-2xl font-bold mb-4 text-center text-green-500">
            Completed
            <span class="ml-10 text-gray-500">
              {{ completedCount }}
            </span>
          </h2>
          <div
            class="border-b-2 border-gray-200 p-5"
            @dragover.prevent
            @drop="drop('completed')"
          >
            <div
              v-for="(storedTask, index) in storedTasks.filter(
                (task) => task.status === 'completed'
              )"
              draggable="true"
              @dragstart="dragStart(storedTask.id)"
              @dragend="dragEnd(storedTask.id)"
              :key="index"
              class="bg-white shadow-md rounded-lg p-6 mb-4 flex justify-between items-start"
            >
              <div>
                <h2 class="text-xl font-bold mb-2">{{ storedTask.title }}</h2>
                <p class="mb-2">{{ storedTask.description }}</p>
              </div>
              <div class="flex flex-col">
                <button
                  @click="editTask(storedTask.id)"
                  class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-1 px-2 rounded mb-2"
                >
                  Edit
                </button>
                <button
                  @click="deleteTask(storedTask.id)"
                  class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-2 rounded"
                >
                  Delete
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <v-dialog v-model="isModalOpen" class="my-8">
        <div class="flex items-center justify-center my-auto mx-auto">
          <div
            class="bg-white w-full sm:w-3/4 md:w-80 lg:w-[500px] h-auto rounded-lg shadow-md"
          >
            <div class="px-6 py-4 font-bold text-2xl">
              {{ isEditing ? "Edit Task" : "Create New Task" }}
            </div>
            <div class="px-6 py-4">
              <label class="block mb-2 font-semibold text-xl">Title</label>
              <input
                v-model="task.title"
                class="w-full px-3 py-2 text-gray-700 border rounded-lg focus:outline-none"
                type="text"
                placeholder="Title"
              />
              <label class="block mt-4 mb-2 font-semibold text-xl"
                >Description</label
              >
              <textarea
                v-model="task.description"
                class="w-full px-5 py-5 text-gray-700 border rounded-lg focus:outline-none"
                placeholder="Description"
              ></textarea>
              <label class="block mt-4 mb-2 font-semibold text-xl"
                >Status</label
              >
              <select
                v-model="task.status"
                class="w-full px-3 py-2 text-gray-700 border rounded-lg focus:outline-none"
              >
                <option disabled value="">Please select a status</option>
                <option value="completed">Completed</option>
                <option value="in progress">In Progress</option>
                <option value="not started">Not Started</option>
              </select>
            </div>
            <div class="px-6 py-4 flex justify-end">
              <button
                class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
                @click="isModalOpen = false"
              >
                Cancel
              </button>
              <button
                class="ml-2 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
                @click="isEditing ? saveTask() : createTask()"
              >
                {{ isEditing ? "Save" : "Create" }}
              </button>
            </div>
          </div>
        </div>
      </v-dialog>
    </v-app>
  </div>
</template>

<style scoped>
.font-large {
  font-size: 2rem;
}
</style>
