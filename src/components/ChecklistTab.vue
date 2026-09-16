<template>
  <div class="checklist-tab-container flex-1 overflow-y-auto no-scrollbar p-3 space-y-4">
    <!-- 头部进度统计卡片 (含 Vant Progress) -->
    <div class="bg-gradient-to-r from-sky-600 to-indigo-600 text-white rounded-2xl p-4 shadow-sm space-y-3">
      <div class="flex items-center justify-between">
        <van-tag round color="rgba(255,255,255,0.25)" text-color="#ffffff">
          行前准备清单
        </van-tag>
        <span class="text-xs font-mono font-bold bg-white/15 px-2 py-0.5 rounded-md">
          已核对 {{ checkedIds.length }}/{{ totalCount }} 项
        </span>
      </div>
      <div>
        <h2 class="text-base font-bold">海岛出行装备核对清单</h2>
        <p class="text-xs text-sky-100 mt-0.5">拖鞋、防晒、数码等备齐装箱，本地自动记忆勾选状态</p>
      </div>

      <!-- Vant Progress 进度条 -->
      <div class="space-y-1 pt-1">
        <van-progress
          :percentage="progressPercentage"
          stroke-width="8px"
          color="linear-gradient(to right, #38bdf8, #34d399)"
          track-color="rgba(255, 255, 255, 0.25)"
          :show-pivot="false"
        />
        <div class="flex items-center justify-between text-[11px] text-sky-200 pt-1">
          <span>{{ checkedIds.length === totalCount ? '🎉 全项准备就绪，随时可以出发！' : '尚有项目未核对' }}</span>
          <span class="font-mono font-bold">{{ progressPercentage }}%</span>
        </div>
      </div>
    </div>

    <!-- Vant CheckboxGroup 交互式核对列表 -->
    <div class="space-y-2.5">
      <van-checkbox-group v-model="checkedIds" @change="saveState">
        <div class="space-y-2.5">
          <div
            v-for="item in checklist"
            :key="item.id"
            class="bg-white rounded-2xl p-3.5 border transition flex items-start gap-3 cursor-pointer select-none"
            :class="checkedIds.includes(item.id) ? 'border-emerald-300 bg-emerald-50/20' : 'border-slate-200/80 shadow-xs'"
            @click="toggleItem(item.id)"
          >
            <div class="pt-0.5 shrink-0" @click.stop>
              <van-checkbox
                :name="item.id"
                shape="square"
                checked-color="#059669"
              />
            </div>

            <div class="flex-1 min-w-0">
              <div class="flex items-center justify-between gap-1">
                <div class="flex items-center gap-1.5">
                  <span class="text-base">{{ item.icon }}</span>
                  <h4
                    class="text-xs font-bold transition"
                    :class="checkedIds.includes(item.id) ? 'line-through text-slate-400' : 'text-slate-900'"
                  >
                    {{ item.name }}
                  </h4>
                </div>
                <van-tag size="small" plain>{{ item.category }}</van-tag>
              </div>
              <p
                class="text-[11px] mt-1 leading-relaxed transition"
                :class="checkedIds.includes(item.id) ? 'text-slate-400' : 'text-slate-500'"
              >
                {{ item.sub }}
              </p>
            </div>
          </div>
        </div>
      </van-checkbox-group>
    </div>

    <!-- 快捷操作按钮 (Vant Buttons) -->
    <div class="flex items-center justify-between gap-2 pt-1">
      <van-button
        size="small"
        class="flex-1 font-bold"
        round
        @click="checkAll"
      >
        全部勾选
      </van-button>
      <van-button
        size="small"
        class="flex-1 font-bold"
        round
        @click="resetAll"
      >
        清空重置
      </van-button>
    </div>

    <!-- 应急安全电话卡片 (Vant NoticeBar 组合) -->
    <div class="bg-amber-50 rounded-2xl p-3.5 border border-amber-200 space-y-2 text-xs text-amber-950 shadow-xs">
      <h4 class="font-bold flex items-center gap-1 text-amber-950">
        <span>🚨</span>
        <span>海岛出行安全应急联络</span>
      </h4>
      <div class="grid grid-cols-2 gap-2 text-[11px] pt-1">
        <div class="bg-white/80 p-2 rounded-xl border border-amber-100">
          <div class="text-slate-500">平潭旅游救援求助</div>
          <div class="font-mono font-bold text-amber-800 mt-0.5">0591-24312301</div>
        </div>
        <div class="bg-white/80 p-2 rounded-xl border border-amber-100">
          <div class="text-slate-500">平潭水上搜救中心</div>
          <div class="font-mono font-bold text-amber-800 mt-0.5">12395</div>
        </div>
      </div>
      <p class="text-[10px] text-amber-800 leading-relaxed pt-1">
        ⚠️ 龙凤头与坛南湾沙滩有暗流暗礁，风大浪急时坚决不可下海野泳；黑石滩与海蚀岩潮湿青苔极滑，务必穿防滑鞋沿安全线游览。
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { CHECKLIST_ITEMS } from '../data/pingtanData.js';

const checklist = CHECKLIST_ITEMS;
const totalCount = checklist.length;
const checkedIds = ref([]);

const progressPercentage = computed(() => {
  return Math.round((checkedIds.value.length / totalCount) * 100);
});

function loadSavedState() {
  try {
    const raw = localStorage.getItem('pingtan_vant_checklist_v1');
    if (raw) {
      checkedIds.value = JSON.parse(raw);
    }
  } catch (e) {
    console.error('Failed to load checklist', e);
  }
}

function saveState() {
  try {
    localStorage.setItem('pingtan_vant_checklist_v1', JSON.stringify(checkedIds.value));
  } catch (e) {
    console.error('Failed to save checklist', e);
  }
}

function toggleItem(id) {
  const index = checkedIds.value.indexOf(id);
  if (index > -1) {
    checkedIds.value.splice(index, 1);
  } else {
    checkedIds.value.push(id);
  }
  saveState();
}

function checkAll() {
  checkedIds.value = checklist.map(it => it.id);
  saveState();
}

function resetAll() {
  checkedIds.value = [];
  saveState();
}

onMounted(() => {
  loadSavedState();
});
</script>

<style scoped>
.checklist-tab-container {
  height: 100%;
}
</style>
