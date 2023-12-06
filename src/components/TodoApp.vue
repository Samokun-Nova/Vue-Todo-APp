<template>
  <div class="todo-container">
    <h2 class="todo-title">TODO APP</h2>

    <!-- todo input -->
    <div class="input">
      <input v-model="newTodo" type="text" placeholder="Enter Todo Text" class="todo-input">
      <input v-model="newDueDate" type="date" class="due-date-input">
      <input v-model="newDueTime" type="time" class="due-time-input">
      <button @click="submitTodo" id="input-button">Submit</button>
    </div>

    <!-- Task Table -->
    <div class="task-table">
      <table class="table table-bordered">
        <thead>
          <tr>
            <th class="table-head" scope="col">Task</th>
            <th class="table-head" scope="col">Status</th>
            <th class="table-head" scope="col">Due Time</th>
            <th class="table-head" scope="col">#</th>
            <th class="table-head" scope="col">#</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(task, index) in todoTasks" :key="task.id || index">
            <td class="task-name">{{ task.name }}</td>
            <td>
              <span @click="changeStatus(index)" 
              :class="{ 'status-finished': task.status === 'finished'}"
              class="pointer status-cell">
                {{ task.status }}
              </span>
            </td>
            <td class="due-time">{{ task.dueDate }} {{ task.dueTime }}</td>
            <td>
              <div class="countdown">
                {{ calculateCountdown(task) }}
              </div>
            </td>
            <td>
              <div class="edit-box actions">
                <button @click="editTask(index)">Edit</button>
              </div>
            </td>
            <td>
              <div>
                <button class="delete" @click="deleteTodo(index)">Delete</button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },

  data() {
    return {
      newTodo: "",
      editedTask: null,
      availableStatuses: ['to-do', 'in-progress', 'finished'],
      todoTasks: this.loadTasksFromLocalStorage(),
      // {
      //   id: 1,
      //   name: '',
      //   status: '',
      //   dueDate: '', // Sample due date
      //   dueTime: '', // Sample due time
      // },
      // {
      //   id: 2,
      //   name: '',
      //   status: '',
      //   dueDate: '', // Sample due date
      //   dueTime: '', // Sample due time
      // }

      newDueDate: "",
      newDueTime: "",
    };
  },

  methods: {
    submitTodo() {
      if (this.newTodo.length === 0 || this.newDueDate === "" || this.newDueTime === "") return;

      if (this.editedTask === null) {
        const newTask = {
          id: Date.now(),
          name: this.newTodo,
          status: 'to-do',
          dueDate: this.newDueDate,
          dueTime: this.newDueTime,

        };

        this.todoTasks.push(newTask);
        this.scheduleNotification(newTask);


      } else {
        this.todoTasks[this.editedTask].name = this.newTodo;
        this.todoTasks[this.editedTask].dueDate = this.newDueDate;
        this.todoTasks[this.editedTask].dueTime = this.newDueTime;
        this.editedTask = null;
      }

      this.newTodo = "";
      this.newDueDate = "";
      this.newDueTime = "";
      this.saveTasksToLocalStorage();
    },

    deleteTodo(index) {
      this.todoTasks.splice(index, 1);
      this.saveTasksToLocalStorage();
    },

    saveTasksToLocalStorage() {
      localStorage.setItem('todoTasks', JSON.stringify(this.todoTasks));
    },

    loadTasksFromLocalStorage() {
      const storedTasks = localStorage.getItem('todoTasks');
      return storedTasks ? JSON.parse(storedTasks) : [];
    },

    editTask(index) {
      this.newTodo = this.todoTasks[index].name;
      this.newDueDate = this.todoTasks[index].dueDate;
      this.newDueTime = this.todoTasks[index].dueTime;
      this.editedTask = index;
    },

    calculateCountdown(task) {
      if (task.status === 'finished') {
        return 'Task Completed';
      }

      const dueDateTime = new Date(`${task.dueDate}T${task.dueTime}`);
      const now = new Date();

      if (dueDateTime > now) {
        const timeUntilDue = dueDateTime - now;

        const hours = Math.floor(timeUntilDue / (1000 * 60 * 60));
        const minutes = Math.floor((timeUntilDue % (1000 * 60 * 60)) / (1000 * 60));
        const seconds = Math.floor((timeUntilDue % (1000 * 60)) / 1000);

        return `${hours}h ${minutes}m ${seconds}s`;
      } else {
        return "Task Expired";
      }
    },


    scheduleNotification(task) {
      const dueDateTime = new Date(`${task.dueDate}T${task.dueTime}`);
      const now = new Date();

      if (dueDateTime > now) {
        const timeUntilDue = dueDateTime - now;

        setTimeout(() => {
          this.showNotification(task);
        }, timeUntilDue);
      }
    },

    showNotification(task) {
      if (Notification.permission === 'granted') {
        new Notification(`Task Reminder: ${task.name}`, {
          body: `Due Time: ${task.dueTime}`,
        });
      } else if (Notification.permission !== 'denied') {
        Notification.requestPermission().then(permission => {
          if (permission === 'granted') {
            this.showNotification(task);
          }
        });
      }
    },

    changeStatus(index) {
      const newIndex = (this.availableStatuses.indexOf(this.todoTasks[index].status) + 1) % this.availableStatuses.length;
      this.todoTasks[index].status = this.availableStatuses[newIndex];
    }
  }
}
</script>

<style scoped>
.pointer {
  cursor: pointer;
}

.due-date-input,
.due-time-input {
  margin-left: 10px;
}

.countdown {
  font-weight: bold;
  color: #ff4500;
  /* or any color you prefer */
}

button {
  color: grey;
  width: 70px;
  height: 30px;
  border: 2px solid;
  border-color: rgba(0, 255, 136, 0.589);
  border-radius: 10px;
}

.todo-input {
  width: 400px;
  height: 50px;
  border-radius: 10px;
  border: 2px solid rgba(0, 255, 136, 0.589);
  ;

}

.todo-container {
  background-color: rgb(205, 234, 237);
}

.todo-title {
  text-align: center;
  justify-content: center;
  display: flex;
  color: rgb(22, 87, 122);
  font-weight: 100;
  font-size: 50px;
}

.input {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 10px;

}

.due-date-input {
  width: 200px;
  height: 40px;
  border-radius: 10px;
  border: 2px solid rgba(0, 255, 136, 0.589);
  ;
  color: grey;
}

.due-time-input {
  width: 200px;
  height: 40px;
  border-radius: 10px;
  border: 2px solid rgba(0, 255, 136, 0.589);
  color: grey;
}

#input-button {
  width: 100px;
  height: 40px;
  margin-bottom: 10px;
}

.table {
  background-color: white;
}

.table-head {
  background-color: rgb(236, 234, 241);
  color: grey;
  border: 3px rgb(20, 165, 198) solid;
}


.status-finished {
  color: green;
}

.due-date-input,
.due-time-input {
  margin-left: 10px;
}

.countdown {
  font-weight: bold;
  color: #ff4500; /* or any color you prefer */
}

.status-cell {
  font-weight: bold;
}

.status-finished {
  color: green;
}

.task-name {
  font-weight: bold;
}

.due-time {
  color: #333; /* or any color you prefer */
}

.countdown {
  font-weight: bold;
  color: #ff4500; /* or any color you prefer */
}

.actions {
  display: flex;
  gap: 5px;
}

.delete {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}
.task-table {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
