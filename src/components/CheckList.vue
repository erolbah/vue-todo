<template>
  <div class="hello">
    <h1>Bouwwerk controlelijst</h1>
    <input type="text" 
      class="todo-input" 
      placeholder="Voer handmatig checklist item in" 
      v-model="newTodo" 
      @keyup.enter="addTodo">
    <transition-group name="fade" enter-active-class="animated fadeInUp" leave-active-class="animated fadeOutDown">
      <div v-for="(todo, index) in todosFiltered" :key="todo.id" class="todo-item">
        <div class="todo-item-left">
          <input type="checkbox" v-model="todo.completed">
          <div v-if="!todo.editing" 
            @dblclick="editTodo(todo)" 
            class="todo-item-label"
            :class="{ completed : todo.completed }">
              {{todo.title}}</div>
          <input v-else 
            class="todo-item-edit" 
            type="text" 
            v-model="todo.title" 
            @blur="doneEdit(todo)" 
            @keyup.enter="doneEdit(todo)" 
            @keyup.esc="cancelEdit(todo)" 
            v-focus>
            <!-- <div v-for="subGroup in todo.subGroups" :key="subGroup.id">{{subGroup.title}}
              <div v-for="task in subGroup.tasks" :key="task.id" class="todo-item-child">{{task.title}}</div>
            </div> -->
        </div>
        <div class="remove-item" @click="removeTodo(index)">
          &times;
        </div>
      </div>
    </transition-group>
    <div class="extra-container">
      <div class="">
        <label for="">
          <input type="checkbox" :checked="!anyRemaining" @change="checkAllTodos">
          Check all
        </label>
      </div>
      <div class="">{{remaining}} items left</div>
    </div>
    <div class="extra-container">
      <div class="">
        <button :class="{ active: filter == 'all' }" @click="filter = 'all'">Alles</button>
        <button :class="{ active: filter == 'active' }" @click="filter = 'active'">Actief</button>
        <button :class="{ active: filter == 'completed' }" @click="filter = 'completed'">Completed</button>
      </div>
      <div class="">
        <transition name="fade">
          <button v-if="showClearCompletedButton" @click="clearCompleted">Clear completed</button>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Todo-list',
  data () {
    return {
      newTodo: '',
      beforeEditCache: '',
      idForTodo: 4, 
      filter: 'all',
      todos: [
        {
          id: 1,
          title: 'Afvoer',
          description: 'Hoofdgroep',
          completed: false,
          editing: false,
          todoTasks:[
            {
            id: 1,
            title: 'Afvoer controleren op afvoercapaciteit',
            description: 'Deze afvoer werkt niet correct en zal opnieuw aangelegt moeten worden SUBGROEP',
            completed: false,
            editing: false
            },
            {
            id: 2,
            title: 'Afvoer opnieuw aanleggen',
            description: 'Afvoer zit verstopt en moet opnieuw aangelegt SUBGROEP',
            completed: false,
            editing: false
            }
          ]
        },
        {
          id: 2,
          title: 'Oplevering glaswerk',
          description: 'Deze afvoer werkt niet correct en zal opnieuw aangelegt moeten worden',
          completed: false,
          editing: false
        },
        {
          id: 3,
          title: 'Beschadigingen op kozijnwerk controleren',
          description: 'Deze afvoer werkt niet correct en zal opnieuw aangelegt moeten worden',
          completed: false,
          editing: false
        },
      ],
    //   todos: [ 
    //     {
    //       id: 1,
    //       title: 'De woning van buiten',
    //       completed: false,
    //       editing: false,
    //       subGroups: [
    //         {
    //           id: 1,
    //           title: '1. Rondom de woning',
    //           description: 'Subgroep',
    //           completed: false,
    //           editing: false,
    //           tasks:[
    //             {
    //             id: 1,
    //             title: 'paden / terrassen',
    //             description: '',
    //             completed: false,
    //             editing: false
    //             },
    //             {
    //             id: 2,
    //             title: 'Tuinafwerking',
    //             description: '',
    //             completed: false,
    //             editing: false
    //             }
    //           ]
    //         },
    //         {
    //           id: 2,
    //           title: '2. Buitenkant van het huis',
    //           description: 'Subgroep',
    //           completed: false,
    //           editing: false,
    //           tasks:[
    //             {
    //             id: 1,
    //             title: 'Dakvlak / dakpannen',
    //             description: '',
    //             completed: false,
    //             editing: false
    //             },
    //             {
    //             id: 2,
    //             title: 'Metsel- / voegwerk',
    //             description: '',
    //             completed: false,
    //             editing: false
    //             },
    //             {
    //             id: 3,
    //             title: 'Geveltimmerwerk',
    //             description: '',
    //             completed: false,
    //             editing: false
    //             },
    //             {
    //             id: 4,
    //             title: 'Kozijnen / ramen',
    //             description: '',
    //             completed: false,
    //             editing: false
    //             }
    //           ]
    //         }
    //       ],
    //     },
    //     {
    //       id: 2,
    //       title: 'In het huis',
    //       completed: false,
    //       editing: false,
    //       subGroups: [
    //         {
    //           id: 1,
    //           title: '1. Installaties',
    //           description: 'Subgroep',
    //           completed: false,
    //           editing: false,
    //           tasks:[
    //             {
    //             id: 1,
    //             title: 'C.V.-ketel',
    //             description: '',
    //             completed: false,
    //             editing: false
    //             },
    //             {
    //             id: 2,
    //             title: 'Warmtevoorziening',
    //             description: '',
    //             completed: false,
    //             editing: false
    //             }
    //           ]
    //         }
    //       ],
    //     },
    //   ]
     }
  },
  computed: {
    remaining() {
      return this.todos.filter(todo => !todo.completed).length
    },
    anyRemaining() {
      // controleerd computed property remaining, als deze niet 0 is dan false.
      return this.remaining != 0
    },
    todosFiltered() {
      if(this.filter === 'all'){
        return this.todos
      } else if (this.filter === 'active') {
        return this.todos.filter(todo => !todo.completed)
      } else if (this.filter === 'completed') {
        return this.todos.filter(todo => todo.completed)
      }
      return this.todos
    },
    showClearCompletedButton() {
      return this.todos.filter(todo => todo.completed).length > 0
    }
  },
  directives: {
    focus: {
      // directive definition
      inserted: function (el) {
        el.focus()
      }
    }
  },
  methods: {
    addTodo () {
      if(this.newTodo.length === 0){
        return
      }
      this.todos.push({
        id: this.idForTodo,
        title: this.newTodo,
        description: this.newTodo,
        completed: false
      })

      this.newTodo = ''
      this.idForTodo++
    },
    removeTodo(index){
      this.todos.splice(index, 1)
    },
    editTodo(todo) {
      this.beforeEditCache = todo.title
      todo.editing = true
    },
    doneEdit(todo){
      if(todo.title.trim() === ''){
        todo.title = this.beforeEditCache
      }
      todo.editing = false
    },
    cancelEdit(todo) {
      todo.title = this.beforeEditCache
      todo.editing = false
    },
    checkAllTodos() {
      this.todos.forEach((todo) => todo.completed = event.target.checked)
    },
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.completed)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
  @import url("https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.5.2/animate.min.css");
  .todo-input {
    width: 100%;
    padding: 10px 18px;
    font-size: 18px;
    margin-bottom: 16px;
    &:focus {
      outline: 0;
    }
  }
  .todo-item {
    margin-bottom: 12px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    animation-duration: 0.3s;
  }
  .remove-item {
    cursor: pointer;
    margin-left: 14px;
    &:hover {
      color: black;
    }
  }
  .todo-item-left { // later
    display: flex;
    align-items: center;
  }
  .todo-item-label {
    padding: 10px;
    border: 1px solid white;
    margin-left: 12px;
  }
  .todo-item-edit {
    font-size: 24px;
    color: #2c3e50;
    margin-left: 12px;
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc; //override defaults
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    &:focus {
      outline: none;
    }
  }
  .completed {
    text-decoration: line-through;
    color: grey;
  }
  .extra-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 16px;
    border-top: 1px solid lightgrey;
    padding-top: 14px;
    margin-bottom: 14px;
  }
  button {
    font-size: 14px;
    background-color: white;
    appearance: none;
    &:hover {
      background: lightgreen;
    }
    &:focus {
      outline: none;
    }
  }
  .active {
    background: lightgreen;
  }
  // CSS Transitions
  .fade-enter-active, .fade-leave-active {
    transition: opacity .2s;
  }
  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>