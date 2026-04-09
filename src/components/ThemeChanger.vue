<script>
/*
  Copyright - 2026  - https://nextjs.org/learn?utm_source=next-site&utm_medium=homepage-cta&utm_campaign=home 

  Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"),
  to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense,
  and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

  The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
  WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
  Modified by Nfire07
*/
import 'material-design-icons-iconfont/dist/material-design-icons.css'

export default {
  name: 'ThemeSwitcher',
  data() {
    return {
      theme: 'light'
    }
  },
  mounted() {
    const saved = localStorage.getItem('theme')
    if (saved) {
      this.theme = saved
      this.applyTheme()
    }
  },
  methods: {
    setTheme(t) {
      this.theme = t
      localStorage.setItem('theme', t)
      this.applyTheme()
    },
    applyTheme() {
      const root = document.documentElement
      root.classList.toggle('dark', this.theme === 'dark')
    }
  }
}
</script>

<style scoped>
.theme-switcher {
  padding: 3px;
  width: fit-content;
  display: flex;
  border-radius: 9999px;
  box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
}

.theme-switcher_switch {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: inherit;
  border: 0;
  background: none;
  cursor: pointer;
}

.theme-switcher_switch.active {
  background-color: rgba(128, 128, 128, 0.35);
}

.theme-switcher_switch:hover > .icon {
  color: #323232;
}

.theme-switcher_switch.active > .icon {
  color: white;
}

.icon {
  font-size: 18px;
}
</style>

<template>
  <div role="radiogroup" class="theme-switcher">
    <button
      type="button"
      role="radio"
      class="theme-switcher_switch"
      :class="{ active: theme === 'light' }"
      aria-label="Light theme"
      :aria-checked="theme === 'light'"
      @click="setTheme('light')"
    >
      <span class="material-icons icon">light_mode</span>
    </button>

    <button
      type="button"
      role="radio"
      class="theme-switcher_switch"
      :class="{ active: theme === 'dark' }"
      aria-label="Dark theme"
      :aria-checked="theme === 'dark'"
      @click="setTheme('dark')"
    >
      <span class="material-icons icon">dark_mode</span>
    </button>
  </div>
</template>