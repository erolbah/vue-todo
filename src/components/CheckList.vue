<template>
  <div>
    <input type="text" class="todo-input" placeholder="Wat moet er gebeuren" v-model="newTodo" @keyup.enter="addTodo">
    <transition-group name="fade" enter-active-class="animated fadeInUp" leave-active-class="animated fadeOutDown">
      <div class="group" v-for="(todo, index) in todos" :key="index">
        <h1>{{todo.title}}</h1>
        <div v-for="(subGroup, key) in todo.subGroups" :key="key">
          <h3>{{subGroup.title}}</h3>
          <!-- <div class="" v-for="(task, key) in subGroup.tasks" :key="key">
            <p>{{task.title}}</p>
          </div> -->
          <todo-item v-for="(task, key) in subGroup.tasks" :key="key" :task="task" :checkAll="!anyRemaining" @removedTodo="removeTodo" @finishedEdit="finishedEdit">
          </todo-item>
        </div>
        <!-- <todo-item v-for="todo in todosFiltered" :key="todo.id" :todo="todo" :checkAll="!anyRemaining" @removedTodo="removeTodo" @finishedEdit="finishedEdit">
        </todo-item> -->
      </div>
    
    </transition-group>

    <div class="extra-container">
      <div><label><input type="checkbox" :checked="!anyRemaining" @change="checkAllTodos"> Alles goedkeuren</label></div>
      <div>{{ remaining }} items over</div>
    </div>

    <div class="extra-container">
      <div>
        <button :class="{ active: filter == 'all' }" @click="filter = 'all'">Alles</button>
        <button :class="{ active: filter == 'active' }" @click="filter = 'active'">Actief</button>
        <button :class="{ active: filter == 'completed' }" @click="filter = 'completed'">Goedgekeurd</button>
        <button :class="{ active: filter == 'decline' }" @click="filter = 'decline'">Afgekeurd</button>
        <button :class="{ active: filter == 'nvt' }" @click="filter = 'nvt'">nvt</button>
      </div>

      <div>
        <transition name="fade">
        <button v-if="showClearCompletedButton" @click="clearCompleted">Verwijder Goedgekeurd</button>
        </transition>
      </div>

    </div>
  </div>
</template>

<script>
import TodoItem from './TodoItem'
export default {
  name: 'todo-list',
  components: {
    TodoItem,
  },
  data () {
    return {
      newTodo: '',
      beforeEditCache: '',
      idForTodo: 4, 
      filter: 'all',
      todos: [ 
        {
          id: 1,
          title: 'De woning van buiten',
          completed: false,
          editing: false,
          subGroups: [
            {
              id: 1,
              title: '1. Rondom de woning',
              description: 'Subgroep',
              completed: false,
              editing: false,
              tasks:[
                {
                id: 1,
                title: 'paden / terrassen',
                description: '',
                completed: false,
                editing: false,
                nvt: false
                },
                {
                id: 2,
                title: 'Tuinafwerking',
                description: '',
                completed: false,
                editing: false,
                nvt: false
                }
              ]
            },
            {
              id: 2,
              title: '2. Buitenkant van het huis',
              description: 'Subgroep',
              completed: false,
              editing: false,
              tasks:[
                {
                id: 1,
                title: 'Dakvlak / dakpannen',
                description: '',
                completed: false,
                editing: false,
                nvt: false
                },
                {
                id: 2,
                title: 'Metsel- / voegwerk',
                description: '',
                completed: false,
                editing: false,
                nvt: false
                },
                {
                id: 3,
                title: 'Geveltimmerwerk',
                description: '',
                completed: false,
                editing: false,
                nvt: false
                },
                {
                id: 4,
                title: 'Kozijnen / ramen',
                description: '',
                completed: false,
                editing: false,
                nvt: false
                }
              ]
            }
          ],
        },
        {
          id: 2,
          title: 'In het huis',
          completed: false,
          editing: false,
          subGroups: [
            {
              id: 1,
              title: '1. Installaties',
              description: 'Subgroep',
              completed: false,
              editing: false,
              tasks:[
                {
                id: 1,
                title: 'C.V.-ketel',
                description: '',
                completed: false,
                editing: false,
                delcine: false,
                nvt: false
                },
                {
                id: 2,
                title: 'Warmtevoorziening',
                description: '',
                completed: false,
                editing: false,
                nvt: false
                }
              ]
            }
          ],
        },
      ]
      // todos: [
      //   {
      //     id: 1,
      //     title: 'Afvoer',
      //     description: '',
      //     completed: false,
      //     decline: false,
      //     nvt: false,
      //     editing: false,
      //     // todoTasks:[
      //     //   {
      //     //   id: 1,
      //     //   title: 'Afvoer controleren op afvoercapaciteit',
      //     //   description: 'Deze afvoer werkt niet correct en zal opnieuw aangelegt moeten worden SUBGROEP',
      //     //   completed: false,
      //     //   editing: false
      //     //   },
      //     //   {
      //     //   id: 2,
      //     //   title: 'Afvoer opnieuw aanleggen',
      //     //   description: 'Afvoer zit verstopt en moet opnieuw aangelegt SUBGROEP',
      //     //   completed: false,
      //     //   editing: false
      //     //   }
      //     // ]
      //   },
      //   {
      //     id: 2,
      //     title: 'Oplevering glaswerk',
      //     description: '',
      //     completed: false,
      //     delcine: false,
      //     nvt: false,
      //     editing: false
      //   },
      //   {
      //     id: 3,
      //     title: 'Beschadigingen op kozijnwerk controleren',
      //     description: '',
      //     completed: false,
      //     delcine: false,
      //     nvt: false,
      //     editing: false
      //   },
      // ],
     }
  },
  computed: {
    remaining() {
      // return this.todos.filter(todo => !todo.completed && !todo.decline && !todo.nvt).length
      return this.todos.filter(todo => !todo.completed).length
    },
    checkedBoxes() {
      // Controle block zodat 1 van de 3 checkboxes actief kan zijn.
      if(this.completed){
        return !this.decline && !this.nvt
      }
      if(this.decline){
        return !this.completed && !this.nvt
      }
      if(this.nvt){
        return !this.completed && !this.decine
      }
      return this.todos
    },
    anyRemaining() {
      return this.remaining != 0
    },
    todosFiltered() {
      if (this.filter == 'all') {
        return this.todos
      } else if (this.filter == 'active') {
        return this.todos.filter(todo => !todo.completed && !todo.decline &&  !todo.nvt)
      } else if (this.filter == 'completed') {
        return this.todos.filter(todo => todo.completed)
      } else if (this.filter == 'decline') {
        return this.todos.filter(todo => todo.decline)
      } else if (this.filter == 'nvt') {
        return this.todos.filter(todo => todo.nvt)
      }
      return this.todos
    },
    showClearCompletedButton() {
      return this.todos.filter(todo => todo.completed).length > 0
    }
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim().length == 0) {
        return
      }
      this.todos.push({
        id: this.idForTodo,
        title: this.newTodo,
        completed: false,
        decline: false,
        nvt: false
      })
      this.newTodo = ''
      this.idForTodo++
    },
    removeTodo(id) {
      const index = this.todos.findIndex((item) => item.id == id)
      this.todos.splice(index, 1)
    },
    checkAllTodos() {
      if(this.todos.filter(todo => todo.decline).length > 0){
        this.todos.filter(todo => todo.decline = false)
      }
      if(this.todos.filter(todo => todo.nvt).length > 0){
        this.todos.filter(todo => todo.nvt = false)
      }
      this.todos.forEach((todo) => todo.completed = event.target.checked)
    },
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.completed)
    },
    finishedEdit(data) {
      const index = this.todos.findIndex((item) => item.id == data.id)
      this.todos.splice(index, 1, data)
    }
  }
}
</script>

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
    // justify-content: space-between;
    animation-duration: 0.3s;
  }
  .remove-item {
    cursor: pointer;
    justify-content: flex-end;
    margin-left: 14px;
    display: flex;
    flex: 1;
    &:hover {
      color: black;
    }
  }
  .description-item{
    cursor: pointer;
    justify-content: flex-end;
    margin-left: 14px;
    display: flex;
    flex: 1;
    &:hover {
      color: black;
    }
  }
  .todo-item-left { // later
    display: flex;
    flex: 1 100%;
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
  .decline {
    text-decoration: line-through;
    color: red;
  }
  .nvt {
    // text-decoration: line-through;
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

