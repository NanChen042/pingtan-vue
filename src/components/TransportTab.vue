<template>
  <div class="transport-tab-container flex-1 overflow-y-auto no-scrollbar p-3 space-y-3">
    <!-- 1. 核心出行策略概览卡 (移动端精致头部) -->
    <div class="bg-gradient-to-br from-sky-600 to-indigo-700 text-white rounded-2xl p-4 shadow-sm space-y-3">
      <div class="flex items-center justify-between">
        <span class="text-xs font-bold px-2.5 py-0.5 rounded-full bg-white/20 backdrop-blur text-white flex items-center gap-1">
          <span>🚗</span>
          <span>全岛出行核心策略</span>
        </span>
        <span class="text-xs font-mono font-bold text-sky-200">4天人均交通约 156 元</span>
      </div>

      <div class="grid grid-cols-3 gap-2 text-center pt-0.5">
        <div class="bg-white/10 backdrop-blur rounded-xl p-2 border border-white/15">
          <div class="text-[10px] text-sky-200">主力出行</div>
          <div class="text-xs font-bold text-white mt-0.5">普通网约快车</div>
        </div>
        <div class="bg-white/10 backdrop-blur rounded-xl p-2 border border-white/15">
          <div class="text-[10px] text-sky-200">最佳组队</div>
          <div class="text-xs font-bold text-white mt-0.5">3人/车 拼座</div>
        </div>
        <div class="bg-white/10 backdrop-blur rounded-xl p-2 border border-white/15">
          <div class="text-[10px] text-sky-200">局部体验</div>
          <div class="text-xs font-bold text-white mt-0.5">北港租小电驴</div>
        </div>
      </div>

      <div class="text-xs text-sky-100 leading-relaxed bg-black/15 rounded-xl p-2.5 border border-white/10">
        💡 <strong>组队核心诀窍：</strong>平潭岛内无合规商务车。多人出行直接分拆为 <strong>3人/车</strong> 在APP同时呼叫普通5座快车，秒接单且机动性最高！
      </div>
    </div>

    <!-- 2. 四种出行方式实测对比卡片 (彻底杜绝星标折行，配色清爽雅致) -->
    <div class="space-y-3">
      <div
        v-for="(item, idx) in transportList"
        :key="idx"
        class="bg-white rounded-2xl p-3.5 border border-slate-200/90 shadow-xs space-y-2.5 transition"
      >
        <!-- 卡片头部：图标、名称与推荐标签 -->
        <div class="flex items-center justify-between gap-2">
          <div class="flex items-center gap-2 min-w-0">
            <span class="w-8 h-8 rounded-xl bg-slate-100 flex items-center justify-center text-lg shrink-0">
              {{ getModeIcon(item.mode) }}
            </span>
            <div class="min-w-0">
              <h3 class="text-xs sm:text-sm font-bold text-slate-900 truncate">{{ item.mode }}</h3>
              <!-- 星级评分：严格单行排列，永不换行折叠 -->
              <div class="flex items-center gap-1.5 mt-0.5 shrink-0">
                <div class="flex items-center text-amber-500 text-xs shrink-0 tracking-tighter">
                  <span v-for="star in 5" :key="star">
                    {{ star <= item.rating ? '★' : '☆' }}
                  </span>
                </div>
                <span class="text-[11px] font-mono font-bold text-amber-700">{{ item.rating }}.0分</span>
              </div>
            </div>
          </div>

          <van-tag :type="getTagType(item.rating)" round size="medium" class="shrink-0 font-bold">
            {{ item.tag.split(' ')[0] }}
          </van-tag>
        </div>

        <!-- 总结说明 -->
        <div class="text-xs text-slate-600 bg-slate-50 p-2 rounded-xl border border-slate-100 leading-relaxed">
          {{ item.summary }}
        </div>

        <!-- 优势、劣势、实战指南清晰分类展示 -->
        <div class="space-y-1.5 text-xs pt-0.5">
          <div class="bg-emerald-50/75 border border-emerald-100/90 p-2.5 rounded-xl flex items-start gap-2 text-emerald-950">
            <span class="px-1.5 py-0.5 rounded bg-emerald-500 text-white font-bold text-[10px] shrink-0 mt-0.5">优势</span>
            <span class="leading-relaxed text-[11px] flex-1">{{ item.pros }}</span>
          </div>

          <div class="bg-rose-50/75 border border-rose-100/90 p-2.5 rounded-xl flex items-start gap-2 text-rose-950">
            <span class="px-1.5 py-0.5 rounded bg-rose-500 text-white font-bold text-[10px] shrink-0 mt-0.5">劣势</span>
            <span class="leading-relaxed text-[11px] flex-1">{{ item.cons }}</span>
          </div>

          <div class="bg-sky-50/75 border border-sky-100/90 p-2.5 rounded-xl flex items-start gap-2 text-sky-950">
            <span class="px-1.5 py-0.5 rounded bg-sky-600 text-white font-bold text-[10px] shrink-0 mt-0.5">指南</span>
            <span class="leading-relaxed text-[11px] flex-1">{{ item.advice }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 3. 平潭约车与避坑守则 -->
    <div class="bg-white rounded-2xl p-3.5 border border-slate-200/90 shadow-xs space-y-2.5 mb-4">
      <div class="flex items-center justify-between pb-1.5 border-b border-slate-100">
        <h3 class="text-xs font-bold text-slate-800 flex items-center gap-1.5">
          <span>⚠️</span>
          <span>平潭岛内出行避坑重要守则</span>
        </h3>
        <span class="text-[10px] text-slate-400">行前必读</span>
      </div>

      <div class="space-y-2 text-xs">
        <div class="p-2.5 rounded-xl bg-amber-50/70 border border-amber-200/80 flex items-start gap-2 text-amber-950">
          <span class="text-base shrink-0">🌅</span>
          <div>
            <div class="font-bold text-amber-900 text-[11px]">长江澳日落集中撤离</div>
            <p class="text-[11px] text-amber-800 leading-relaxed mt-0.5">18:30 退潮日落需提前 15-20 分钟在滴滴或高德呼叫网约快车，避免落日后数百人同时排队滞留。</p>
          </div>
        </div>

        <div class="p-2.5 rounded-xl bg-rose-50/70 border border-rose-200/80 flex items-start gap-2 text-rose-950">
          <span class="text-base shrink-0">🛑</span>
          <div>
            <div class="font-bold text-rose-900 text-[11px]">坚决拒乘非法黑车</div>
            <p class="text-[11px] text-rose-800 leading-relaxed mt-0.5">高铁站与各大景区出口无资质拉客黑车漫天要价且无合规营运保险，必须通过正规软件叫车。</p>
          </div>
        </div>

        <div class="p-2.5 rounded-xl bg-indigo-50/70 border border-indigo-200/80 flex items-start gap-2 text-indigo-950">
          <span class="text-base shrink-0">⚡</span>
          <div>
            <div class="font-bold text-indigo-900 text-[11px]">小电驴还车时限与防断电</div>
            <p class="text-[11px] text-indigo-800 leading-relaxed mt-0.5">北港短租小电驴务必在 15:00 前骑回北港村原店归还，严禁骑出环岛风情段，防止半路山道断电推车。</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { TRANSPORT_EVALUATION } from '../data/pingtanData.js';

const transportList = TRANSPORT_EVALUATION;

function getModeIcon(mode) {
  if (mode.includes('网约车')) return '🚗';
  if (mode.includes('短租')) return '🛵';
  if (mode.includes('环岛')) return '⚠️';
  if (mode.includes('公交')) return '🚌';
  return '🚲';
}

function getTagType(rating) {
  if (rating >= 4) return 'success';
  if (rating === 1) return 'danger';
  if (rating === 2) return 'warning';
  return 'primary';
}
</script>

<style scoped>
.transport-tab-container {
  height: 100%;
}
</style>
