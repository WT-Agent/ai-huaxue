<template>
  <div class="app-container">
    <!-- 常驻悬浮分享按钮 (H5 / 移动端与桌面端通用) -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg class="share-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享化学专家工具</span>
    </button>

    <header>
      <div class="user-status-bar" style="margin-bottom: 0.75rem; font-size: 0.8rem; text-align: center;">
        <span v-if="isLoggedIn" class="status-badge logged-in" style="background: rgba(192, 132, 252, 0.15); color: #c084fc; padding: 4px 12px; border-radius: 12px; border: 1px solid rgba(192, 132, 252, 0.3);">
          已登录 (每日 15 次额度 · 今日已用: {{ authUsesCount }}/15)
        </span>
      </div>
      <h1>{{ appTitle }}</h1>
      <p>化学反应方程式配平 · 电子转移与机理推导 · 高考考点与推断题 · 实验室安全规范</p>
    </header>

    <!-- 动态广播轮播 -->
    <UserTicker />

    <!-- 核心操作区卡片 -->
    <main ref="inputCardRef" class="glass-card input-group">
      <!-- 化学解析类型选择 -->
      <div class="selector-group">
        <label class="selector-label">选择化学解析与推导类型</label>
        <div class="style-selector">
          <button 
            v-for="ctype in chemistryTypeOptions" 
            :key="ctype.value"
            class="style-option"
            :class="{ active: activeChemistryType === ctype.value }"
            @click="activeChemistryType = ctype.value"
          >
            {{ ctype.label }}
          </button>
        </div>
      </div>

      <!-- 适用学段与知识板块 -->
      <div class="options-row" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem;">
        <div class="selector-group">
          <label class="selector-label">适用学段</label>
          <div class="style-selector">
            <button 
              v-for="level in educationLevelOptions" 
              :key="level"
              class="style-option"
              :class="{ active: selectedEducationLevel === level }"
              @click="selectedEducationLevel = level"
            >
              {{ level }}
            </button>
          </div>
        </div>

        <div class="selector-group">
          <label class="selector-label">知识板块侧重</label>
          <div class="style-selector">
            <button 
              v-for="topic in topicAreaOptions" 
              :key="topic"
              class="style-option"
              :class="{ active: selectedTopicArea === topic }"
              @click="selectedTopicArea = topic"
            >
              {{ topic }}
            </button>
          </div>
        </div>
      </div>

      <!-- 化学反应/考点需求输入框 -->
      <div class="selector-group">
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <label class="selector-label">输入化学方程式、反应物生成物、有机结构或高考考点题目</label>
          <div style="display: flex; gap: 0.5rem;">
            <button v-if="userInput" class="text-link-btn" @click="userInput = ''">清空输入</button>
            <button class="text-link-btn" @click="showChemistryTipsModal = true">实验安全与配平指南</button>
          </div>
        </div>
        <textarea 
          v-model="userInput" 
          placeholder="请输入您需要解析的化学方程式、反应系统或考点题目...（例如：分析浓硫酸与铜加热反应的化学方程式配平、得失电子转移方向及生成气体检验方法；或分析乙烯与溴水发生加成反应的机理过程。）"
          style="min-height: 130px;"
        ></textarea>
        <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-secondary);">
          <span>字符数: {{ userInput.length }} 字</span>
          <span>建议包含反应物、催化条件、化合价变化或机理疑问</span>
        </div>
      </div>

      <!-- 操作按钮区 -->
      <div style="display: flex; gap: 0.75rem;">
        <button 
          class="action-btn" 
          :disabled="loading || !userInput.trim()"
          @click="handleGenerate"
        >
          {{ loading ? '正在严谨推导与生成化学学术报告中...' : '开始生成化学深度解析报告' }}
        </button>
        <button class="icon-btn" style="padding: 0 1rem; border-radius: 10px;" @click="toggleHistoryDrawer">
          历史解析 ({{ historyList.length }})
        </button>
      </div>

      <!-- 异常提示 -->
      <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
        {{ errorMsg }}
      </div>
    </main>

    <!-- 生成结果卡片 -->
    <section v-if="result || loading" class="glass-card">
      <div class="result-header">
        <span class="result-title">化学知识与反应方程式深度解析报告</span>
        <div class="button-actions">
          <button v-if="result" class="icon-btn" @click="copyText">
            {{ copied ? '已复制解析全文' : '复制化学报告' }}
          </button>
          <button v-if="result" class="icon-btn" @click="resetResult">
            重置
          </button>
        </div>
      </div>

      <!-- 加载中骨架屏 -->
      <div v-if="loading" class="skeleton">
        <div class="skeleton-line" style="width: 85%"></div>
        <div class="skeleton-line" style="width: 95%"></div>
        <div class="skeleton-line" style="width: 70%"></div>
        <div class="skeleton-line" style="width: 90%"></div>
        <div class="skeleton-line" style="width: 60%"></div>
      </div>

      <!-- 渲染结果 -->
      <div v-else-if="result">
        <!-- AI 共识打分可视化看板 -->
        <div v-if="aiScores" class="scores-container" style="margin-bottom: 1.5rem; padding: 1.25rem; background: rgba(0,0,0,0.25); border-radius: 12px; border: 1px solid rgba(255,255,255,0.06);">
          <div style="font-weight: 700; font-size: 0.95rem; margin-bottom: 1rem; color: #a5b4fc; display: flex; justify-content: space-between; align-items: center;">
            <span>AI 化学考据与机理质量评估看板</span>
            <span style="font-size: 0.8rem; font-weight: normal; color: var(--text-secondary);">综合考据分: {{ getAverageScoreFromMap(aiScores) }} / 5.0</span>
          </div>
          <div class="metrics-grid" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1rem;">
            <div v-for="metric in metricsList" :key="metric.key" class="metric-item">
              <div style="display: flex; justify-content: space-between; font-size: 0.8rem; margin-bottom: 0.3rem;">
                <span style="color: var(--text-secondary);">{{ metric.label }}</span>
                <span style="font-weight: bold; color: var(--accent-color);">{{ aiScores[metric.key] || 4 }} / 5</span>
              </div>
              <div class="bar-bg" style="height: 6px; background: rgba(255,255,255,0.08); border-radius: 3px; overflow: hidden;">
                <div class="bar-fill" :style="{ width: ((aiScores[metric.key] || 4) * 20) + '%', background: 'var(--primary-gradient)', height: '100%', borderRadius: '3px', transition: 'width 0.5s ease' }"></div>
              </div>
            </div>
          </div>
        </div>

        <div class="output-content">{{ displayResultText }}</div>
      </div>
    </section>

    <!-- 历史记录面板 -->
    <section v-if="showHistory" class="glass-card" style="margin-top: 1rem;">
      <div class="result-header">
        <span class="result-title">本地化学解析历史记录</span>
        <button class="icon-btn" @click="showHistory = false">关闭记录</button>
      </div>

      <div v-if="historyList.length === 0" style="text-align: center; color: var(--text-secondary); padding: 1.5rem; font-size: 0.85rem;">
        暂无历史化学解析记录，开始生成第一个案例吧！
      </div>

      <div v-else class="history-grid" style="display: flex; flex-direction: column; gap: 0.75rem; max-height: 320px; overflow-y: auto;">
        <div v-for="item in historyList" :key="item.id" class="history-item" style="padding: 1rem; background: rgba(0,0,0,0.2); border-radius: 10px; border: 1px solid var(--card-border);">
          <div style="display: flex; justify-content: space-between; font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 0.4rem;">
            <span>{{ item.timestamp }} · [{{ item.chemistryType }} / {{ item.educationLevel }}]</span>
            <span style="color: var(--primary-color);">评分: {{ getAverageScore(item) }}</span>
          </div>
          <div style="font-size: 0.85rem; font-weight: bold; margin-bottom: 0.4rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; color: var(--text-primary);">
            关键词: {{ item.input }}
          </div>
          <div style="display: flex; gap: 0.5rem;">
            <button class="icon-btn" style="font-size: 0.75rem;" @click="applyHistory(item)">套用提示词</button>
            <button class="icon-btn" style="font-size: 0.75rem;" @click="viewHistoryOutput(item)">查看解析全文</button>
          </div>
        </div>
      </div>
    </section>

    <!-- 化学考察模版 Showcase -->
    <NomadsShowcase
      @apply-template="handleApplyTemplate"
    />

    <!-- 化学实验安全与配平指南 Modal -->
    <div v-if="showChemistryTipsModal" class="modal-overlay" @click.self="showChemistryTipsModal = false">
      <div class="modal-content" style="max-width: 480px;">
        <h3>化学实验安全防范与方程式配平指南</h3>
        <p style="text-align: left; font-size: 0.825rem; margin-bottom: 1rem; color: var(--text-secondary);">
          确保化学方程式推导准确、反应机理清晰且符合实验室安全操作标准：
        </p>
        <div class="modal-scroll-area" style="text-align: left; font-size: 0.825rem;">
          <div v-for="(rule, idx) in chemistryGuideRules" :key="idx" style="margin-bottom: 0.75rem; padding: 0.5rem 0.75rem; background: rgba(255,255,255,0.03); border-radius: 8px; border: 1px solid rgba(255,255,255,0.05);">
            <div style="color: var(--accent-color); font-weight: bold; margin-bottom: 0.2rem;">{{ rule.title }}</div>
            <div style="color: var(--text-primary); margin-bottom: 0.2rem;">规范要求: {{ rule.req }}</div>
            <div style="color: var(--text-secondary); font-size: 0.775rem;">避坑指南: {{ rule.avoid }}</div>
          </div>
        </div>
        <button class="modal-btn" style="margin-top: 1rem;" @click="showChemistryTipsModal = false">关闭</button>
      </div>
    </div>

    <!-- 微信 H5 悬浮分享引导 Modal -->
    <div v-if="showShareGuide" class="modal-overlay" @click.self="showShareGuide = false">
      <div class="modal-content">
        <h3>分享化学知识与反应方程式专家</h3>
        <p>扫码关注或将链接转发给化学研究员、高考考生与教研团队，探索化学科学魅力。</p>
        
        <div class="qr-code-placeholder">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" width="100%" height="100%">
            <rect width="100" height="100" fill="white"/>
            <rect x="5" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="9" width="17" height="17" fill="white"/>
            <rect x="13" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="70" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="74" y="9" width="17" height="17" fill="white"/>
            <rect x="78" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="5" y="70" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="74" width="17" height="17" fill="white"/>
            <rect x="13" y="78" width="9" height="9" fill="#110e24"/>
            <rect x="35" y="10" width="8" height="8" fill="#110e24"/>
            <rect x="48" y="5" width="6" height="12" fill="#110e24"/>
            <rect x="60" y="15" width="5" height="5" fill="#110e24"/>
            <rect x="35" y="35" width="10" height="10" fill="#110e24"/>
            <rect x="50" y="45" width="15" height="8" fill="#110e24"/>
            <rect x="40" y="70" width="8" height="16" fill="#110e24"/>
            <rect x="55" y="65" width="10" height="10" fill="#110e24"/>
            <rect x="75" y="40" width="12" height="12" fill="#110e24"/>
            <rect x="75" y="75" width="15" height="15" fill="#110e24"/>
            <rect x="45" y="80" width="8" height="8" fill="#110e24"/>
          </svg>
        </div>

        <div style="font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 1.5rem;">
          微信号: <span style="color: var(--primary-color); font-weight: bold;">{{ wechatId }}</span>
        </div>

        <button class="modal-btn" @click="showShareGuide = false">关闭</button>
      </div>
    </div>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
      <a href="https://api.wuxian.xyz/sign-up?aff=OyRY" target="_blank" rel="noopener noreferrer" class="footer-link-btn">API 平台</a>
      <a href="https://www.kutuyun.com/aff/IPJKCKWF" target="_blank" rel="noopener noreferrer" class="footer-link-btn">酷兔云</a>
      <a href="https://bandwagonhost.com/aff.php?aff=48115" target="_blank" rel="noopener noreferrer" class="footer-link-btn">搬瓦工</a>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们高度重视化学实验与教研数据隐私。您在本工具中输入的化学方程式、反应物生成物及考查课题仅用于大模型实时分析生成，系统不会在云端长期保存或公开。</p>
          <p>为了保障免费使用额度，本应用会在您的浏览器本地（localStorage）记录试用次数与解锁状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用网腾无限 AI 化学知识与反应方程式专家。本工具生成的化学方程式配平、机理推导及实验规范供学术研究、教学备课与自学参考。</p>
          <p>进行实际化学实验时，请务必在具备资质的实验室进行，并严格遵守国家化学危险品安全规程及老师/导师现场指导。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信交流" class="contact-qr-img" />
              <span class="contact-qr-label">微信交流</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉联系" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉联系</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import NomadsShowcase from './components/NomadsShowcase.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

const appTitle = ref(appConfig.title || '网腾无限AI - 化学知识与反应方程式专家');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inputCardRef = ref<HTMLElement | null>(null);
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showChemistryTipsModal = ref(false);
const showHistory = ref(false);

interface HistoryRecord {
  id: string;
  timestamp: string;
  input: string;
  chemistryType: string;
  educationLevel: string;
  topicArea: string;
  result: string;
  scores: Record<string, number> | null;
}

const historyList = ref<HistoryRecord[]>([]);

// 4 种化学解析类型
const chemistryTypeOptions = [
  { label: '氧化还原与离子方程式配平', value: '氧化还原与离子方程式配平' },
  { label: '有机化学合成与官能团推导', value: '有机化学合成与官能团推导' },
  { label: '元素周期律与物质结构解析', value: '元素周期律与物质结构解析' },
  { label: '实验现象描述与操作安全规范', value: '实验现象描述与操作安全规范' },
];
const activeChemistryType = ref(chemistryTypeOptions[0].value);

// 适用学段与知识板块选项
const educationLevelOptions = ['初中化学', '高中高考化学', '大学无机有机', '科普常识'];
const selectedEducationLevel = ref('高中高考化学');

const topicAreaOptions = ['元素化合物', '化学反应原理', '有机合成', '化学实验'];
const selectedTopicArea = ref('元素化合物');

// 指南规则
const chemistryGuideRules = [
  {
    title: '1. 方程式与离子反应配平守恒原则',
    req: '严格遵守质量守恒、电荷守恒与得失电子守恒，正确标明沉淀符号、气体符号与反应条件（加热、催化剂等）。',
    avoid: '切忌电荷不守恒、漏标反应条件或错写反应产物化合价。'
  },
  {
    title: '2. 有机化学反应机理与官能团演绎',
    req: '清晰梳理反应试剂类型（如亲核试剂、亲电试剂）、中间体构型、断键与成键位置及主要副产物。',
    avoid: '切忌混淆取代与加成、消去反应条件（如氢氧化钠水溶液与醇溶液的区别）。'
  },
  {
    title: '3. 实验现象描述客观精准性',
    req: '严格区分“现象”（如产生无色刺激性气体、溶液由无色变为蓝色）与“结论”（如生成二阶铜离子）。',
    avoid: '切忌把结论直接写成现象，避免遗漏微观沉淀溶解或气体溢出过程。'
  },
  {
    title: '4. 实验室危险品防护与安全应急',
    req: '针对强氧化剂、易燃有机溶剂及剧毒气体，提供防倒吸装置、尾气吸收液及酸碱灼伤应急处置规范。',
    avoid: '切忌将水直接倒入浓硫酸，切忌无通风设施进行有毒气体实验。'
  }
];

// 评估看板指标
const metricsList = [
  { key: 'chemicalFormulaAccuracy', label: '化学式与方程式准确度' },
  { key: 'reactionMechanismClarity', label: '反应机理清晰度' },
  { key: 'experimentalSafetyRigor', label: '实验安全规范度' },
  { key: 'syllabusAlignment', label: '考纲知识点契合度' },
  { key: 'popularScienceAccessibility', label: '化学科普通俗度' }
];

const aiScores = ref<Record<string, number> | null>(null);

// 解析 Cookie
const getCookie = (name: string): string | null => {
  const nameEQ = name + '=';
  const ca = document.cookie.split(';');
  for (let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) === ' ') c = c.substring(1, c.length);
    if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
  }
  return null;
};

// SSO 用户状态
const userToken = ref(getCookie('wuxian_session'));
const isLoggedIn = computed(() => !!userToken.value);
const authUsesCount = ref(parseInt(localStorage.getItem('auth_uses') || '0', 10));

// 免费限制
const isLimitReached = computed(() => {
  if (isLoggedIn.value) {
    return authUsesCount.value >= 15;
  }
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

// API 端点
const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

onMounted(() => {
  loadHistory();
});

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('huaxue_history_records');
    if (raw) {
      historyList.value = JSON.parse(raw);
    }
  } catch (e) {
    console.error('读取历史记录失败', e);
  }
};

const saveHistory = (record: HistoryRecord) => {
  try {
    historyList.value.unshift(record);
    if (historyList.value.length > 20) {
      historyList.value = historyList.value.slice(0, 20);
    }
    localStorage.setItem('huaxue_history_records', JSON.stringify(historyList.value));
  } catch (e) {
    console.error('保存历史记录失败', e);
  }
};

const parseScores = (text: string) => {
  const match = text.match(/\[HUAXUE_SCORES\](.*?)\[\/HUAXUE_SCORES\]/);
  if (!match) return null;
  const content = match[1];
  const scores: Record<string, number> = {};
  const pairs = content.split(',');
  pairs.forEach(p => {
    const [k, v] = p.split(':');
    if (k && v) {
      const num = parseInt(v.trim(), 10);
      if (!isNaN(num)) {
        scores[k.trim()] = num;
      }
    }
  });
  return Object.keys(scores).length > 0 ? scores : null;
};

const displayResultText = computed(() => {
  if (!result.value) return '';
  return result.value.replace(/\[HUAXUE_SCORES\](.*?)\[\/HUAXUE_SCORES\]/g, '').trim();
});

const getAverageScoreFromMap = (scoresMap: Record<string, number> | null) => {
  if (!scoresMap) return '4.8';
  const vals = Object.values(scoresMap);
  if (vals.length === 0) return '4.8';
  const sum = vals.reduce((a, b) => a + b, 0);
  return (sum / vals.length).toFixed(1);
};

const getAverageScore = (item: HistoryRecord) => {
  return getAverageScoreFromMap(item.scores);
};

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const combinedPrompt = `${promptTopic.value}\n【解析类型】：${activeChemistryType.value}\n【适用学段】：${selectedEducationLevel.value}\n【知识板块】：${selectedTopicArea.value}\n【用户查询】：${userInput.value}`;
    
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: combinedPrompt
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      const parsed = parseScores(data.result);
      aiScores.value = parsed;

      // 保存到本地历史记录
      saveHistory({
        id: Date.now().toString(),
        timestamp: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' }),
        input: userInput.value,
        chemistryType: activeChemistryType.value,
        educationLevel: selectedEducationLevel.value,
        topicArea: selectedTopicArea.value,
        result: data.result,
        scores: parsed
      });
      
      if (isLoggedIn.value) {
        const nextAuthUses = authUsesCount.value + 1;
        localStorage.setItem('auth_uses', nextAuthUses.toString());
        authUsesCount.value = nextAuthUses;
      } else {
        const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
        localStorage.setItem('free_uses', (currentUses + 1).toString());
      }
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleApplyTemplate = (payload: { prompt: string; chemistryType?: string; educationLevel?: string; topicArea?: string }) => {
  userInput.value = payload.prompt;
  if (payload.chemistryType) {
    activeChemistryType.value = payload.chemistryType;
  }
  if (payload.educationLevel) {
    selectedEducationLevel.value = payload.educationLevel;
  }
  if (payload.topicArea) {
    selectedTopicArea.value = payload.topicArea;
  }
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const toggleHistoryDrawer = () => {
  showHistory.value = !showHistory.value;
};

const applyHistory = (item: HistoryRecord) => {
  userInput.value = item.input;
  if (item.chemistryType) activeChemistryType.value = item.chemistryType;
  if (item.educationLevel) selectedEducationLevel.value = item.educationLevel;
  if (item.topicArea) selectedTopicArea.value = item.topicArea;
  showHistory.value = false;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const viewHistoryOutput = (item: HistoryRecord) => {
  result.value = item.result;
  aiScores.value = item.scores;
  showHistory.value = false;
};

const resetResult = () => {
  result.value = '';
  aiScores.value = null;
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>
