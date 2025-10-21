<script lang="ts" setup>
import {  NDataTable, NSwitch, useMessage, NButton, NTag, useDialog } from 'naive-ui'
import { defineProps, h , onMounted, reactive, ref} from 'vue'
import type { Task } from '../assets/types';


const msg = useMessage();
const dialog = useDialog();

const tasks = ref<Task[]>([]);

defineProps({
  tasks: Array<Task>
})

const getPriorityType = (id: number) => {
  switch(id){
    case 0:
      return 'success'
    case 1:
      return 'info'
    case 2:
      return 'warning'
    case 3:
      return 'error'
    }
}
const getPriorityName = (id: number) => {
    switch(id){
    case 0:
      return 'Low'
    case 1:
      return 'Normal'
    case 2:
      return 'High'
    case 3:
      return 'Urgent'
    }
}

const columns = [
  {
    title: 'number',
    key: 'number',
    render: (_: Task, idx: number) => idx + 1
  },
  {
    title: 'Title',
    key: 'title',
  },
  {
    title: 'Completed',
    key: 'completed',
    render(row: Task) {
      return h(NSwitch, {
        value: row.isCompleted,
        disabled: row.isCompleted,
        "on-update:value": () => {
                    updateTaskStatus(row.id)
                }
      });
    }
  },
  {
    title: 'Priority',
    key: 'priority',
    render(row: Task){
      return h(NTag, {
        type: getPriorityType(row.priorityLevel),
        bordered: false}
        ,
        getPriorityName(row.priorityLevel)
      )
    }
  },

  {
    title: 'Delete',
    key: 'delete',
    render(row : Task) {
      return h(NButton, {
        type: 'error',
        disabled: !row.isCompleted,
        onClick: () => {
          dialog.error({
              title: 'Confirm',
              content: `Delete task "${row.title}" ?`,
              positiveText: 'Delete',
              negativeText: 'Cancel',
              draggable: true,
              onPositiveClick: () => {
                   fetch(`${import.meta.env.VITE_API_URL}/api/todo/${row.id}`, {
                     method: "DELETE",
                     headers: {
                       "Content-Type": "application/json"
                     },
                   })
                   .then(response => {
                     if (!response.ok) throw new Error('Failed to delete')
                     return response.json().catch(() => null)
                   })
                   .then(() => msg.success('Task deleted successfully!'))
                   .catch(error => msg.error("Error: " + error.message))
                   .finally(() => { refresh() })
              },
              onNegativeClick: () => {
                //fine
              }
            })
          }
      }, 'Delete')
    }
  }
]

const updateTaskStatus = async (id: string) => {
    fetch(`${import.meta.env.VITE_API_URL}/api/todo/${id}/complete`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
})
  .then(response => response.json())
  .then(data => data && msg.success('Task marked as completed!'))
  .catch(error => msg.error("Error:", error))
  .finally(() => { refresh() });

  

}

const isLoadingData = ref(false);

const refresh = async () => {
  try {
    isLoadingData.value = true; // start loading

    const response = await fetch(`${import.meta.env.VITE_API_URL}/api/todo`, {
      method: "GET",
    });

    if (!response.ok) {
      throw new Error("Failed to fetch tasks");
    }

    const data = await response.json();
    tasks.value = data;

    msg.success("Task list refreshed!");
  } catch (error) {
    console.error("Error:", error);
    msg.error("Failed to refresh tasks!");
  } finally {
    isLoadingData.value = false; // stop loading
  }
};

onMounted(() => {
  refresh();
});

const paginationReactive = reactive({
  page: 2,
  pageSize: 10,
  showSizePicker: true,
  pageSizes: [5, 10, 25, 50],
  onChange: (page :number) => {
    paginationReactive.page = page
  },
  onUpdatePageSize: (pageSize :number) => {
    paginationReactive.pageSize = pageSize
    paginationReactive.page = 1
  }
})

</script>


<template>
  <div>
    <NDataTable :columns="columns" :data="tasks" :bordered="true" :pagination="paginationReactive" :loading="isLoadingData"/>
  </div>
</template>


<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
