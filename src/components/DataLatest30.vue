<template>
  <div class="data-list-template">
    <div class="box" style="margin-top: 0.8em">
      <div class="prompt-box" :class="{ expanded: isPromptExpanded }">
        <p style="margin-bottom: 0.2em">
          <b v-html="formatDescription(description)"></b>
        </p>
        <div class="prompt-content">
          <b>prompt：</b>
          <pre>{{ prompt || "null" }}</pre>
        </div>
        <el-button class="expand-btn" @click="togglePrompt" text size="small">
          {{ isPromptExpanded ? "收起" : "展开" }}
        </el-button>
      </div>
    </div>
    <div class="summary-box">
      <p>Showing new listings for {{ getCurrentDate() }}</p>
      <p>
        Total of {{ props.dataList.length }} entries ({{ level_tatistics?.core || "0"}} Core,
        {{ level_tatistics?.partial || "0" }} Partial, {{ level_tatistics?.["none-irrelevant"] || "0" }} Irrelevant)
      </p>
    </div>
    <!-- Data Latest 30 标签页添加分页功能 -->
    <div>
      <!-- 日期范围筛选 -->
      <!-- <selectComponent></selectComponent> -->
      <div class="filter-box">
        <div class="filter-container">
          <div>
            <span class="demonstration" style="margin-right: em"
              >Subject : &nbsp;&nbsp;</span
            >
            <el-select
              v-model="subjectValue"
              filterable
              placeholder="Select subject"
              style="width: 400px"
            >
              <el-option label="All" value="All" />
              <el-option
                v-for="item in subjects"
                :key="item"
                :label="item"
                :value="item"
              />
            </el-select>
          </div>
          <div>
            <span class="demonstration" style="margin-right: em"
              >日期范围 : &nbsp;&nbsp;</span
            >
            <el-date-picker
              v-model="dateRange"
              type="daterange"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              format="YYYY-MM-DD"
              value-format="YYYY-MM-DD"
              :disabled-date="disabledDate"
              @change="handleDateRangeChange"
              style="width: 400"
            />
            <el-button @click="clearDateFilter" style="margin-left: 0.2em"
              >清除日期筛选</el-button
            >
          </div>

          <div>
            <span class="demonstration">Relevance : &nbsp;&nbsp;</span>
            <el-select
              v-model="relevanceValue"
              filterable
              placeholder=""
              style="width: 200px"
              class="relevance-select"
            >
              <!-- <el-option label="All" value="All" /> -->
              <el-option
                v-for="item in relevanceList"
                :key="item"
                :label="item"
                :value="item"
                style="text-transform: capitalize"
              />
            </el-select>
          </div>

          <!-- <div>
          <span class="demonstration"
            >version : &nbsp;&nbsp;</span
          >
          <el-select
            v-model="versionValue"
            filterable
            placeholder=""
            style="width: 200px"
          >
            <el-option
              v-for="item in versionList"
              :key="item"
              :label="item"
              :value="item"
            />
          </el-select>
        </div> -->
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
        <span v-if="dateRange && dateRange.length === 2" class="filter-info">
          （已筛选：{{ dateRange[0] }} 至 {{ dateRange[1] }}）
        </span>

        <!-- 统一展开/收起控制 -->
        <div class="toggle-all-control" @click="toggleAllDetails">
          <el-icon :class="{ rotate: isAllExpanded }">
            <ArrowDown />
          </el-icon>
          <span>{{ isAllExpanded ? "Collapse All" : "Expand All" }}</span>
        </div>
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
import { ArrowDown } from "@element-plus/icons-vue";
import selectComponent from "@/components/selectComponent.vue";
import DataItemComponent from "@/components/DataItemComponent.vue";

const props = defineProps({
  dataList: {
    type: Array,
    default: () => [],
  },
  subjects: {
    type: Array,
    default: () => [],
  },
  type: {
    type: String,
    required: false, // 确保type是必需的
  },
  description: {
    type: String,
    required: false, // 确保type是必需的
  },
  prompt: {
    type: String,
    required: false, // 确保type是必需的
  },
  arxiv_update_date: {
    type: String,
    required: false, // 确保type是必需的
  },
  level_tatistics: {
    type: Object,
    Default: () => ({}), // 确保level_tatistics是可选的
    required: false,
  },
});

const subjectValue = ref("All"); // 主题筛选值，默认为All
const relevanceValue = ref("All"); // 相关性筛选值，默认为All
const versionValue = ref("All"); // version筛选值，默认为All

const relevanceList = ref([]);
relevanceList.value = ["All", "core", "partial", "irrelevant"];

const versionList = ref([]);
versionList.value = ["All", "v1", "v2", "v3"];

// if (props.type === "Creativity") {

//   relevanceValue.value = "All"; // 默认选择Creativity
// } else {
//   relevanceList.value = ["All"];
// }

// 分页相关状态
const currentPage = ref(1);
const pageSize = ref(100);

// 日期筛选相关状态
const dateRange = ref(null);

// 禁用今天之后的日期
const disabledDate = (time) => {
  const today = new Date();
  today.setHours(23, 59, 59, 999);
  return time.getTime() > today.getTime();
};

// 计算筛选后的数据（基于父组件传入的dataList）
const filteredData = computed(() => {
  let result = props.dataList;

  // 按subject筛选
  if (subjectValue.value && subjectValue.value !== "All") {
    result = result.filter((item) => {
      return item.subjects && item.subjects.includes(subjectValue.value);
    });
  }

  // 按relevance.level筛选
  if (relevanceValue.value && relevanceValue.value !== "All") {
    result = result.filter((item) => {
      return item.relevance && 
        item.relevance.level && 
        item.relevance.level.toLowerCase().includes(relevanceValue.value.toLowerCase());
    });
  }

  // 按version筛选（通过submission_historys中最后一项的version）
  if (versionValue.value && versionValue.value !== "All") {
    result = result.filter((item) => {
      if (
        !item.submission_historys ||
        !Array.isArray(item.submission_historys) ||
        item.submission_historys.length === 0
      ) {
        return false;
      }
      const lastSubmission =
        item.submission_historys[item.submission_historys.length - 1];
      return lastSubmission && lastSubmission.version === versionValue.value;
    });
  }

  // 按日期范围筛选
  if (dateRange.value && dateRange.value.length === 2) {
    const [startDate, endDate] = dateRange.value;
    result = result.filter((item) => {
      if (!item.last_revised_date) return false;

      // 将日期格式从 2025/06/20 转换为 2025-06-20 以便比较
      const itemDate = item.last_revised_date.replace(/\//g, "-");
      return itemDate >= startDate && itemDate <= endDate;
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
    subjectValue.value = "All";
    dateRange.value = null;
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

// 日期范围变化处理
const handleDateRangeChange = () => {
  currentPage.value = 1; // 重置到第一页
};

// 清除日期筛选
const clearDateFilter = () => {
  dateRange.value = null;
  currentPage.value = 1;
};

// 清除所有筛选
const clearAllFilters = () => {
  dateRange.value = null;
  subjectValue.value = "All";
  relevanceValue.value = "All";
  versionValue.value = "All";
  currentPage.value = 1;
};

// Subject筛选变化处理
watch(subjectValue, () => {
  currentPage.value = 1; // 重置到第一页
});

// Relevance筛选变化处理
watch(relevanceValue, () => {
  currentPage.value = 1; // 重置到第一页
});

// Version筛选变化处理
watch(versionValue, () => {
  currentPage.value = 1; // 重置到第一页
});

const isPromptExpanded = ref(false);

const togglePrompt = () => {
  isPromptExpanded.value = !isPromptExpanded.value;
};

// 格式化描述文本，将URL转换为链接
const formatDescription = (text) => {
  if (!text) return "";

  // URL 正则表达式
  const urlRegex = /(https?:\/\/[^\s]+)/g;

  return text.replace(urlRegex, '<a href="$1" target="_blank">$1</a>');
};

// 计算各种相关性级别的数量（使用原始数据）
const coreCount = computed(() => {
  return props.dataList.filter(
    (item) => item.relevance && 
      item.relevance.level && 
      item.relevance.level.toLowerCase().includes("core")
  ).length;
});

const partialCount = computed(() => {
  return props.dataList.filter(
    (item) => item.relevance && 
      item.relevance.level && 
      item.relevance.level.toLowerCase().includes("partial")
  ).length;
});

const irrelevantCount = computed(() => {
  return props.dataList.filter(
    (item) => item.relevance && 
      item.relevance.level && 
      item.relevance.level.toLowerCase().includes("irrelevant")
  ).length;
});

// 获取当前日期的前一天
const getCurrentDate = () => {
  const today = new Date(props.arxiv_update_date);
  const yesterday = new Date(today);
  yesterday.setDate(today.getDate());

  return yesterday.toLocaleDateString("en-US", {
    weekday: "long",
    year: "numeric",
    month: "long",
    day: "numeric",
  });
};

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
:deep(.relevance-select){
  .el-select__selected-item {
    text-transform: capitalize;
  }
}
</style>
