<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";

import IconPlay from "./icons/IconPlay.vue";
import IconReset from "./icons/IconReset.vue";
import IconRegenerate from "./icons/IconRegenerate.vue";

const input = ref([1, 8, 6, 2, 5, 4, 8, 3, 7]);
const left = ref(0);
const right = ref(input.value.length - 1);

const viewport = ref({
  width: window.innerWidth,
  height: window.innerHeight,
});

const chartDimensions = computed(() => ({
  width: Math.min(450, viewport.value.width - 50),
  height: Math.min(450, viewport.value.height / 2.5),
}));

const barWidth = computed(() => chartDimensions.value.width / 20);
const barGap = computed(() => barWidth.value * 1.5);

function getCurrentArea(): number {
  const width = right.value - left.value;
  const height = Math.min(input.value[left.value], input.value[right.value]);
  return width * height;
}

const initialArea = getCurrentArea();
const results = ref({ area: initialArea, maximum: initialArea });

const isIconSpinning = ref(false);

function movePointer(): void {
  if (input.value[left.value] <= input.value[right.value]) {
    left.value++;
  } else {
    right.value--;
  }
  const newArea = getCurrentArea();
  results.value = { area: newArea, maximum: Math.max(results.value.maximum, newArea) };
}

function resetPointers(): void {
  left.value = 0;
  right.value = input.value.length - 1;
  const newArea = getCurrentArea();
  results.value = { area: newArea, maximum: newArea };
}

function regenerateInput(): void {
  isIconSpinning.value = true;
  input.value = input.value.map(() => Math.floor(Math.random() * 10));
  resetPointers();
}

function resizeChart(): void {
  viewport.value = {
    width: window.innerWidth,
    height: window.innerHeight,
  };
}

onMounted(() => {
  window.addEventListener("resize", resizeChart);
});

onUnmounted(() => {
  window.removeEventListener("resize", resizeChart);
});
</script>

<template>
  <main>
    <div class="input">
      <div>[{{ input.join(", ") }}]</div>
      <button
        class="regenerate"
        :class="{ spinning: isIconSpinning }"
        @click="regenerateInput"
        @animationend="isIconSpinning = false"
      >
        <IconRegenerate />
      </button>
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
        @click="resetPointers"
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

  @media (width <= 380px) {
    word-spacing: -0.3rem;
  }
}

.regenerate {
  svg {
    width: 1.5rem;
    height: 1.5rem;
  }

  &.spinning {
    svg {
      animation: spin 0.3s;
    }

    @keyframes spin {
      0% {
        rotate: 0deg;
      }
      100% {
        rotate: 180deg;
      }
    }
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

  @media (width <= 380px) {
    font-size: 1rem;
  }
}

.bar {
  background-color: #000000;
  transition: height 0.3s;
}

.water {
  position: absolute;
  bottom: 0;
  z-index: -1;
  background-color: #0183fd;
  transition: all 0.1s;
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
  transition: left 0.1s;
}

.buttons {
  padding-inline: 0.5rem;
  display: flex;
  gap: 1rem;

  @media (width <= 360px) {
    font-size: 0.8rem;
  }
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
  transition: opacity 0.1s;

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
