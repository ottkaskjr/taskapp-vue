<template>
  <div class="container">
    <div id="notifyParent">
      <div
        v-for="(notification, index) in notifications"
        class="notification p-3 mb-1"
      >
        <p class="">{{ notification }}</p>
        <b-button v-on:click="dismiss(index)" variant="success" class="btn-sm"
          >Close</b-button
        >
      </div>
    </div>
    <div class="mt-3 mb-4">
      <b-button v-b-modal.taskModal variant="success" class="btn btn-lg"
        >Create task</b-button
      >

      <b-modal id="taskModal" title="Create task" hide-footer>
        <label class="">Task objective*</label>
        <b-form-input
          v-on:input="validateForm()"
          v-model="newTask.task"
          placeholder="Task"
        ></b-form-input>
        <label>Task deadline*</label>
        <b-form-input
          v-on:input="validateForm()"
          class="mb-1"
          v-model="date"
          type="date"
        ></b-form-input>
        <b-form-input
          v-on:input="validateForm()"
          v-model="time"
          type="time"
        ></b-form-input>
        <div class="text-center py-3">
          <b-button
            v-on:click="createTask()"
            id="createTaskBtn"
            variant="success"
            class="btn-lg"
            :disabled="validated == false"
            >Create</b-button
          >
        </div>
      </b-modal>
      <b-modal id="errorModal" title="Error">
        <p class="my-4">{{ error }}</p>
      </b-modal>
    </div>
    <div id="tasksContainer">
      <div
        v-for="task in tasks"
        v-bind:class="{ critical: isDue(task.deadline) }"
        class="taskContainer mx-1 mx-md-0 row p-3 border-sm shadowSm mb-3 text-md-center text-left"
      >
        <div class="col-12 col-md-4">
          <p class="mb-0"><b>Objective</b></p>
          <p>{{ task.task }}</p>
        </div>
        <div class="col-12 col-md-3">
          <p class="mb-0"><b>Deadline</b></p>
          <p>{{ buildDate(task.deadline) }}</p>
        </div>
        <div class="col-12 col-md-3">
          <p class="mb-0"><b>Created</b></p>
          <p>{{ buildDate(task.createdAt) }}</p>
        </div>
        <div class="col-12 col-md-2 text-center">
          <button
            :id="task.id"
            v-on:click="resolveTask(task.id, task.task)"
            v-bind:class="{ 'btn-warning': isDue(task.deadline) }"
            class="btn btn-success"
          >
            Resolve
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Tasks',
  data: function() {
    return {
      newTask: {
        task: '',
      },
      date: '',
      time: '',
      validated: false,
      tasks: [],
      error: '',
      connection: null,
      notifications: ['message'],
    };
  },
  methods: {
    dismiss: function(index) {
      this.notifications.splice(index, 1);
    },
    validateForm: function() {
      this.validated = true;
      if (this.newTask.task.length < 1) {
        this.validated = false;
      }
      if (this.date.length < 1) {
        this.validated = false;
      }
      if (this.time.length < 1) {
        this.validated = false;
      }
    },
    createTask: function() {
      let year = parseInt(this.date.slice(0, 4));
      let month = parseInt(this.date.slice(5, 7)) - 1;
      let day = parseInt(this.date.slice(8, 10));
      let hour = parseInt(this.time.slice(0, 2));
      let minutes = parseInt(this.time.slice(3, 5));
      let deadline = new Date(year, month, day, hour, minutes);
      this.newTask.deadline = deadline.getTime();
      let errorTest = { a: 135 };
      let taskName = this.newTask.task;
      this.$http
        .post(this.$host + 'task/', this.newTask)
        .then((data) => {
          this.newTask = { task: '' };
          this.date = '';
          this.time = '';
          this.validated = false;
          this.$bvModal.hide('taskModal');
          this.sendMessage(taskName + ' created');
        })
        .catch((e) => {
          this.$bvModal.hide('taskModal');
          this.error = e.message;
          this.$bvModal.show('errorModal');
        });
    },
    sortTasks: function() {
      this.tasks.sort((task1, task2) => {
        return task1.deadline - task2.deadline;
      });
    },
    getTasks: function() {
      this.$http
        .get(this.$host + 'task/')
        .then((data) => {
          this.tasks = data.data.sort((task1, task2) => {
            return task1.deadline - task2.deadline;
          });
        })
        .catch((e) => {
          this.error = e.message;
          this.$bvModal.show('errorModal');
        });
    },
    buildDate: function(dateString) {
      let date = new Date(parseInt(dateString));
      return (
        this.fixNumber(date.getHours()) +
        ':' +
        this.fixNumber(date.getMinutes()) +
        ' ' +
        this.fixNumber(date.getDate()) +
        '-' +
        this.fixNumber(date.getMonth() + 1) +
        '-' +
        date.getFullYear()
      );
    },
    fixNumber: function(number) {
      if (number < 10) {
        return 0 + number.toString();
      }
      return number;
    },
    isDue: function(milliseconds) {
      let now = new Date().getTime();
      if (milliseconds - now < 3600000) {
        return true;
      }
      return false;
    },
    resolveTask: function(id, task) {
      this.$http
        .delete(this.$host + 'task/' + id)
        .then((data) => {
          this.sendMessage(task + ' is resolved');
        })
        .catch((e) => {
          this.error = e.message;
          this.$bvModal.show('errorModal');
        });
    },
    sendMessage: function(message) {
      this.connection.send(message);
    },
  },
  created: function() {
    this.getTasks();

    /* OPEN CONNECTION */
    this.connection = new WebSocket('ws://localhost:8080/update/1');
    this.connection.onopen = function(event) {
      //console.log(event);
      //console.log('Successfully connected to the echo WebSocket Server');
    };

    this.connection.onmessage = (event) => {
      //console.log(event);
      this.notifications.push(event.data);
      this.getTasks();
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.taskContainer {
  background-color: #cfd8ff;
}
.critical {
  background-color: #ff3d3d !important;
}
.border-sm {
  border-radius: 5px;
}
.shadowSm {
  -webkit-box-shadow: 0px 0px 13px -1px #000000;
  box-shadow: 0px 0px 13px -1px #000000;
}
#notifyParent {
  position: absolute;
  z-index: 1;
  top: 5vh;
  left: 50%;
  transform: translateX(-50%);
  width: 300px;
}
.notification {
  position: relative;
  width: inherit;
  background-color: #c4ecff;
  border-radius: 5px;
}
</style>
