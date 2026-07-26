<template>
  <section class="nomads-showcase-section">
    <div class="showcase-header">
      <div class="header-left">
        <h2 class="showcase-title">化学知识与反应方程式实战模板库</h2>
        <p class="showcase-subtitle">精选氧化还原、有机机理、周期律推断与实验室安全实战模板，点击“一键套用”快速解析</p>
      </div>
      <span class="showcase-badge">已收录 {{ showcaseItems.length }} 个化学实战模板</span>
    </div>

    <div class="showcase-grid">
      <div 
        v-for="item in showcaseItems" 
        :key="item.id" 
        class="glass-card showcase-card"
      >
        <div class="card-header">
          <span class="scenario-tag">{{ item.tag }}</span>
          <span class="usage-count">{{ item.usageCount }} 次解析</span>
        </div>

        <div class="card-content">
          <h3 class="item-title">{{ item.title }}</h3>
          <p class="item-prompt">“{{ item.prompt }}”</p>
        </div>

        <div class="card-action">
          <button class="apply-btn" @click="applyTemplate(item)">
            <span>一键套用</span>
            <svg class="arrow-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const emit = defineEmits<{
  (e: 'apply-template', payload: { prompt: string; chemistryType?: string; educationLevel?: string; topicArea?: string }): void;
}>();

export interface ShowcaseItem {
  id: string;
  tag: string;
  title: string;
  prompt: string;
  chemistryType?: string;
  educationLevel?: string;
  topicArea?: string;
  usageCount: string;
}

const showcaseItems = computed<ShowcaseItem[]>(() => [
  {
    id: 'huaxue-1',
    tag: '氧化还原',
    title: '浓硫酸脱水性与强氧化性反应解析',
    prompt: '请解析浓硫酸与蔗糖脱水炭化过程，以及加热条件下浓硫酸与金属铜反应的化学方程式、电子转移数与气体产物检验。',
    chemistryType: '氧化还原与离子方程式配平',
    educationLevel: '高中高考化学',
    topicArea: '元素化合物',
    usageCount: '62.4k'
  },
  {
    id: 'huaxue-2',
    tag: '有机机理',
    title: '乙烯与溴水加成反应机理与官能团演变',
    prompt: '详细推导乙烯与溴水发生加成反应的亲电加成机理，分析溴鎓离子中间体生成过程及产物1,2-二溴乙烷的结构特点。',
    chemistryType: '有机化学合成与官能团推导',
    educationLevel: '高中高考化学',
    topicArea: '有机合成',
    usageCount: '54.1k'
  },
  {
    id: 'huaxue-3',
    tag: '离子配平',
    title: '复杂离子反应方程式与分步沉淀配平',
    prompt: '分析向偏铝酸钠溶液中通入过量二氧化碳，以及向氯化铝溶液中逐滴加入氢氧化钠溶液的离子方程式配平与图像特征。',
    chemistryType: '氧化还原与离子方程式配平',
    educationLevel: '高中高考化学',
    topicArea: '化学反应原理',
    usageCount: '48.8k'
  },
  {
    id: 'huaxue-4',
    tag: '电化学',
    title: '铜锌原电池与工业电解精炼铜电极反应',
    prompt: '剖析铜锌原电池电子流向与盐桥作用，比较工业电解精炼铜与粗铜精炼过程中阴阳极电极反应式及阳极泥形成原理。',
    chemistryType: '元素周期律与物质结构解析',
    educationLevel: '高中高考化学',
    topicArea: '化学反应原理',
    usageCount: '58.2k'
  },
  {
    id: 'huaxue-5',
    tag: '周期律结构',
    title: '短周期元素周期律与杂化轨道分子构型推断',
    prompt: '根据短周期主族元素原子半径、第一电离能及最高价氧化物水化物酸碱性递变规律，结合杂化轨道理论推断分子空间构型。',
    chemistryType: '元素周期律与物质结构解析',
    educationLevel: '大学无机有机',
    topicArea: '元素化合物',
    usageCount: '41.9k'
  },
  {
    id: 'huaxue-6',
    tag: '实验安全',
    title: '实验室氯气制备与危险化学品安全防护',
    prompt: '总结实验室用二氧化锰与浓盐酸加热制取氯气的装置连接、除杂干燥、尾气多级吸收及防倒吸等安全规范。',
    chemistryType: '实验现象描述与操作安全规范',
    educationLevel: '高中高考化学',
    topicArea: '化学实验',
    usageCount: '51.3k'
  }
]);

function applyTemplate(item: ShowcaseItem) {
  emit('apply-template', {
    prompt: item.prompt,
    chemistryType: item.chemistryType,
    educationLevel: item.educationLevel,
    topicArea: item.topicArea
  });
}
</script>

<style scoped>
.nomads-showcase-section {
  margin-top: 2rem;
  width: 100%;
}

.showcase-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 1.25rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--card-border);
}

.showcase-title {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text-primary);
  background: var(--primary-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.showcase-subtitle {
  font-size: 0.825rem;
  color: var(--text-secondary);
  margin-top: 0.25rem;
}

.showcase-badge {
  font-size: 0.75rem;
  color: #a5b4fc;
  background: rgba(99, 102, 241, 0.12);
  border: 1px solid rgba(99, 102, 241, 0.25);
  padding: 4px 10px;
  border-radius: 20px;
}

.showcase-grid {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  gap: 1.25rem;
}

@media (min-width: 640px) {
  .showcase-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .showcase-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.showcase-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
  padding: 1.25rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid var(--card-border);
  border-radius: 14px;
  transition: all 0.25s ease;
}

.showcase-card:hover {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(99, 102, 241, 0.4);
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.scenario-tag {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(168, 85, 247, 0.15);
  color: #c084fc;
  border: 1px solid rgba(168, 85, 247, 0.3);
}

.usage-count {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.card-content {
  margin-bottom: 1rem;
  flex: 1;
}

.item-title {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0.4rem;
}

.item-prompt {
  font-size: 0.825rem;
  color: var(--text-secondary);
  line-height: 1.45;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  font-style: italic;
}

.card-action {
  padding-top: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.04);
}

.apply-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 0.5rem 1rem;
  background: rgba(99, 102, 241, 0.1);
  border: 1px solid rgba(99, 102, 241, 0.3);
  border-radius: 8px;
  color: #a5b4fc;
  font-size: 0.825rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.showcase-card:hover .apply-btn {
  background: var(--primary-gradient);
  border-color: transparent;
  color: white;
}

.arrow-icon {
  width: 14px;
  height: 14px;
  transition: transform 0.2s ease;
}

.apply-btn:hover .arrow-icon {
  transform: translateX(3px);
}
</style>
