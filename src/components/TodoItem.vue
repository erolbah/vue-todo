<template>
  <div class="todo-item">
    <div class="todo-item-left">
        <input type="checkbox" v-model="completed">
        <div v-if="!editing" 
        @dblclick="editTodo" 
        class="todo-item-label"
        :class="{ completed : completed }">
            {{title}}</div>
        <input v-else 
        class="todo-item-edit" 
        type="text" 
        v-model="todo.title" 
        @blur="doneEdit" 
        @keyup.enter="doneEdit" 
        @keyup.esc="cancelEdit" 
        v-focus>
        <!-- <div v-for="subGroup in todo.subGroups" :key="subGroup.id">{{subGroup.title}}
            <div v-for="task in subGroup.tasks" :key="task.id" class="todo-item-child">{{task.title}}</div>
        </div> -->
    </div>
    <div class="remove-item" @click="removeTodo(index)">
        &times;
    </div>
  </div>
</template>

<script>
export default {
    name: 'todoItem',
    props: {
        todo: {
            type: Object,
            required: true
        },
        index: {
            type: Number,
            required: true
        }
    },
    data() {
        return {
            id: this.todo.id,
            title: this.todo.title,
            completed: this.todo.completed,
            editing: this.todo.editing,
            beforeEdit: ''
        }
    },
    methods: {
        removeTodo(index) {
            this.$emit('removedTodo', index)
        },
        editTodo() {
            this.beforeEditCache = this.title
            this.editing = true
        },
        doneEdit(){
            if(this.title.trim() === ''){
                this.title = this.beforeEditCache
            }
            this.editing = false
            this.$emit('finishedEdit', {
                index: this.index,
                todo: {
                    id: this.id,
                    title: this.title,
                    completed: this.completed,
                    editing: this.editing
                }
            })
        },
        cancelEdit() {
            this.title = this.beforeEditCache
            this.editing = false
        },
    }
}
</script>

<style>

</style>template>