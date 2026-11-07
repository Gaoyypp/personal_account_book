<script setup>
/**
 * 【占位骨架】本文件属于 9 号（记录展示），由 7 号在骨架期提供最小可运行外壳。
 * 交付前必须由 9 号替换：正式加载/空态/失败提示与最近记录排版。
 * 契约要求保持不变：props 只有 refreshKey；挂载及 refreshKey 改变时请求 limit=10。
 */
import { ref, watch } from 'vue'
import { listRecords, isApiError } from '../../api/client.js'
import RecordTable from './RecordTable.vue'

const props = defineProps({
  refreshKey: { type: Number, default: 0 }
})

const items = ref([])
const total = ref(0)
const loading = ref(false)
const errorMessage = ref('')

let requestSeq = 0

async function load() {
  const seq = ++requestSeq
  loading.value = true
  errorMessage.value = ''
  try {
    const page = await listRecords({ limit: 10 })
    if (seq !== requestSeq) return // 只认最后一次请求的结果
    items.value = page.items
    total.value = page.total
  } catch (error) {
    if (seq !== requestSeq) return
    items.value = []
    total.value = 0
    errorMessage.value = isApiError(error) ? error.message : '最近记录加载失败'
  } finally {
    if (seq === requestSeq) loading.value = false
  }
}

// 首次挂载与每次 refreshKey 变化都重新请求：App 递增标记即触发
watch(() => props.refreshKey, load, { immediate: true })
</script>

<template>
  <div class="card">
    <div class="recent-head">
      <h2>最近记录</h2>
      <span v-if="!loading && !errorMessage" class="hint">共 {{ total }} 条</span>
    </div>

    <p v-if="loading" class="state-block state-loading">正在加载最近记录…</p>

    <div v-else-if="errorMessage" class="state-block state-error" role="alert">
      <p>{{ errorMessage }}</p>
      <button type="button" class="btn" @click="load">重新加载</button>
    </div>

    <p v-else-if="items.length === 0" class="state-block">还没有记录，先在左侧保存一笔。</p>

    <RecordTable v-else :records="items" />
  </div>
</template>

<style scoped>
.recent-head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: var(--space-3);
  margin-bottom: var(--space-4);
}
</style>
