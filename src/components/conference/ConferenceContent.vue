<template>
  <div class="data-list-template">
    <!-- Data Latest 30 标签页添加分页功能 -->
    <div>
      <!-- 日期范围筛选 -->
      <!-- <selectComponent></selectComponent> -->
      <div class="filter-box">
        <div class="filter-container">
          <div>
            <span class="demonstration" style="margin-right: em"
              >years : &nbsp;&nbsp;</span
            >
            <el-select
              v-model="yearValue"
              filterable
              placeholder="Select Years"
              style="width: 400px"
            >
              <el-option label="All" value="All" />
              <el-option
                v-for="item in years"
                :key="item"
                :label="item"
                :value="item"
              />
            </el-select>
          </div>
        </div>

        <div class="button-box">
          <el-button @click="clearAllFilters">清除所有筛选</el-button>
          <!-- <el-button type="primary">保存此次筛选</el-button> -->
        </div>
      </div>

      <!-- 分页信息显示 -->
      <div class="pagination-info">
        共 {{ filteredData.length }} 条数据，当前显示第
        {{ (currentPage - 1) * pageSize + 1 }} -
        {{ Math.min(currentPage * pageSize, filteredData.length) }} 条

        <!-- 统一展开/收起控制 -->
        <!-- <div class="toggle-all-control" @click="toggleAllDetails">
          <el-icon :class="{ rotate: isAllExpanded }">
            <ArrowDown />
          </el-icon>
          <span>{{ isAllExpanded ? "Collapse All" : "Expand All" }}</span>
        </div> -->
      </div>

      <ul class="result-list" style="margin-top: 1em">
        <li
          key=""
          v-for="(item, index) in paginatedData"
          :key="index"
          class="result-item"
        >
          <DataItemComponent
            :dataItem="item"
            :num="(currentPage - 1) * pageSize + index + 1"
            :ref="(el) => setItemRef(el, index)"
          ></DataItemComponent>
        </li>
      </ul>

      <!-- 分页组件 -->
      <div class="pagination-container">
        <el-pagination
          v-model:current-page="currentPage"
          v-model:page-size="pageSize"
          :page-sizes="[100, 200, 500, 1000]"
          :small="false"
          :disabled="false"
          :background="true"
          layout="total, sizes, prev, pager, next, jumper"
          :total="filteredData.length"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, defineProps, nextTick } from "vue";
import DataItemComponent from "@/components/conference/ConferenceItem.vue";

const props = defineProps({
  dataList: {
    type: Array,
    default: () => [],
  },
  years: {
    type: Array,
    default: () => [],
  },
});

const yearValue = ref("All"); // 主题筛选值，默认为All

// 分页相关状态
const currentPage = ref(1);
const pageSize = ref(100);

// 计算筛选后的数据（基于父组件传入的dataList）
const filteredData = computed(() => {
  let result = props.dataList;

  // 按subject筛选
  if (yearValue.value && yearValue.value !== "All") {
    result = result.filter((item) => {
      return item.year && item.year == yearValue.value;
    });
  }

  return result;
});

// 计算分页后的数据（基于筛选后的数据）
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return filteredData.value.slice(start, end);
});

// 监听dataList变化，重置分页和筛选
watch(
  () => props.dataList,
  () => {
    currentPage.value = 1;
    yearValue.value = "All";
  },
  { immediate: true }
);

// 滚动到 #content 顶部
const scrollToTop = () => {
  setTimeout(() => {
    const contentElement = document.getElementById("content");
    if (contentElement) {
      contentElement.scrollTo({
        top: 0,
        // behavior: 'smooth'
      });
    } else {
      // 如果找不到 #content，则滚动整个页面
      window.scrollTo({
        top: 0,
        // behavior: 'smooth'
      });
    }
  }, 100);
};

// 分页事件处理
const handleSizeChange = (newSize) => {
  pageSize.value = newSize;
  currentPage.value = 1; // 重置到第一页
  scrollToTop();
};

const handleCurrentChange = (newPage) => {
  currentPage.value = newPage;
  scrollToTop();
};

// 清除所有筛选
const clearAllFilters = () => {
  dateRange.value = null;
  yearValue.value = "All";
  relevanceValue.value = "All";
  versionValue.value = "All";
  currentPage.value = 1;
};

const isPromptExpanded = ref(false);

const isAllExpanded = ref(false);
const itemRefs = ref([]);

// 设置子组件引用
const setItemRef = (el, index) => {
  if (el) {
    itemRefs.value[index] = el;
  }
};

// 统一展开/收起所有详情
const toggleAllDetails = () => {
  isAllExpanded.value = !isAllExpanded.value;

  nextTick(() => {
    itemRefs.value.forEach((item) => {
      if (item && item.setExpandedState) {
        item.setExpandedState(isAllExpanded.value);
      }
    });
  });
};

// 监听分页数据变化，重置引用数组
watch(paginatedData, () => {
  itemRefs.value = [];
  isAllExpanded.value = false;
});
</script>

<style scoped lang="scss">
.data-list-template {
  .summary-box {
    margin: 2em 0 1em;
    font-size: 14px;
    & > p:nth-child(1) {
      font-weight: bold;
      font-size: 17px;
      margin-bottom: 0.5em;
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
    margin-bottom: 0em;
    display: flex;
    align-items: center;
    justify-content: space-between;

    .toggle-all-control {
      display: flex;
      align-items: center;
      gap: 0.5em;
      cursor: pointer;
      color: #409eff;
      font-size: 14px;

      &:hover {
        color: #66b1ff;
      }

      .el-icon {
        &.rotate {
          transform: rotate(180deg);
        }
      }
    }
  }

  .pagination-container {
    display: flex;
    justify-content: center;
    margin-top: 2em;
    padding: 1em 0;
  }
  .filter-box {
    margin-bottom: 1em;
    padding: 1em 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1em;
    .button-box {
      align-self: flex-end;
    }
    .filter-container {
      display: flex;
      align-items: center;
      padding: 1em 0 0;
      gap: 1em 2em;
      flex-wrap: wrap;
      // background: #f8f9fa;
      // border-radius: 8px;
      // border: 1px solid #e9ecef;
      .demonstration {
        font-weight: bold;
      }
    }
  }

  .filter-info {
    color: #28a745;
    font-weight: 500;
    margin-left: 0.5em;
  }
}

.box {
  border: 1px solid #e4e7ed;
  border-radius: 4px;
  background: #f9f9f9;
  padding: 0.5em 1em;
}
.prompt-box {
  position: relative;
  height: 4.4em;
  min-height: 4.4em;
  overflow: hidden;
  transition: height 0.3s ease;

  &.expanded {
    height: auto;
  }

  .prompt-content {
    display: flex;
    flex-direction: row;
    gap: 1em;
    font-size: 1em;

    pre {
      margin: 0;
      font-size: 0.9em;
      line-height: 1.4;
      white-space: pre-wrap;
      flex: 1;
    }
  }

  .expand-btn {
    position: absolute;
    right: 8px;
    top: 8px;
    z-index: 1;
    background: rgba(255, 255, 255, 0.9);
    border: 1px solid #ddd;

    &:hover {
      background: rgba(255, 255, 255, 1);
    }
  }
}
:deep(.relevance-select) {
  .el-select__selected-item {
    text-transform: capitalize;
  }
}
</style>
