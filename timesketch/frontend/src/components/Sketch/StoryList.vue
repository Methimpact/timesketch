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
    <ul class="content-list">
      <li style="padding:10px;border-bottom:none;" v-for="story in meta.stories" :key="story.id">
        <div>
          <router-link :to="{ name: 'SketchStoryContent', params: {sketchId: sketch.id, storyId: story.id}}">{{ story.title }}</router-link>
          <div class="field is-grouped is-pulled-right" style="margin-top:10px;">
            <p class="control">
              <button v-if="controls" v-on:click="remove(story)" class="button is-small is-rounded is-danger">
                <span class="icon is-small">
                  <i class="fas fa-trash"></i>
                </span>
                <span>Remove</span>
              </button>
            </p>
          </div>
          <br>
          <span class="is-size-7">Last activity {{ story.updated_at | moment("YYYY-MM-DD HH:mm") }}</span>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import ApiClient from '../../utils/RestApiClient'

export default {
  props: ['controls'],
  data () {
    return {
      stories: []
    }
  },
  methods: {
    remove (story) {
      ApiClient.deleteStory(this.sketch.id, story.id)
        .then((response) => {
          this.$store.dispatch('updateSketch', this.sketch.id)
        }).catch((e) => {
          console.error(e)
        })
    }
  },
  computed: {
    sketch () {
      return this.$store.state.sketch
    },
    meta () {
      return this.$store.state.meta
    }
  }
}
</script>

<!-- CSS scoped to this component only -->
<style scoped lang="scss"></style>
