<template>
  <div class="data-item-component">
    <div class="dt">
      <a style="color: #000">[{{ num }}]&nbsp;&nbsp;</a>
    </div>
    <div class="dd">
      <div class="title">
        <a :href="dataItem.url" target="_blank">{{ dataItem.title  || "title" }}</a>
      </div>
      <div class="author">{{ dataItem.author  || "author" }}</div>
      <div class="abstract">{{ dataItem.abstract }}</div>
      <div class="doi"><b>doi: </b>{{ dataItem.doi }}</div>
      <div class="booktitle"><b>booktitle: </b>{{ dataItem.booktitle }}</div>
      <div class="year"><b>year: </b>{{ dataItem.year }}</div>
      <div class="publisher"><b>publisher: </b>{{ dataItem.publisher }}</div>
      <div class="series"><b>series: </b>{{ dataItem.series }}</div>
      
      

      <!-- 折叠/展开控制 -->
      <!-- <div class="toggle-header" @click="toggleExpanded">
        <span>{{ isExpanded ? "Hide" : "Show" }} Details</span>
        <el-icon :class="{ rotate: isExpanded }">
          <ArrowDown />
        </el-icon>
      </div> -->

      <!-- 可折叠的内容 -->
      <!-- <div class="toggle-content" :class="{ expanded: isExpanded }">
        <p><b>Submission_historys:</b> {{ dataItem.submission_historys }}</p>
      </div> -->
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, defineProps, nextTick, defineExpose } from "vue";
import { ArrowDown } from "@element-plus/icons-vue";

const props = defineProps({
  dataItem: {
    type: Object,
    required: true,
    default: () => ({}),
  },
  num: {
    type: Number,
    required: true,
    default: 0,
  },
});

const isExpanded = ref(false);

const toggleExpanded = () => {
  isExpanded.value = !isExpanded.value;
};

// 外部控制展开状态的方法
const setExpandedState = (state) => {
  isExpanded.value = state;
};

// 暴露方法给父组件
defineExpose({
  setExpandedState,
});
</script>

<style lang="scss" scoped>
.data-item-component {
  font-size: 14px;
  color: #000;
  display: flex;
  .title{
    margin-bottom: .2em;
    a{
      color: #000;
      &:hover{
        color: #0000ee;
      }
    }
  }
  a {
    color: #0000ee;
  }
  .dt {
    a {
      font-weight: bold;
    }
  }
  .dd {
    line-height: 130%;
    margin-left: 3em;
    .title {
      font-size: 18px;
      font-weight: bold;
      line-height: 120%;
    }
    .abstract {
      margin-top: 1em;
    }
    .subjects {
      // font-size: 13px;
    }
    .authors {
    }
    b {
      // text-transform: capitalize;
    }
  }

  .flex-item {
    justify-content: flex-start;
    display: flex;
    flex-direction: row;
    & > b {
      margin-right: 0.2em;
    }
  }

  .toggle-header {
    font-size: 12px;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    cursor: pointer;
    padding: 0.5em 0;
    margin-top: 1em;
    user-select: none;
    gap: 0 0.5em;
    &:hover {
      color: #409eff;
    }

    .el-icon {
      &.rotate {
        transform: rotate(180deg);
      }
    }
  }

  .toggle-content {
    overflow: hidden;
    background: #fafafa;
    padding: 0 1em;
    display: none;
    font-size: 13px;

    &.expanded {
      display: block;
      padding: 1em;
    }
  }
}
</style>