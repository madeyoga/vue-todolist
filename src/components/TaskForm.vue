<template>
  <v-form ref="taskForm" lazy-validation @submit.prevent="verifyTaskForm">
    <v-text-field
      v-model="newTodoItem.title"
      counter="72"
      label="Todo title"
      :rules="taskFormRules.title"
    ></v-text-field>
    <v-textarea
      v-model="newTodoItem.description"
      :rules="taskFormRules.description"
      counter="144"
      label="Description"
    ></v-textarea>
    <v-spacer></v-spacer>
    <div class="row mt-2">
      <v-spacer></v-spacer>
      <v-btn 
        class="mr-2" 
        dark 
        color="red"
        @click="$emit('dialogState', false)"
        >Cancel</v-btn>
      <v-btn type="submit" dark color="indigo accent-2">Add task</v-btn>
      <v-spacer></v-spacer>
    </div>
  </v-form>
</template>

<script>
export default {
  name: 'taskform',

  data: function () {
    return {
      newTodoItem: { title: "", description: "", checked: false, show: false },
      taskFormRules: {
        title: [
          (v) => !!v || "Title cannot be empty",
          (v) => v.length < 72 || "Length must be less then 72 characters",
        ],
        description: [
          (v) => !!v || "Description cannot be empty",
          (v) => v.length < 144 || "Length must be less then 144 characters",
        ],
      },
    };
  },

  methods: {
    verifyTaskForm() {
      if (this.$refs.taskForm.validate()) {
        // add new task to todoList
        this.$emit('addTodo', JSON.parse(JSON.stringify(this.newTodoItem)));
        this.$emit('dialogState', false);
      }
    },
  }
};
</script>
