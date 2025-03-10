<!--
Copyright 2019 Google Inc. All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->
<template>

  <div>

    <!-- Timeline detail modal -->
    <b-modal :active.sync="showInfoModal" :width="1024" scroll="keep">
      <div class="modal-background"></div>
      <div class="modal-content">
        <div class="card">
          <header class="card-header">
            <p class="card-header-title">Detailed information for {{ timeline.name }}</p>
          </header>
          <div class="card-content">
              <ul>
                <li>Elasticsearch index: {{ timeline.searchindex.index_name }}</li>
                <li v-if="meta.stats_per_timeline[timeline.id]">Number of events: {{ meta.stats_per_timeline[timeline.id]['count'] | compactNumber }} ({{ meta.stats_per_timeline[timeline.id]['count']}})</li>
                <li>Created by: {{ timeline.user.username }}</li>
                <li>Created at: {{ timeline.created_at | moment("YYYY-MM-DD HH:mm") }}</li>
              </ul>
              <br>

              <b-message :type="{ 'is-success': !datasource.error_message, 'is-danger': datasource.error_message }" :title="datasource.created_at" :closable="false" v-for="datasource in timeline.datasources" :key="datasource.id">
                <ul>
                  <li><strong>Provider:</strong> {{ datasource.provider }}</li>
                  <li><strong>Context:</strong> {{ datasource.context }}</li>
                  <li><strong>User:</strong> {{ datasource.user.username }}</li>
                  <li><strong>File on disk:</strong> {{ datasource.file_on_disk }}</li>
                  <li><strong>File size:</strong> {{ datasource.file_size | compactBytes }}</li>
                  <li><strong>Original filename:</strong> {{ datasource.original_filename }}</li>
                  <li><strong>Data label:</strong> {{ datasource.data_label }}</li>
                </ul>
                <br>
                <div v-if="datasource.error_message">
                  <strong style="font-size:1.2rem; margin-bottom:10px;">Error detail</strong>
                  <pre style="margin-top:10px;">{{ datasource.error_message }}</pre>
                </div>
              </b-message>

            </div>
          </div>

      </div>
      <button class="modal-close is-large" aria-label="close" v-on:click="showInfoModal = !showInfoModal"></button>
    </b-modal>

    <!-- Timeline edit modal -->
    <b-modal v-if="controls" :active.sync="showEditModal" :width="640" scroll="keep">
      <div class="modal-background"></div>
      <div class="modal-content">
        <div class="card">
          <header class="card-header">
            <p class="card-header-title">Rename timeline</p>
          </header>
          <div class="card-content">
            <div class="content">
              <form v-on:submit.prevent="saveTimeline">
                <div class="field">
                  <div class="control">
                    <input v-model="timeline.name" class="input" type="text" required autofocus>
                  </div>
                </div>
                <div class="field">
                  <div class="control">
                    <input class="button is-success" type="submit" value="Save">
                  </div>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>
      <button class="modal-close is-large" aria-label="close" v-on:click="showEditModal = !showEditModal"></button>
    </b-modal>

    <div v-if="timelineStatus === 'processing'" class="ts-timeline-color-box is-pulled-left blink" style="background-color: #f5f5f5;"></div>
    <div v-else-if="timelineStatus === 'fail'" v-on:click="showInfoModal =! showInfoModal" class="ts-timeline-color-box is-pulled-left" style="background-color: #f5f5f5;"></div>
    <div v-else-if="timelineStatus === 'ready' && controls" class="dropdown is-pulled-left" v-bind:class="{'is-active': colorPickerActive}">
      <div class="dropdown-trigger">
        <div class="ts-timeline-color-box" v-bind:style="timelineColorStyle" v-on:click="colorPickerActive = !colorPickerActive"></div>
      </div>
      <div class="dropdown-menu" id="dropdown-menu" role="menu">
        <div class="dropdown-content" style="padding:0;">
          <div class="dropdown-item" style="padding:0;">
            <color-picker v-model="initialColor" @input="updateColor"></color-picker>
          </div>
        </div>
      </div>
    </div>
    <div v-else-if="timelineStatus === 'ready'" class="ts-timeline-color-box is-pulled-left" v-bind:style="timelineColorStyle" v-on:click="colorPickerActive = !colorPickerActive"></div>
    <div v-else class="ts-timeline-color-box is-pulled-left" style="background-color: #f5f5f5;"></div>

    <div v-if="!controls" class="field is-grouped is-pulled-right" style="margin-top:10px;">
      <span class="is-size-7">{{ timeline.updated_at | moment("YYYY-MM-DD HH:mm") }}</span>
    </div>

    <div v-if="controls" class="field is-grouped is-pulled-right" style="margin-top:10px;">
      <p v-if="!isCompact" class="control">
        <button class="button is-rounded is-small is-outlined" v-on:click="showInfoModal = !showInfoModal">
                <span class="icon is-small">
                  <i class="fas fa-info-circle"></i>
                </span>
          <span>Info</span>
        </button>
      </p>
      <p v-if="meta.permissions.write && timelineStatus === 'ready' && controls" class="control">
        <button class="button is-rounded is-small is-outlined" v-on:click="showEditModal = !showEditModal">
          <span class="icon is-small">
            <i class="fas fa-edit"></i>
          </span>
          <span>Rename</span>
        </button>
      </p>

      <!-- Disabled 2020-12-17. Too expensive for large sketches. TODO: Refactor to do lazy loading instead.
      <p v-if="timelineStatus === 'ready'" class="control">
        <b-dropdown position="is-bottom-left" aria-role="menu" trap-focus append-to-body :scrollable="true" :max-height="300">
          <button class="button is-outlined is-rounded is-small" slot="trigger">
            <span class="icon is-small">
              <i class="fas fa-info-circle"></i>
            </span>
            <span>Data types</span>
          </button>
          <b-dropdown-item aria-role="menu-item" :focusable="false" custom>
            <div style="width:350px;">
              <div class="field" v-for="(dt) in meta.indices_metadata[timeline.searchindex.index_name]['data_types']" :key="dt.data_type">
                <b-checkbox v-model="checkedDataTypes" :native-value="dt.data_type" type="is-info">{{ dt.data_type }} ({{ dt.count | compactNumber }})</b-checkbox>
              </div>
              <button class="button is-success is-fullwidth" v-on:click="openFilteredTimeline(timeline.searchindex.index_name, checkedDataTypes)" :disabled="!checkedDataTypes.length">Open Filtered</button>
            </div>
          </b-dropdown-item>
        </b-dropdown>
      </p>
       -->

      <p v-if="timelineStatus === 'ready' && controls" class="control">
        <button class="button is-small is-rounded is-outlined" @click="showAnalysisHistory = !showAnalysisHistory">
          <span class="icon is-small">
            <i class="fas fa-history"></i>
          </span>
          <span>Analysis History</span>
        </button>
      </p>

      <p v-if="meta.permissions.write && controls" class="control">
        <button v-on:click="remove(timeline)" class="button is-small is-rounded is-danger">
          <span class="icon is-small">
            <i class="fas fa-trash"></i>
          </span>
          <span>Remove</span>
        </button>
      </p>
    </div>

    <router-link v-if="timelineStatus === 'ready'" :to="{ name: 'SketchExplore', query: {timeline: timeline.id}}">{{ timeline.name }}</router-link>
    <span v-if="timelineStatus !== 'ready'">{{ timeline.name }}</span>
    <br>

    <span v-if="timelineStatus === 'ready'" class="is-size-7">
      <span class="is-small" :title="meta.stats_per_timeline[timeline.id]['count'] + ' events in index'">{{ meta.stats_per_timeline[timeline.id]['count'] | compactNumber }} events</span>
      <span v-if="timeline.datasources.length > 1"> ({{ timeline.datasources.length }} imports: <span v-on:click="showInfoModal =! showInfoModal" style="cursor:pointer;text-decoration: underline;">details</span>)</span>
      <span v-if="timeline.datasources.length === 1"> (imported with {{ timeline.datasources[0].provider }})</span>
      <span v-if="datasourceErrors.length" style="margin-left:10px;">
        <span class="icon is-small" style="color:orange;">
          <i class="fas fa-exclamation-triangle"></i>
        </span>
        <span v-on:click="showInfoModal =! showInfoModal" style="cursor:pointer;text-decoration: underline; margin-left:5px;">{{ datasourceErrors.length }} failed imports</span>
      </span>
    </span>

    <span v-else-if="timelineStatus === 'fail'" class="is-size-7">
      <span class="icon is-small" style="color:var(--font-color-red);">
        <i class="fas fa-exclamation-triangle"></i>
      </span>
      ERROR: <span v-on:click="showInfoModal =! showInfoModal" style="cursor:pointer;text-decoration: underline">Click here for details</span>
    </span>
    <span v-else-if="timelineStatus === 'processing'" class="is-size-7">
      Indexing in progress...
    </span>
    <span v-else class="is-size-7">
      Unknown status: {{ timelineStatus }}
    </span>

    <div v-if="showAnalysisHistory">
      <ts-analyzer-history :timeline="timeline" @closeHistory="showAnalysisHistory = false"></ts-analyzer-history>
    </div>

  </div>
</template>

<script>
import Vue from 'vue'
import { Chrome } from 'vue-color'
import _ from 'lodash'

import ApiClient from '../../utils/RestApiClient'

import TsAnalyzerHistory from './AnalyzerHistory'

import EventBus from '../../main'

export default {
  components: {
    'color-picker': Chrome,
    TsAnalyzerHistory
  },
  props: ['timeline', 'controls', 'isCompact'],
  data () {
    return {
      checkedDataTypes: [],
      initialColor: {},
      newColor: '',
      newTimelineName: '',
      colorPickerActive: false,
      showInfoModal: false,
      showEditModal: false,
      analysisSessionId: false,
      showAnalysisDetail: false,
      showAnalysisHistory: false,
      timelineStatus: null,
      autoRefresh: false,
      isOpen: false,
      isDarkTheme: false
    }
  },
  computed: {
    sketch () {
      return this.$store.state.sketch
    },
    meta () {
      return this.$store.state.meta
    },
    timelineColorStyle () {
      let backgroundColor = this.newColor || this.timeline.color
      if (!backgroundColor.startsWith('#')) {
        backgroundColor = '#' + backgroundColor
      }
      if (this.isDarkTheme) {
        return {
          'background-color': backgroundColor,
          'filter': 'grayscale(25%)',
          'color': '#333'
        }
      }
      return {
        'background-color': backgroundColor
      }
    },
    datasourceErrors () {
      return this.timeline.datasources.filter(datasource => datasource.error_message)
    }
  },
  methods: {
    remove (timeline) {
      this.$emit('remove', timeline)
    },
    updateColor: _.debounce(function (color) {
      this.newColor = color.hex
      if (this.newColor.startsWith('#')) {
        this.newColor = this.newColor.substring(1)
      }
      Vue.set(this.timeline, 'color', this.newColor)
      this.$emit('save', this.timeline)
    }, 300),
    saveTimeline () {
      this.showEditModal = false
      this.$emit('save', this.timeline)
    },
    fetchData () {
      ApiClient.getSketchTimeline(this.sketch.id, this.timeline.id).then((response) => {
        this.timelineStatus = response.data.objects[0].status[0].status
        if (this.timelineStatus !== 'ready') {
          this.autoRefresh = true
        }
        this.$store.dispatch('updateSketch', this.$store.state.sketch.id)
      }).catch((e) => {})
    },
    openFilteredTimeline: function (index, dataTypes) {
      if (dataTypes.length === 0) {
        return false
      }
      let searchQuery = ''
      for (let i = 0; i < dataTypes.length; i++) {
        const dt = dataTypes[i]
        if (i !== 0) {
          searchQuery += ' OR '
        }
        searchQuery += 'data_type:"' + dt + '"'
      }
      this.$router.push({ name: 'SketchExplore', query: { index: index, q: searchQuery } })
    },
    toggleTheme: function () {
      this.isDarkTheme = !this.isDarkTheme
    }
  },
  mounted () {
    // Hide color picket when clicked outside.
    let self = this
    window.addEventListener('click', function (e) {
      if (!self.$el.contains(e.target)) {
        self.colorPickerActive = false
      }
    })
  },
  created () {
    this.isDarkTheme = localStorage.theme === 'dark'
    EventBus.$on('isDarkTheme', this.toggleTheme)

    this.initialColor = {
      hex: this.timeline.color
    }
    this.timelineStatus = this.timeline.status[0].status
    if (this.timelineStatus !== 'ready') {
      this.autoRefresh = true
    }
  },
  beforeDestroy () {
    clearInterval(this.t)
    this.t = false
  },
  watch: {
    autoRefresh (val) {
      if (val && !this.t) {
        this.t = setInterval(function () {
          this.fetchData()
          if (this.timelineStatus === 'ready') {
            this.autoRefresh = false
          }
        }.bind(this), 5000)
      } else {
        clearInterval(this.t)
        this.t = false
      }
    }
  }
}
</script>

<!-- CSS scoped to this component only -->
<style scoped lang="scss">
.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active, .list-leave-active {
  transition: all 0.5s;
}
.list-enter, .list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
.vc-sketch {
  box-shadow: none;
}
.blink {
  animation: blinker 1s linear infinite;
}
.checkbox-margin {
  margin-left: 10px;
  margin-right: 6px;
}
.small-top-margin {
  margin-top: 4px;
}

@keyframes blinker {
  50% {
    opacity: 40%;
  }
}

.table th {
  color: var(--default-font-color);
}
</style>
