<script setup lang="ts">
import { ref, nextTick } from "vue";
import { Input as KInput } from "@progress/kendo-vue-inputs";
const input = ref<HTMLInputElement | null>(null);
const active = ref(false);
const value = ref("");
const emit = defineEmits<{
  (e: "create", payload: string): void;
  (e: "cancel"): void;
}>();
const handleEnter = () => {
  emit("create", value.value);
  value.value = "";
  active.value = false;
};
const handleEsc = () => {
  value.value = "";
  emit("cancel");
  active.value = false;
};
const handleActivate = () => {
  active.value = true;
  nextTick(() => input.value?.focus());
};
</script>

<template>
  <div class="w-full">
    <KInput
      ref="input"
      type="text"
      v-if="active"
      v-model="value"
      @keypress.enter="handleEnter"
      @keypress.esc="handleEsc"
      @blur="handleEsc"
    />
    <button
      v-else
      @click="handleActivate"
      class="text-gray-600 block w-full text-left p-1"
    >
      + Create Task
    </button>
  </div>
</template>
