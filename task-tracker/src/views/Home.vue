<template>
  <div>
    <!-- v-show simply toggles display and the code stays there -->
    <AddTask v-if="showAddTask" @add-task="addTask" />
    <Tasks
      @toggle-reminder="toggleReminder"
      @delete-task="deleteTask"
      :tasks="tasks"
    />
  </div>
</template>

<script>
import Tasks from '../components/Tasks';
import AddTask from '../components/AddTask';
export default {
  name: 'Home',
  props: {
    showAddTask: Boolean,
  },
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async addTask(task) {
      console.log(task);
      const res = await fetch('api/tasks', {
        method: 'POST',
        headers: {
          'content-type': 'Application/json',
        },
        body: JSON.stringify(task),
      });
      const data = await res.json();
      this.tasks = [...this.tasks, data];
    },
    async deleteTask(taskId) {
      if (confirm('Are you sure?')) {
        const res = await fetch(`api/tasks/${taskId}`, {
          method: 'DELETE',
        });
        res.status === 200
          ? (this.tasks = this.tasks.filter(task => task.id !== taskId))
          : alert('Error Deleting this Task');
      }
    },
    async toggleReminder(taskId) {
      const taskToToggle = await this.fetchTask(taskId);
      const updTask = {
        ...taskToToggle,
        reminder: !taskToToggle.reminder,
      };
      const res = await fetch(`api/tasks/${taskId}`, {
        method: 'PUT',
        headers: {
          'content-type': 'Application/json',
        },
        body: JSON.stringify(updTask),
      });
      const data = await res.json();
      this.tasks = this.tasks.map(task =>
        task.id === taskId ? { ...task, reminder: data.reminder } : task
      );
    },
    async fetchTasks() {
      const res = await fetch('api/tasks');
      const data = await res.json();
      return data;
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);
      const data = await res.json();
      return data;
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>
