<script setup lang="ts">
import { ref, computed } from "vue";

import IconPlay from "./icons/IconPlay.vue";
import IconReset from "./icons/IconReset.vue";
import IconRegenerate from "./icons/IconRegenerate.vue";

const input = ref([1, 8, 6, 2, 5, 4, 8, 3, 7]);
const left = ref(0);
const right = ref(input.value.length - 1);

const chartDimensions = ref({ width: 450, height: 450 });
const barWidth = computed(() => chartDimensions.value.width / 20);
const barGap = computed(() => barWidth.value * 1.5);

function calculateArea(input: number[], left: number, right: number): number {
  const width = right - left;
  const height = Math.min(input[left], input[right]);
  return width * height;
}

const initialArea = calculateArea(input.value, left.value, right.value);
const results = ref({ area: initialArea, maximum: initialArea });

function movePointer(): void {
  if (input.value[left.value] <= input.value[right.value]) {
    left.value++;
  } else {
    right.value--;
  }
  const newArea = calculateArea(input.value, left.value, right.value);
  results.value = { area: newArea, maximum: Math.max(results.value.maximum, newArea) };
}

function resetPointers(input: number[]): void {
  left.value = 0;
  right.value = input.length - 1;
  const defaultArea = (right.value - left.value) * Math.min(input[left.value], input[right.value]);
  results.value = { area: defaultArea, maximum: defaultArea };
}
</script>

<template>
  <main>
    <div class="input">
      <div>[{{ input.join(", ") }}]</div>
      <button class="regenerate"><IconRegenerate /></button>
    </div>

    <div class="results">
      <div>Площадь: {{ results.area }}</div>
      <div>Максимум: {{ results.maximum }}</div>
    </div>

    <div class="chart" :style="{ height: `${chartDimensions.height}px`, gap: `${barGap}px` }">
      <div v-for="(height, index) in input" :key="index">
        <div class="bar-label">{{ height }}</div>
        <div
          class="bar"
          :style="{
            height: `${(height / Math.max(...input)) * chartDimensions.height}px`,
            width: `${barWidth}px`,
          }"
        ></div>
      </div>

      <div
        class="water"
        :style="{
          height: `${
            (Math.min(input[left], input[right]) * chartDimensions.height) / Math.max(...input)
          }px`,
          width: `${(right - left + 1) * barWidth + barGap * (right - left)}px`,
          left: `${left * barWidth + left * barGap}px`,
        }"
      ></div>

      <div
        class="pointer"
        :style="{
          left: `${left * barWidth + left * barGap}px`,
          bottom: `${-16 - barWidth}px`,
          borderLeftWidth: `${barWidth / 2}px`,
          borderRightWidth: `${barWidth / 2}px`,
          borderBottomWidth: `${barWidth}px`,
        }"
      ></div>

      <div
        class="pointer"
        :style="{
          left: `${right * barWidth + right * barGap}px`,
          bottom: `${-16 - barWidth}px`,
          borderLeftWidth: `${barWidth / 2}px`,
          borderRightWidth: `${barWidth / 2}px`,
          borderBottomWidth: `${barWidth}px`,
        }"
      ></div>
    </div>

    <div class="buttons">
      <button class="button" @click="movePointer" :disabled="right - left <= 1">
        <IconPlay />
        Следующий шаг
      </button>

      <button
        class="button"
        @click="resetPointers(input)"
        :disabled="left === 0 && right === input.length - 1"
      >
        <IconReset />
        Сбросить
      </button>
    </div>
  </main>
</template>

<style scoped lang="scss">
main {
  height: 100dvh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.input {
  display: flex;
  gap: 0.5rem;
  font-family: "Ubuntu Mono", monospace;
  font-weight: 700;
  font-size: 1.5rem;
}

.regenerate {
  svg {
    width: 1.5rem;
    height: 1.5rem;
  }
}

.results {
  margin-top: 1rem;
  font-size: 1.2rem;
  text-align: center;
}

.chart {
  position: relative;
  box-sizing: content-box;
  margin-top: 2.5rem;
  display: flex;
  align-items: flex-end;
  border-bottom: 0.5rem solid #000000;
}

.bar-label {
  font-family: "Ubuntu Mono", monospace;
  font-weight: 400;
  font-size: 1.3rem;
  line-height: 1.4;
  text-align: center;
}

.bar {
  background-color: #000000;
}

.water {
  position: absolute;
  bottom: 0;
  z-index: -1;
  background-color: #0183fd;
}

.pointer {
  position: absolute;
  width: 0;
  height: 0;
  border-left-style: solid;
  border-right-style: solid;
  border-bottom-style: solid;
  border-left-color: transparent;
  border-right-color: transparent;
  border-bottom-color: #ff0000;
}

.buttons {
  padding-inline: 0.5rem;
  display: flex;
  gap: 1rem;
}

.button {
  margin-top: 3.5rem;
  padding: 0.7rem 1rem;
  border: 0.19rem solid #000000;
  border-radius: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.3rem;
  font-weight: 700;
  background-color: transparent;

  &:disabled {
    opacity: 0.5;
    cursor: default;
  }

  svg {
    width: 1rem;
    height: 1rem;
  }
}
</style>
