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

  <section class="section" style="background-color:var(--navbar-background);padding:0;border-bottom: 1px solid var(--navbar-header-border-color);">
    <div class="container is-fluid" style="padding-bottom:0;">

      <nav class="navbar" role="navigation" aria-label="main navigation">
        <div class="navbar-item" v-if="currentAppContext === 'sketch'">
          <div class="tabs is-left">
            <ul>
              <li v-bind:class="{'is-active': currentPage === 'overview'}">
                <router-link :to="{ name: 'SketchOverview' }">
                  <span class="icon is-small"><i class="fas fa-cubes" aria-hidden="true"></i></span>
                  <span>Overview</span>
                </router-link>
              </li>
              <li v-bind:class="{'is-active': currentPage === 'explore'}">
                <router-link :to="{ name: 'SketchExplore' }">
                  <span class="icon is-small"><i class="fas fa-search" aria-hidden="true"></i></span>
                  <span>Explore</span>
                </router-link>
              </li>
              <li v-bind:class="{'is-active': currentPage === 'graph'}">
                <router-link :to="{ name: 'SketchGraphOverview' }">
                  <span class="icon is-small"><i class="fas fa-project-diagram" aria-hidden="true"></i></span>
                  <span>Graph</span>
                </router-link>
              </li>
              <li v-bind:class="{'is-active': currentPage === 'aggregate'}">
                <router-link :to="{ name: 'SketchAggregate' }">
                  <span class="icon is-small"><i class="fas fa-chart-bar" aria-hidden="true"></i></span>
                  <span>Aggregate</span>
                </router-link>
              </li>
              <li v-bind:class="{'is-active': currentPage === 'analyzers'}">
                <router-link :to="{ name: 'SketchAnalyzersOverview' }">
                  <span class="icon is-small"><i class="fas fa-magic" aria-hidden="true"></i></span>
                  <span>Analyze</span>
                </router-link>
              </li>
              <li v-bind:class="{'is-active': currentPage === 'timelines'}">
                <router-link :to="{ name: 'SketchManageTimelines' }">
                  <span class="icon is-small"><i class="fas fa-stream" aria-hidden="true"></i></span>
                  <span>Timelines</span>
                </router-link>
              </li>
              <li v-bind:class="{'is-active': currentPage === 'stories'}">
                <router-link :to="{ name: 'SketchStoryOverview' }">
                  <span class="icon is-small"><i class="fas fa-book" aria-hidden="true"></i></span>
                  <span>Stories</span>
                </router-link>
              </li>
            </ul>
          </div>
        </div>
        <div class="navbar-end">
          <div class="navbar-item">
            <slot></slot>
          </div>
        </div>
      </nav>
    </div>
  </section>

</template>

<script>
export default {
  name: 'ts-navbar-secondary',
  props: ['currentAppContext', 'currentPage']
}
</script>

<!-- CSS scoped to this component only -->
<style scoped lang="scss">
  .navbar {
    background: transparent;
  }
  .tabs {
    margin-left: -20px;
  }
  .tabs a {
    padding: 0.5em 1em;
  }
</style>
