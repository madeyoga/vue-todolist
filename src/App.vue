<template>
  <v-app>
    <v-fab-transition>
      <v-btn
        color="indigo accent-2"
        dark
        fixed
        bottom
        right
        fab
        @click="dialog = !dialog"
      >
        <v-icon>mdi-plus</v-icon>
      </v-btn>
    </v-fab-transition>

    <!-- Side nav -->
    <v-navigation-drawer
      v-model="drawer"
      app
      temporary
      gradient="to top right, rgba(19,84,122,.5), rgba(128,208,199,.8)"
    >
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title>My todo list</v-list-item-title>
        </v-list-item-content>
      </v-list-item>

      <v-divider></v-divider>

      <v-list rounded dense>
        <v-list-item
          v-for="item in drawerItems"
          :key="item.title"
          link
        >
          <v-list-item-icon>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar
      app
      fixed
      color="#fcb69f"
      dark
      extended
      extension-height="156"
      scroll-threshold="500"
      src="https://picsum.photos/1920/1080?random"
    >
      <template v-slot:img="{ props }">
        <v-img
          v-bind="props"
          gradient="to top right, rgba(100,115,201,.7), rgba(25,32,72,.7)"
        ></v-img>
      </template>

      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-spacer></v-spacer>

      <v-btn icon @click="switchTheme">
        <v-icon>{{ !$vuetify.theme.dark ? 'mdi-weather-night' : 'mdi-weather-cloudy' }}</v-icon>
      </v-btn>

      <template v-slot:extension>
        <div class="col-6">
          <h2 class="display-1 mb-4">Your<br>Things</h2>

          <small>{{ currentDate }}</small>
        </div>
        <div class="col-6 row d-flex flex-column">
          <div class="d-flex">
            <div class="col-6 text-right">
              <h4>{{ todoList.length }}</h4>
              <small>Tasks</small>
            </div>
            <div class="col-6 text-right">
              <h4>{{ completedTasksCount }}</h4>
              <small>Completed</small>
            </div>
          </div>
          <div class="col-12 d-flex justify-end">
            <div class="d-flex align-center">
              <v-progress-circular
                rotate="-90"
                size="32"
                width="4"
                :value="completePercentage"
                :color="completePercentage > 50 ? 'green' : 'orange'"
              >
              </v-progress-circular>
              <div class="ml-2">{{ completePercentage }}% done</div>
            </div>
          </div>
        </div>
      </template>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <v-subheader>
          Your Tasks
          <v-spacer></v-spacer>
          <v-btn icon @click="sorting=!sorting" color="indigo accent-2">
            <v-icon v-if="sorting">mdi-sort-variant-lock-open</v-icon>
            <v-icon v-else>mdi-sort-variant-lock</v-icon>
          </v-btn>
        </v-subheader>
        <draggable class="list-group row" tag="div" v-model="todoList" group="todos" @start="drag=true" @end="drag=false" 
          v-bind="dragOptions">
          <div class="col-12 col-md-6 col-lg-4 col-xl-3" style="width:100%;" v-for="(item, index) in todoList" :key="index">
            <v-card class="rounded-xl" flat :color="$vuetify.theme.dark ? '' : 'indigo lighten-5'">
                <v-list-item>
                  <v-list-item-action>
                    <v-checkbox v-model="item.checked"></v-checkbox>
                  </v-list-item-action>
                  <v-list-item-content  @click="item.show = !item.show">
                    <v-list-item-title>{{ item.title }}</v-list-item-title>
                    <v-list-item-subtitle>{{ item.description }}</v-list-item-subtitle>
                  </v-list-item-content>
                  <v-list-item-action v-if="item.checked">
                    <v-btn icon color="red" @click="done(index)">
                      <v-icon>mdi-close</v-icon>
                    </v-btn>
                  </v-list-item-action>
              </v-list-item>

              <v-expand-transition>
                <div v-show="item.show">
                  <v-card-text>
                    {{item.description}}
                  </v-card-text>
              </div>
            </v-expand-transition>
            </v-card>
          </div>
        </draggable>
      </v-container>
    </v-main>

    <v-dialog
      v-model="dialog"
      fullscreen
      hide-overlay
      transition="dialog-bottom-transition">
      <v-card>
          <v-toolbar
            dark
            color="indigo accent-2"
          >
            <v-btn
              icon
              dark
              @click="dialog = false"
            >
              <v-icon>mdi-close</v-icon>
            </v-btn>
            <v-toolbar-title>Add new task</v-toolbar-title>
          </v-toolbar>

          <v-container>
            <taskform @addTodo="addTodo" @dialogState="dialogState"/>
          </v-container>
        </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import draggable from 'vuedraggable';
import taskform from './components/TaskForm.vue';

const dateFormatOptions = {
  weekday: "long",
  year: "numeric",
  month: "long",
  day: "numeric"
};

const locale = "en-AU";

export default {
  name: 'App',
  
  components: {
    // HelloWorld,
    draggable,
    taskform,
  },

  data: function () {
    return {
      currentDate: new Date().toLocaleDateString(locale, dateFormatOptions),
      dialog: false,
      doneTasks: [],
      drag: false,
      drawer: false,
      drawerItems: [
        { title: 'Home', icon: 'mdi-view-dashboard' },
        { title: 'FAQ', icon: 'mdi-help-circle-outline' }
      ],
      sorting: false,
      todoList: [
        { datetime: new Date(), title: 'Workout', description: 'Working out some sweat', due: new Date(), checked: false, show: false, },
        { datetime: new Date(), title: 'Eat', description: 'Big mac', due: new Date(), checked: true, show: false, },
        { datetime: new Date(), title: 'Play some game', description: 'No play no gain', due: new Date(), checked: true, show: false, },
        { datetime: new Date(), title: 'Study', description: 'Read an article or a book', due: new Date(), checked: false, show: false, },
        { datetime: new Date(), title: 'Sleep', description: 'At 09:00', due: new Date(), checked: true, show: false, },
      ],
    }
  },

  methods: {
    switchTheme() {
      this.$vuetify.theme.dark = !this.$vuetify.theme.dark;
      localStorage.setItem('theme', this.$vuetify.theme.dark ? 'dark' : 'light');
    },
    done(index) {
      this.doneTasks.push(this.todoList[index]);
      this.todoList.splice(index, 1);
    },
    addTodo(todo) {
      this.todoList.push(todo);
    },
    dialogState(val) {
      this.dialog = val;
    }
  },

  computed: {
    completedTasksCount() {
      let counter = 0;
      for (const item of this.todoList) {
        if (item.checked)
          counter += 1;
      }
      return counter;
    },
    completePercentage() {
      return this.todoList.length > 0 ? parseInt((this.completedTasksCount / this.todoList.length) * 100) : 0;
    },
    dragOptions() {
      return {
        animation: 200,
        group: "description",
        disabled: !this.sorting,
        ghostClass: "ghost"
      };
    },
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif!important;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.v-toolbar__extension {
  padding: 0!important;
}

.flip-list-move {
  transition: transform 0.5s;
}
.no-move {
  transition: transform 0s;
}
.ghost {
  opacity: 0.5;
}
.list-group {
  min-height: 20px;
  width: 100%;
}
.list-group-item {
  cursor: move;
}
.list-group-item i {
  cursor: pointer;
}
</style>
