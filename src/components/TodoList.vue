<script setup lang="ts">
import { computed } from 'vue'
import { debounce } from 'lodash'
import TodoItem from '~/components/TodoItem.vue'
import { useTodosStore } from '~/store/todos'
import TheIcon from '~/components/TheIcon.vue'

const todosStore = useTodosStore()

const debounced = debounce((val: boolean) => {
  todosStore.updateCheckboxes(val)
}, 1000)

const isAllChecked = computed({
  get() {
    return todosStore.todos.every((todo) => todo.done)
  },
  set(val: boolean) {
    todosStore.todos.forEach((todo) => {
      todo.done = val
    })
    debounced(val)
  }
})

todosStore.fetchTodos()

function toggleAllCheckboxes() {
  isAllChecked.value = !isAllChecked.value
}
</script>

<template>
  <div class="todo-head">
    <TheIcon
      :active="isAllChecked"
      @click="toggleAllCheckboxes">
      done_all
    </TheIcon>
  </div>
  <div class="todo-list">
    <TodoItem
      v-for="todo in todosStore.todos"
      :key="todo.id"
      :todo="todo" />
  </div>
</template>

<style scoped lang="scss"></style>
