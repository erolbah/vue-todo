<template>
  <div class="todo-item">
    <div class="todo-item-left">
      <input type="checkbox" v-model="completed" @change="doneEdit">
      <div v-if="!editing" @dblclick="editTodo" class="todo-item-label" :class="{ completed : completed }">{{ title }}</div>
      <input v-else class="todo-item-edit" type="text" v-model="title" @blur="doneEdit" @keyup.enter="doneEdit" @keyup.esc="cancelEdit" v-focus>
    </div>
    <div v-if="!todo.description==''" class="description-item" @click="editRemark(todo.id)">
      [0]
    </div>
    <div class="remove-item" @click="removeTodo(todo.id)">
      &times;
    </div>
    <modal v-if="showModal" @close="showModal = false">
      <!--
        you can use custom content here to overwrite
        default content
      -->
      <p slot="body">{{todo.description}}</p>
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
    todo: {
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
      id: this.todo.id,
      title: this.todo.title,
      description: this.description,
      completed: this.todo.completed,
      editing: this.todo.editing,
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
      this.completed = this.checkAll ? true : this.todo.completed
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
        editing: this.editing,
      })
    },
    cancelEdit() {
      this.title = this.beforeEditCache
      this.editing = false
    },
    editRemark() {
      this.showModal = true
    }
  }
}
</script>