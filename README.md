# Визуализация задачи про контейнеры с водой

![Vue.js](https://img.shields.io/badge/vuejs-%2335495e.svg?style=for-the-badge&logo=vuedotjs&logoColor=%234FC08D)
![TypeScript](https://img.shields.io/badge/typescript-%23007ACC.svg?style=for-the-badge&logo=typescript&logoColor=white)
![SASS](https://img.shields.io/badge/SASS-hotpink.svg?style=for-the-badge&logo=SASS&logoColor=white)

https://tsykunov.ru/containers

Интерактивная визуализация задачи [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/description/) с LeetCode. График анимированный и адаптивный.

<img src="https://github.com/user-attachments/assets/18681ade-02ba-4ac1-8cb0-4732f96f4190" width="600" />

Задача решается методом двух указателей. На каждом шаге нужно сдвигать указатель, который указывает на меньшую высоту.

Вот пример решения на JavaScript:

```js
function calculateMaxArea(heights) {
  let maxArea = 0;
  let left = 0;
  let right = heights.length - 1;

  while (left < right) {
    const height = Math.min(heights[left], heights[right]);
    const width = right - left;
    const area = height * width;
    maxArea = Math.max(maxArea, area);

    if (heights[left] <= heights[right]) {
      left++;
    } else {
      right--;
    }
  }

  return maxArea;
}
```
