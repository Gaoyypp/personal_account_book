<script setup>
/**
 * 【占位骨架】本文件属于 9 号（记录展示），由 7 号在骨架期提供最小可运行外壳。
 * 交付前必须由 9 号替换：正式列结构、收支标记样式、加载/空态/备注安全展示。
 * 契约要求保持不变：只接收 records，不发请求、不修改父组件数据；备注只用文本插值。
 */
import { formatSignedMoney } from '../../utils/format.js'

defineProps({
  records: { type: Array, default: () => [] }
})
</script>

<template>
  <table class="record-table">
    <caption class="sr-only">收支记录</caption>
    <thead>
      <tr>
        <th scope="col">日期</th>
        <th scope="col">类型</th>
        <th scope="col">类别</th>
        <th scope="col" class="col-amount">金额（元）</th>
        <th scope="col">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="record in records" :key="record.id">
        <td>{{ record.date }}</td>
        <td>{{ record.type === 'income' ? '收入' : '支出' }}</td>
        <td>{{ record.category }}</td>
        <td class="col-amount amount" :class="record.type === 'income' ? 'amount-income' : 'amount-expense'">
          {{ formatSignedMoney(record.type, record.amount_cents) }}
        </td>
        <!-- 备注按纯文本展示，禁止 v-html -->
        <td class="col-note">{{ record.note }}</td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped>
.record-table {
  width: 100%;
  border-collapse: collapse;
  font-size: var(--font-size-sm);
}

th,
td {
  padding: var(--space-2) var(--space-3);
  border-bottom: 1px solid var(--color-border);
  text-align: left;
  vertical-align: top;
}

th {
  color: var(--color-muted);
  font-weight: 500;
  white-space: nowrap;
}

.col-amount {
  text-align: right;
  white-space: nowrap;
}

.col-note {
  color: var(--color-muted);
  word-break: break-word;
}
</style>
