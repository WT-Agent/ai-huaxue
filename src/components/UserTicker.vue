<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[解析生成完成]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  { text: '用户 138****9281 完成了浓硫酸与铜加热反应方程式与电子转移解析' },
  { text: '用户 177****4019 完成了乙烯与溴水加成反应机理及立体化学历程推导' },
  { text: '用户 189****3320 生成了过量二氧化碳通入偏铝酸钠溶液的离子方程式' },
  { text: '用户 131****8829 完成了元素周期律原子半径与电负性递变规律剖析' },
  { text: '用户 153****1290 完成了实验室氯气制备与尾气多级吸收安全规范指南' },
]);
</script>
