<script lang="ts" setup>
import { NButton, NInput, NSelect, useMessage } from 'naive-ui'
import { reactive,defineEmits } from 'vue' 

// Define the event the child can emit
const emit = defineEmits(['close'])

const message = useMessage();

const prioritySelectOptions = [
  {value : -1, label: "Priority", disabled: true},
  {value : 0, label: "low", style: {color: 'green'} },
  {value : 1, label: "normal", style: {color: 'blue'}},
  {value : 2, label: "high", style: {color: '#A0A020'} },
  {value : 3, label: "urgent", style: {color: 'red'}},
];

const Task = reactive({
  title: '',
  priority: 0
})

// Function to create a task
const createTask = async () => {
  if (!Task.title  ) {
    message.error('Please fill in both the title!')
    return
  }

  fetch(`${import.meta.env.VITE_API_URL}/api/todo`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify( { title : Task.title, priority : Task.priority}) 
})
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error("Error:", error));

  message.success('Task created successfully!')

  // Reset form
  Task.title = ''

  emit('close')

}
</script>

<template>
  <div class="task-form bg-light p-4 rounded shadow d-flex flex-column gap-4">
    <h1>Create New Task</h1>

    <div class="d-flex flex-row gap-3">
      <NInput 
        v-model:value="Task.title" 
        placeholder="Task Title" 
        clearable
      />  
      <NSelect 
        v-model:value="Task.priority" 
        placeholder="Task Priority"
        :options="prioritySelectOptions"
        clearable
      /> 
    </div>

    <div class="d-flex justify-end">
      <NButton @click="createTask" type="primary">Create</NButton>
    </div>
  </div>
</template>

<style scoped>
.task-form {
  max-width: 600px;
  margin: 20px auto;
}

h1 {
  font-size: 1.8rem;
  margin-bottom: 20px;
}

.d-flex {
  display: flex;
}

.justify-end {
  justify-content: flex-end;
}

.gap-3 {
  gap: 1rem;
}

.gap-4 {
  gap: 1.5rem;
}
</style>
