<script setup lang="ts">
import { watch, reactive, toRaw } from "vue";
import { cloneDeep } from "lodash-es";
import { v4 as uuidv4 } from "uuid";
import draggable from "vuedraggable";
import type { Board, Column, Task } from "@/types";
import TaskCard from "./TaskCard.vue";
import { useAlerts } from "@/stores/alerts";
import TaskCreator from "./TaskCreator.vue";

const alerts = useAlerts();
// props
const props = defineProps<{
  board: Board;
  tasks: Task[];
  addTask(task: Partial<Task>): Promise<Task>;
}>();

//events
const emit = defineEmits<{
  (e: "update", payload: Partial<Board>): void;
}>();

// local data
const tasks = reactive(cloneDeep(props.tasks));
const board = reactive(cloneDeep(props.board));
const columns = reactive<Column[]>(JSON.parse(board.order as string));

// methods
function addColumn() {
  columns.push({
    id: uuidv4(),
    title: "New Column",
    taskIds: [],
  });
}

watch(columns, () => {
  emit("update", cloneDeep({ ...board, order: JSON.stringify(toRaw(columns)) }));
});

const addTask = async ({ title, column }: { title: string; column: Column }) => {
  const newtask = { title };
  try {
    const savedTask = await props.addTask(newtask);
    tasks.push({ ...savedTask });
    column.taskIds.push(savedTask.id);
  } catch (err) {
    alerts.error("Error creating task!");
  }
};
</script>
<template>
  <div class="flex py-12 items-start">
    <draggable
      :list="columns"
      group="columns"
      item-key="id"
      class="flex overflow-x-auto flex-grow-0 flex-shrink-0 pb-8"
    >
      <template #item="{ element: column }">
        <div
          class="column bg-gray-100 flex flex-col justify-between rounded-lg px-3 py-3 rounded mr-4 w-[300px] hover-card"
        >
          <div>
            <h3>
              <input
                type="text"
                :value="column.title"
                class="bg-transparent mb-2"
                @keydown.enter="($event.target as HTMLInputElement).blur()"
                @blur="column.title = ($event.target as HTMLInputElement).value"
              />
            </h3>
            <draggable
              :list="column.taskIds"
              group="tasks"
              item-key="id"
              :animation="200"
              ghost-class="ghost-card"
              class="min-h-[400px]"
            >
              <template #item="{ element: taskId }">
                <TaskCard
                  v-if="tasks.find((t: Task) => t.id === taskId)"
                  :task="(tasks.find((t: Task) => t.id === taskId) as Task)"
                  class="mt-3 cursor-move"
                />
              </template>
            </draggable>
            <TaskCreator @create="addTask({ title: $event, column })" />
          </div>
        </div>
      </template>
    </draggable>
    <button class="text-gray-500" @click="addColumn">New Column +</button>
  </div>
</template>
