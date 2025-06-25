<template>
  <div class="">
    <div class="box" style="margin-top: .8em;">
      <div
        class="prompt-box"
        :class="{ expanded: isPromptExpanded }"
      >
        <p style="margin-bottom: 0.2em">
          <b v-html="formatDescription(description)"></b>
        </p>
        <div class="prompt-content">
          <b>prompt：</b>
          <pre>{{ prompt }}</pre>
        </div>
        <el-button class="expand-btn" @click="togglePrompt" text size="small">
          {{ isPromptExpanded ? "收起" : "展开" }}
        </el-button>
      </div>
    </div>
    <!-- Data Latest 30 标签页添加分页功能 -->
    <div>
      <!-- 日期范围筛选 -->
      <!-- <selectComponent></selectComponent> -->
      <div class="filter-container" style="margin-top: 2em">
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
          <el-button @click="clearDateFilter" style="margin-left: 1em"
            >清除日期筛选</el-button
          >
        </div>

        <div>
          <span class="demonstration" style="margin-right: em"
            >相关性 : &nbsp;&nbsp;</span
          >
          <el-select
            v-model="relevanceValue"
            filterable
            placeholder=""
            style="width: 400px"
          >
            <!-- <el-option label="All" value="All" /> -->
            <el-option
              v-for="item in relevanceList"
              :key="item"
              :label="item"
              :value="item"
            />
          </el-select>
        </div>
      </div>
      <div style="display: flex; justify-content: end">
        <el-button @click="clearAllFilters">清除所有筛选</el-button>
        <!-- <el-button type="primary">保存此次筛选</el-button> -->
      </div>

      <!-- 分页信息显示 -->
      <div class="pagination-info">
        共 {{ filteredData.length }} 条数据，当前显示第
        {{ (currentPage - 1) * pageSize + 1 }} -
        {{ Math.min(currentPage * pageSize, filteredData.length) }} 条
        <span v-if="dateRange && dateRange.length === 2" class="filter-info">
          （已筛选：{{ dateRange[0] }} 至 {{ dateRange[1] }}）
        </span>
      </div>

      <ul class="result-list" style="margin-top: 1em">
        <li
          key=""
          v-for="(item, index) in paginatedData"
          :key="index"
          class="result-item"
        >
          <div>
            <span>{{ (currentPage - 1) * pageSize + index + 1 }}</span>
          </div>
          <div>
            <p v-if="item.relevance"><b>relevance: </b>{{ item.relevance }}</p>
            <p v-if="item.reason"><b>reason: </b> {{ item.reason }}</p>
            <p v-if="item.source"><b>source: </b>{{ item.source }}</p>
            <p v-if="item.type"><b>type: </b> {{ item.type }}</p>

            <h3>title: {{ item.title }}</h3>
            <p>
              <b>authors: </b>
              <span v-for="(author, index) in item.authors" :key="index">
                {{ author }}
                <span v-if="index < item.authors.length - 1"
                  >, &nbsp;&nbsp;</span
                >
              </span>
            </p>
            <p v-if="item.comments"><b>comments: </b>{{ item.comments }}</p>
            <p>
              <b>last_revised_date:</b>
              {{ item.last_revised_date ? item.last_revised_date : "null" }}
            </p>
            <p>
              <b>subjects : </b>
              <span v-for="(subject, index) in item.subjects" :key="index">
                {{ subject
                }}<span v-if="index < item.subjects.length - 1"
                  >, &nbsp;&nbsp;</span
                >
              </span>
            </p>

            <p>
              <b>links: </b>
              <span v-for="(value, key, index) in item.links" :key="index">
                {{ key }}: <a :href="value" target="_blank">{{ value }}</a
                ><span v-if="index < Object.entries(item.links).length - 1"
                  >, &nbsp;&nbsp;
                </span>
              </span>
            </p>
            <p v-if="item.repo_urls">
              <b>repo_urls: </b>
              <span v-for="(value, index) in item.repo_urls" :key="index">
                <a :href="value" target="_blank">{{ value }}</a
                ><span v-if="index < item.repo_urls.length - 1"
                  >, &nbsp;&nbsp;
                </span>
              </span>
            </p>
            <p v-if="item.conference">
              <b>conference: </b>{{ item.conference }}
            </p>
            <p v-if="item.conference_url_abs">
              <b>conference_url_abs:</b
              ><a :href="item.conference_url_abs" target="_blank">{{
                item.conference_url_abs
              }}</a>
            </p>
            <p v-if="item.tasks">
              <b>tasks: </b>
              <span v-for="(value, index) in item.tasks" :key="index">
                {{ value
                }}<span v-if="index < item.tasks.length - 1"
                  >, &nbsp;&nbsp;
                </span>
              </span>
            </p>

            <div class="flex-item" v-if="item.datasets">
              <b>datasets:</b>
              <div>
                <p v-for="(dataset, index) in item.datasets" :key="index">
                  <span v-for="(value, key, index) in dataset" :key="index">
                    {{ key }}:
                    <template v-if="key == 'link'">
                      <a :href="value" target="_blank">{{ value }}</a>
                    </template>
                    <template v-else>
                      {{ value }}
                    </template>
                    <span v-if="index < Object.entries(dataset).length - 1"
                      >, &nbsp;&nbsp;
                    </span>
                  </span>
                </p>
              </div>
            </div>
            <div class="flex-item" v-if="item.models">
              <b>models:</b>
              <div>
                <p v-for="(dataset, index) in item.models" :key="index">
                  <span v-for="(value, key, index) in dataset" :key="index">
                    {{ key }}:
                    <template v-if="key == 'link'">
                      <a :href="value" target="_blank">{{ value }}</a>
                    </template>
                    <template v-else>
                      {{ value }}
                    </template>
                    <span v-if="index < Object.entries(dataset).length - 1"
                      >, &nbsp;&nbsp;
                    </span>
                  </span>
                </p>
              </div>
            </div>
            <p><b>abstract</b>: {{ item.abstract }}</p>
          </div>
        </li>
      </ul>

      <!-- 分页组件 -->
      <div class="pagination-container">
        <el-pagination
          v-model:current-page="currentPage"
          v-model:page-size="pageSize"
          :page-sizes="[10, 20, 50, 100]"
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
import selectComponent from "@/components/selectComponent.vue";

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
});

const subjectValue = ref("All"); // 主题筛选值，默认为All
const relevanceValue = ref("All"); // 相关性筛选值，默认为All

const relevanceList = ref([]);
if (props.type === "Creativity") {
  relevanceList.value = ["Creativity"];
  relevanceValue.value = "Creativity"; // 默认选择Creativity
} else {
  relevanceList.value = ["All"];
}

// 分页相关状态
const currentPage = ref(1);
const pageSize = ref(10);

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
  currentPage.value = 1;
};

// Subject筛选变化处理
watch(subjectValue, () => {
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
    margin-bottom: 0em;
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
    padding: 1em 0;
    gap: 1em 2em;
    flex-wrap: wrap;
    // background: #f8f9fa;
    // border-radius: 8px;
    // border: 1px solid #e9ecef;
    .demonstration {
      font-weight: bold;
    }
  }

  .filter-info {
    color: #28a745;
    font-weight: 500;
    margin-left: 0.5em;
  }
}

.flex-item {
  display: flex;
  flex-direction: row;
  & > b {
    margin-top: 0.2em;
    margin-right: 0.2em;
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
</style>
