<template>
  <div class="home-template main-container">
    <div style="margin: 0.3em 0;display: flex;justify-content: space-between;align-items:end">
      <h2>Conference</h2>
      <!-- <div>The data is expected to be updated daily between 9:00 AM and 10:00 AM.</div> -->
    </div>
    <div class="button-container" style="margin-top: 2em">
      <el-button
        :type="currentIndex == 0 ? 'primary' : ''"
        :class="currentIndex == 0 ? 'on' : ''"
        @click="currentIndex = 0"
        >Creativity</el-button
      >
    </div>

    <div v-show="currentIndex == 0">
      <DataLatest
        :dataList="CreativityMetting?.data || []"
        :years="CreativityMetting?.years || []"

      />
    </div>

    
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";
import DataLatest from "@/components/conference/ConferenceContent.vue";
const currentIndex = ref(0);

// 创建响应式变量
const CreativityMetting = ref([]);

// 生命周期钩子：组件挂载时请求 JSON 数据
onMounted(async () => {
  try {
    const response = await axios.get(
      "./data/creativity_metting.json?v=" + new Date().getTime()
    );

    CreativityMetting.value = response.data;

    
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
