<template>
  <div class="data-item-component">
    <div class="dt">
      <a style="color: #000">[{{ num }}]&nbsp;&nbsp;</a>
      <template v-if="dataItem.links">
        <a :href="dataItem.links.Abstract" title="Abstract" target="_blank"
        >ACM:{{ dataItem.id }}</a
      >
      &nbsp;&nbsp; [
      <template v-for="(value, key, index) in dataItem.links" :key="index">
        <template v-if="key !== 'Abstract'">
          <a :href="value" target="_blank">{{ key }}</a>
          <template v-if="index < Object.entries(dataItem.links).length - 1"
            >,&nbsp;
          </template>
        </template> </template
      >]
      </template>
      <template v-else>
        <b>{{ dataItem.source }}:{{ dataItem.id }}</b>
      </template>
      
    </div>
    <div class="dd">
      <div class="title">{{ dataItem.title  || "title" }}</div>
      <div v-if="dataItem.author" class="author">{{ dataItem.author  || "author" }}</div>
      <div class="abstract" style="margin-bottom: 1em;">{{ dataItem.abstract }}</div>

      <div class="relevance" style="margin-top: 1em" v-if="dataItem.relevance">
        <p>
          <b>Relevance Rating: </b>
          <span style="text-transform: capitalize">{{
            dataItem.relevance.level
          }}</span>
        </p>
        <p><b>Relevance Summary:</b> {{ dataItem.relevance.reason }}</p>
      </div>

      
     
      
      

      <!-- 折叠/展开控制 -->
      <div class="toggle-header" @click="toggleExpanded">
        <span>{{ isExpanded ? "Hide" : "Show" }} Details</span>
        <el-icon :class="{ rotate: isExpanded }">
          <ArrowDown />
        </el-icon>
      </div>

      <!-- 可折叠的内容 -->
      <div class="toggle-content" :class="{ expanded: isExpanded }">
        <div class="year"><b class="capitalize">year: </b>{{ dataItem.year }}</div>
        <p><b class="capitalize">conference_name:</b> {{ dataItem.conference_name }}</p>
        <p><b class="capitalize">keywords:</b> {{ dataItem.keywords }}</p>
        <p><b class="capitalize">address:</b> {{ dataItem.address }}</p>
        <p><b class="capitalize">booktitle:</b> {{ dataItem.booktitle }}</p>
        
        <p><b class="capitalize">id:</b> {{ dataItem.id }}</p>
        <p><b class="capitalize">isbn:</b> {{ dataItem.isbn }}</p>
        
        <p><b class="capitalize">location:</b> {{ dataItem.location }}</p>
        
        <p><b class="capitalize">publisher:</b> {{ dataItem.publisher }}</p>
        
        <p><b class="capitalize">series:</b> {{ dataItem.series }}</p>

        <p><b class="capitalize">numpages:</b> {{ dataItem.numpages }}</p>
        <p><b class="capitalize" v-if="dataItem.pages">pages:</b> {{ dataItem.pages }}</p>
        <p><b class="capitalize">number_of_citation:</b> {{ dataItem.number_of_citation }}</p>
        <p><b class="capitalize">number_of_total_download:</b> {{ dataItem.number_of_total_download }}</p>
        <p><b class="capitalize">open_access:</b> {{ dataItem.open_access }}</p>
        
      </div>
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
    margin-left: 5em;
    margin-top: 0.5em;
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
    b.capitalize {
      text-transform: capitalize;
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