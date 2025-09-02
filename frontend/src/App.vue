<template>
  <NMessageProvider>
    <NDialogProvider>
      <div id="header">
        <img
          alt="Vue logo"
          src="./assets/logo.png"
          height="60"
          style="border-radius: 10px;"
        />
        <div>
          <NButton type="primary" style="margin-right: 10px;" @click="openAddTaskWindow = true">
            Add Task
          </NButton>
        </div>
      </div>

      <NModal v-model:show="openAddTaskWindow" title="Add New Task" closable>
        <TaskForm @close="handleClose" />
      </NModal>

      <br />

      <TasksList :tasks="tasks" :key="refresher" />
    </NDialogProvider>
  </NMessageProvider>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { NButton, NModal, NMessageProvider, NDialogProvider } from 'naive-ui';
import TasksList from './components/TasksList.vue';
import TaskForm from './components/TaskForm.vue';
import type { Task } from './assets/types.ts';

const openAddTaskWindow = ref<boolean>(false);
const refresher = ref<number>(0);


const tasks = ref<Task[]>([]);

// Methods
const refreshTasks = (): void => {
  refresher.value += 1;
};

const handleClose = (): void => {
  openAddTaskWindow.value = false;
  refreshTasks();
};

</script>

<style scoped>
#header {
  background-color: #cbeede;
  padding: 20px;
  color: white;
  height: 70px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>