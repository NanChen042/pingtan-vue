<template>
  <div class="food-tab-container flex-1 overflow-y-auto no-scrollbar p-3 space-y-4">
    <!-- 头部横幅 -->
    <div class="bg-gradient-to-r from-amber-500 to-orange-600 text-white rounded-2xl p-4 shadow-sm space-y-1.5">
      <div class="flex items-center justify-between">
        <van-tag round color="rgba(255,255,255,0.25)" text-color="#ffffff">
          海岛寻味指南
        </van-tag>
        <span class="text-xs opacity-90 font-mono">人均自理 320 元</span>
      </div>
      <h2 class="text-base font-bold">地道风味小吃 · 明码标价避雷指南</h2>
      <p class="text-xs text-amber-100 leading-relaxed">
        百元吃透平潭老城市井碳水与现捞海鲜，认准本地排档与老铺，拒绝路边拉客宰客。
      </p>
    </div>

    <!-- 四大地道必吃小吃 -->
    <div class="space-y-2">
      <h3 class="text-xs font-bold text-slate-800 uppercase tracking-wider flex items-center gap-1">
        <span>🥟</span>
        <span>四大必吃传统风味</span>
      </h3>
      <div class="grid grid-cols-2 gap-2.5">
        <div
          v-for="(snack, idx) in foodData.snacks"
          :key="idx"
          class="bg-white rounded-2xl p-3 border border-slate-200/80 shadow-xs flex flex-col justify-between space-y-2"
        >
          <div>
            <div class="flex items-center justify-between">
              <span class="text-2xl">{{ snack.icon }}</span>
              <van-tag type="warning" size="medium" round>
                {{ snack.price }}
              </van-tag>
            </div>
            <h4 class="text-xs font-bold text-slate-900 mt-1.5">{{ snack.name }}</h4>
            <van-tag type="primary" plain size="small" class="mt-0.5">
              {{ snack.tag }}
            </van-tag>
            <p class="text-[11px] text-slate-500 mt-1 leading-relaxed">
              {{ snack.desc }}
            </p>
          </div>
          <div class="pt-1.5 border-t border-slate-100 text-[10px] text-slate-400 truncate">
            📍 {{ snack.location }}
          </div>
        </div>
      </div>
    </div>

    <!-- 4 大海鲜防坑避雷法则 -->
    <div class="bg-rose-50/70 border border-rose-200 rounded-2xl p-3.5 space-y-2.5">
      <h3 class="text-xs font-bold text-rose-900 flex items-center gap-1.5">
        <span>🛡️</span>
        <span>海鲜排档防坑四步黄金法则</span>
      </h3>
      <div class="space-y-2">
        <div
          v-for="(rule, idx) in foodData.antiScamRules"
          :key="idx"
          class="bg-white p-2.5 rounded-xl border border-rose-100 flex items-start gap-2 shadow-2xs"
        >
          <span class="w-4 h-4 rounded-full bg-rose-600 text-white text-[10px] font-bold flex items-center justify-center shrink-0 mt-0.5">
            {{ rule.step }}
          </span>
          <div>
            <h5 class="text-xs font-bold text-slate-800">{{ rule.title }}</h5>
            <p class="text-[11px] text-slate-600 mt-0.5 leading-relaxed">{{ rule.rule }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 4 家行程严选餐厅 (Vant Cell 组合) -->
    <div class="space-y-2 pb-4">
      <h3 class="text-xs font-bold text-slate-800 uppercase tracking-wider flex items-center gap-1">
        <span>🍽️</span>
        <span>行程严选正餐据点</span>
      </h3>
      <div class="space-y-2.5">
        <div
          v-for="(rest, idx) in foodData.restaurants"
          :key="idx"
          class="bg-white rounded-2xl p-3.5 border border-slate-200/80 shadow-xs space-y-1.5"
        >
          <div class="flex items-start justify-between gap-2">
            <div>
              <h4 class="text-xs font-bold text-slate-900">{{ rest.name }}</h4>
              <van-tag type="success" size="small" plain class="mt-0.5">{{ rest.tag }}</van-tag>
            </div>
            <van-tag color="#f59e0b" text-color="#ffffff" round size="medium">
              {{ rest.budget }}
            </van-tag>
          </div>
          <div class="text-[11px] text-slate-600 bg-slate-50 p-2 rounded-xl border border-slate-100 space-y-0.5">
            <div><strong class="text-slate-700">招牌菜品:</strong> {{ rest.specialty }}</div>
            <div class="text-slate-500 truncate"><strong class="text-slate-700">地址:</strong> {{ rest.address }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { FOOD_DATA } from '../data/pingtanData.js';

const foodData = FOOD_DATA;
</script>

<style scoped>
.food-tab-container {
  height: 100%;
}
</style>
