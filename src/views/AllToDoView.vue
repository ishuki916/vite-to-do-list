<script setup>
import { defineProps, defineEmits } from 'vue'
const props = defineProps({
  todo: {
    type: Object,
    required: true
  }
})
const emits = defineEmits(['delete', 'toggle'])
</script>
<template>
  <li>
    <label class="todoList_label">
      <input
        class="todoList_input"
        type="checkbox"
        :checked="todo.status"
        @change="emits('toggle')"
      />
      <span>{{ todo.content }}</span>
    </label>
    <a @click="emits('delete')">
      <i class="fa fa-times"></i>
    </a>
  </li>
</template>
