<template>    
    <AddTask v-show="showAddTask" @new-task="addTask"/>    
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" />
</template>

<script>
import Tasks from "../components/Tasks.vue"
import AddTask from "../components/AddTask.vue"

export default {
    name: "Home",
    props: {
        showAddTask: Boolean
    },
    components: {
        AddTask,
        Tasks
    },
    data() {
        return {
            tasks: []
        }        
    },
    methods: {
    async addTask(newTask) {
      const res = await fetch('api/tasks',{
        method: 'POST',
        headers: {
          'Content-Type': "application/json",
        },
        body: JSON.stringify(newTask)
      })

      const data = await res.json();

      this.tasks = [...this.tasks, data]
    },
    async deleteTask(id) {
      if(confirm("You want to delete task")) {
        const res = await fetch(`api/tasks/${id}`,{
          method: 'DELETE',
        })

        res.status === 200 
        ? (this.tasks = this.tasks.filter((task) => task.id !== id)) 
        : alert("Oops! Errot occured while deleting tasks")         
      }
    },
    async toggleReminder(id) {  
      const taskToToggle = await this.fetchTask(id)
      const updateTask = {...taskToToggle, reminder: !taskToToggle.reminder}
      
      const res = await fetch(`api/tasks/${id}`,{
        method: 'PUT',
        headers: {
          'Content-Type': "application/json",
        },
        body: JSON.stringify(updateTask)
      })

      const data = await res.json();

      this.tasks = this.tasks.map((task) => task.id === id 
      ? {...task, reminder: data.reminder} 
      : task
      )
    },
    async fetchTasks() {
      const res = await fetch('api/tasks');
      const data = await res.json();
      return data;
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);
      const data = (res) ? await res.json() : null
      return data;
    }
  },
  async created() {
    this.tasks = await this.fetchTasks()
  }
}
</script>