<template>
  <div class="budget-tab-container flex-1 overflow-y-auto no-scrollbar p-3 space-y-4">
    <!-- 成团人数选择器 (Vant Tabs 卡片风格) -->
    <div class="bg-white rounded-2xl p-3.5 border border-slate-200/80 shadow-xs space-y-2.5">
      <div class="flex items-center justify-between">
        <h3 class="text-xs font-bold text-slate-800 flex items-center gap-1">
          <span>👥</span>
          <span>选择同行成团规模测算</span>
        </h3>
        <span class="text-[10px] text-slate-400">点击切换人均预算</span>
      </div>

      <van-tabs
        v-model:active="selectedGroupSize"
        type="card"
        color="#0284c7"
        class="van-budget-group-tabs"
      >
        <van-tab :name="3" title="3人及多人同行 (553元)" />
        <van-tab :name="2" title="2人同行 (622元)" />
      </van-tabs>
    </div>

    <!-- 动态自费预算汇总卡片 -->
    <div class="bg-gradient-to-br from-sky-600 to-indigo-700 text-white rounded-2xl p-4 shadow-sm space-y-3">
      <div class="flex items-start justify-between">
        <div>
          <van-tag round color="rgba(255,255,255,0.25)" text-color="#ffffff">
            {{ activePlan.recommendation }}
          </van-tag>
          <div class="mt-2 flex items-baseline gap-1">
            <span class="text-xs opacity-85">人均自费总计约</span>
            <span class="text-3xl font-black font-mono tracking-tight">{{ activePlan.total }}</span>
            <span class="text-sm font-bold">元</span>
          </div>
        </div>
        <div class="text-right">
          <span class="text-[11px] text-sky-200">岛内交通人均</span>
          <div class="text-xl font-bold font-mono">{{ activePlan.transport }} <span class="text-xs">元</span></div>
        </div>
      </div>

      <!-- 每日人均分布 -->
      <div class="bg-white/10 backdrop-blur rounded-xl p-2.5 text-xs grid grid-cols-4 gap-1 text-center border border-white/15 font-mono">
        <div>
          <div class="text-[10px] text-sky-200">Day 1</div>
          <div class="font-bold">{{ activePlan.daily[0] }} 元</div>
        </div>
        <div>
          <div class="text-[10px] text-sky-200">Day 2</div>
          <div class="font-bold">{{ activePlan.daily[1] }} 元</div>
        </div>
        <div>
          <div class="text-[10px] text-sky-200">Day 3</div>
          <div class="font-bold">{{ activePlan.daily[2] }} 元</div>
        </div>
        <div>
          <div class="text-[10px] text-sky-200">Day 4</div>
          <div class="font-bold">{{ activePlan.daily[3] }} 元</div>
        </div>
      </div>

      <!-- 详细说明 -->
      <p class="text-xs text-sky-100 leading-relaxed pt-1">
        {{ activePlan.detail }}
      </p>
    </div>

    <!-- 团队组队出行核心建议 (Vant NoticeBar) -->
    <div class="rounded-2xl overflow-hidden border border-sky-200 shadow-2xs">
      <van-notice-bar
        wrapable
        :scrollable="false"
        color="#0369a1"
        background="#f0f9ff"
        left-icon="info-o"
      >
        <strong>💡 多人/团队组队出行核心建议：</strong><br />
        平潭岛内合规网约车以 5 座轿车为主，多人同行时直接按 <strong>3人/车</strong> 分组同时呼叫多辆快车，秒接单且机动性最佳，人均交通费与 3 人方案完全一致（人均 156 元，总计 553 元）。
      </van-notice-bar>
    </div>

    <!-- 费用四大构成卡片 (Vant Cell 组合) -->
    <div class="space-y-2">
      <h3 class="text-xs font-bold text-slate-800 uppercase tracking-wider flex items-center gap-1">
        <span>🧾</span>
        <span>四大项费用明细构成</span>
      </h3>
      <div class="space-y-2">
        <div
          v-for="(cat, idx) in budgetCategories"
          :key="idx"
          class="bg-white rounded-2xl p-3.5 border border-slate-200/80 shadow-xs flex items-start gap-3"
        >
          <span class="text-2xl p-2 rounded-xl bg-slate-50 border border-slate-100 shrink-0">
            {{ cat.icon }}
          </span>
          <div class="flex-1 min-w-0">
            <div class="flex items-center justify-between gap-1">
              <h4 class="text-xs font-bold text-slate-900">{{ cat.name }}</h4>
              <van-tag type="primary" size="medium" round>
                {{ cat.cost }}
              </van-tag>
            </div>
            <p class="text-[11px] text-slate-500 mt-1 leading-relaxed">{{ cat.note }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 成团方案横向对照表 -->
    <div class="bg-white rounded-2xl p-3.5 border border-slate-200/80 shadow-xs space-y-2 pb-6">
      <h3 class="text-xs font-bold text-slate-800 flex items-center gap-1">
        <span>📊</span>
        <span>成团规模费用横向对照</span>
      </h3>
      <div class="overflow-x-auto">
        <table class="w-full text-xs text-center border-collapse">
          <thead>
            <tr class="bg-slate-100 text-slate-700">
              <th class="p-2.5 text-left font-bold rounded-l-lg">项目</th>
              <th class="p-2.5 font-bold" :class="selectedGroupSize === 3 ? 'text-sky-600 bg-sky-50' : ''">3人/多人组队 (推荐)</th>
              <th class="p-2.5 font-bold rounded-r-lg" :class="selectedGroupSize === 2 ? 'text-sky-600 bg-sky-50' : ''">2人同行</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 text-[11px]">
            <tr>
              <td class="p-2.5 text-left text-slate-500 font-medium">岛内交通</td>
              <td class="p-2.5 font-mono font-bold text-emerald-600">156 元</td>
              <td class="p-2.5 font-mono">225 元</td>
            </tr>
            <tr>
              <td class="p-2.5 text-left text-slate-500 font-medium">景点门票</td>
              <td class="p-2.5 font-mono">77 元</td>
              <td class="p-2.5 font-mono">77 元</td>
            </tr>
            <tr>
              <td class="p-2.5 text-left text-slate-500 font-medium">自理正餐</td>
              <td class="p-2.5 font-mono">320 元</td>
              <td class="p-2.5 font-mono">320 元</td>
            </tr>
            <tr class="bg-slate-50 font-bold">
              <td class="p-2.5 text-left text-slate-800">总计人均</td>
              <td class="p-2.5 font-mono text-sky-600 font-bold text-sm">553 元</td>
              <td class="p-2.5 font-mono text-slate-700">622 元</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { GROUP_PLANS, BUDGET_CATEGORIES } from '../data/pingtanData.js';

const selectedGroupSize = ref(3);
const budgetCategories = BUDGET_CATEGORIES;

const activePlan = computed(() => GROUP_PLANS[selectedGroupSize.value] || GROUP_PLANS[3]);
</script>

<style scoped>
.budget-tab-container {
  height: 100%;
}
.van-budget-group-tabs :deep(.van-tabs__nav--card) {
  height: 32px;
  margin: 0;
  border-radius: 9999px;
  overflow: hidden;
}
.van-budget-group-tabs :deep(.van-tab) {
  font-size: 11px;
  line-height: 30px;
  font-weight: 700;
}
</style>
