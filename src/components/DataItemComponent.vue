<template>
  <div class="data-item-component">
    <div class="dt">
      <a>[{{ num }}]&nbsp;&nbsp;</a>
      <a :href="dataItem.links.Abstract" title="Abstract" target="_blank"
        >{{ dataItem.source }}:{{ dataItem.id }}</a
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
    </div>
    <div class="dd">
      <div class="title">{{ dataItem.title }}</div>
      <div class="authors">
        <span v-for="(author, index) in dataItem.authors" :key="index">
          {{ author }}
          <span v-if="index < dataItem.authors.length - 1">, &nbsp;&nbsp;</span>
        </span>
      </div>
      <div class="subjects">
        <span>Subjects: </span>
        <span v-for="(subject, index) in dataItem.subjects" :key="index">
          <b>{{ subject }}</b
          ><span v-if="index < dataItem.subjects.length - 1"
            >, &nbsp;&nbsp;</span
          >
        </span>
      </div>
      <div class="abstract">{{ dataItem.abstract }}</div>
      <div class="relevance" style="margin-top: 1em">
        <p><b>Relevance Rating:</b> <span style="text-transform: capitalize;">{{ dataItem.relevance.level }}</span></p>
        <p><b>Relevance Summary:</b> {{ dataItem.relevance.reason }}</p>
      </div>
      <div>
        <b>Last_revised_date:</b>
        {{ dataItem.last_revised_date }}
      </div>
      <div>
        <b>License:</b>
        <a :href="dataItem.license" target="_blank">{{ dataItem.license }}</a>
      </div>
      <p v-if="dataItem.type"><b>Type: </b> {{ dataItem.type }}</p>
      <p v-if="dataItem.repo_urls">
        <b>Repo_urls: </b>
        <span v-for="(value, index) in dataItem.repo_urls" :key="index">
          <a :href="value" target="_blank">{{ value }}</a
          ><span v-if="index < dataItem.repo_urls.length - 1"
            >, &nbsp;&nbsp;
          </span>
        </span>
      </p>
      <p v-if="dataItem.conference">
        <b>Conference: </b>{{ dataItem.conference }}
      </p>
      <p v-if="dataItem.conference_url_abs">
        <b>Conference_url_abs:</b
        ><a :href="dataItem.conference_url_abs" target="_blank">{{
          dataItem.conference_url_abs
        }}</a>
      </p>
      <p v-if="dataItem.tasks">
        <b>Tasks: </b>
        <span v-for="(value, index) in dataItem.tasks" :key="index">
          {{ value
          }}<span v-if="index < dataItem.tasks.length - 1"
            >, &nbsp;&nbsp;
          </span>
        </span>
      </p>
      <p v-if="dataItem.comments"><b>comments: </b>{{ dataItem.comments }}</p>

      <div class="flex-item" v-if="dataItem.datasets">
        <b>Datasets:</b>
        <div>
          <p v-for="(dataset, index) in dataItem.datasets" :key="index">
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
      <div class="flex-item" v-if="dataItem.models">
        <b>Models:</b>
        <div>
          <p v-for="(dataset, index) in dataItem.models" :key="index">
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
      <p><b>Submission_historys:</b> {{ dataItem.submission_historys }}</p>
      <p><b>submitted_date:</b> {{ dataItem.submitted_date }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, defineProps, nextTick } from "vue";
import selectComponent from "@/components/selectComponent.vue";

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
    b {
      // text-transform: capitalize;
    }
  }

  .flex-item {
    justify-content: flex-start;
    display: flex;
    flex-direction: row;
    & > b {
      margin-top: 0.2em;
      margin-right: 0.2em;
    }
  }
}
</style>