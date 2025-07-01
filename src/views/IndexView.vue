<template>
  <div class="home-template main-container">
    <div style="margin: 0.3em 0;display: flex;justify-content: space-between;align-items:end">
      <h2>Daily Paper</h2>
      <!-- <div>The data is expected to be updated daily between 9:00 AM and 10:00 AM.</div> -->
    </div>
    <div class="button-container" style="margin-top: 2em">
      <el-button
        :type="currentIndex == 0 ? 'primary' : ''"
        :class="currentIndex == 0 ? 'on' : ''"
        @click="currentIndex = 0"
        >Creativity</el-button
      >
      <el-button
        :type="currentIndex == 1 ? 'primary' : ''"
        :class="currentIndex == 1 ? 'on' : ''"
        @click="currentIndex = 1"
        >LLM Training Data</el-button
      >
      <!-- <el-button
        :type="currentIndex == 2 ? 'primary' : ''"
        :class="currentIndex == 2 ? 'on' : ''"
        @click="currentIndex = 2"
        >Data Latest 30</el-button
      > -->
    </div>

    <div v-show="currentIndex == 0">
      <DataLatest30
        :dataList="HCIResultList?.data || []"
        :subjects="HCIResultList?.subjects || []"
        :description="HCIResultList?.description || ''"
        :prompt="HCIResultList?.prompt?.creativity || ''"
        type="Creativity"
        :arxiv_update_date="HCIResultList?.arxiv_update_date || ''"
        :level_tatistics="HCIResultList?.level_tatistics || {}"
        :updated_at="HCIResultList?.updated_at || ''"
      />
    </div>

    <div v-show="currentIndex == 1">
      <DataLatest30
        :dataList="DataResultList?.data || []"
        :subjects="DataResultList?.subjects || []"
        :description="DataResultList?.description || ''"
        :prompt="DataResultList?.prompt?.train_data || ''"
        :arxiv_update_date="DataResultList?.arxiv_update_date || ''"
        :level_tatistics="DataResultList?.level_tatistics || {}"
        :updated_at="DataResultList?.updated_at || ''"
      />
    </div>

    <!-- Data Latest 30 标签页添加分页功能 -->
    <!-- <div v-show="currentIndex == 2">
      <DataLatest30
        :dataList="data_latest_30_result?.data || []"
        :subjects="data_latest_30_result?.subjects || []"
      ></DataLatest30>
    </div> -->
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";
import DataLatest30 from "@/components/DataLatest30.vue";
const currentIndex = ref(0);

// 创建响应式变量
const HCIResultList = ref([]);
const DataResultList = ref([]);
const data_latest_30_result = ref([]);

// 生命周期钩子：组件挂载时请求 JSON 数据
onMounted(async () => {
  try {
    const response = await axios.get(
      "./data/creativity.json?v=" + new Date().getTime()
    );
    
    // 过滤数据，只保留 relevance 为 'creativity' 的数据
    // const originalData = response.data;
    // const filteredData = {
    //   ...originalData,
    //   data: originalData.data?.filter(item => 
    //     item.relevance && item.relevance.keyword  === 'creativity'
    //   ) || []
    // };
    HCIResultList.value = response.data;

    const response2 = await axios.get(
      "./data/llm_training_data.json?v=" + new Date().getTime()
    );
    DataResultList.value = response2.data;

    const response3 = await axios.get(
      "./data/data_latest_30.json?v=" + new Date().getTime()
    );
    data_latest_30_result.value = response3.data;
  } catch (error) {
    console.error("获取 JSON 数据失败:", error);
  }
});
</script>




<style scoped lang="scss">
.home-template {
  padding: 1em 0;
  margin: 0 auto;
  h2 {
    font-size: 2.5em;
  }
  .button-container {
    .el-button {
      font-size: 1.2em;
      padding: 0.5em 1.5em;
      min-width: 7em;
      height: 2.5em;
    }
  }

  .result-item {
    padding: 1em;
    border-bottom: 1px solid #ccc;

    display: flex;
    flex-direction: row;
    gap: 2em;
    p {
      margin: 0.2em 0;
    }
    & > div:nth-child(1) {
      min-width: 3em;
      span {
        font-size: 2em;
      }
    }
    & > div:nth-child(2) {
      flex: 1;
      padding: 0.5em 0;
    }
  }

  .pagination-info {
    color: #666;
    font-size: 14px;
    margin-bottom: 1em;
  }

  .pagination-container {
    display: flex;
    justify-content: center;
    margin-top: 2em;
    padding: 1em 0;
  }

  .filter-container {
    display: flex;
    align-items: center;
    margin-bottom: 1em;
    padding: 1em;
    background: #f8f9fa;
    border-radius: 8px;
    border: 1px solid #e9ecef;
  }

  .filter-info {
    color: #28a745;
    font-weight: 500;
    margin-left: 0.5em;
  }
}

</style>
