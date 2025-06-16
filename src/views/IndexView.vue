<template>
  <div class="home-template main-container">
    <h2>Daily Paper</h2>
    <div style="margin-top: 2em;">
      <el-button type="primary">HCI</el-button>
      <el-button type="primary">Data</el-button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// 创建响应式变量
const caseList = ref([])

// 生命周期钩子：组件挂载时请求 JSON 数据
onMounted(async () => {
  try {
    const response = await axios.get('/data/cs_HC.json')
    caseList.value = response.data
  } catch (error) {
    console.error('获取 JSON 数据失败:', error)
  }
})
</script>




<style scoped lang="scss">
.home-template {
  padding: 1em 0;
  margin: 0 auto;
  color: #070707;
  h2 {
    font-size: 2.5em;
    margin: 0.3em 0;
  }
  .el-button {
    
    font-size: 1.2em;
    padding: 0.5em 1.5em;
    width: 7em;
    height: 2.5em;
  }
}
</style>
