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
    <section class="section" style="background-color:var(--navbar-background);padding:0;border-bottom: 1px solid var(--navbar-border-color);">
        <div class="container is-fluid">

          <nav class="navbar" role="navigation" aria-label="main navigation">
            <div class="navbar-brand">
              <router-link class="navbar-item" to="/">
                <div class="logo" style="margin-top:7px;">
                  <img src="/dist/timesketch-color.png">
                </div>
                <span style="color: var(--default-title-font-color); margin-left: 7px; margin-top: 1px; font-size: var(--font-size-title);">time<b>sketch</b></span>
              </router-link>
            </div>

            <div class="navbar-item" style="margin-left: 20px; margin-top:5px; font-size: var(--font-size-large);">
              <slot name="left"></slot>
            </div>

            <div class="navbar-item navbar-center">
              <slot name="center"></slot>
            </div>

            <div class="navbar-end">
              <div class="navbar-item">
                <b-switch
                  v-model="isDarkTheme"
                  v-on:input="switchTheme"
                  size="is-small"
                  passive-type='is-dark'
                  type='is-dark'>
                  Dark Mode
                </b-switch>
              </div>
              <div class="navbar-item">
                {{ currentUser }}
              </div>
              <div class="navbar-item">
                <a href="/logout">Logout</a>
              </div>
            </div>
          </nav>

        </div>
    </section>

</template>

<script>
import EventBus from '../main'

export default {
  name: 'ts-navbar-main',
  data () {
    return {
      isDarkTheme: null
    }
  },

  computed: {
    currentUser () {
      return this.$store.state.currentUser
    }
  },
  methods: {
    switchTheme () {
      let element = document.body
      switch (element.dataset.theme) {
        case 'light':
          element.dataset.theme = 'dark'
          localStorage.theme = 'dark'
          this.isDarkTheme = true
          EventBus.$emit('isDarkTheme', true)
          break
        case 'dark':
          element.dataset.theme = 'light'
          localStorage.theme = 'light'
          this.isDarkTheme = false
          EventBus.$emit('isDarkTheme', false)
          break
      }
    }
  },
  created () {
    this.isDarkTheme = localStorage.theme === 'dark'
  }
}
</script>

<!-- CSS scoped to this component only -->
<style scoped lang="scss">
  .navbar {
    height: 70px;
  }

  .navbar-item {
    padding-left: 0;
  }

  .logo img {
    width: 20px;
    height: 25px;
  }

  .navbar-item img {
    width: 20px;
    height: 25px;
    max-height: 25px;
  }

  .navbar-item.navbar-center {
    flex-grow: 1;
    flex-direction: column;
    justify-content: center;
  }

  .navbar > .container .navbar-brand, .container > .navbar .navbar-brand {
    margin-left: 0;
}
</style>
