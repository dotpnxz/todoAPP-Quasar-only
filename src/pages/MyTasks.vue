<template>
  <div class="flex justify-center">
    <div class="full-width q-pa-xl">
      <h6 class="q-my-none q-mb-md">My Tasks</h6>
      <q-form ref="form" @submit="!selectedTodo ? addTodo() : updateTodo()">
        <div class="row gap">
          <q-input
            dense
            outlined
            label="What's your todo?"
            class="q-pa-none col-10"
            lazy-rules
            :rules="[(val) => !!val || 'Field is required']"
            v-model="text"
          />
          <div class="col-2 q-pl-sm">
            <q-btn
              class="bg-primary fit"
              color="white"
              glossy
              dense
              flat
              type="submit"
              :icon="!selectedTodo ? 'add' : 'edit'"
            />
          </div>
        </div>

        <q-card
          @click="selectTodo(row)"
          class="q-mt-sm"
          v-for="row in MyTasks"
          :key="row.id"
        >
          <q-card-section
            :class="`bg-${
              selectedTodo && selectedTodo.id === row.id ? 'secondary' : 'primary'
            } text-white q-pa-none flex justify-between items-center`"
          >
            <div class="text-bold q-pl-lg">{{ row.todo }}</div>
            <div class="bg-white q-pa-sm">
              <q-btn
                @click.stop="toggleDialog(row, 'mark-done')"
                flat
                icon="check_circle_outline"
                color="green"
              />
              <q-btn 
              @click.stop="toggleDialog(row, 'delete')"
                flat
                icon="delete_outline" color="red" />
            </div>
          </q-card-section>
        </q-card>
      </q-form>

      <q-dialog v-model="showDialog" persistent>
  <q-card>
    <q-card-section class="row items-center">
      <div v-if="toMarkAsDone" class="q-ml-sm">
        Are you sure you want to mark
        <span class="text-green">"{{ toMarkAsDone.todo }}"</span> as done?
      </div>
      <div v-else-if="toDelete" class="q-ml-sm">
        Are you sure you want to delete
        <span class="text-green">"{{ toDelete.todo }}"</span>?
      </div>
    </q-card-section>

    <q-card-actions align="right">
      <q-btn flat label="Cancel" color="primary" v-close-popup />
      <q-btn
        @click="toMarkAsDone ? markAsDone() : removeTodo()"
        flat
        label="Yes"
        color="primary"
        v-close-popup
      />
    </q-card-actions>
  </q-card>
</q-dialog>

    </div>
  </div>
</template>


<script setup>
import { ref } from "vue";
import axios from 'axios';
import { MyTasks, FinishedTasks, DeletedTasks } from "../composables/Tasks";



// Reactive variables
const text = ref(null);
const form = ref(null);
const selectedTodo = ref(null);
const showDialog = ref(false);

// Variables for dialog actions
const toMarkAsDone = ref(null);
const toDelete = ref(null);

// Add a new todo
const addTodo = () => {
  if (text.value) {
    MyTasks.value.push({
      id: MyTasks.value.length + 1,
      todo: text.value,
    });
    text.value = ""; // Clear the input field
  }
};

// Select a todo for editing
const selectTodo = (row) => {
  selectedTodo.value = row;
  text.value = row.todo;
};

// Update an existing todo
const updateTodo = () => {
  let index = MyTasks.value.findIndex((t) => t.id === selectedTodo.value.id);
  if (index !== -1) {
    MyTasks.value[index].todo = text.value;
  }
  text.value = null;
  selectedTodo.value = null;
  form.value.reset();
};

// Toggle the dialog for specific actions
const toggleDialog = (row, status) => {
  showDialog.value = true;
  toMarkAsDone.value = null;
  toDelete.value = null;

  if (status === "mark-done") {
    toMarkAsDone.value = row;
  } else if (status === "delete") {
    toDelete.value = row;
  }
};

// Mark a task as done
const markAsDone = () => {
  let index = MyTasks.value.findIndex((t) => t.id === toMarkAsDone.value.id);
  if (index !== -1) {
    FinishedTasks.value.push(toMarkAsDone.value);
    MyTasks.value.splice(index, 1);
  }
  toMarkAsDone.value = null;
  showDialog.value = false;
};

// Remove a todo
const removeTodo = () => {
  let index = MyTasks.value.findIndex((t) => t.id === toDelete.value.id);
  if (index !== -1) {
    DeletedTasks.value.push(toDelete.value);
    MyTasks.value.splice(index, 1);
  }
  toDelete.value = null;
  showDialog.value = false;
};
</script>

