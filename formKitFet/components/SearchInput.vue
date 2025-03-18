<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  modelValue: { type: String, required: true }
})

const emit = defineEmits(['update:modelValue'])

const inputValue = ref(props.modelValue)
let timeoutId = null // <-- Simplificamos la declaración

watch(inputValue, (newVal) => {
  clearTimeout(timeoutId)
  timeoutId = setTimeout(() => {
    emit("update:modelValue", newVal)
  }, 300)
})

watch(
  () => props.modelValue,
  (newVal) => {
    inputValue.value = newVal
  }
)
</script>

<template>
  <div class="relative">
    <IconSearch
      class="w-3 w-3 absolute top-[50%] translate-y-[-50%] left-2 opacity-30"
    />
    
    <input
      type="text"
      placeholder="Search"
      class="pl-10 p-2 rounded"
      v-model="inputValue"
    />
  </div>
</template>