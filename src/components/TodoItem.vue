<template>
  <div class="todo-item">
    <div class="todo-item-left">
      <input type="checkbox" v-model="completed" @change="doneEdit">
      <input type="checkbox" v-model="decline" @change="doneEdit">
      <input type="checkbox" v-model="nvt" @change="doneEdit">
      <div v-if="!editing" @dblclick="editTodo" class="todo-item-label" :class="{ completed : completed, decline : decline, nvt : nvt }">{{ task.title }}</div>
      <input v-else class="todo-item-edit" type="text" v-model="title" @blur="doneEdit" @keyup.enter="doneEdit" @keyup.esc="cancelEdit" v-focus>
    </div>
    <!-- Wel een description -->
    <div v-if="!task.description==''||task.description" class="description-item" @click="editRemark">
      &#9917;
    </div>
    <!-- Geen description -->
    <div v-if="task.description==''|| task.description === undefined" class="description-item" @click="editRemark">
      &#9918;
    </div>
    <div class="remove-item" @click="removeTodo(task.id)">
      &times;
    </div>
    <modal v-bind="$props" v-if="showModal" @finishedEditModal="finishedEditModal">
      <h2 slot="header">{{task.title}}</h2>
      <h3 slot="subHeader">Opmerking:</h3>
      <!-- <p slot="body" v-if="!editing" @dblclick="editTodo" :class="{ completed : completed }">{{ description }}</p> -->
      <input class="todo-item-edit" type="text" v-model="description" slot="body" @blur="doneEdit" @keyup.enter="doneEdit" @keyup.esc="cancelEdit" v-focus>
    </modal>
  </div>
</template>

<script>
import Modal from './Modal.vue'

export default {
  name: 'todo-item',
  components: {
    Modal
  },
  props: {
    task: {
      type: Object,
      required: true,
    },
    checkAll: {
      type: Boolean,
      required: true,
    }
  },
  data() {
    return {
      id: this.task.id,
      title: this.task.title,
      description: this.description,
      completed: this.task.completed,
      decline: this.task.decline,
      nvt: this.task.nvt,
      editing: this.task.editing,
      beforeEditCache: '',
      showModal: false
    }
  },
  watch: {
    checkAll() {
      // if (this.checkAll) {
      //   this.completed = true
      // } else {
      //   this.completed = this.todo.completed
      // }

      this.nvt = false
      this.decline = false
      
      this.completed = this.checkAll ? true : this.task.completed
    }
  },
  directives: {
    focus: {
      inserted: function (el) {
        el.focus()
      }
    }
  },
  methods: {
    removeTodo(id) {
      this.$emit('removedTodo', id)
    },
    editTodo() {
      this.beforeEditCache = this.title
      this.editing = true
    },
    doneEdit() {
      if (this.title.trim() == '') {
        this.title = this.beforeEditCache
      }
      this.editing = false

      this.$emit('finishedEdit', {
        id: this.id,
        title: this.title,
        description: this.description,
        completed: this.completed,
        decline: this.decline,
        nvt: this.nvt,
        editing: this.editing,
      })
    },
    cancelEdit() {
      this.title = this.beforeEditCache
      this.editing = false
    },
    editRemark() {
      this.showModal = true
    },
    finishedEditModal(data) {
      if(data.description === null){
        return this.task.description === ''
      }
      this.task.description === data.decription
      this.showModal = false
    }
  }
}
</script>