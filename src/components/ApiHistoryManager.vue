<template>
  <el-card class="api-history-manager">
    <template #header>
      <div class="card-header">
        <span>API历史管理</span>
      </div>
    </template>
    <el-table :data="history" style="width: 100%">
      <el-table-column prop="apiKey" label="API密钥" width="320" />
      <el-table-column prop="apiBaseUrl" label="API地址" />
      <el-table-column label="操作" width="180">
        <template #default="scope">
          <el-button size="small" type="primary" @click="insertHistory(scope.row)">插入</el-button>
          <el-button size="small" type="danger" @click="deleteHistory(scope.$index)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </el-card>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'

const history = ref([])
const emit = defineEmits(['selectHistory'])

// 读取历史
const loadHistory = () => {
  try {
    // 优先localStorage，其次json文件
    const local = localStorage.getItem('apiHistory')
    if (local) {
      history.value = JSON.parse(local)
    } else {
      fetch('/src/config/api_history.json')
        .then(res => res.json())
        .then(data => { history.value = data })
    }
  } catch (e) {
    history.value = []
  }
}

// 插入（自动去重）
const insertHistory = (row) => {
  let apiHistory = JSON.parse(localStorage.getItem('apiHistory') || '[]')
  if (apiHistory.some(item => item.apiKey === row.apiKey)) {
    ElMessage.info('该密钥已存在，自动跳过')
    emit('selectHistory', row)
    return
  }
  apiHistory.push(row)
  localStorage.setItem('apiHistory', JSON.stringify(apiHistory))
  ElMessage.success('插入成功')
  loadHistory()
  emit('selectHistory', row)
}

// 删除（不去重）
const deleteHistory = (idx) => {
  let apiHistory = JSON.parse(localStorage.getItem('apiHistory') || '[]')
  apiHistory.splice(idx, 1)
  localStorage.setItem('apiHistory', JSON.stringify(apiHistory))
  ElMessage.success('删除成功')
  loadHistory()
}

onMounted(() => {
  loadHistory()
})
</script>

<style scoped>
.api-history-manager {
  max-width: 800px;
  margin: 0 auto;
}
.card-header {
  display: flex;
  align-items: center;
  font-size: 18px;
  font-weight: bold;
}
</style> 