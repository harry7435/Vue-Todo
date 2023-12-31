<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { debounce } from 'lodash'
import Sortable from 'sortablejs'
import TodoItem from '~/components/TodoItem.vue'
import { useTodosStore } from '~/store/todos'
import TheIcon from '~/components/TheIcon.vue'
import TheBtn from '~/components/TheBtn.vue'

const todosStore = useTodosStore()
const todoListEl = ref<HTMLDivElement | null>(null)

const debounced = debounce((val: boolean) => {
  todosStore.updateCheckboxes(val)
}, 1000)

const isAllChecked = computed({
  get() {
    return (
      !!todosStore.filteredTodos.length &&
      todosStore.filteredTodos.every((todo) => todo.done)
    )
  },
  set(val: boolean) {
    todosStore.todos.forEach((todo) => {
      todo.done = val
    })
    debounced(val)
  }
})

todosStore.fetchTodos()

onMounted(() => {
  initSorable()
})

function toggleAllCheckboxes() {
  isAllChecked.value = !isAllChecked.value
}

function initSorable() {
  if (todoListEl.value) {
    new Sortable(todoListEl.value, {
      handle: '.drag-handle',
      animation: 0,
      forceFallback: true,
      onEnd: (event) => {
        const { oldIndex, newIndex } = event
        todosStore.reorderTodos({
          oldIndex: oldIndex as number,
          newIndex: newIndex as number
        })
      }
    })
  }
}
</script>

<template>
  <div class="todos-wrap shadow">
    <div class="todo-head">
      <TheIcon
        :active="isAllChecked"
        @click="toggleAllCheckboxes">
        done_all
      </TheIcon>
      <div class="btn-group">
        <TheBtn
          v-for="filter in todosStore.filters"
          :key="filter.name"
          :active="todosStore.filterStatus === filter.name"
          @click="todosStore.filterStatus = filter.name">
          {{ filter.label }}
        </TheBtn>
        <TheBtn
          danger
          @click="todosStore.deleteDoneTodos">
          완료만 삭제
        </TheBtn>
      </div>
    </div>
    <div
      ref="todoListEl"
      class="todo-list">
      <TodoItem
        v-for="todo in todosStore.filteredTodos"
        :key="todo.id"
        :todo="todo" />
    </div>
  </div>
</template>

<style scoped lang="scss">
.todos-wrap {
  border-radius: 6px;
  overflow: hidden;
}
</style>
